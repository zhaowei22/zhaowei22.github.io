---
title: docker 容器搭建
tags:
  - docker
categories: 环境搭建
abbrlink: 3677998467
date: 2019-01-19 21:43:28
updated: 2019-02-15 22:22:22
---
# docker 学习笔记
## docker 安装
1. 在ubuntu下的安装

> sudo apt install docker.io

在生产环境下应该安装指定版本
> sudo apt-get install docker-ce=<VERSION>

注意：在此处可能会出现包引用的错误，具体原因自行解决（没搞懂...）

2. 在win下安装

在windows下安装需要满足的条件：
> **Docker for Windows** 支持 64 位版本的 Windows 10 Pro，且必须开启 Hyper-V。

Windows 10 Pro, Windows旗舰版和教育版可行。

Hyper-V 为Windows上的虚拟机，如果电脑上还有VM虚拟机，则这两个会发生冲突，VM在运行时需要关闭Hyper-V。

3. 在mac下安装

参考win下安装

4. docker 需要用户具有sudo权限，为了避免每次都输入sudo，可以添加用户到docker用户组

创建用户组,添加用户，重启：

> $ sudo groupadd docker 

> $ sudo gpasswd -a ${USER} docker

> $ sudo service docker restart

切换当前会话到新 group 或者重启 X 会话
> newgrp - docker

如果已有分组，直接添加用户
> $ sudo usermod -aG docker $USER

### docker 镜像（Image）

**镜像加速：**

Docker 官方提供的中国 registry mirror https://registry.docker-cn.com

七牛云加速器 https://reg-mirror.qiniu.com/

对于使用 upstart 的系统而言，编辑 /etc/default/docker 文件，在其中的 DOCKER_OPTS 中配置加速器地址：

DOCKER_OPTS="--registry-mirror=https://registry.docker-cn.com"

重新启动服务。

$ sudo service docker restart

对于使用 systemd 的系统，请在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）

```
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ]
}
```
之后重新启动服务。
> $ sudo systemctl daemon-reload
> $ sudo systemctl restart docker

对于使用 Windows 10 的系统，在系统右下角托盘 Docker 图标内右键菜单选择 Settings，打开配置窗口后左侧导航菜单选择 Daemon。在 Registry mirrors 一栏中填写加速器地址 https://registry.docker-cn.com，之后点击 Apply 保存后 Docker 就会重启并应用配置的镜像地址了

**获取镜像：**

> $ docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

一般从Docker Store上下载，搜索需要的镜像并按Tag下载，一般为官方原版镜像

也可以自己创建自定义镜像，创建Dockerfile

```
FROM python:3.6-alpine3.8  
# 官方基于alpine3.8下的python3.6 镜像
# alpine为精简版Linux
# 在此基础上叠加镜像包

LABEL maintainer="Seven test"
# 创建人信息

COPY ./app /usr/src/app
# 拷贝本地文件./app 到镜像文件/use/src/app下

RUN cd /usr/src/app && pip install -r requirements.txt
# 执行操作 进入/usr/src/app 并下载相关安装包

WORKDIR /usr/src/app
# 指定工作目录

EXPOSE 8888
# 对外暴露的端口

CMD python xxx/xxx.py
# 启动容器时执行的操作
```


列出镜像：

> $ docker image ls

删除镜像：

> $ docker image rm [选项] <镜像1> [<镜像2> ...]



### docker 容器（Container）

每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为容器存储层。容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。

按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 数据卷（Volume）、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。

数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。

运行容器：

> $ sudo docker run --name webserver -d -p 80:80 nginx

> $ docker exec -it webserver bash

当然，你应该还会用到其它的一些参数，比如-name指定容器名字，后面再 start 这个容器就不用查 id 了；-v挂载文件，将你本地的代码挂载进 docker；或者-p映射端口，将 docker 的端口映射到本机，以便提供http 等服务。

例如，创建一个名字为 webserver的镜像，将我本地 code下的代码挂载到镜像/root/app目录下，并将虚拟机的80端口映射到本机8080，命令如下：
> docker run -it --name webserver -v ~/code:/root/app -p 8080:80 ubuntu:16.04 /bin/bash

### Docker Compose 的使用

在多容器连接的情况下，避免重复去创建容器，所以采用docker-compose更方便

Linux下需要另外下载docker-compose:
> $ sudu apt install docker-compose

在项目文件下新建 docker-compose.yml 文件：

docker-compose 遵循YAML格式

```
mysql:
  image: mysql:5.7
  environment:
    - MYSQL_ROOT_PASSWORD=123456
    - MYSQL_DATABASE=wordpress
web:
  image: wordpress
  links:
    - mysql
  environment:
    - WORDPRESS_DB_PASSWORD=123456
  ports:
    - "127.0.0.3:8080:80"
  working_dir: /var/www/html
  volumes:
    - wordpress:/var/www/html
```

两个顶层标签表示有两个容器mysql和web。每个容器的具体设置，前面都已经讲解过了，还是挺容易理解的。

启动所有服务
> $ docker-compose up

关闭所有服务
> $ docker-compose stop

关闭以后，这两个容器文件还是存在的，写在里面的数据不会丢失。下次启动的时候，还可以复用。下面的命令可以把这两个容器文件删除（容器必须已经停止运行）。
> $ docker-compose rm

#### 注意点：
1. 

---
#### 参考文档：

[Docker —— 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)

[docker docs](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#cmd)