---
title: "内核模块编译"
description: 
date: 2025-07-18
image: 
math: 
license: 
hidden: false
comments: true
draft: true
---


# 模块源码获取
模块编译要注意匹配内核版本，所以不建议直接下载Linux源码，而是从软件源中下载，以Debain系为例：
```bash
zci@GEM:/usr/src$ apt list | grep linux | grep source

linux-source-6.8.0/noble-updates,noble-security 6.8.0-64.67 all   # 通用的 Linux 内核源代码包，适用于标准的 x86/ARM 等架构，基于 Linux 内核 6.8.0。
linux-source/noble-updates,noble-security 6.8.0-64.67 all 
```
下载源码包：  
```bash
sudo apt source linux-source-6.8.0  # 下载源码到/usr/src目录
```
/usr/src目录多出了如下文件:   
```bash
drwxr-xr-x 29 root root      4096  7月 18 16:17 linux-6.8.0                       # 内核源码a源文件
-rw-r--r--  1 root root   5617250  6月 23 22:06 linux_6.8.0-64.67.diff.gz
-rw-r--r--  1 root root      9342  6月 23 22:06 linux_6.8.0-64.67.dsc
-rw-r--r--  1 root root 230060117  3月 15  2024 linux_6.8.0.orig.tar.gz
```


