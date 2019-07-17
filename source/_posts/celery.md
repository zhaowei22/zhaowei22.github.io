---
title: celery
tags:
  - 组件
categories: 组件使用
abbrlink: 1699526646
date: 2019-01-19 21:43:28
updated: 2019-02-15 22:22:22
---
# celery 和 Flask + redis

Celery 是一个简单、灵活且可靠的，处理大量消息的分布式系统，并且提供维护这样一个系统的必需工具。

它是一个专注于实时处理的任务队列，同时也支持任务调度。

Celery 需要一个发送和接受消息的传输者。RabbitMQ 和 Redis 中间人的消息传输支持所有特性，但也提供大量其他实验性方案的支持，包括用 SQLite 进行本地开发。

Celery 可以单机运行，也可以在多台机器上运行，甚至可以跨越数据中心运行。

### celery 配置
文件目录树：

```
server/
├── apis
│     └── api.raml
├── xxxx
│   ├── xxxx.py
│   ├── models.py
│   ├── transforms.py
│   ├── users.py
│   └── views
│         ├── __init__.py
│         ├── tasks.py
│         └── xxxx.py
├── celery_worker.py
├── run.py
└── settings.py

```
在settings.py文件中配置cellery

```
class Config(object):
    # celery config
    CELERY_BROKER_URL = 'redis://:@localhost:6379/'
    CELERY_RESULT_BACKEND = 'redis://:@localhost:6379/'
```

在__init__.py文件下和app一起配置：

```
from flask import Flask, render_template, redirect, url_for,\
     _request_ctx_stack, request, abort
from flask.ext.security import Security, SQLAlchemyUserDatastore
from .models import db, User, Role
from celery import Celery
import logging

CELERY = Celery('ddu',broker='redis://:@localhost:6379/1')

def _log_config(app):
    if not app.debug and not app.testing:
        from logging.handlers import RotatingFileHandler
        file_handler = RotatingFileHandler(
            app.config.get('LOGGING_PATH'),
            maxBytes=app.config.get('LOGGING_SIZE'))
        file_handler.setLevel(logging.WARNING)
        app.logger.addHandler(file_handler)

def create_app(config):
    app = Flask(
        __name__,
        template_folder='../../votingmanage/templates',
        static_folder='../../votingmanage/dist',
        static_url_path="/static")
    app.config.from_object(config)
    
    app.db = db
    db.init_app(app)
    
    user_datastore = SQLAlchemyUserDatastore(db, User, Role)
    security = Security(app, user_datastore)
    app.user_datastore = user_datastore
    
    @app.route('/')
    def to_app():
        return redirect(url_for('ngapp.home'))

    app.apns = APNs(use_sandbox=True,
                    cert_file=path.join(app.root_path, 'ck.pem'),
                    key_file=path.join(app.root_path, 'ck.pem'))
                    
    import users
    app.register_blueprint(users.bp, url_prefix='/api/users')  
    
    # Poplulate user information if the token is specified
    @app.before_request
    def populate_user():
        header_key = app.config.get('SECURITY_TOKEN_AUTHENTICATION_HEADER', 'Authentication-Token')
        args_key = app.config.get('SECURITY_TOKEN_AUTHENTICATION_KEY', 'token')
        header_token = request.headers.get(header_key, None)
        token = request.args.get(args_key, header_token)
        if request.get_json(silent=True):
            token = request.json.get(args_key, token)

        if token:
            user = app.extensions['security'].login_manager.token_callback(token)
            _request_ctx_stack.top.user = user
    
    global CELERY #全局变量CELERY
    
    def make_celery(app):   #此处app为创建的实例对象
        celery = Celery(app.import_name, broker=app.config['CELERY_BROKER_URL'],
                        backend=app.config['CELERY_RESULT_BACKEND'])
        celery.conf.update(app.config)
        TaskBase = celery.Task

        class ContextTask(TaskBase):
            abstract = True

            def __call__(self, *args, **kwargs):
                with app.app_context():
                    return TaskBase.__call__(self, *args, **kwargs)

        celery.Task = ContextTask
        return celery

    CELERY = make_celery(app) 
    
    return app
```
创建celery的启动文件celery_worker.py：

```
from ddu import create_app, CELERY

from settings import DEV as PROD
app = create_app(PROD)
app.app_context().push()

```
编写tasks.py文件，完成定时任务：

```
# -*- coding:utf-8 -*-
from .. import CELERY
from ..models import User, Work, Remid, GZHUser
from datetime import datetime,timedelta
from flask import current_app, request
import requests
import json

@CELERY.on_after_configure.connect
def setup_periodic_tasks(sender, **kwargs):
    # 添加定时任务，设置时间
    # sender.add_periodic_task(10.0, test1.s('world'), expires=10)
    sender.add_periodic_task(60.0, weixin_remid.s())
    sender.add_periodic_task(10.0, user_unionid.s())
    sender.add_periodic_task(120.0, get_list.s())
    
@CELERY.task
def test1(arg1="arg1"):
    print arg1,111
    return 5
    
@CELERY.task
def user_unionid():
    count = get_people_unionid()
    print str(count) + 'user unionid...'
```
编写xxx.py文件，完成异步任务：

```
task.add.apply_async(args=[3,7],expires=10)
# expires：任务过期时间，参数类型可以是 int，也可以是 datetime
# eta (estimated time of arrival)：指定任务被调度的具体时间，参数类型是 datetime
# countdown：指定多少秒后执行任务
```

### celery 启动
在celery_worker.py文件下：
> celery -A celery_worker.CELERY worker --loglevel=info --beat

### 注意：
1. 编写完tasks.py文件后，在启动Flask项目时，应该在主文件中导入tasks.py文件
> from .tasks import test1
2. 定时任务，也可以在配置文件中进行配置描述（此方法未试验成功，setting配置无响应）

```
from datetime import timedelta

CELERYBEAT_SCHEDULE = {
    'add-every-30-seconds': {
        'task': 'tasks.add',
        'schedule': timedelta(seconds=30),
        'args': (16, 16)
    },
}

CELERY_TIMEZONE = 'UTC'
```

```
from celery.schedules import crontab

CELERYBEAT_SCHEDULE = {
    # Executes every Monday morning at 7:30 A.M
    'add-every-monday-morning': {
        'task': 'tasks.add',
        'schedule': crontab(hour=7, minute=30, day_of_week=1),
        'args': (16, 16),
    },
}
```

3. celery 和 Flask 同时使用时一定要注意实例的config配置问题，以免出现混乱
4. 上文为celery读取数据库进行定时任务（未涉及任务结果的重新保存）