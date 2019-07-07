---
title: 使用Vundle管理Vim插件
date: 2019-07-07 11:00:49 +0800
categories: vim
tags: vim plugin

---

Vim凭借其本身的功能，已经称得上地表最强大文本编辑器之一。在各种插件的加持下，它更是如有神助，让文本输入变成了一种让人享受的超然体验。

但是，正如人无完人，Vim也绝非完美，它自带的插件管理就不是很好用。好在有许多优秀的工具可以帮助我们完成这一工作。今天，我就将向大家介绍Vundle，一款优秀的Vim插件管理器。

# 茹毛饮血的手动管理方式

在我们直接步入工业4.0时代之前，大家还是有必要了解一下如何手动地安装Vim插件。正所谓忆苦方能思甜，了解了原本生活的艰辛，才会对美好的进步感恩。

我们这里使用[vim-markdown-quote-syntax](https://github.com/joker1007/vim-markdown-quote-syntax)这款插件作为示例。它为我们的Markdown文本提供了代码块高亮功能。在安装这个插件之前，如果我们打开一个含有代码块的`.md`文件，其显示效果是这样的：

![no-vim-markdown-quote-syntax](/assets/imgs/vundle-tutorial/no-vim-markdown-quote-syntax.png)

大家可以看到，我们嵌入的pythond代码泛着惨白的颜色，丝毫不能体现出这段代码的**高深和精妙**。因此，我们必须解决这一问题！

## 下载

点击上面的链接🔗，会跳转到插件的Github主页。以Zip压缩包的方式下载项目后，将其解压到`~/.vim/bundle/`目录中（这一目录只是约定俗成的路径，并非必须）。

## 配置

单纯把解压得到的文件夹放入`bundle`目录还不够，你需要编辑你的`runtimepath`，让其包含`~/.vim/bundle/vim-markdown-quote-syntax-master`。

1. `$ vim ~/.vimrc`
2. 增加一行`set runtimepath^=~/.vim/bundle/vim-markdown-quote-syntax-master`
3. 退出Vim，重新打开`.md`文件，你会发现我们的代码拥有了高贵的语法高亮

![with-vim-markdown-quote-syntax](/assets/imgs/vundle-tutorial/with-vim-markdown-quote-syntax.png)

# 文明优雅的Vundle管理方式

美好的人生不应如此多艰，装个插件咋能这么麻烦？一定是姿势不对啊！问，理想情况下，安装插件总共分几步？

![three-steps](/assets/imgs/vundle-tutorial/3-steps.png)

1. 找到想要的插件
2. 安装
3. 开始用！

如果使用Vundle，我们要做的就是
1. 在`.vimrc`中添加`Plugin 'joker1007/vim-markdown-quote-syntax'`
2. 运行`:PluginInstall`
3. 开始用!

是不是非常简单？不需要再去下载代码，也不需要自己操作`runtimepath`，一下步入了小康社会啊！

## 安装Vundle

万事开头难，为了以后的幸福，我们先要吃点苦，也就是安装下Vundle。好在Vundle的安装相对来说还是很容易的。

我们首先打开[Vundle](https://github.com/VundleVim/Vundle.vim)的GitHub首页，然后找到[Quick Start](https://github.com/VundleVim/Vundle.vim#quick-start)。文章的主体思想就是：

1. 先把Vundle下载下来，放到你的`~/.vim/bundle/`文件夹下。`git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim`
2. 把这一段内容复制到你`.vimrc`文件中
    ```vim
    set nocompatible              " be iMproved, required
    filetype off                  " required
    
    " set the runtime path to include Vundle and initialize
    set rtp+=~/.vim/bundle/Vundle.vim
    call vundle#begin()
    " alternatively, pass a path where Vundle should install plugins
    "call vundle#begin('~/some/path/here')
    
    " let Vundle manage Vundle, required
    Plugin 'VundleVim/Vundle.vim'
    
    " The following are examples of different formats supported.
    " Keep Plugin commands between vundle#begin/end.
    " plugin on GitHub repo
    Plugin 'tpope/vim-fugitive'
    " plugin from http://vim-scripts.org/vim/scripts.html
    " Plugin 'L9'
    " Git plugin not hosted on GitHub
    Plugin 'git://git.wincent.com/command-t.git'
    " git repos on your local machine (i.e. when working on your own plugin)
    Plugin 'file:///home/gmarik/path/to/plugin'
    " The sparkup vim script is in a subdirectory of this repo called vim.
    " Pass the path to set the runtimepath properly.
    Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
    " Install L9 and avoid a Naming conflict if you've already installed a
    " different version somewhere else.
    " Plugin 'ascenator/L9', {'name': 'newL9'}
    
    " All of your Plugins must be added before the following line
    call vundle#end()            " required
    filetype plugin indent on    " required
    " To ignore plugin indent changes, instead use:
    "filetype plugin on
    "
    " Brief help
    " :PluginList       - lists configured plugins
    " :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
    " :PluginSearch foo - searches for foo; append `!` to refresh local cache
    " :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
    "
    " see :h vundle for more details or wiki for FAQ
    " Put your non-Plugin stuff after this line
    ```
3. 打开vim然后运行`:PluginInstall`就可以安装你在`.vimrc`中指定的插件了。或者你也可以直接在Terminal中运行`vim +PluginInstall +qall`来安装插件。

如果以后想安装新的插件，只要根据上面的描述，根据插件来源在相应的位置添加插件的名字就好了。而如果你想删除插件，只要把它从`.vimrc`中移除，然后运行`:PluginClean`就好了。


# 总结

虽然说有了Vundle之后，妈妈再也不用担心我安装Vim插件的时候掉头发了，但是在此还是劝诫大家，劳逸结合，适度编程！
