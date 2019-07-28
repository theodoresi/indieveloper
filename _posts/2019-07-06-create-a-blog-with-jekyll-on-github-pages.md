---
title: 使用Jekyll在GitHub Pages上创建自己的博客
date: 2019-07-06 12:40:49 +0800
categories: jekyll
---
## 为何使用Jekyll + GitHub Pages搭建博客

创建自己的博客有各种各样的途径。入门级玩家可以直接使用新浪、网易、CSDN或者cnblogs等平台提供的服务。这些服务的优点在于它们提供了十分成熟，稳定的博客功能，你只需要注册一个账号，就可以与所有人分享你的知识和观点，而且由于使用人数众多，相对来说你的文章更可能被别人看到。但是它们的缺陷也是显而易见的，例如广告众多，只能使用服务提供商的域名，无法灵活配置页面样式，无法增加定制化功能等等。

如果你是硬核玩家，则可以购买VPS或者Webhosting服务，然后使用WordPress这样的软件来打造自己的个人网站。这一方案能为你提供非常强大的扩展性，你可以让你的博客拥有任何你所想要的功能，但是你必须为之支付一定的费用。而且更重要的是，你需要为网站性能以及安全负责，这些事务会极大地分散你的精力，如果你没有相关经验，那么在你发表第一篇博客之前可能已经想放弃了。

而Jekyll + GitHub Pages则在上述两种方案之间找到了平衡：

* 你只需新建一个GitHub repo，其中的文件就是你的网站内容。例如你在repo中存储一个index.html，则用户在访问你的网站时，其浏览器显示的就是这个文件的内容。
* 使用Jekyll，你只要以Markdown格式写博客，Jekyll就可以将其转换成HTML页面。
* 通过`git push`命令你就可以把本地网站文件推送到GitHub，随后GitHub会使用Jekyll对网站源文件进行“编译”，生成实际的HTML文件。
* Jekyll支持模板，你可以使用、修改、甚至创建模板，让自己的博客与众不同。
* GitHub Pages支持自定义域名，这样更有利于你打造个人品牌。
* 完全免费。

## 从简单的事情开始——使用GitHub Pages创建一个网站

[GitHub Pages](https://pages.github.com/)是一个静态网站托管服务，你可以[在这里找到官方文档](https://help.github.com/en/categories/github-pages-basics)。接下来我们就通过简单的几个步骤，实践一下GitHub Pages版的Hello World：

1. 用你的GitHub账户创建一个新的**public** repo，名字是`<your_user_name>.github.io`，其中`<your_user_name>`必须是你的用户名。由于我们随后要使用Jekyll来构建网站，所以我建议在`Add .gitignore`那里选择Jekyll，这样Jekyll编译生成的副产品就不会被同步到我们的repo中。
1. 将repo克隆到本地，然后在其中创建一个index.html，写入任意内容后push到GitHub。
1. 稍等片刻，打开`https://<your_user_name>.github.io`，你会发现自己的网站已经可以访问了！


## 安装与配置开发环境

事实上，如果你愿意写HTML，或者以纯文本方式展示你的文章，你的个人博客已经构建完成了！但是那显然是一件非常痛苦的事情（不论对你还是对你的读者而言，都是一种非人道主义行为），因此我们需要Jekyll帮助我们构建更加美观，易用的网站。Jekyll是一个静态页面生成器，你可以在[这里](https://jekyllrb.com/docs/)了解更多关于Jekyll的信息。

接下来我们就开始安装Jekyll，这一步骤还是有点挑战的，尤其是如果你像我一样对Ruby没有任何使用经验，对其生态环境也不了解的话。不过坑我已经帮你踩过了，只要你紧随我的脚步，相信你一定能顺利度过雷区，开启自己的写作生涯。

我们这里介绍两个平台上的安装方法，Windows 10和Linux(Ubuntu 19.04/Manjaro 18.04)，Mac用户遵照Linux的方式应该也能轻松完成安装。

### 安装Ruby以及Jekyll

#### Linux

Ubuntu仓库中的Ruby版本较低，因此我们将使用[RVM(Ruby Version Manager)](http://rvm.io/)安装Ruby。首先要安装gnupg2以及curl，然后添加GPG keys并且安装RVM。紧接着就可以使用RVM安装指定版本的Ruby。

```bash
# 安装后面可能用到的工具，build-essential或base-devel的作用在于
# 如果rvm无法找到对应当前平台的ruby二进制文件，就会用这些工具对源码进行编译安装
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
$ gem install bundler jekyll
$ jekyll -v
```

#### Windows 10

Windows不是Jekyll官方正式支持的平台，但是通过一些“小技巧”还是可以在Windows上使用Jekyll的。详情可以参考[Jekyll on Windows](https://jekyllrb.com/docs/installation/windows/)。

1. 下载安装[Rubyinstaller](https://rubyinstaller.org/downloads/)
1. 在安装的最后一步，运行`ridk install`
1. 安装结束后，打开新的命令行窗口（在开始菜单中搜索ruby，找到`Start Command Prompt with Ruby`），运行`gem install jekyll bundler`
1. 测试jekyll是否正常`jekyll -v`


### 生成一个Jekyll网站

接下来就是主角登场的时间了，执行以下命令，我们将在本地创建一个基于Jekyll的网站：

```bash
$ jekyll new my-awesome-site
$ cd my-awesome-site
$ bundle exec jekyll serve
```

打开浏览器，访问[http://localhost:4000](http://localhost:4000)，你应该看到一个Jekyll为你生成的网站。


## 强强联合——使用Jekyll在GitHub Pages上的内容创建网站

假设你之前克隆的repo存在`${MY_SITE}`目录下，则你可以将`my-awesome-site`中的文件移动到`${MY_SITE}`，并且移除你之前创建的index.html（这一步很重要，否则你访问时GitHub Pages还是会显示这个文件的内容），然后push到GitHub。稍等片刻后再次打开你的首页，你会发现它的内容已经变了。

接下来你就可以在`_posts`目录下创建新的`.markdown`或者`.md`文件来写作博客了。这里需要注意的是，文件名应该遵守一定的格式，也就是以日期开始，紧接着你的标题。具体格式可以参考新建项目时Jekyll自动生成的那个`.markdown`文件。


## 总结

到此为止，你就学会了如何使用Jekyll来创建一个博客网站，并且将其部署到GitHub Pages上，但是我们的征程还远未结束。我们的网站看起来还太过简陋，需要使用主题对其进行美化。

而且你现在心里一定充满了疑问：

* 我应该怎么对所有的文章进行整理归类
* 怎么在文章中插入图片
* 我可以生成一些非博客页面，例如一些自我介绍吗
* 怎么使用自己的域名取代`github.io`

大家别急，我接下来就会对这些内容一一做出解答。这里给大家留一个小的作业，聪明的你一定能自己解决：Jekyll默认使用的主题是`minima`，请你尝试找到一个自己喜欢的主题，并且进行替换(Hint: 在你的GitHub repo中的Settings里找找看)。