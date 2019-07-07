---
title: 使用Jekyll在GitHub Pages上创建自己的博客
date: 2019-07-06 12:40:49 +0800
categories: jekyll
---

## 安装与配置开发环境

我们这里将在Ubuntu 19.04上构建开发环境。

### 安装Ruby以及Jekyll

Ubuntu仓库中的Ruby版本较低，因此我们将使用[RVM(Ruby Version Manager)](http://rvm.io/)安装Ruby。首先要安装gnupg2以及curl，然后添加GPG keys并且安装RVM。紧接着就可以使用RVM安装指定版本的Ruby。

```bash
$ sudo apt install gnupg2 curl
$ gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
$ \curl -sSL https://get.rvm.io | bash -s stable
$ rvm install ruby-2.6
```

rvm会帮你安装相关的依赖，并安装ruby。安装结束后，你可以尝试跑一下`rvm use ruby`，你会看到一些错误提示信息，要求你allow login shell。不同的Terminal设置可能不同，在Ubuntu，你可以这样做：`Profile`->`Command`->`Run command as a login shell`。重新打开你的Terminal，查看下自己的ruby和gem命令是否可以正常运行了：

```bash
$ ruby -v
$ gem -v
```

接下来就是主角登场的时间了，执行以下命令，你的网站就建好并且启动了！

```bash
$ gem install bundler jekyll
$ jekyll new my-awesome-site
$ cd my-awesome-site
$ bundle exec jekyll serve
```

打开浏览器，访问[http://localhost:4000](http://localhost:4000)。

### Jekyll的基本配置

### 写一篇图文并茂的博客

接下来，我们开始写第一篇博客了！在这篇博客中，我们将会使用各种Markdown语法。

```python
def my_function():
    print('Hi, this is my function')
```
