---
title: python 代码规范PEP8
tags:
  - python
categories: 学习笔记
abbrlink: 1884161696
date: 2019-03-01 18:45:34
updated: 2019-03-01 18:45:34
---
### 缩进

使用四个空格表示每个缩进级别。

```python
# 对齐
foo = long_function_name(var_one, var_two,
                         var_three, var_four)
                         
# 比之后的内容多一层缩进
# 使用更多的缩进以和其他的代码单元区别开来
def long_function_name(
        var_one, var_two, var_three,
        var_four):
    print(var_one)

# 悬垂的缩进，多加一层
foo = long_function_name(
    var_one, var_two,
    var_three, var_four)
# 或者：
foo = long_func_name(
    var_one, var_two,
    var_three, var_four
)
# 另外其实有一种可选情况，也就是悬垂缩进可以缩进不为4个空格，比如用两个
```
不规范示例

```python
# 当不适用垂直对齐时，禁止在第一行使用参数
# 换句话说，在垂直对齐时，才可在第一行使用参数
foo = lone_func_name(var_one, var_two, 
    var_three, var_four)

# 当缩进不足以区分代码结构时，增加一个缩进级别
def long_func_name(
    var_one, var_two, var_three
    var_four):
    print(var_one)
```
### 最大长度
所有行的最大长度均为79个字符

对于文档字符串或者注释，最长72字符

```python
# 当超出最大长度时，使用反斜杠来换行
with open('') as file_1, \
     open('') as file_2:
    file_2.write(file_1.read())
```
使用正确的换行位置。推荐的位置在二元操作符（binary operator，如下述代码中的and、or以及%）之后，二元运算符( + -)之前

```python
# 同一括号换行，不需要反斜杠，因为是隐式换行
# 在 and or % 之后回车换行
class Rectangle(Shape):
    def __init__(self, width, height,
                 color='black', emphasis=None, highlight=0):
        if (width == 0 and height == 0 and
                color == 'red' and emphasis == 'strong' or
                height > 100):
            raise ValueError("sorry, you lose")
        if width == 0 and height == 0 and (color == 'red' or emphasis is None):
            raise ValueError("I don't think so -- values are %s, %s" %
                             (width, height))
        Shape.__init__(self, width, height, color,
                       emphasis, highlight)

```

```python
# 在 + - 之前换行，增加可读性
income = (gross_wages
          + taxable_interest
          + (dividends - qualified_dividends)
          - ira_deduction
          - student_loan_interest)
```
### 空行
- 顶级函数（当前文件中的第一个函数）或者顶级类（当前文件的第一个类）之前要有两个空行
- 定义在类内部的函数（成员函数）之间要留有一个空行
- 可以使用额外的空行（但要注意节制）以区分不同的函数组，
- 在一堆只有一行的函数之间不要使用空行（比如一些函数的空实现）
- 在函数内部使用空行，来标识不同的逻辑单元

### 导入
import应该总是在文件的最上面，在模块注释和文档字符串之后，在模块变量和常量之前

注意import的顺序，各个import的组需要用空行隔开，顺序为（各个import独立成行）
 - 标准库import
 - 相关的第三方import
 - 本地应用和库的import
 - 
 一般情况使用绝对的import，但是在包层次比较复杂的时候使用相对import可以更加简洁，则使用相对import

在import一个class的时候，如果不会引起命名冲突，则可以使用from进行import，否则则直接import并且使用全名

应该避免使用利用通配符进行import，也就是避免使用from xxx import *

模块级的特殊名字，此处特指那些前后都有双下划线的名字， 比如author等，应该被放置在模块文档字符串之后，除from future import的任何import之前，比如

```python
"""This is the example module.

This module does stuff.
"""

from __future__ import barry_as_FLUFL

__all__ = ['a', 'b', 'c']
__version__ = '0.1'
__author__ = 'Cardinal Biggles'

import os
import sys

import numpy
import pandas
import matplotlib

import tool
```
### 空格

以下几种情况不要额外加空格：

- 在各种括号之中，比如spam(ham[1], {eegs: 2})而不是spam( ham[ 1 ], { eggs: 2 } )
- 在逗号分号和冒号之前
- 但是如果冒号作为分隔符，则前后都加空格
- 后面立即跟了一个括号，比如函数调用的函数和括号之间不应该加空格
- 后面跟的是索引或者切片的中括号，比如a[1]而不是a [1]
- 对于赋值或者其他操作符，不要为了多个语句对齐而加很多空格，前后一个即可

其他的建议:
- 一行的尾部不要有空格
- 二元运算符前后始终都最好有一个空格
- 在一个表达式中有不同优先级的运算符，可以添加空格以区别优先级
- 在调用函数时作为参数的那个等号则前后不要有空格（虽然看起来像个二元运算符）,比如func(a=3, b=4)而不是func(a = 3, b = 4)
- 带箭头的函数，箭头两端也应该和二元运算符一样，前后有空格def func() -> AnyStr: ...
- 用于指示关键字参数或默认参数值时，不要在 = 符号周围使用空格

```python
def complex(real, imag=0.0):
    return magic(r=real, i=imag)
```
- 将参数注释与默认值组合时，请在=符号周围使用空格（但仅限于那些同时具有注释和默认值的参数

```python
def munge(sep: AnyStr = None): ...
def munge(input: AnyStr, sep: AnyStr = None, limit=1000): ...
```
### 文档
- 为所有公共模块或者函数、类以及方法编写文档。不必为非公共方法编写doc文档，但应有一个注释描述算法的功能，这条注释应当出现在def之后
- 结尾的"""应当独占一行

```
"""Return a foobang

Optional plotz says to frobnicate the bizbaz first.
"""
```
### 注释
- 注释和内容冲突的，比不要注释还好，改代码一定要改注释，保证注释最新
- 注释应该是完整的句子，如果一个注释是一个短语或者句子，首字母大写，除非是小写字母开头的标识符作为开头
- 短注释结尾的句号可以省略，包含一个或多个段落的块注释每一个句子都应该有句号
- 在句子结束的句号之后应该有两个句号
- 用英语写的时候，遵守Strunk and White写作风格
- 除非你十分确定你的代码不会被任何不用你这个语言的人使用，否则都用英语写注释

### 编程推荐
- 代码应该不使得其他python实现有劣势，也就是不要依靠特定的python实现来提高效率
- 和单例(Singleton)，比如None比较，应该一直使用is和is not而不是等于符号和不等于
- 使用is not运算符而不是使用not … is
- 如果实现比较运算密集的有序操作时，最好实现所有的六种操作(eq, ne, lt, le, gt, ge)
- 定义有名字的函数始终使用def而不是通过lambda表达式来赋值
- 从Exception继承而不是从BaseException
- 恰当的使用exception链
- 在py2里边raise一个异常的时候，使用raise ValueError('message')而不是raise ValuError, 'message'， py3中后面的方法会出错
- 捕捉异常的时候，使用具体的异常，而不是用一个单纯的except:xxx，否则连SystemExit和KeyboardInterrupt也会被捕捉到
- 给异常绑定名字的时候使用py2.6添加的as的方法，except Exception as exc:...
- 捕捉系统错误的时候，使用py3.3添加的异常阶层而不是使用errno值
- try语句包含的东西越少越好
- 如果一个资源是一段代码本地使用的，使用with语句保证被正确释放
- 上下文管理器始终应该使用函数或者方法来调用
- 保证返回语句一致，要么所有情况都有返回，要么都没有
- 使用字符串方法，而不是使用字符串模块
- 使用''.startswith()和''.endswith()而不是字符串切片来检查前缀后缀
- 对象类型的比较应该使用isinstance()而不是直接比较
- 对于序列(字符串，列表，元组)，可以利用空序列为false
- 字符串字面量不要依赖有意义的尾部空白
- 布尔值不要用==和is来比较，直接用if is_true:...而不是if is_true == True:更不要用if is_true is True:...

### 参考连接：
[官方PEP8文档](https://www.python.org/dev/peps/pep-0008/#fn-hi)

[pep8 要求归纳](https://blog.csdn.net/qq_29343201/article/details/54660570)

