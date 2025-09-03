---
title: "free命令"  
description: "查看系统内存使用情况。"
date: 2024-05-08
hidden: false
comments: true
draft: false
categories:
  - Linux内核
---

# free命令
命令结果如下：   
```bash
zcli@MiBook:~$ free -m
              total        used       free          shared     buff/cache   available
内存：         31412        6177       19370         127        6443         25234      # 物理内存
交换：          8191           0        8191                                            # 交换空间
zcli@MiBook:~$ 
zcli@MiBook:~$ free -g
                 total        used       free         shared      buff/cache   available
内存：            30           6          18           0           6            24
交换：             7           0           7
zcli@MiBook:~$ 
```
参数选项：  -m 以MB为单位显示内存使用情况，-g 以GB为单位显示内存使用情况。   

# 字段解释
- `物理内存:  `就是我们主板上的内存条。  
- `交换空间（Swap）:  `是磁盘上的一块虚拟内存区域。当物理内存不足时，操作系统会将不活跃的内存页移动到 Swap 中，释放物理内存供其他程序使用。  
- `shared:  `  共享内存，即多个进程共享的内存，如共享内存段（System V 共享内存、POSIX 共享内存）等等。  
- `used:  `已经使用的物理内存，total-available或者total-free-buff/cache。   
- `buff/cache: `  缓冲/缓存。  
- `free与available:  ` free是表面上的可用物理内存，available是实际可用的物理内存，available=free+buff/cache，当内存（free）不够时，系统会从buff/cache中回收内存。  
