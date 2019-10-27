---
title: 为多个GitHub账户设置SSH Key
date: 2019-10-27 09:00:49 +0800
categories: git
tags: linux github git
---

在使用GitHub管理代码时，我们可能会遇到这样的情况：我有两个甚至更多的GitHub账户，我希望让它们全都能通过SSH pull/push代码，应该怎么做？
如果耿直的你直接把自己`~/.ssh/id_rsa.pub`中的内容添加到第二个账户的SSH keys中，你会发现GitHub提醒你，这个Key已经被使用了。今天我就来向你介绍一下如何正确进行设置。

### 生成第二个SSH Key
既然那个Key已经被用了，那就再来一个吧
```bash
$ ssh-keygen -t rsa -C "your comment"
```

这里通过`-C`参数填入一个注释，让你之后可以分辨不同的key。根据提示，输入一个新的绝对路径文件名（**注意**，这里不要直接按回车，否则会把`~/.ssh/id_rsa`和`~/.ssh/id_rsa.pub`替换掉），例如`/home/drizzlex/.ssh/id_rsa_test`。随后的步骤就和你之前生成SSH Key的步骤一样了。

当你发现在`~/.ssh`目录下多了两个文件，你的新Key就生成好了。

### 在GitHub中设置新Key
这一步很简单，打开你的个人账户设置，然后在SSH and GPG Keys中添加新的Key就好了。记得取一个帮助你记忆的名字。

### 配置`~/.ssh/config`
如果该文件不存在的话，就新建一个，然后在里面填入
```bash
#primary account
Host github.com
	HostName github.com
	User git
	IdentityFile ~/.ssh/id_rsa

#secondary account
Host github.com-test
	HostName github.com
	User git
	IdentityFile ~/.ssh/id_rsa_test

```
配置的内容不难理解，我在这里配置了两个账户，一个主账户(primary account)和一个辅助账户(secondary account)。使用主账户时，和之前没有任何区别，而使用辅助账户的时候，需要对remote URL做一个修改，也就是你在`git clone`的时候，需要把`git@github.com`换成`git@github.com-test`（因为我这里Host设置的是`github.com-test`，你需要根据自己的设置做相应变更。

### 小结
至此我们就在同一台机器上配置了两个SSH Key，并让它们服务于两个GitHub账户。希望本文对你有所帮助！