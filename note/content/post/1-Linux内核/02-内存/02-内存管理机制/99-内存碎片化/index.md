---
title: "内存碎片化"
description: "认识内存碎片化以及解决方法"
date: 2025-05-13
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - Linux内核
---

# 1. 什么是内存碎片化
随着系统的运行，内存会不断被分配和释放，被分割成大量不连续的小块空闲区域，这会致系统无法分配较大连续物理内存页面，当我们发现空闲内存总量可观，但是大块内存数量少，以及大块内存申请效率低下时，可能就是内存碎片化比较严重。

# 2. 检查内存碎片化程度
先了解一个重要文件：   
```bash
zcli@fedora:~$ cat /proc/buddyinfo
Node 0, zone      DMA      0      0      0      0      0      0      0      0      1      1      2 
Node 0, zone    DMA32      8     11      6      7      7      7     10      8      7      9    573 
Node 0, zone   Normal   1958    592    249    207    133     32     98     87     36     11   4422 
```
文件每一行的格式为：  
```bash
#                                          4KB      8KB           2M       4M
Node   <node_id>,   zone   <zone_name>   <order0> <order1> ... <order9> <order10>
```
- `Node 0`：  表示单个 NUMA 节点（无 NUMA 架构时仅一个节点）。     
- `zone <zone_name>`：内存区域类型，常见类型包括：     
  - DMA：直接内存访问（DMA）区域，用于 32 位设备的内存访问（地址范围有限）。    
  - DMA32：扩展的 32 位 DMA 区域（地址范围比 DMA 更广）。    
  - Normal：普通内存区域，用于大多数内存分配。  
   
- `<orderN>`：   
表示阶（order）为 N 的空闲内存块数量(N从0开始)。每个阶的内存块大小为 2^N * PAGE_SIZE（通常 PAGE_SIZE = 4KB），order9是2M，order10是4M。

对于32位的系统，内存区域类型: 
```bash 
  DMA
  Normal
  HighMem  # 高内存区，在32位系统上用于扩展可寻址内存范围，对于64位系统，没有这东西。
```

如果**内存总量充足，但是大块内存申请效率低**，甚至引发OOM，或者系统重启，此时可以查看order9和order10的内存块数量，**高阶阶空闲块越少**代表内存碎片化越严重。

# 3. 如何解决
## 3.1. 内存整理
内核虚拟文件系统中有这么一个文件：/proc/sys/vm/compact_memory      
```
root@fedora:~# ls -al /proc/sys/vm/compact_memory
--w-------. 1 root root 0  5月20日 22:44 /proc/sys/vm/compact_memory # 写入1或0
```
0： 禁止内存整理功能（默认值）。
1： 启用内存压缩，并触发一次内存整理操作。  


**1. 临时启用内存压缩功能**   
```bash
sudo susctl vm.compact_memory=1
```
以上方法是一次性的，重启后失效。

**2. 永久启用**   
将下面内容写入/etc/sysctl.conf文件中：   
```bash
vm.compact_memory=1
```
然后执行：  
```bash
sudo sysctl -p
```

`但是要注意`:     
1. 内存压缩是一个同步、阻塞式的操作，会增加CPU和IO开销，不建议在生产环境频繁使用。    
2. 内存压缩需要启用内核**CONFIG_COMPACTION**选项，可以通过以下命令查看该功能是否启用：  
```bash
grep CONFIG_COMPACTION /boot/config-$(uname -r) # 输出CONFIG_COMPACTION=y表示启用了该功能。
```

## 3.2. 增加预留内存
`/proc/sys/vm/min_free_kbytes`，这个文件可以查看系统预留内存的大小。文件中的值定义了在系统中应始终保持空闲的最小内存量（以KB为单位）。这个参数对于防止内存完全耗尽，确保系统即使在高负载下也能平稳运行至关重要。当可用内存降至 min_free_kbytes 设置的阈值之下时，Linux 内核的内存回收机制会主动尝试释放内存，例如通过回收缓存或交换出不活跃的页面到磁盘。

## 3.3. 增加预留内存的方法
**1. 临时措施**  
```bash
sudo sysctl -w vm.min_free_kbytes=<new_value>
# sudo sysctl -w vm.min_free_kbytes=2097152 设置为2G
```
**1. 永久措施**  
向/etc/sysctl.conf写入`vm.min_free_kbytes = <new_value>`;  
执行命令`sudo sysctl -p`;    

适当增加预留内存，可以使系统更加积极的整理内存，而不是等到有程序申请大块内存时才进行整理。  
