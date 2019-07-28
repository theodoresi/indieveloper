---
title: 使用Jekyll在GitHub Pages上创建自己的博客
date: 2019-07-06 12:40:49 +0800
categories: jekyll
---

## 安装与配置开发环境

我们这里介绍两个平台，Windows 10和Linux(以Ubuntu 19.04/Manjaro 18.04为例)。

### 安装Ruby以及Jekyll

#### Linux

Ubuntu仓库中的Ruby版本较低，因此我们将使用[RVM(Ruby Version Manager)](http://rvm.io/)安装Ruby。首先要安装gnupg2以及curl，然后添加GPG keys并且安装RVM。紧接着就可以使用RVM安装指定版本的Ruby。

```bash
# 安装后面可能用到的工具，build-essential或base-devel的作用在于，如果rvm无法找到对应当前平台的ruby二进制文件会用源码进行编译
$ sudo apt install gnupg2 curl build-essential # Manjaro中则使用 sudo pacman -Syu gnupg curl base-devel
$ gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
$ \curl -sSL https://get.rvm.io | bash -s stable
# 如果上述步骤出错，请根据错误提示跑相应的命令
# 如果成功，则会提示你
# * To start using RVM you need to run `source /home/drizzlex/.rvm/scripts/rvm`
#     in all your open shell windows, in rare cases you need to reopen all shell windows.
$ source /home/drizzlex/.rvm/scripts/rvm
$ rvm install ruby-2.6
```

rvm会帮你安装相关的依赖，并安装ruby。安装结束后，你可以尝试跑一下`rvm use ruby`，你会看到一些错误提示信息，要求你allow login shell。不同的Terminal设置可能不同，如果你使用的是Ubuntu(Gnome Terminal)，你可以这样做：`Profile`->`Command`->`Run command as a login shell`。重新打开你的Terminal，查看下自己的ruby和gem命令是否可以正常运行了：

```bash
$ ruby -v
$ gem -v
```

接下来就是主角登场的时间了，执行以下命令，我们将在本地创建一个基于jekyll的网站：

```bash
$ gem install bundler jekyll
$ jekyll new my-awesome-site
$ cd my-awesome-site
$ bundle exec jekyll serve
```

打开浏览器，访问[http://localhost:4000](http://localhost:4000)。

#### Windows 10

Windows不是官方正式支持的平台，但是通过一些“小技巧”还是可以在Windows上使用Jekyll的。详情可以参考[Jekyll on Windows](https://jekyllrb.com/docs/installation/windows/)。

1. 下载安装[Rubyinstaller](https://rubyinstaller.org/downloads/)
1. 在安装的最后一步，运行`ridk install`
1. 安装结束后，打开新的命令行窗口，运行`gem install jekyll bundler`
1. 测试jekyll是否正常`jekyll -v`

### Jekyll的基本配置

#### 使用模板

### 写一篇图文并茂的博客

接下来，我们开始写第一篇博客了！在这篇博客中，我们将会使用各种Markdown语法。

```python
def my_function():
    print('Hi, this is my function')
```
