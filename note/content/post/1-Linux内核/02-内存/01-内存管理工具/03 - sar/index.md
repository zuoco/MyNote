---
title: "sar -r命令与sar -S命令"
description: "动态显示内存使用情况"
date: 2024-06-17
hidden: false
comments: true
draft: false
categories:
  - Linux内核
---

# 1. sar -r命令
```bash
sar -r [interval] [count]
```
`interval`：指定采样间隔。     
`count`：指定采样次数。

例如：   
```bash
zcli@MiBook:~$ sar -r 1 3
Linux 6.11.0-26-generic (MiBook) 	2025年06月13日 	_x86_64_	(16 CPU)

14时40分09秒 kbmemfree   kbavail kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
14时40分10秒  18907448  25056588   6238488     19.39    459788   5979568  22043760     54.36   6992188   4599208       840
14时40分11秒  18954764  25103904   6191164     19.25    459788   5979568  22043568     54.36   6936048   4599208         8
14时40分12秒  18958924  25108068   6187000     19.23    459796   5979568  22043576     54.36   6930696   4599216       100
平均时间:  18940379  25089520   6205551     19.29    459791   5979568  22043635     54.36   6952977   4599211       316
```
- `kbmemfree： `对应于free命令的free。       
- `kbavail： `对应 free 中的 available 字段。   
- `kbmemused： `对应 free 中的 used 字段。    
- `%memused`： kbmemused字段所占百分比。  
- `kbbuffers`+`kbcached: `对应free命令输出的buff/cache字段。    

# sar -S
```bash
zcli@MiBook:~$ sar -S 1 2

14时54分27秒 kbswpfree kbswpused  %swpused  kbswpcad   %swpcad
14时54分28秒   8388604         0      0.00         0      0.00
14时54分29秒   8388604         0      0.00         0      0.00
平均时间:      8388604         0      0.00         0      0.00
```
参数选项：  -S 显示swap空间使用情况。   
