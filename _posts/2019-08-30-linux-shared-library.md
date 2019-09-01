---
title: Linux共享库(shared library)的那些事
date: 2019-08-30 19:00:49 +0800
categories: linux
tags: linux library

---

## 缘起

我首先想介绍一下为什么会写这篇文章，我十分建议你了解一下背景故事，因为这对于你认清全局会很有帮助。在这一章节的结尾我还谈到了一点对于学习计算机知识的经验和感悟。如果你时间有限，可以跳过这一部分，直接进入shared memory相关的章节。

这篇文章起源于最近在Python(2.7.9)中使用[ijson](https://pypi.org/project/ijson/)和[yajl](https://lloyd.github.io/yajl/)库的时候遇到的问题。ijson是一个iterative JSON parser，它与standard library中json模块的区别在于它不会把整个文件load到内存中，这样可以显著减小程序的memory footprint。它们的关系可以类比于处理XML文件时的DOM和SAX parser。

ijson可以使用不同的的backend，例如pure Python或者yajl。由于yajl的速度会更快，所以我希望ijson使用它作为backend

```python
import ijson.backends.yajl2 as ijson
```

在执行过程中报了一个错，是关于yajl库缺少某个方法的。原来公司服务器上的yajl库是1.x的某个古老版本，该版本缺少那个必要的方法。在找到一个2.x版本的yajl库后（公司不允许直接从外部下载程序，因此只能使用公司已经存在的库版本），我使用了`export LD_LIBRARY_PATH=/path/to/yajl2`，因为我之前写C/C++程序时遇到过找不到正确.so文件的问题，知道这样可以解决。可惜在我重新运行程序后依然报同样的错，似乎它对我刚刚的努力无动于衷。

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

[ctypes](https://docs.python.org/2/library/ctypes.html)是Python标准库的一部分，它为我们从Python代码中调用其他语言，例如C编写的shared library提供帮助。可以看出我们希望`util.find_library`能通过yajl这个名字找到正确的so（shared object）文件。

事情到这里变得有趣起来。让我们看看`find_library`是怎么实现的

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

从这里我们可以看出来，`util.find_library`会使用yajl作为参数调用`_findSoname_ldconfig`，进而使用`ldconfig -p`的输出进行正则表达式匹配

```bash
# 为方便大家，我这里贴上我笔记本Ubuntu系统中`ldconfig -p`的一行输出
# 公司服务器中的输出和我的不同，那里会找到一个.so.1
    libyajl.so.2 (libc6,x86-64) => /lib/x86_64-linux-gnu/libyajl.so.2
```

可以看出，如果在我的笔记本上运行，最后会把字符串`libyajl.so.2`返回，也就是作为`so_name`。这个`so_name`会被传入`cdll.LoadLibrary`，载入真正的yajl库。在我的笔记本上运行结果是正常的，但由于服务器上`ldconfig -p`会返回1.x版本的yajl库的位置，自然也就出错了。

到这里我们已经了解到了许多信息，我们知道在载入shared library的过程中涉及到`so name`，`ldconfig`，以及`cdll`等的参与。等等，怎么没有`LD_LIBRARY_PATH`呢？这一点和我的预期有所出入。事实上也有人提出了相同的问题。在较新版本的Python中（如2.7.16，3.7.3，这是我机器上安装了的两个Python版本），在`find_library`的注释里有一行`# See issue #9998`，大家可以自己打开[Issue 9998](https://bugs.python.org/issue9998)看一看。

这个Issue的主旨就是为什么`find_library`没有考虑`LD_LIBRARY_PATH`。在回贴中有人提出文档已经说明，`find_library`的目的就是像编译器（准确的说是linker）一样找出库文件的名字（例如你使用gcc编译时传入-lm，gcc会找到libm.so.6）。我们来看下文档中的说法(Python 2)

> The purpose of the find_library() function is to locate a library in a way similar to what the compiler does (on platforms with several versions of a shared library the most recent should be loaded), while the ctypes library loaders act like when a program is run, and call the runtime loader directly.

按照文档的意思，`find_library`的行为会像compile-time linker那样（在Python 3.6以后有所变化，我下面有说到）。我对这一点感到有些惊讶，因为**代码实现中使用`ldconfig -p`来寻找库的名字，但是如果我们看一下ldconfig的man page，会发现ldconfig的作用是configure dynamic linker run-time bindings**。因此我对这一实现其实**依然不太理解**。

在我看来，`find_library`**原本的目标是通过一个传给linker的-l参数那样，找到一个库的so name，就像文档中的代码示例那样。但是它的实现却没有优先使用ld -t的输出，而是使用了ldconfig，这在我看来很奇怪。**

我们只能暂且假设`find_library`如其所说，和compile-time linker表现一致，忽略它的代码实现。文档中又提到

> If wrapping a shared library with ctypes, it may be better to determine the shared library name at development time, and hardcode that into the wrapper module instead of using find_library() to locate the library at runtime.

在有上述假设的情况下，看来是ijson库的`find_yajl_ctypes`这个方法的实现有些问题。它不应该使用`so_name = util.find_library('yajl')`来寻找shared library的so name，而应该把名字写死，这样`cdll.LoadLibrary(so_name)`就可以打开正确的so文件了（背后使用了`dlopen`）。

但ijson的作者为何没有那么写呢？正如在Issue 9998中一些人提出的那样，如果写死（限定使用yajl2，即cdll.LoadLibrary('libyajl.so.2')），更换一个平台就无法找到正确的文件了，因为并不是所有平台的shared library都是libxxx.so.N这个格式的，也就是说保证跨平台性的任务就被迁移到了ijson的开发者身上。

帖子中的讨论还是挺有趣的，从这些讨论中我们可以真实地看到由于各个平台之间的割裂，导致跨平台开源编程语言或软件开发的复杂性。

在经过Issue 9998的讨论后，3.6以后的版本已经加入了对`LD_LIBRARY_PATH`的搜索（把其中的value作为-L参数传入ld，对ld的输出进行regex匹配，这里我**觉得也很奇怪**，把`LD_LIBRARY_PATH`中的值传入ld真的是正确的做法吗？我为什么会这么问，你看完这篇文章应该就懂了），而且文档中对于这个方法的描述也有了变化

> The purpose of the find_library() function is to locate a library in a way similar to what the compiler or runtime loader does

不过这一搜索是在最后才进行的，即其他途径中都没有找到这个库的时候才会做，这和dynamic linker/loader载入shared library的顺序还是不太一样。而且即便有了这一变化，依然无法解决我最开始遇到的问题，因为此时`find_library`会优先使用`ldconfig -p`的输出，而它会返回旧版本，`LD_LIBRARY_PATH`中的值会被忽略掉。有兴趣的话可以看一下Python的源码。

我觉得背景介绍应该在此打住，毕竟我们这篇文章的主题是关于Linux中shared library本身的，而不是如何在Python extension中使用这些shared library。

不过，在开始正题之前容我再谈两句自己的感想。一些看似已经过时或者过于底层的知识，对于当今使用高级语言进行编程的我们依旧不可或缺。现在对青少年进行编程教育，大多会使用Python或者Java这样的高级语言（C虽然也算高级语言，但是显然不被包括其中）甚至图形化编程游戏。我认为这对激发他们的兴趣很有意义，因为这样一来在较短的时间内他们就可以得到一个不错的成果，毕竟一个具有图形界面的小游戏要比黑底白字的Hello World有趣多了。但是如果他们有意将这一领域作为职业的话，对包括操作系统（特别是Unix-like系统），编译器以及稍微底层一些的C语言的了解依旧必不可少，因为在这些知识都是计算机的基础，而且其中包含许多在这一领域的范式或者习惯。这些知识可以让他们在之后少走许多弯路。只有根基牢固，才能让高楼耸立，不然它就只是一座精美的沙堡，无法经受海浪涛涛。

而我更想说的一点是，如果正在阅读文章的你刚刚接触计算机这一领域，那么请你努力把这篇文章中提到的**每一点都弄清楚**，最好自己做下笔记。我之所以这样说，完全来自自己的切身体会。

在我上大学时，操作系统这门课（计算机操作系统，汤子瀛）中其实对shared library有简单的介绍，但是里面的知识过于陈旧，例如文中提到“近几年流行起来的运行时动态链接方式”，我不知道你会怎么想，我会认为这件事发生最多不到十年，但这显然与事实不符。在随后的编程中我遇到了动态库的一些问题，但是**课本中的抽象描述只能让我知道它们是相关的，对于如何解决手头的实际问题却没有太大帮助**，因此我不得不在网上疯狂搜索。

可是正如计算机领域的许多问题一样，**shared library涉及的知识点太过庞杂**。这是学习计算机知识中最让人头疼的一点，因为你总是在学习A的时候冒出来一个B，然后你发现不了解B的话就无法了解A（或者无法完全了解），因此你转而学习B。最后导致的结果就是

1. 忽略B，对A拥有一个大致的了解，随着时间迁移，忘记A是怎么回事，因为你并没有把问题真正弄懂
1. 学习B，但是发现B又涉及其他知识，从而形成了“递归学习”，真正实现了“学无止境”

这都是让人沮丧的结果。那我们应该怎么办呢？我们需要做的是，了解“足够的B以及它所涉及的知识”。**那么问题来了，如何判断多少算“足够”？**，对于一个缺乏经验的人来说其难度并没有减少，因为如果你没有去看某些知识，你怎么知道它是否必要？

这就是为什么我们需要前人的经验。因为他们遭遇过和你一样的问题，他们就知道多少知识算是“足够”的，学习这些被验证过的知识能让你对所面临的问题有一个较为全面的理解，又不至于陷入茫茫大海无法自拔。

而这篇文章就是希望为你提供**足够多的关于shared library**的知识。

## Shared Library基础知识

### 什么是Shared Library

### 使用Shared Library

### 创建自己的Shared Library


## Shared Library载入顺序

### LD_LIBRARY_PATH
### 编译时指定路径(compiled-in path)
### 系统ld.so搜索路径


