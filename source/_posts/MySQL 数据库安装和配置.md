---
title: MySQL数据库
tags:
  - mysql
categories: 数据库
abbrlink: 2653527150
date: 2019-01-19 21:43:28
updated: 2019-02-15 22:22:22
---
# MySQL数据库相关操作

#### 1.安装MySQL数据库

查看mysql支持的存储引擎，"show engines;" 

| Engine| Support | Comment | Transactions | XA   | Savepoints |
---|---|---|----|---|---
| MyISAM  | YES | MyISAM storage engine           | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |

关于存储引擎的介绍：
[四种mysql存储引擎](https://www.cnblogs.com/wcwen1990/p/6655416.html)


DEFAULT为默认存储引擎，因为MyISAM引擎（一般为默认）不支持事务也不支持外键，所以需要用到Innodb引擎，于是将mysql的默认引擎设置为innodb

在配置文件my.cnf中的 [mysqld] 下面修改default-storage-engine=INNODB，如果没有则添加

重启mysql服务器：service mysqld restart 

查看数据库默认使用的存储引擎：

> show variables like '%storage_engine%';

查看已有表的存储引擎：

> show create table Tablename(表名);


#### 2.设置MySQL数据库的编码格式
utf-8编码可能2个字节、3个字节、4个字节的字符，但是MySQL的utf8编码只支持3字节的数据，而移动端的表情数据是4个字节的字符。如果直接往采用utf-8编码的数据库中插入表情数据，Java程序中将报SQL异常

可以对4字节的字符进行编码存储，然后取出来的时候，再进行解码。但是这样做会使得任何使用该字符的地方都要进行编码与解码。

utf8mb4编码是utf8编码的超集，兼容utf8，并且能存储4字节的表情字符。 
采用utf8mb4编码的好处是：存储与获取数据的时候，不用再考虑表情字符的编码与解码问题

##### 更改数据库的编码为utf8mb4
**MySQL的版本**:utf8mb4的最低mysql版本支持版本为5.5.3+，若不是，请升级到较新版本

**修改MySQL配置文件**:

修改mysql配置文件my.cnf（windows为my.ini）

my.cnf一般在etc/mysql/my.cnf位置。找到后请在以下三部分里添加如下内容： 

[client] 

default-character-set = utf8mb4 

[mysql] 

default-character-set = utf8mb4 

[mysqld] 

character-set-client-handshake = FALSE 

character-set-server = utf8mb4 

collation-server = utf8mb4_unicode_ci 

init_connect='SET NAMES utf8mb4'

**重启数据库，检查变量**
> SHOW VARIABLES WHERE Variable_name LIKE 'character_set_%' OR Variable_name LIKE 'collation%';

Variable_name  | Value | 描述
---|---|---
character_set_client | utf8mb4 | 客户端来源数据使用的字符集
character_set_connection |  utf8mb4|连接层字符集
character_set_database | utf8mb4 | 当前选中数据库的默认字符集
character_set_filesystem | binary
character_set_results | utf8mb4 | 查询结果字符集
character_set_server | utf8mb4 | 默认的内部操作字符集
character_set_system | utf8
collation_connection | utf8mb4_unicode_ci
collation_database | utf8mb4_unicode_ci
collation_server | utf8mb4_unicode_ci

**数据库连接的配置**

数据库连接参数中: 

characterEncoding=utf8会被自动识别为utf8mb4，也可以不加这个参数，会自动检测。 

而autoReconnect=true是必须加上的。

**将数据库和已经建好的表也转换成utf8mb4**

**更改数据库编码**：ALTER DATABASE caitu99 CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

**更改表编码**：ALTER TABLE TABLE_NAME CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci; 

如有必要，还可以更改列的编码

#### 3.更改数据库的存储路径

**在mysql下查看数据存储路径**(默认位置如下)
```
mysql> show variables like '%datadir%';

+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.04 sec)
```
**关闭mysql**
> [root@mysql ~]# systemctl stop mysql　　　　 //停止mysql

> [root@mysql ~]# mv /var/lib/mysql   /var/lib/mysql.BAK　　　　//数据改名备份一下，方便回滚

> [root@mysql ~]# mkdir /data　　　　//这里新建一个假设的新目录/data/

> [root@mysql ~]# rsync -av  /var/lib/mysql   /data/　　　　//复制数据到新目录。rsync命令没有可以用yum安装；也可以用cp命令复制;此处也有很多人在用mv直接移动,省去权限问题的麻烦,如果直接用mv记得要备份好原数据

**更改my.cnf文件**
> [root@mysql ~]# vim /etc/my.cnf　　　//编辑 my.cnf。如果默认没有，可以"cp /usr/share/mysql/my-default.cnf  /etc/my.cnf"

[client]

port = 3306
socket = /data/mysql/mysql.sock

[mysqld]

datadir = /data/mysql
 
socket = /data/mysql/mysql.sock

备注：其实socke可以不用改的  只要改下 datadir就行了 如果安装innodb 那么 innodbdir 也要改

[mysqld_safe]

socket = /home/mysql/mysql.sock

[mysql.server]

socket = /home/mysql/mysql.sock

**再次启动mysql**
> [root@mysql ~]# systemctl start mysql

**启动失败解决办法**

在出现报错情况下，优先去查看日志去分析问题

1. 检查selinux

一个简单的解决办法是使用命令暂时关闭selinux，以便让你的操作可以继续下去

setenforce 0

但最好使用一个永久方法，以便在重启后继续不要这货

修改/etc/selinux/config文件中设置SELINUX=disabled ，然后重启或等待下次重启。（如果没有这个就看下面的方法)

2. 检查apparmor

它也对mysql所能使用的目录权限做了限制

在 /etc/apparmor.d/usr.sbin.mysqld 这个文件中，有这两行，规定了mysql使用的数据文件路径权限

/var/lib/mysql/ r,

/var/lib/mysql/** rwk,

我想把数据文件移动到/data/mysql下，那么为了使mysqld可以使用/data/mysql这个目录，照上面那两条，增加下面这两条就可以了

/data/mysql/ r,

/data/mysql/** rwk,

重启: sudo service apparmor restart

如有必要,将socket新目录也添加上
