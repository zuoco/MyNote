---
title: "ps命令"
description: "ps命令查看内存使用量大的进程。" 
date: 2023-06-13
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories: 
  - Linux内核
---

# 1. 用法  
```bash
# 显示内存使用量最大的前10个进程
ps -eo pid,comm,%mem --sort -%mem | head -n 10
```
参数解释：
- `-e`：显示所有进程。  
- `-o`：自定义输出格式，指定需要显示的字段：  pid(进程ID),comm(进程名),%mem(进程使用的内存百分比)。  
- `--sort -%mem`：按照内存使用百分比降序排序。  
- `head -n 10`：显示前10行。  