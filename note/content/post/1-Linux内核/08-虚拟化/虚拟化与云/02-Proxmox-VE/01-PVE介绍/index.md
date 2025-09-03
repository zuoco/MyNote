---
title: "第一节 PVE介绍"
description: 
date: 2024-05-27
hidden: false
comments: true
draft: true
categories:
  - 虚拟化
---

# 1. 基本介绍
Proxmox VE（Proxmox Virtual Environment），奥地利企业“Proxmox Server Solutions GnbH”开发的虚拟化产品，基于Debian，于2008年发布第一个版本稳定版Proxmox VE 1.0。

# 2. 功能特性
1. `服务器虚拟化`     
基于KVM技术提供虚拟化支持，可以运行Linux和Windows操作系统。   
2. `Linux容器`      
基于LXC技术提供Linux容器支持，通过PCT(Proxmox Container Tools)就可以直接使用Proxmox VE的存储资源和网络资源。   
3. `基于Web的管理`   
通过web登陆对服务器进行管理，不需要安装客户端软件。   
4. `高可用`   
基于Linux HA技术。   
5. `去中心化`   
Prexmox VE基于数据库设计了一种专用的文件系统用于保存配置文件，这个文件系统通过Corosync将配置文件实时复制到PVE集群的所有节点，不再需要部署一个单独的管理端服务器，这就是PVE的去中心化设计。    
6. `集成Ceph分布式存储`   
通过Web端即可运行管理Ceph。
7. `集成备份与还原`  

8. `企业级备份`   
Proxomx Backup Server与集成的备份与还原功能相比更强大。  
9.  `集成防火墙`     
通过Web页面对虚拟机以及容器的网络流量进行过滤。   
10. `支持多种身份认证`   
包括Microsoft活动目录、LDAP、双因素身份认证等等。   
