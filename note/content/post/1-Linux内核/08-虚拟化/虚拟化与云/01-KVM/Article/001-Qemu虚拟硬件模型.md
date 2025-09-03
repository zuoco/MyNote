---
title: "虚拟硬件模型"
description: 
date: 2024-05-03
hidden: false
comments: true
draft: false
categories:
  - 虚拟化 
---

# 1. 虚拟硬件模型
&emsp;&emsp;简单的讲“虚拟化”就是使用软件模拟硬件，一个硬件设备，例如QXl（一种虚拟显卡），或者一套硬件，例如虚拟机（可以运行操作系统）。`虚拟硬件模型`虚拟的就是一个完整的主板环境，如QEMU中的q35或i440fx。那么这个主板环境有那些东西呢？     
- 芯片组（Chipset）：如北桥、南桥（或现代的MCH/ICH），负责协调CPU、内存、I/O设备等的通信。   
- 内存控制器：管理内存的访问和分配。   
- PCI/PCIe总线：支持外设的连接和数据传输。   
- 存储控制器：如SATA、NVMe接口。   
- 其他硬件特性：如UEFI固件、IOMMU（用于设备直通）、电源管理等。   

# 2. q35与i440fx   
`i440FX`  
这个比较老了，是QEMU最早支持的虚拟硬件模型之一，基于1996年发布的Intel 82443FX（i440FX）芯片组。   
`q35`   
这是QEMU中较新的虚拟硬件模型，基于Intel 82801（ICH9）芯片组。相比于i440FX，它提供了更多的现代硬件特性支持，适合更高要求的应用场景支持很多现代硬件特性，如：   
- IOMMU支持：通过vIOMMU（虚拟IOMMU），可以实现设备直通（如GPU、网卡等），这对于高性能计算和图形处理非常重要。    
- UEFI支持：通过OVMF（Open Virtual Machine Firmware），可以支持UEFI启动，这在现代操作系统中越来越普遍。     
- PCIe支持：相比i440FX，q35提供了更好的PCI Express（PCIe）支持，允许更高的带宽和更低的延迟。    
- 电源管理：支持S3（挂起到RAM）和S4（挂起到磁盘）等高级电源状态，这些功能在现代PC中很常见。   
- VirtIO支持：推荐使用VirtIO SCSI或VirtIO Block控制器以获得更好的性能和维护性。     
