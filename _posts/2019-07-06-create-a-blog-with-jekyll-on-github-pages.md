---
layout: post
title:  "使用Jekyll在Github Pages上创建自己的博客"
date:   2019-07-06 12:40:49 +0800
categories: jekyll
---


# 安装与配置开发环境

我们这里将在Ubuntu 19.04上构建开发环境。

## 安装Ruby以及Jekyll

Ubuntu仓库中的Ruby版本较低，因此我们将使用[RVM(Ruby Version Manager)](http://rvm.io/)安装Ruby。首先要安装gnupg2以及curl，然后添加GPG keys并且安装RVM。紧接着就可以使用RVM安装指定版本的Ruby。

```Bash
$ sudo apt install gnupg2 curl
$ gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
$ \curl -sSL https://get.rvm.io | bash -s stable
$ rvm install ruby-2.6
```

rvm会帮你安装相关的依赖，并安装ruby。至此，你可以查看下自己的Ruby和Gem版本
```Bash
$ ruby -v
$ gem -v
```

如果上述命令可以正常运行，那么你的Ruby就装好了。

接下来就是主角登场的时间了，执行以下命令，你的网站就建好并且启动了！

```Bash
$ gem install bundler jekyll
$ jekyll new my-awesome-site
$ cd my-awesome-site
$ bundle exec jekyll serve
```

打开浏览器，访问[http://localhost:4000](http://localhost:4000)
