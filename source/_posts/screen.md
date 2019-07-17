---
title: screen操作笔记
tags:
  - Linux
categories: 操作系统
abbrlink: 2547638477
date: 2019-01-19 21:43:28
updated: 2019-02-15 22:22:22
---
# screen 操作笔记
### 简介：
**Screen**是一款由GNU计划开发的用于命令行终端切换的自由软件。用户可以通过该软件同时连接多个本地或远程的命令行会话，并在其间自由切换。GNU Screen可以看作是窗口管理器的命令行界面版本。它提供了统一的管理多个会话的界面和相应的功能。
### 安装screen
有些Linux会自带screen，如果没有可以在GNU screen官网下载


> [seven@TT ~]# yuminstall screen

> [seven@TT ~]# sudo apt install screen

也可以从官网下载安装包，解压安装

[GNU screen官网](http://www.gnu.org/software/screen/)
### 操作命令：
#### 创建一个新窗口
安装完成后，直接敲命令screen就可以启动它。但是这样启动的screen会话没有名字，实践上推荐为每个screen会话取一个名字，方便分辨：
> seven@TT:~/Desktop$ screen -S xxx

也可以直接加上所要运行的脚本程序
> seven@TT:~/Desktop$ screen vim aaa.txt

> seven@TT:~/Desktop$ screen -S acac vim bbb.txt

screen创建一个执行vim aaa.txt的单窗口会话，退出vim 将退出该窗口/会话。

需要长时间运行的后台程序
> seven@TT:~/Desktop$ screen python run.py

#### 会话分离与恢复
screen 的好处就是可以在不中断程序运行的状态,而暂时断开会话窗口,在随后可以重新连接该会话,重新控制运行的程序
```
seven@TT:~/Desktop$ screen -S xxx
[detached from 27802.xxx]
'''创建并进入会话'''
'''在会话窗口按 Ctrl+a+d 可以不中断程序,而暂时退出会话'''
seven@TT:~/Desktop$ screen -ls
There is a screen on:
	27802.xxx	(2018年11月20日 10时15分57秒)	(Detached)
1 Socket in /var/run/screen/S-seven.
'''查看会话列表'''
seven@TT:~/Desktop$ screen -r xxx
'''重新连接指定会话'''
```
当然，如果你在另一台机器上没有分离一个Screen会话，就无从恢复会话了。这时可以使用下面命令强制将这个会话从它所在的终端分离，转移到新的终端上来：
```
seven@TT:~/Desktop$ screen -ls
There are screens on:
	28018.awaw	(2018年11月20日 10时40分51秒)	(Attached)
	27869.acac	(2018年11月20日 10时22分53秒)	(Detached)
	27802.xxx	(2018年11月20日 10时15分58秒)	(Detached)
3 Sockets in /var/run/screen/S-seven.
'''Attached 为未分离的会话'''
seven@TT:~/Desktop$ screen -d 28018
[28018.awaw detached.]
'''强制分离会话'''
seven@TT:~/Desktop$ screen -r 28018
[detached from 28018.awaw]
'''重新连接会话'''
```
#### 清除dead 会话
如果由于某种原因其中一个会话死掉了（例如人为杀掉该会话），这时screen -list会显示该会话为dead状态。使用screen -wipe命令清除该会话：

```
seven@TT:~/Desktop$ screen -ls
There are screens on:
	28018.awaw	(2018年11月20日 10时40分51秒)	(Detached)
	27869.acac	(2018年11月20日 10时22分53秒)	(Detached)
	27802.xxx	(2018年11月20日 10时15分58秒)	(Detached)
3 Sockets in /var/run/screen/S-seven.
seven@TT:~/Desktop$ kill -9 27802
seven@TT:~/Desktop$ screen -ls
There are screens on:
	28018.awaw	(2018年11月20日 10时40分50秒)	(Detached)
	27869.acac	(2018年11月20日 10时22分52秒)	(Detached)
	27802.xxx	(2018年11月20日 09时44分52秒)	(Dead ???)
Remove dead screens with 'screen -wipe'.
3 Sockets in /var/run/screen/S-seven.

seven@TT:~/Desktop$ screen -wipe 27802
There is a screen on:
	27802.xxx	(2018年11月20日 09时44分53秒)	(Removed)
1 socket wiped out.
No Sockets found in /var/run/screen/S-seven.

seven@TT:~/Desktop$ screen -ls
There are screens on:
	28018.awaw	(2018年11月20日 10时40分51秒)	(Detached)
	27869.acac	(2018年11月20日 10时22分53秒)	(Detached)
2 Sockets in /var/run/screen/S-seven.
```
#### 关闭或杀死窗口
正常情况下，当你退出一个窗口中最后一个程序（通常是bash）后，这个窗口就关闭了。另一个关闭窗口的方法是使用C-a k，这个快捷键杀死当前的窗口，同时也将杀死这个窗口中正在运行的进程。

如果一个Screen会话中最后一个窗口被关闭了，那么整个Screen会话也就退出了，screen进程会被终止。

除了依次退出/杀死当前Screen会话中所有窗口这种方法之外，还可以使用快捷键C-a :，然后输入quit命令退出Screen会话。需要注意的是，这样退出会杀死所有窗口并退出其中运行的所有程序。其实C-a :这个快捷键允许用户直接输入的命令有很多，包括分屏可以输入split等，这也是实现Screen功能的一个途径，不过个人认为还是快捷键比较方便些
#### 会话共享
> seven@TT:~/Desktop$ screen -x

假设你在和朋友在不同地点以相同用户登录一台机器，然后你创建一个screen会话，你朋友可以在他的终端上命令,
这个命令会将你朋友的终端Attach到你的Screen会话上，并且你的终端不会被Detach。这样你就可以和朋友共享同一个会话了，如果你们当前又处于同一个窗口，那就相当于坐在同一个显示器前面，你的操作会同步演示给你朋友，你朋友的操作也会同步演示给你。当然，如果你们切换到这个会话的不同窗口中去，那还是可以分别进行不同的操作的

#### 会话锁定与解锁
Screen允许使用快捷键C-a s锁定会话。锁定以后，再进行任何输入屏幕都不会再有反应了。但是要注意虽然屏幕上看不到反应，但你的输入都会被Screen中的进程接收到。快捷键C-a q可以解锁一个会话。

也可以使用C-a x锁定会话，不同的是这样锁定之后，会话会被Screen所属用户的密码保护，需要输入密码才能继续访问这个会话。

#### 屏幕分割
现在显示器那么大，将一个屏幕分割成不同区域显示不同的Screen窗口显然是个很酷的事情。可以使用快捷键C-a S将显示器水平分割，Screen 4.00.03版本以后，也支持垂直分屏，快捷键是C-a |。分屏以后，可以使用C-a 在各个区块间切换，每一区块上都可以创建窗口并在其中运行进程。

可以用C-a X快捷键关闭当前焦点所在的屏幕区块，也可以用C-a Q关闭除当前区块之外其他的所有区块。关闭的区块中的窗口并不会关闭，还可以通过窗口切换找到它

#### C/P模式和操作
screen的另一个很强大的功能就是可以在不同窗口之间进行复制粘贴了。使用快捷键C-a 或者C-a [可以进入copy/paste模式，这个模式下可以像在vi中一样移动光标，并可以使用空格键设置标记。其实在这个模式下有很多类似vi的操作，譬如使用/进行搜索，使用y快速标记一行，使用w快速标记一个单词等。关于C/P模式下的高级操作，其文档的这一部分有比较详细的说明。

一般情况下，可以移动光标到指定位置，按下空格设置一个开头标记，然后移动光标到结尾位置，按下空格设置第二个标记，同时会将两个标记之间的部分储存在copy/paste buffer中，并退出copy/paste模式。在正常模式下，可以使用快捷键C-a ]将储存在buffer中的内容粘贴到当前窗口

#### 更多screen功能
同大多数UNIX程序一样，GNU Screen提供了丰富强大的定制功能。你可以在Screen的默认两级配置文件/etc/screenrc和$HOME/.screenrc中指定更多，例如设定screen选项，定制绑定键，设定screen会话自启动窗口，启用多用户模式，定制用户访问权限控制等等。如果你愿意的话，也可以自己指定screen配置文件。

以多用户功能为例，screen默认是以单用户模式运行的，你需要在配置文件中指定multiuser on 来打开多用户模式，通过acl*（acladd,acldel,aclchg...）命令，你可以灵活配置其他用户访问你的screen会话

---

#### 命令查看

> \# screen [-AmRvx -ls -wipe][-d <作业名称>][-h <行数>][-r <作业名称>][-s ][-S <作业名称>]

##### 命令行参数
-A 　将所有的视窗都调整为目前终端机的大小。

-d <作业名称> 　将指定的screen作业离线。

-h <行数> 　指定视窗的缓冲区行数。

-m 　即使目前已在作业中的screen作业，仍强制建立新的screen作业。

-r <作业名称> 　恢复离线的screen作业。

-R 　先试图恢复离线的作业。若找不到离线的作业，即建立新的screen作业。

-s 　指定建立新视窗时，所要执行的shell。

-S <作业名称> 　指定screen作业的名称。

-v 　显示版本信息。

-x 　恢复之前离线的screen作业。

-ls或--list 　显示目前所有的screen作业。

-wipe 　检查目前所有的screen作业，并删除已经无法使用的screen作业

##### 窗口命令
C-a ? -> 显示所有键绑定信息
C-a c -> 创建一个新的运行shell的窗口并切换到该窗口
C-a n -> Next，切换到下一个 window 

C-a p -> Previous，切换到前一个 window 

C-a 0..9 -> 切换到第 0..9 个 window

Ctrl+a [Space] -> 由视窗0循序切换到视窗9

C-a C-a -> 在两个最近使用的 window 间切换

C-a x -> 锁住当前的 window，需用用户密码解锁

C-a d -> detach，暂时离开当前session，将目前的 screen session (可能含有多个 windows) 丢到后台执行，并会回到还没进 screen 时的状态，此时在 screen session 里，每个 window 内运行的 process (无论是前台/后台)都在继续执行，即使 logout 也不影响。 

C-a z -> 把当前session放到后台执行，用 shell 的 fg 命令则可回去。

C-a w -> 显示所有窗口列表

C-a t -> time，显示当前时间，和系统的 load 

C-a k -> kill window，强行关闭当前的 window

C-a [ -> 进入 copy mode，在 copy mode 下可以回滚、搜索、复制就像用使用 vi 一样    

C-b Backward，PageUp     

C-f Forward，PageDown     

H(大写) High，将光标移至左上角     

L Low，将光标移至左下角     

0 移到行首     

$ 行末     

w forward one word，以字为单位往前移     

b backward one word，以字为单位往后移     

Space 第一次按为标记区起点，第二次按为终点 

Esc 结束 

copy mode 

C-a ] -> paste，把刚刚在 copy mode 选定的内容贴上
