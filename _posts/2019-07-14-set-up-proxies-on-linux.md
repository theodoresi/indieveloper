---
title: 设置HTTP, HTTPS, FTP代理
date: 2019-07-13 11:00:49 +0800
categories: vim
tags: vim plugin

---


## Linux设置HTTP

除了可以通过GUI在系统设置中操作，还可以在终端中通过设置环境变量进行操作。以下三种格式都可以：

```bash
export http_proxy="http://127.0.0.1:8123"
export https_proxy="https://127.0.0.1:8123"
export ftp_proxy="ftp://127.0.0.1:8123"

export http_proxy=http://127.0.0.1:8123
export https_proxy=https://127.0.0.1:8123
export ftp_proxy=ftp://127.0.0.1:8123

export http_proxy=127.0.0.1:8123
export https_proxy=127.0.0.1:8123
export ftp_proxy=127.0.0.1:8123
```

### SOCKS代理转HTTP代理

如果你有SOCKS代理，那也可以通过`polipo`这个软件将其进行转换（在Windows下可以试试`privoxy`）。

```bash
$ sudo apt install polipo
# SOCKS代理在1090端口
$ sudo polipo socksParentProxy=localhost:1090
```
