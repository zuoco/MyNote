---
title: "用户态协议栈与内核旁路"
description: 
date: 2025-02-17
image: 
math: 
license: 
hidden: false
comments: true
draft: true
categories:
    - 内核旁路与用户态协议栈
---



现在有很多高性能大吞吐量的网络服务使用DPDK + F-Stack。DPDK是英特尔网卡支持的内核旁路软件包，所谓的内核旁路的就是绕过内核协议栈，将网络数据包的收发和处理直接转移到用户态，减少内核与用户态之间的上下文切换、系统调用和数据拷贝开销。F-Stack就是一个开源的用户态协议栈，就是在用户空间实现完整的TCP/IP协议栈功能，替代内核协议栈的处理流程。它通过旁路内核，直接操作硬件资源（如网卡、内存），类似的还有LWIP、MTCP。   







