---
title: "TCP的“三次握手”与“四次挥手”"
description: "通过抓取TCP报文，了解TCP的“三次握手”与“四次挥手”。"
date: 2023-06-17
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - TCP
    - 网络协议原理与设计
---

- [1. tcpdump 抓取Tcp报文](#1-tcpdump-抓取tcp报文)
- [2. 分析抓包结果](#2-分析抓包结果)
  - [2.1. 三次握手](#21-三次握手)
  - [2.2. 数据发送与响应](#22-数据发送与响应)
  - [2.3. 四次挥手](#23-四次挥手)
- [3. 其他](#3-其他)


# 1. tcpdump 抓取Tcp报文
**1.** `现在有一对tcp程序`     
- 服务端地址：192.168.52.39：8080。       
- 客户端地址：192.168.52.67。   
- 连接建立后客户端发送数据： ABCDEF。
- 服务器收到数据后，发送数据：ABCDEFGH。    
- 客户端断开连接。   


**2.** `tcpdump抓取Tcp报文`  
```bash
sudo tcpdump -i any 'tcp and host 192.168.52.39 and host 192.168.52.67' -vvn
```

**3.** `抓取结果`   
```bash
tcpdump: data link type LINUX_SLL2
tcpdump: listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
13:29:25.569020 wlp4s0 In  IP (tos 0x0, ttl 64, id 57345, offset 0, flags [DF], proto TCP (6), length 60)
    192.168.52.67.33762 > 192.168.52.39.8080: Flags [S], cksum 0x3356 (correct), seq 3666571938, win 64240, options [mss 1460,sackOK,TS val 1128528921 ecr 0,nop,wscale 7], length 0
13:29:25.569049 wlp4s0 Out IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 60)
    192.168.52.39.8080 > 192.168.52.67.33762: Flags [S.], cksum 0xe9e9 (incorrect -> 0x92ef), seq 394631164, ack 3666571939, win 65160, options [mss 1460,sackOK,TS val 4121163671 ecr 1128528921,nop,wscale 7], length 0
13:29:25.571512 wlp4s0 In  IP (tos 0x0, ttl 64, id 57346, offset 0, flags [DF], proto TCP (6), length 52)
    192.168.52.67.33762 > 192.168.52.39.8080: Flags [.], cksum 0xbe4a (correct), seq 1, ack 1, win 502, options [nop,nop,TS val 1128528925 ecr 4121163671], length 0
13:29:25.571513 wlp4s0 In  IP (tos 0x0, ttl 64, id 57347, offset 0, flags [DF], proto TCP (6), length 58)
    192.168.52.67.33762 > 192.168.52.39.8080: Flags [P.], cksum 0xfa78 (correct), seq 1:7, ack 1, win 502, options [nop,nop,TS val 1128528925 ecr 4121163671], length 6: HTTP
13:29:25.571554 wlp4s0 Out IP (tos 0x0, ttl 64, id 45420, offset 0, flags [DF], proto TCP (6), length 52)
    192.168.52.39.8080 > 192.168.52.67.33762: Flags [.], cksum 0xe9e1 (incorrect -> 0xbe39), seq 1, ack 7, win 510, options [nop,nop,TS val 4121163674 ecr 1128528925], length 0
13:29:25.571658 wlp4s0 Out IP (tos 0x0, ttl 64, id 45421, offset 0, flags [DF], proto TCP (6), length 60)
    192.168.52.39.8080 > 192.168.52.67.33762: Flags [P.], cksum 0xe9e9 (incorrect -> 0xb520), seq 1:9, ack 7, win 510, options [nop,nop,TS val 4121163674 ecr 1128528925], length 8: HTTP
13:29:25.577532 wlp4s0 In  IP (tos 0x0, ttl 64, id 57348, offset 0, flags [DF], proto TCP (6), length 52)
    192.168.52.67.33762 > 192.168.52.39.8080: Flags [.], cksum 0xbe35 (correct), seq 7, ack 9, win 502, options [nop,nop,TS val 1128528929 ecr 4121163674], length 0
13:29:30.578167 wlp4s0 In  IP (tos 0x0, ttl 64, id 57349, offset 0, flags [DF], proto TCP (6), length 52)
    192.168.52.67.33762 > 192.168.52.39.8080: Flags [F.], cksum 0xaaab (correct), seq 7, ack 9, win 502, options [nop,nop,TS val 1128533930 ecr 4121163674], length 0
13:29:30.618353 wlp4s0 Out IP (tos 0x0, ttl 64, id 45422, offset 0, flags [DF], proto TCP (6), length 52)
    192.168.52.39.8080 > 192.168.52.67.33762: Flags [.], cksum 0xe9e1 (incorrect -> 0x96ec), seq 9, ack 8, win 510, options [nop,nop,TS val 4121168721 ecr 1128533930], length 0
13:29:35.571866 wlp4s0 Out IP (tos 0x0, ttl 64, id 45423, offset 0, flags [DF], proto TCP (6), length 52)
    192.168.52.39.8080 > 192.168.52.67.33762: Flags [F.], cksum 0xe9e1 (incorrect -> 0x8392), seq 9, ack 8, win 510, options [nop,nop,TS val 4121173674 ecr 1128533930], length 0
13:29:35.576375 wlp4s0 In  IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 52)
    192.168.52.67.33762 > 192.168.52.39.8080: Flags [.], cksum 0x7014 (correct), seq 8, ack 10, win 502, options [nop,nop,TS val 1128538928 ecr 4121173674], length 0
^C
11 packets captured
11 packets received by filter
0 packets dropped by kernel
```

# 2. 分析抓包结果
```bash
11 packets captured
11 packets received by filter
0 packets dropped by kernel
```
三次握手、四次挥手、一个数据报文，一个数据报文的ACK，算起来9个报文，下面分析一下抓取的报文。     


## 2.1. 三次握手
```bash
    192.168.52.67.33762 > 192.168.52.39.8080: Flags [S], cksum 0x3356 (correct), seq 3666571938, win 64240, options [mss 1460,sackOK,TS val 1128528921 ecr 0,nop,wscale 7], length 0
    192.168.52.39.8080 > 192.168.52.67.33762: Flags [S.], cksum 0xe9e9 (incorrect -> 0x92ef), seq 394631164, ack 3666571939, win 65160, options [mss 1460,sackOK,TS val 4121163671 ecr 1128528921,nop,wscale 7], length 0
    192.168.52.67.33762 > 192.168.52.39.8080: Flags [.], cksum 0xbe4a (correct), seq 1, ack 1, win 502, options [nop,nop,TS val 1128528925 ecr 4121163671], length 0
```
简单分析一下：   
`192.168.52.67.33762 > 192.168.52.39.8080`： 报文的源地址 > 目标地址    
`Flags [S]`:   同步报文段，由客户端发送，表示请求建立一个连接。   
`Flags [S.]`:  同步+确认报文段，由服务端发送，做了两件事： 1.同步,表示同意客户端的连接请求。2.响应客户端的SYN报文。    
`Flags [.]`:   确认报文。   
`seq 3666571938`:   客户端ISN值，就是报文序号的初始值。   
`seq 394631164`:    服务端ISN值，两个传输方向分别有各自的ISN值。    
`ack 3666571939`:   确认序号，表示服务端已经收到客户端的报文,确认序号就是接收到的报文的seq+1。    
`options [mss 1460,sackOK,TS val 1128528921 ecr 0,nop,wscale 7]`:   TCP头部结构中的头部选项。   
`length 0`:  报文的数据长度，SYN 包不携带数据，所以为0。    

以上就是“三次握手”。   

## 2.2. 数据发送与响应   
```bash
    # 客户端发送数据包，6个字节
    192.168.52.67.33762 > 192.168.52.39.8080: Flags [P.], cksum 0xfa78 (correct), seq 1:7, ack 1, win 502, options [nop,nop,TS val 1128528925 ecr 4121163671], length 6: HTTP
    # 服务器收到数据包，发送确认消息
    192.168.52.39.8080 > 192.168.52.67.33762: Flags [.], cksum 0xe9e1 (incorrect -> 0xbe39), seq 1, ack 7, win 510, options [nop,nop,TS val 4121163674 ecr 1128528925], length 0
    # 服务器发送数据包，8个字节
    192.168.52.39.8080 > 192.168.52.67.33762: Flags [P.], cksum 0xe9e9 (incorrect -> 0xb520), seq 1:9, ack 7, win 510, options [nop,nop,TS val 4121163674 ecr 1128528925], length 8: HTTP
    # 客户端接收数据包，发送确认消息
    192.168.52.67.33762 > 192.168.52.39.8080: Flags [.], cksum 0xbe35 (correct), seq 7, ack 9, win 502, options [nop,nop,TS val 1128528929 ecr 4121163674], length 0
```
`seq 1:7`:         数据报文的起始序列号为1（方向： 客户端->服务器），报文所携带的字节编号是[1, 7)。     
`length 6: HTTP`:  报文携带的数据长度为6字节，识别为HTTP报文是因为使用了8080端口。    
`seq 1:9`:         数据报文的起始序列号为1（方向： 服务器->客户端）。    
`ack 7`:           确认序号为7，就是对于来自客户端的数据报文进行响应，表示收到了这个报文，ack=seq+length+1，也就是下一个报文的起始序列号。

**注意：**   
 tcp虽然也是一个一个报文发送的，但是tcp是流式协议，序列号用于表示一次网络通信过程中，所传输数据的每一个`字节`的编号，注意了是字节的编号，在一次通信活动中，数据字节的编号是连续的。因为是流式协议，对于应用层协议来说，tcp数据流是没有边界的，会发生所谓的`粘包`问题，这需要应用层去解决，毕竟tcp只负责数据的可靠传输，而不管数据从哪里开始，从到哪里结束。   
**ACK报文：**   
例如一个报文，字节编号是[100, 201)，这个报文的应答报文的ACK就是201，也就是下一个报文的数据字节的起始编号，通过这个方式告诉发送端，这个报文已经收到，可以开始发送下一个报文了。  

## 2.3. 四次挥手   
```bash
    192.168.52.67.33762 > 192.168.52.39.8080: Flags [F.], cksum 0xaaab (correct), seq 7, ack 9, win 502, options [nop,nop,TS val 1128533930 ecr 4121163674], length 0
    192.168.52.39.8080 > 192.168.52.67.33762: Flags [.], cksum 0xe9e1 (incorrect -> 0x96ec), seq 9, ack 8, win 510, options [nop,nop,TS val 4121168721 ecr 1128533930], length 0
    192.168.52.39.8080 > 192.168.52.67.33762: Flags [F.], cksum 0xe9e1 (incorrect -> 0x8392), seq 9, ack 8, win 510, options [nop,nop,TS val 4121173674 ecr 1128533930], length 0
    192.168.52.67.33762 > 192.168.52.39.8080: Flags [.], cksum 0x7014 (correct), seq 8, ack 10, win 502, options [nop,nop,TS val 1128538928 ecr 4121173674], length 0
```
`192.168.52.67.33762 > 192.168.52.39.8080: Flags [F.]`:   客户端发送了关闭连接的请求。   
`192.168.52.39.8080 > 192.168.52.67.33762: Flags [.]`:    服务器收到客户端请求后，发送了确认包。     
`192.168.52.39.8080 > 192.168.52.67.33762: Flags [F.]`:   服务器发送了关闭连接的请求。    
`192.168.52.67.33762 > 192.168.52.39.8080: Flags [.]`:    客户端收到服务器的关闭请求后，发送了确认包。   

双方都要发送关闭连接请求，保证双方都没有数据要发送。有些机器挥手只有3个报文，因为中间的两个报文合并在一起了。    


# 3. 其他
```bash
# 时间戳            网络接口     数据方向     IP报文相关信息                                                  
13:29:25.571513    wlp4s0      In         IP (tos 0x0, ttl 64, id 57347, offset 0, flags [DF], proto TCP (6), length 58)
13:29:25.571554    wlp4s0      Out        IP (tos 0x0, ttl 64, id 45420, offset 0, flags [DF], proto TCP (6), length 52)
```
