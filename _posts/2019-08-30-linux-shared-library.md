---
title: 关于Linux共享库(shared library)的那些事
date: 2019-08-30 19:00:49 +0800
categories: linux
tags: linux library

---

这篇文章起源于最近在Python(2.7.9)中使用[ijson](https://pypi.org/project/ijson/)和[yajl](https://lloyd.github.io/yajl/)库的时候遇到的一个问题。

我希望ijson使用yajl2作为backend，于是使用了

```Python
import ijson.backends.yajl2 as ijson
```

但是在执行过程中报了一个错，是关于yajl库缺少某个方法的，原来服务器上的yajl库是1.x的某个古老版本，该版本缺少那个必要的方法。在找到一个2.x版本的yajl库后，我使用了`export LD_LIBRARY_PATH=/path/for/yajl2`，因为我之前写C/C++程序时遇到过找不到正确.so文件的问题，知道这样可以解决。在我重新运行程序后发现依然无效。

无奈我只好从ijson的源代码中找寻蛛丝马迹，`find_yajl_ctypes`方法中有如下两行，其中`util.find_library`是Python标准库中的方法

```python
from ctypes import util, cdll
so_name = util.find_library('yajl')
if so_name is None:
	raise YAJLImportError('YAJL shared object not found.')
try:
	yajl = cdll.LoadLibrary(so_name)
except OSError:
	raise YAJLImportError('Unable to load YAJL.')
```

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

到这里我们已经了解到了许多信息，我们知道在载入shared library的过程中有`so name`，`ldconfig`，以及`cdll`等的参与，但是等等，怎么没有`LD_LIBRARY_PATH`呢？事实上也有人提出了相同的问题。在较新版本的Python中（如2.7.16，3.7.3，这是我机器上安装了的两个Python版本），在`find_library`的注释里有一行`# See issue #9998`，大家可以自己打开[Issue 9998](https://bugs.python.org/issue9998)看一看。我可以看到在3.7.3的`find_library`中已经加入了对`LD_LIBRARY_PATH`的搜索，不过它是在最后才进行的，和系统loader载入shared library的顺序还是不太一样（上面Issue中有一些关于`find_library`应该表现的像build-time linker还是run-time loader的讨论，感兴趣的话可以看一下）。

不过我觉得对问题的背景介绍应该在此打住，毕竟我们这篇文章的主题是关于Linux中shared library本身的，而不是如何在Python extension中加载这些shared library的。

## Shared Library基础知识

### 什么是Shared Library

### 使用Shared Library

### 创建自己的Shared Library


## Shared Library载入顺序

### LD_LIBRARY_PATH
### 编译时指定路径(compiled-in path)
### 系统ld.so搜索路径


