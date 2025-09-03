---
title: "Linux网络管理（03） — tcpdump抓包"
description: "tcpdump网络抓包的常用命令组合"
date: 2024-07-05
hidden: false
comments: true
draft: false
categories:
   - "Linux内核"
   - "网络"
---

# 1. 基本命令选项
|命令选项|用途||
|--------|--------|--------|
|-i      | 指定网络接口|tcpdump   -i   eth0|
|-X      | （大写X）以 ASCII 和十六进制的形式输出捕获的数据包内容，减去链路层的包头信息|  |
|-XX     | （大写X）以 ASCII 和十六进制的形式输出捕获的数据包内容，包括链路层的包头信息|  |
|-n      | ip 不以别名的形式显示||
|-nn     | ip 和端口不以别名的形式显示||
|-S      | （大写S）以绝对值显示包的 ISN 号（包序列号），默认以上一包的偏移量显示||
|-vv     | 抓包的信息详细地显示||
|-vvv    | 抓包的信息更详细地显示||
|-w      | 抓包信息写入文件pcap，后面跟文件名|tcpdump -w test.pcap|
|-r      | 读取文件pcap（就是-w保存的文件）||  

# 2. 过滤器

## 2.1. 指定协议类型
```bash
tcpdump tcp                    # 仅 TCP 流量
tcpdump udp                    # 仅 UDP 流量
tcpdump icmp                   # 仅 ICMP 流量（如 ping）
```

## 2.2. 指定IP地址
```bash
tcpdump host 192.168.1.100                                # 抓取IP为 192.168.1.100 的数据包
tcpdump src  192.168.1.100                                # 源 IP 为 192.168.1.100
tcpdump dst  192.168.1.100                                # 目标 IP 为 192.168.1.100
```

## 2.3. 指定端口
```bash
tcpdump port 80                # HTTP 流量（端口 80）
tcpdump portrange 8000-8080    # 端口范围 8000 到 8080
tcpdump src port 22            # 源端口为 22（SSH）
tcpdump dst port 443           # 目标端口为 443（HTTPS）
```

## 2.4. 使用and组合过滤器
当有多个过滤条件时，推荐使用单因号。
```bash
tcpdump 'src 192.168.1.100 and dst 192.168.1.200'                      # IP组合
tcpdump 'host 192.168.1.100 or host 192.168.1.200'
tcpdump 'host 192.168.1.100 and port 80'                               # IP-Port组合
tcpdump 'host 192.168.1.100 and not port 22'                           # 排除 SSH 流量
tcpdump 'src 192.168.1.100 and (dst port 80 or dst port 443)'          # 嵌套过滤器，源 IP 为 192.168.1.100，目标端口为 80 或 443
```

## 2.5. 示例组合
```bash
tcpdump -i wlp4s0 'tcp and src 192.168.1.100 and (dst port 80 or dst port 443)'  
tcpdump -i wlp4s0 'src 192.168.1.100 and (dst port 80 or dst port 443) and (tcp or udp)'  
```

# 3. 其他
tcpdump也可以过滤数据包的数据部分，例如过滤http的GET、POST请求等等。