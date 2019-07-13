---
title: 使用Manjaro
date: 2019-07-13 11:00:49 +0800
categories: vim
tags: vim plugin

---

## 安装Manjaro

### 配置

用`pacman`安装软件的时候可能会比较慢，可以切换以下镜像源:

```bash
$ sudo pacman-mirrors -c China
```

我们来感受以下差距，没有对比就没有伤害...

```

**Before**

Total Installed Size:  27.68 MiB

:: Proceed with installation? [Y/n] y
:: Retrieving packages...
nmap-7.70-3-x86_64        943.8 KiB  12.9K/s 06:34 [####-----------------------]  15%

**After**

Total Installed Size:  27.68 MiB

:: Proceed with installation? [Y/n] Y
:: Retrieving packages...
 nmap-7.70-3-x86_64          5.0 MiB  7.02M/s 00:01 [###########################] 100%

```

### pacman常用命令

