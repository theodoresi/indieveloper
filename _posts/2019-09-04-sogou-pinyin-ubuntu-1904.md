---
title: 在Ubuntu 19.04安装使用搜狗拼音输入法
date: 2019-09-04 09:00:49 +0800
categories: linux
tags: pinyin linux ubuntu
---

### 安装fcitx
Ubuntu自带的Input Method Framework是IBUS，我们需要首先将其替换为fcitx

```bash
sudo apt-get install fcitx-bin
sudo apt-get install fcitx-table
```

### 安装搜狗拼音
打开[官网](https://pinyin.sogou.com/linux/?r=pinyin)，下载和自己系统对应的deb包，然后安装

```bash
sudo dpkg -i <installation_package.deb>
```

### 配置fcitx
1. 打开Settings -> Region & Language -> Manage Installed Languages，把Keyboard input method system改成fcitx
1. 重启电脑
1. 系统托盘应该多了一个键盘图标，右键点击，打开configure
1. 点击+，去掉Only Show Current Language的钩，搜素sogou
1. 选中搜狗拼音，点击OK
1. 选中sogou，点击向上的箭头，把它移动到第一个位置，保存退出
1. 按Ctrl+空格就可以切换输入法了

### 可能遇到的问题
1. 无法使用shift切换中英文输入法
打开Region & Language的配置页面，然后点击拼音输入法的齿轮，在shortcut页中把Switch Chinese/English的快捷键去掉。点击状态栏的fcitx图标，点击restart,稍等片刻就好啦！
