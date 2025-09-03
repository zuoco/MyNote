---
title: "常用Linux系统目录"
description: 
date: 2023-05-18
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - Linux内核
---

- [1. boot](#1-boot)
- [2. usr/src](#2-usrsrc)
- [3. /usr/lib/modules](#3-usrlibmodules)


# 1. boot
```bash
zci@GEM:/boot$ ls -lah
总计 187M
-rw-r--r--  1 root root 286K  6月 26 20:36 config-6.11.0-29-generic    # 安装内核时自动生成，记录对应内核版本的编译配置，包含内核编译时的选项（如启用的硬件驱动、模块支持等），用于调试或重新编译内核模块。
-rw-r--r--  1 root root 290K  7月  7 22:27 config-6.14.0-24-generic

drwxr-xr-x  4 root root 4.0K  1月  1  1970 efi                         # 启动分区挂载位置，存放 UEFI 系统所需的引导文件。包含 grubx64.efi 等 UEFI 启动管理器文件。
drwxr-xr-x  5 root root 4.0K  7月 18 11:17 grub                        # 存放 GRUB（GRand Unified Bootloader）的配置文件和模块。

lrwxrwxrwx  1 root root   28  7月 18 11:16 initrd.img -> initrd.img-6.14.0-24-generic  # 临时根文件系统，用于加载启动时需要的驱动和工具，在挂载真实根文件系统前，加载必要的模块（如 RAID、LVM、加密驱动）。。
-rw-r--r--  1 root root  69M  7月 14 11:09 initrd.img-6.11.0-29-generic
-rw-r--r--  1 root root  70M  7月 18 11:19 initrd.img-6.14.0-24-generic
lrwxrwxrwx  1 root root   28  7月 18 11:16 initrd.img.old -> initrd.img-6.11.0-29-generic

-rw-r--r--  1 root root 140K  4月  9  2024 memtest86+ia32.bin   # 内存测试工具，用于检测硬件故障。
-rw-r--r--  1 root root 141K  4月  9  2024 memtest86+ia32.efi
-rw-r--r--  1 root root 145K  4月  9  2024 memtest86+x64.bin
-rw-r--r--  1 root root 146K  4月  9  2024 memtest86+x64.efi

-rw-------  1 root root 9.1M  6月 26 20:36 System.map-6.11.0-29-generic   # 内核符号表，用于调试和分析内核问题，。
-rw-------  1 root root 8.8M  7月  7 22:27 System.map-6.14.0-24-generic 

lrwxrwxrwx  1 root root   25  7月 18 11:16 vmlinuz -> vmlinuz-6.14.0-24-generic   # 压缩后的 Linux 内核镜像，系统启动时加载到内存并运行的核心程序，负责初始化硬件、挂载根文件系统并启动用户空间进程。  
-rw-------  1 root root  15M  6月 26 20:39 vmlinuz-6.11.0-29-generic
-rw-------  1 root root  15M  7月  7 22:32 vmlinuz-6.14.0-24-generic
lrwxrwxrwx  1 root root   25  7月 18 11:16 vmlinuz.old -> vmlinuz-6.11.0-29-generic
```

# 2. usr/src
```bash
/usr/src
├── linux-headers-6.11.0-29-generic/    # 存放特定内核版本的头文件（Header Files），包含ubuntu中的特有文件，用于编译内核模块或开发与内核交互的应用程序。
├── linux-headers-6.14.0-24-generic/
├── linux-hwe-6.11-headers-6.11.0-29/   # 提供对新硬件的支持（如更新的 GPU、CPU 驱动）。
├── linux-hwe-6.14-headers-6.14.0-24/
├── nvidia-575.57.08/
├── nvidia-575.64/
└── python3.12/
```



# 3. /usr/lib/modules
存放内核模块的目录。内核模块是动态可加载的代码片段，用于扩展内核功能（如硬件驱动、文件系统支持等），无需重新编译整个内核即可按需加载或卸载，内核模块可以在编译内核时编译到内核中，也可以单独编译成模块，然后动态加载到内核中。  
```bash
/usr/lib/modules
├── 6.11.0-26-generic
├── 6.11.0-28-generic
├── 6.11.0-29-generic
└── 6.14.0-24-generic
```

