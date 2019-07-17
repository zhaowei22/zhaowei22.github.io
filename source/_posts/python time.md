---
title: python time 笔记
tags:
  - python
categories: 学习笔记
abbrlink: 2653527150
date: 2019-05-25 23:22:28
updated: 2019-02-27 22:33:54
---
在 python 中，操作时间 和 日期的模块有 time 和 datetime，calendar

time 简单的时间输出或转换

datetime 简单和复杂的时间格式化

calendar 操作日历的模块

### 相关概念
> **时间戳**： 是指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至现在的总秒数。通俗的讲， 时间戳是一份能够表示一份数据在一个特定时间点已经存在的完整的可验证的数据。 它的提出主要是为用户提供一份电子证据， 以证明用户的某些数据的产生时间。 在实际应用上， 它可以使用在包括电子商务、 金融活动的各个方面， 尤其可以用来支撑公开密钥基础设施的 “不可否认” 服务。  
时间戳单位最适于做日期运算。但是1970年之前的日期就无法以此表示了。太遥远的日期也不行，UNIX和Windows只支持到2038年。

> **时间元组**： 用元组形式表示一个时间点  

属性 | 注释 | 取值
---|---|---
tm_year | 4位数年 | 4位数，2019
tm_mon | 月 | 1 到 12
tm_mday | 日 | 1到31
tm_hour | 时 | 0到23
tm_min | 分 | 0到59
tm_sec | 秒 | 0到61 (60或61 是闰秒)
tm_wday | 一周的第几日 | 0到6 (0是周一)
tm_yday | 一年的第几日 | 1到366 (儒略历)
tm_isdst | 夏令时 | -1, 0, 1, -1是决定是否为夏令时的旗帜

> python 格式化时间符号

```
%y    # 两位数的年份表示（00-99）
%Y    # 四位数的年份表示（000-9999）
%m    # 月份（01-12）
%d    # 月内中的一天（0-31）
%H    # 24小时制小时数（0-23）
%I    # 12小时制小时数（01-12）
%M    # 分钟数（00=59）
%S    # 秒（00-59）
%a    # 本地简化星期名称
%A    # 本地完整星期名称
%b    # 本地简化的月份名称
%B    # 本地完整的月份名称
%c    # 本地相应的日期表示和时间表示
%j    # 年内的一天（001-366）
%p    # 本地A.M.或P.M.的等价符
%U    # 一年中的星期数（00-53）星期天为星期的开始
%w    # 星期（0-6），星期天为星期的开始
%W    # 一年中的星期数（00-53）星期一为星期的开始
%x    # 本地相应的日期表示
%X    # 本地相应的时间表示
%Z    # 当前时区的名称
%%    # %号本身
```


## time 模块

```python
# -*- coding: UTF-8 -*-
import time

# 获取当前时间戳
# 1558162277.7014651
timestamp = time.time()

# 获取时间元组；可获取指定时间
# time.struct_time(tm_year=2019, tm_mon=5, tm_mday=18, tm_hour=14, tm_min=51, tm_sec=17, tm_wday=5, tm_yday=138, tm_isdst=0)
localtime = time.localtime(timestamp)

# 将时间元组格式化为可读时间
# 'Sat May 18 14:51:17 2019'
localtime = time.asctime(localtime)

# 使用 strftime 方法来格式化日期
# '2019-05-18 15:06:05'
time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
# Sat May 18 15:07:06 2019'
time.strftime("%a %b %d %H:%M:%S %Y", time.localtime())


dt = "2019-04-27 20:22:45"

# 将字符串时间转换成时间数组；
# time.struct_time(tm_year=2019, tm_mon=4, tm_mday=27, tm_hour=20, tm_min=22, tm_sec=45, tm_wday=5, tm_yday=117, tm_isdst=-1)
time_tuple = time.strptime(dt, "%Y-%m-%d %H:%M:%S")

# 将时间元组转换成时间戳
# 556367765.0 时间间隔是以秒为单位的浮点小数
timestamp = time.mktime(time_tuple)

# 推迟调用线程的运行
time.sleep(secs)

# 属性
# 当地时区（未启动夏令时）距离格林威治的偏移秒数（>0，美洲;<=0大部分欧洲，亚洲，非洲）
# -28800
time.timezone

# 属性time.tzname包含一对根据情况的不同而不同的字符串，分别是带夏令时的本地时区名称，和不带的
# ('CST', 'CST')
time.tzname
```






## datetime 模块

datetime 模块包含的类

```date``` 日期，属性: year, month, day

```time``` 时间，属性: hour, minute, second, microsecond, tzinfo

```datetime``` 日期和时间的组合
```python
from datetime import date, time, datetime

# datetime(year, month, day[, hour[, minute[, second[, microsecond[, tzinfo]]]]])

date.today()
# datetime.date(2019, 5, 25)

datetime.now()
# datetime.datetime(2019, 5, 25, 22, 15, 49, 423978)
datetime.utcnow()
# datetime.datetime(2019, 5, 27, 14, 16, 28, 998506)

datetime.strptime('2019-5-25 20:25','%Y-%m-%d %H:%M')
# datetime.datetime(2019, 5, 25, 20, 25)
datetime.strftime("%Y-%m-%d %H:%M:%S")
# '2019-05-25 22:19:28'

a = datetime.datetime.now()
datetime.datetime.combine(a.date(),a.time())
# 将date 和 time 合并为 datetime
```


```timedelta``` 日期/时间差
```python
from datetime import timedelta

# timedelta([days[, seconds[, microseconds[, milliseconds[, minutes[, hours[, weeks]]]]]]])

td = timedelta(days=30)

td.days()
# 30

td.total_seconds()
# 2592000.0 == 30 * 3600 * 24

str(td)
# '30 days, 0:00:00'

# timedelta 支持数学运算
# + - * // abs 正 负
```

```tzinfo``` 时区信息



## calendar 模块

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import calendar
 
cal = calendar.month(2019, 5)
print("以下输出2019年5月份的日历:")
print(cal)
#       May 2019
# Mo Tu We Th Fr Sa Su
#        1  2  3  4  5
#  6  7  8  9 10 11 12
# 13 14 15 16 17 18 19
# 20 21 22 23 24 25 26
# 27 28 29 30 31

timegm(tuple)
# 将datetime 元组格式转换成 时间戳格式

calendar.isleap(year)
# 判断是否为闰年，返回bool值

calendar.leapdays(y1, y2)
# 返回y1 至 y2(不包括) 年份之间包含的闰年数

calendar.firstweekday()
# 查询当前设置的一周的第一天

calendar.weekday(year, month, day)
# 返回给定 年月日 是星期几
```