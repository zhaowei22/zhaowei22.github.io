---
title: python读取大文件
tags:
  - python
categories: 学习笔记
abbrlink: 1232847187
date: 2019-01-19 21:43:28
updated: 2019-02-15 22:22:22
---
## python 读取大文件

python读取文件一般情况是利用open()函数以及read()函数来完成：

```
f = open(filename,'r')
f.read()
```
这种方法读取小文件，即读取大小远远小于内存的文件显然没有什么问题。但是如果是将一个10G大小的日志文件读取，即文件大小大于内存，这么处理就有问题了，会造成MemoryError ... 也就是发生内存溢出。

这里发现跟read()类似的还有其他的方法：read(参数)、readline()、readlines()
### read(参数)：
通过参数指定每次读取的大小长度,这样就避免了因为文件太大读取出问题。

```
while True:
    block = f.read(1024)
    if not block:
        break
```
### readline()：
每次读取一行

```
while True:
    line = f.readline()
    if not line:
        break
```
### readlines()：
读取全部的行，构成一个list，通过list来对文件进行处理，但是这种方式依然会造成MemoyError

```
for line in f.readlines():
    ....
```
### 分块读取：
处理大文件是很容易想到的就是将大文件分割成若干小文件处理，处理完每个小文件后释放该部分内存。这里用了iter 和 yield：

```
def read_in_chunks(filePath, chunk_size=1024*1024):
    """
    Lazy function (generator) to read a file piece by piece.
    Default chunk size: 1M
    You can set your own chunk size
    """
    file_object = open(filePath)
    while True:
        chunk_data = file_object.read(chunk_size)
        if not chunk_data:
            break
        yield chunk_data
if __name__ == "__main__":
    filePath = './path/filename'
    for chunk in read_in_chunks(filePath):
        process(chunk) # <do something with chunk>
```
### 使用With open()：
with语句打开和关闭文件，包括抛出一个内部块异常。for line in f文件对象f视为一个迭代器，会自动的采用缓冲IO和内存管理，所以你不必担心大文件

```
# If the file is line based
with open(...) as f:
　　for line in f:
　　　　process(line) # <do something with line>
```

##### 关于with open()的优化：
面对百万行的大型数据使用with open 是没有问题的，但是这里面参数的不同也会导致不同的效率。经过测试发现参数为"rb"时的效率是"r"的6倍。由此可知二进制读取依然是最快的模式。

```
with open(filename,"rb") as f: 
  for fLine in f: 
  　　pass 
```
测试结果：rb方式最快，100w行全遍历2.9秒。基本能满足中大型文件处理效率需求。如果从rb(二级制读取)读取改为r(读取模式)，慢5-6倍。
### linecache模块
以上几种方式都不支持对于文件按行随机访问。在这样的背景下，能够支持直接访问某一行内容的linecache模块是一种很好的补充。
我们可以使用linecache模块的getline方法访问某一具体行的内容，官方文档中给出了如下用法：

```
import linecache
linecache.getline('filename', 6)
```
在使用过程中我注意到，基于linecache的getline方法的日志分析会在跑满CPU资源之前首先占用大量内存空间，也就是在CPU使用率仍然很低的情况下，内存空间就会被迅速地消耗。
这一现象引起了我的兴趣。我猜测linecache在随机读取文件时，是首先依序将文件读入内存，之后寻找所要定位的行是否在内存当中。若不在，则进行相应的替换行为，直至寻找到所对应的行，再将其返回。
对linecache代码的阅读证实了这一想法。
在linecache.py中，我们可以看到getline的定义为：

```
def getline(filename, lineno, module_globals=None):
    lines = getlines(filename, module_globals)
    if 1 <= lineno <= len(lines):
        return lines[lineno-1]
    else:
        return ''
```
不难看出，getline方法通过getlines得到了文件行的List，以此来实现对于文件行的随机读取。继续查看getlines的定义。

```
def getlines(filename, module_globals=None):
    """Get the lines for a file from the cache.
    Update the cache if it doesn't contain an entry for this file already."""

    if filename in cache:
        return cache[filename][2]
    else:
        return updatecache(filename, module_globals)
```
由此可见，getlines方法会首先确认文件是否在缓存当中，如果在则返回该文件的行的List，否则执行updatecache方法，对缓存内容进行更新。因此，在程序启动阶段，linecache不得不首先占用内存对文件进行缓存，才能进行后续的读取操作。
而在updatecache方法中，我们可以看到一个有趣的事实是：

```
def updatecache(filename, module_globals=None):
    """Update a cache entry and return its list of lines.
    If something's wrong, print a message, discard the cache entry,
    and return an empty list."""

    ## ... 省略...

    try:
        fp = open(fullname, 'rU')
        lines = fp.readlines()
        fp.close()
    except IOError, msg:
##      print '*** Cannot open', fullname, ':', msg
        return []
    if lines and not lines[-1].endswith('\n'):
        lines[-1] += '\n'
    size, mtime = stat.st_size, stat.st_mtime
    cache[filename] = size, mtime, lines, fullname
    return lines
```
也就是说，linecache依然借助了文件对象的readlines方法。这也给了我们一个提示，当文件很大不适用readlines方法直接获取行的List进行读取解析时，linecache似乎也并不会成为一个很好的选择。


### 内存检测工具介绍
#### memory_profiler
首先先用pip安装memory_profiler

```
pip install memory_profiler
```
memory_profiler是利用python的装饰器工作的，所以我们需要在进行测试的函数上添加装饰器。

```
from hashlib import sha1
import sys
@profile
def my_func():
    sha1Obj = sha1()
    with open(sys.argv[1], 'rb') as f:
        while True:
            buf = f.read(1024 * 1024)
            if buf:
                sha1Obj.update(buf)
            else:
                break
    print(sha1Obj.hexdigest())
if __name__ == '__main__':
    my_func()
```
之后在运行代码时加上** -m memory_profiler**

就可以了解函数每一步代码的内存占用了
![image](https://upload-images.jianshu.io/upload_images/8552201-762556af60ff1b61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### guppy
首先，通过pip先安装guppy

```
pip install guppy
```
之后可以在代码之中利用guppy直接打印出对应各种python类型（list、tuple、dict等）分别创建了多少对象，占用了多少内存。

```
from guppy import hpy
import sys
def my_func():
    mem = hpy()
    with open(sys.argv[1], 'rb') as f:
        while True:
            buf = f.read(10 * 1024 * 1024)
            if buf:
                print(mem.heap())
            else:
                break
```
通过上述两种工具guppy与memory_profiler可以很好地来监控python代码运行时的内存占用问题。


---

### 参考链接：

[Python读取大文件的"坑“与内存占用检测](https://www.pythontab.com/html/2018/pythonhexinbiancheng_0828/1340.html)

[使用Python读取大文件的方法](https://www.jb51.net/article/135002.htm)

[python linecache读取过程
](https://www.cnblogs.com/wennn/p/6725194.html?utm_source=itdadao&utm_medium=referral)