---
title: Linux共享库(shared library)的那些事
date: 2019-08-30 19:00:49 +0800
categories: linux
tags: linux library

---

这篇文章起源于最近在Python(2.7.9)中使用[ijson](https://pypi.org/project/ijson/)和[yajl](https://lloyd.github.io/yajl/)库的时候遇到的一个问题。ijson是一个iterative JSON parser，它与standard library中json模块的区别在于它不会把整个文件load到内存中，这样可以显著减小程序的memory footprint。它们的关系可以类比于处理XML文件时的DOM和SAX parser。

ijson可以使用不同的的backend，例如pure Python或者yajl。由于yajl的速度会更快，所以我希望ijson使用它作为backend

```python
import ijson.backends.yajl2 as ijson
```

但是在执行过程中报了一个错，是关于yajl库缺少某个方法的，原来服务器上的yajl库是1.x的某个古老版本，该版本缺少那个必要的方法。在找到一个2.x版本的yajl库后（公司内不允许直接从外部下载程序，因此只能使用公司已经存在的库），我使用了`export LD_LIBRARY_PATH=/path/to/yajl2`，因为我之前写C/C++程序时遇到过找不到正确.so文件的问题，知道这样可以解决。可惜在我重新运行程序后依然无效，似乎它对我刚刚的努力无动于衷，程序找到的还是那个1.x版本的库。

无奈我只好从ijson的源代码中找寻蛛丝马迹。

```python
# backends/yajl2.py
yajl = backends.find_yajl_ctypes(2)

# backends/__init__.py
from ctypes import util, cdll
so_name = util.find_library('yajl')
if so_name is None:
	raise YAJLImportError('YAJL shared object not found.')
try:
	yajl = cdll.LoadLibrary(so_name)
except OSError:
	raise YAJLImportError('Unable to load YAJL.')
```

[ctypes](https://docs.python.org/2/library/ctypes.html)是Python标准库的一部分，它为我们从Python代码中调用其他语言，例如C编写的shared library提供帮助。可以看出我们希望`util.find_library`能通过`yajl`这个名字找到正确的.so文件。

事情到这里变得有趣起来。

```python
# 以下代码摘自Python 2.7.9源码

def _findSoname_ldconfig(name):
    import struct
    if struct.calcsize('l') == 4:
        machine = os.uname()[4] + '-32'
    else:
        machine = os.uname()[4] + '-64'
    mach_map = {
        'x86_64-64': 'libc6,x86-64',
        'ppc64-64': 'libc6,64bit',
        'sparc64-64': 'libc6,64bit',
        's390x-64': 'libc6,64bit',
        'ia64-64': 'libc6,IA-64',
        }
    abi_type = mach_map.get(machine, 'libc6')

    # XXX assuming GLIBC's ldconfig (with option -p)
    expr = r'\s+(lib%s\.[^\s]+)\s+\(%s' % (re.escape(name), abi_type)
    f = os.popen('/sbin/ldconfig -p 2>/dev/null')
    try:
        data = f.read()
    finally:
        f.close()
    res = re.search(expr, data)
    if not res:
        return None
    return res.group(1)

def find_library(name):
    return _findSoname_ldconfig(name) or _get_soname(_findLib_gcc(name))
```

从这里我们可以看出来，`util.find_library`会使用yajl作为参数调用`_findSoname_ldconfig`，进而使用`ldconfig -p`的输出进行正则表达式的匹配

```bash
# 为方便大家，我这里贴上我笔记本Ubuntu系统中`ldconfig -p`的一行输出
    libyajl.so.2 (libc6,x86-64) => /lib/x86_64-linux-gnu/libyajl.so.2
```

可以很容易看出，最后会把字符串`libyajl.so.2`返回，也就是作为`so_name`。这个`so_name`会被传入`cdll.LoadLibrary`，载入真正的yajl库。在我的笔记本上运行结果是正常的，但由于服务器上`ldconfig -p`会返回1.x版本的yajl库的位置，自然也就出错了。

到这里我们已经了解到了许多信息，我们知道在载入shared library的过程中有`so name`，`ldconfig`，以及`cdll`等的参与。等等，怎么没有`LD_LIBRARY_PATH`呢？事实上也有人提出了相同的问题。在较新版本的Python中（如2.7.16，3.7.3，这是我机器上安装了的两个Python版本），在`find_library`的注释里有一行`# See issue #9998`，大家可以自己打开[Issue 9998](https://bugs.python.org/issue9998)看一看。

这个Issue的主旨就是为什么`find_library`没有考虑`LD_LIBRARY_PATH`。有人提出在文档中已经说明，`find_library`的目的就是像编译器（准确的说是linker, ld）一样找出库文件的名字（例如你使用gcc编译时传入-lm，gcc会找到libm.so.6）

> The purpose of the find_library() function is to locate a library in a way similar to what the compiler does (on platforms with several versions of a shared library the most recent should be loaded), while the ctypes library loaders act like when a program is run, and call the runtime loader directly.


我可以看到在3.7.3的`find_library`中已经加入了对`LD_LIBRARY_PATH`的搜索（Python 2中依然没有）而且文档中对于这个方法的描述也有了变化

> The purpose of the find_library() function is to locate a library in a way similar to what the compiler or runtime loader does

不过它是在最后才进行的，和系统loader载入shared library的顺序还是不太一样。

我觉得对问题的背景介绍应该在此打住，毕竟我们这篇文章的主题是关于Linux中shared library本身的，而不是如何在Python extension中加载这些shared library的。

## Shared Library基础知识

### 什么是Shared Library

### 使用Shared Library

### 创建自己的Shared Library


## Shared Library载入顺序

### LD_LIBRARY_PATH
### 编译时指定路径(compiled-in path)
### 系统ld.so搜索路径


