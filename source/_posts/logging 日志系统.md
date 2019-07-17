---
title: logging 日志模块
tags:
  - python
categories: 学习笔记
abbrlink: 2728557103
date: 2019-01-19 21:43:28
updated: 2019-01-25 22:22:22
---
# logging 日志系统
[toc]
### 日志级别


 等级 | 数值 | 描述
---|---|---
NOTSET | 0 |
DEBUG | 10 | 详细信息，通常仅在诊断问题时才有意义
INFO | 20 | 确认事情按预期工作
WARNING | 30 | 警告信息
ERROR | 40 | 由于更严重的问题，该软件无法执行某些功能
CRITICAL | 50 | 严重错误，表明程序本身可能无法继续运行

默认级别为 WARNING，这意味着将仅跟踪此级别及更高级别的事件，除非日志包已配置为执行其他操作。

### 记录器
Logger对象有三重的工作。首先，它们向应用程序代码公开了几种方法，以便应用程序可以在运行时记录消息 其次，记录器对象根据严重性（默认过滤工具）或过滤器对象确定要处理的日志消息。第三，记录器对象将相关的日志消息传递给所有感兴趣的日志处理程序。

记录器对象上使用最广泛的方法分为两类：配置和消息发送。

- Logger.setLevel() 指定记录器将处理的最低严重性日志消息，其中debug是最低内置严重性级别，critical是最高内置严重性级别。例如，如果严重性级别为INFO，则记录器将仅处理INFO，WARNING，ERROR和CRITICAL消息，并将忽略DEBUG消息。

- Logger.addHandler()并Logger.removeHandler()从logger对象添加和删除处理程序对象。处理程序中详细介绍了处理程序。

- Logger.addFilter()并Logger.removeFilter()从记录器对象中添加和删除过滤器对象。过滤器对象中详细介绍了 过滤器。

### 处理程序
Handler对象负责将适当的日志消息（基于日志消息的严重性）分派给处理程序的指定目标。 Logger对象可以使用addHandler()方法向自己添加零个或多个处理程序对象。作为示例场景，应用程序可能希望将所有日志消息发送到日志文件，将错误或更高的所有日志消息发送到标准输出，以及对电子邮件地址至关重要的所有消息。此方案需要三个单独的处理程序，其中每个处理程序负责将特定严重性的消息发送到特定位置。

标准库包含很多处理程序类型（请参阅 有用的处理程序）; 教程主要使用StreamHandler并 FileHandler在其示例中使用。

处理程序中很少有方法可供应用程序开发人员关注。与使用内置处理程序对象（即不创建自定义处理程序）的应用程序开发人员相关的唯一处理程序方法是以下配置方法：

- 该setLevel()方法与logger对象一样，指定将分派到适当目标的最低严重性。为什么有两种setLevel()方法？记录器中设置的级别确定将传递给其处理程序的消息的严重性。每个处理程序中设置的级别确定处理程序将发送哪些消息。
- setFormatter() 选择要使用的此处理程序的Formatter对象。
- addFilter()并removeFilter()分别在处理程序上配置和取消配置过滤器对象。

应用程序代码不应直接实例化和使用实例 Handler。相反，Handler该类是一个基类，它定义了所有处理程序应具有的接口，并建立了子类可以使用（或覆盖）的一些默认行为

### 常见处理程序
除了基Handler类之外，还提供了许多有用的子类：

- StreamHandler 实例将消息发送到流（类文件对象）。
- FileHandler 实例将消息发送到磁盘文件。
- BaseRotatingHandler是在某个点旋转日志文件的处理程序的基类。它并不意味着直接实例化。相反，使用RotatingFileHandler或 TimedRotatingFileHandler。
- RotatingFileHandler 实例将消息发送到磁盘文件，支持最大日志文件大小和日志文件轮换。
- TimedRotatingFileHandler 实例将消息发送到磁盘文件，以特定的时间间隔旋转日志文件。
- SocketHandler实例将消息发送到TCP / IP套接字。从3.4开始，也支持Unix域套接字。
- DatagramHandler实例将消息发送到UDP套接字。从3.4开始，也支持Unix域套接字。
- SMTPHandler 实例将消息发送到指定的电子邮件地址。
- SysLogHandler 实例将消息发送到Unix syslog守护程序，可能在远程计算机上。
- NTEventLogHandler 实例将消息发送到Windows NT / 2000 / XP事件日志。
- MemoryHandler 实例将消息发送到内存中的缓冲区，只要满足特定条件，就会刷新。
- HTTPHandler实例使用任一语义GET或POST语义将消息发送到HTTP服务器。
- WatchedFileHandler实例会监视他们要登录的文件。如果文件发生更改，则会关闭该文件并使用文件名重新打开。此处理程序仅在类Unix系统上有用; Windows不支持使用的基础机制。
- QueueHandler实例将消息发送到队列，例如在queue或multiprocessing模块中实现的那些队列。
- NullHandler实例不执行任何错误消息。它们由想要使用日志记录的库开发人员使用，但是希望避免“如果库用户没有配置日志记录，则可以显示”没有找到记录器XXX的处理程序“消息。有关更多信息，请参阅配置库的日志记录。

新的3.1版：的NullHandler类。

新版本3.2：在QueueHandler类。

### 基本配置
> logging.basicConfig(filename='example.log', filemode='w', level=logging.DEBUG)

基本配置过程为：获取logger，获取对应Handler并设置参数，然后将Handler添加到logger，有些模块包含创建logger的方法

在flask中的配置

```
import logging

def _log_config(app):
    if not app.debug and not app.testing:
        from logging.handlers import RotatingFileHandler
        file_handler = RotatingFileHandler(
            app.config.get('LOGGING_PATH'),
            maxBytes=app.config.get('LOGGING_SIZE'))
        file_handler.setLevel(logging.WARNING)
        app.logger.addHandler(file_handler)
        
# 然后在创建app的时候执行此函数 _log_config(app)
def create_app(config):
    app = Flask()
    ...
    _log_config(app)
    ...
```

参考连接：

[Logging Cookbook](https://docs.python.org/3.6/howto/logging-cookbook.html#logging-cookbook)

[Logging handlers](https://docs.python.org/3.6/library/logging.handlers.html#logging.handlers.RotatingFileHandler)

[logging](https://docs.python.org/3.6/library/logging.html#logging.Formatter)

[flask logging](http://flask.pocoo.org/docs/1.0/logging/?highlight=logging)