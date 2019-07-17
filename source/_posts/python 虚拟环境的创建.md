---
title: python 虚拟环境的搭建
tags:
  - python
categories: 学习笔记
abbrlink: 945033781
date: 2019-01-19 21:43:28
updated: 2019-02-15 22:22:22
---
# python 虚拟环境的创建

#### 基于python2 的虚拟环境搭建
virtualenv 是一个创建隔绝的Python环境的工具，用来管理虚拟环境

安装 virtualenv：
> pip install virtualenv

创建虚拟环境：

```
$ cd my_project_dir
$ virtualenv venv　　#venv为虚拟环境目录名，目录名自定义
'''激活虚拟环境：'''
$ source venv/bin/activate
'''安装需要的包：'''
$ pip install requests
'''当有requirements.txt的时候可以'''
$ pip install -r requirements.txt  #直接安装全部包
'''退出虚拟环境：'''
$ . venv/bin/deactivate
```



#### 基于python3 的虚拟环境搭建
使用venv搭建的虚拟环境同virtualenv搭建的虚拟环境，即venv可替代virtualenv

python3自带venv

创建虚拟环境：
```
$ mkdir myenv    #venv为虚拟环境目录名，目录名自定义
$ cd myenv
$ python3 -m venv .　#(venv之后一个空格加上一个点 “ . ”)

'''激活虚拟环境：'''
$ cd Scripts
$ activate.bat / activate

'''安装需要的包：'''
$ pip3 install requests    #requests 即为包名，需要版本号时为：request==版本号
'''退出虚拟环境：'''
$ deactivate.bat / deactivate
```
##### pip3升级：
```
sudo pip3 install --upgrade pip
sudo vim /usr/bin/pip3
    from pip import __main__
         ......(__main__._main())
```

