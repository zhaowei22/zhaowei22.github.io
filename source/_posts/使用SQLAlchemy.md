---
title: SQLAlchemy的使用
tags:
  - mysql
categories: 数据库
abbrlink: 1260038886
date: 2019-01-19 21:43:28
updated: 2019-02-15 22:22:22
---
[toc]
# 使用SQLAlchemy
ORM技术：Object-Relational Mapping，把关系数据库的表结构映射到对象上。

在Python中，最有名的ORM框架是SQLAlchemy。
#### pip安装SQLAlchemy：
> pip install sqlalchemy

首先，导入SQLAlchemy，并初始化DBSession：

```
# 导入:
from sqlalchemy import Column, String, create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.declarative import declarative_base

# 创建对象的基类:
Base = declarative_base()

# 定义User对象:
class User(Base):
    # 表的名字:
    __tablename__ = 'user'

    # 表的结构:
    id = Column(String(20), primary_key=True)
    name = Column(String(20))

# 初始化数据库连接:
engine = create_engine('mysql+mysqlconnector://root:password@localhost:3306/test')
# 创建DBSession类型:
DBSession = sessionmaker(bind=engine)
```
下面，我们看看如何向数据库表中添加一行记录。

由于有了ORM，我们向数据库表中添加一行记录，可以视为添加一个User对象

```
# 创建session对象:
session = DBSession()
# 创建新User对象:
new_user = User(id='5', name='Bob')
# 添加到session:
session.add(new_user)
# 提交即保存到数据库:
session.commit()
# 关闭session:
session.close()

```
SQLAlchemy提供的查询接口如下：

```
# 创建Session:
session = DBSession()
# 创建Query查询，filter是where条件，最后调用one()返回唯一行，如果调用all()则返回所有行:
user = session.query(User).filter(User.id=='5').one()
# 打印类型和对象的name属性:
print('type:', type(user))
print('name:', user.name)
# 关闭Session:
session.close()
```
ORM框架也可以提供两个对象之间的一对多、多对多等功能。

例如，如果一个User拥有多个Book，就可以定义一对多关系如下：

```
class User(Base):
    __tablename__ = 'user'

    id = Column(String(20), primary_key=True)
    name = Column(String(20))
    # 一对多:
    books = relationship('Book')

class Book(Base):
    __tablename__ = 'book'

    id = Column(String(20), primary_key=True)
    name = Column(String(20))
    # “多”的一方的book表是通过外键关联到user表的:
    user_id = Column(String(20), ForeignKey('user.id'))
```
当我们查询一个User对象时，该对象的books属性将返回一个包含若干个Book对象的list。

#### 关系定义
1. **一对多**:
 
 表示一对多的关系时，在子表类中通过 foreign key (外键)引用父表类。然后，在父表类中通过 relationship() 方法来引用子表的类
```
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    children = relationship("Child")
    # 在父表类中通过 relationship() 方法来引用子表的类集合
    
class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))
    # 在子表类中通过 foreign key (外键)引用父表的参考字段
```
在一对多的关系中建立双向的关系，这样的话在对方看来这就是一个多对一的关系，在子表类中附加一个 relationship() 方法，并且在双方的 relationship() 方法中使用 relationship.back_populates 方法参数：

```
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    children = relationship("Child", back_populates="parent")
    
class Child(Base):
    __tablename__ = 'child'    
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))    
    parent = relationship("Parent", back_populates="children")
    
# 子表类中附加一个 relationship() 方法
# 并且在(父)子表类的 relationship() 方法中使用 relationship.back_populates 参数
```
或者，可以在单一的 relationship() 方法中使用 backref 参数来代替 back_populates 参数

2. **一对一**

一对一是两张表之间本质上的双向关系.要做到这一点，只需要在一对多关系基础上的父表中使用 uselist 参数来表示，uselist=False。

3. **多对多**
多对多关系会在两个类之间增加一个关联的表。这个关联的表在 relationship() 方法中通过 secondary 参数来表示。通常的，这个表会通过 MetaData 对象来与声明基类关联，所以这个 ForeignKey 指令会使用链接来定位到远程的表

```
# 多对多关系中的两个表之间的一个关联表
association_table = Table('association', Base.metadata, Column('left_id', Integer, ForeignKey('left.id')), Column('right_id', Integer, ForeignKey('right.id')))
class Parent(Base):
    __tablename__ = 'left'
    id = Column(Integer, primary_key=True)
    children = relationship("Child",secondary=association_table)
    # 在父表中的 relationship() 方法传入 secondary 参数，其值为关联表的表名
    
class Child(Base):
    __tablename__ = 'right'
    id = Column(Integer, primary_key=True)
```
指定使用 relationship.back_populates 参数，并且为每一个 relationship() 方法指定共用的关联表：

```
 association_table = Table('association', Base.metadata, Column('left_id', Integer, ForeignKey('left.id')), Column('right_id', Integer, ForeignKey('right.id')))
 class Parent(Base):
    __tablename__ = 'left'
    id = Column(Integer, primary_key=True)
    children = relationship("Child", secondary=association_table, back_populates="parents")
class Child(Base):
    __tablename__ = 'right'
    id = Column(Integer, primary_key=True)
    parents = relationship("Parent", secondary=association_table, back_populates="children")
```



---

常见的SQLAlchemy列类型.配置选项和关系选项
类型名称 | python类型 | 描述
---|---|---
Integer | int | 常规整形，通常为32位
SmallInteger | int | 短整形，通常为16位
BigInteger | int/long | 精度不受限整形
Float | float | 浮点数
Numeric | decimal.Decimal | 定点数
String | str | 可变长度字符串
Text | str | 可变长度字符串，适合大量文本
Unicode | unicode | 浮点数
Boolean | bool | 布尔型
Date | datetime.date | 日期类型
Time | datetime.time | 时间类型
Interval |   datetime.timedelta | 时间间隔
Enum  |  str|字符列表
PickleType | 任意Python对象 | 自动Pickle序列化
LargeBinary | str | 二进制

常见的SQLAlchemy列选项
可选参数 | 描述
---|---
primary_key | 如果设置为True，则为该列表的主键
unique | 如果设置为True，该列不允许相同值
index  | 如果设置为True，为该列创建索引，查询效率会更高
nullable  |  如果设置为True，该列允许为空。如果设置为False，该列不允许空值
default | 定义该列的默认值

### 错误解决
当使用migrate upgrade更新数据库时，如果删除以前创建的外键关联字段，会报错：
> 1553, "Cannot drop index 'xxx': needed in a foreign key constraint

解决办法：

进入数据库，手动删除外键所创建的索引：
> show create table TABLE_NAME;

找到对应的索引，并删除：
> ALTER TABLE table_name DROP FOREIGN KEY exchange_ibfk_3;

