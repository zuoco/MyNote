---
title: "C++包管理器之vcpkg"
description: 
date: 2024-07-31
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - "C++"
---


1. [1. vcpkg](#1-vcpkg)
2. [2. 安装](#2-安装)
3. [3. 使用](#3-使用)
  1. [3.1. 安装软件包](#31-安装软件包)
  2. [3.2. 查看软件包信息](#32-查看软件包信息)
4. [4. 使用清单](#4-使用清单)
    1. [4.0.1. 手动安装每个依赖](#401-手动安装每个依赖)
  1. [4.1. CMake构建](#41-cmake构建)




# 1. vcpkg
Vcpkg 由 Microsoft 和 C++ 社区维护，可在 Windows、macOS 和 Linux 上运行，使用 C++ 和 CMake 脚本编写。

# 2. 安装    
1. 我这里就安装到~/.APP/目录下：   
```bash
cd ~/.app   
# clone vcpkg
git clone https://github.com/microsoft/vcpkg.git   
# 初始化vcpkg  
cd vcpkg && ./bootstrap-vcpkg.sh
```

2. 设置软连接：     
```bash
sudo ln -s /home/zcli/.app/vcpkg/vcpkg /usr/local/bin/vcpkg
```

# 3. 使用
## 3.1. 安装软件包
```bash
vcpkg install fmt
vcpkg install fmt:x64-linux           # 指定平台
vcpkg install fmt:x64-windows-static  # 静态库
vcpkg install fmt:x64-windows         # 动态库
```

## 3.2. 查看软件包信息
```bash
vcpkg list                               # 查找已经安装的软件包
vcpkg search                             # 查看所有软件包
vcpkg search opencv                      # 查看包含“opencv”关键词的软件包
vcpkg info   opencv                      # 查看软件包信息
pvckg help   triplet                     # 查看支持的平台
vcpkg search opencv --triplet x64-linux  # 查找特定平台的包	
```

# 4. 使用清单
1. 在项目根目录下：   
```bash
vcpkg new --application
```
2. 生成两个文件：   
```bash
vcpkg-configuration.json
vcpkg.json    # 依赖清单
```
vcpkg-configuration.json文件如下：  
```bash
{
  "default-registry": {
    "kind": "git",
    "baseline": "89dc8be6dbcf18482a5a1bf86a2f4615c939b0fb",
    "repository": "https://github.com/microsoft/vcpkg"
  },
  "registries": [
    {
      "kind": "artifact",
      "location": "https://github.com/microsoft/vcpkg-ce-catalog/archive/refs/heads/main.zip",
      "name": "microsoft"
    }
  ]
}
```
vcpkg.json文件如下：  
```bash
{}
```
3. 添加依赖
```bash
vcpkg add port opencv
```
添加结果如下：  
```bash
{
  "dependencies": [
    "opencv"
  ]
}
```
也可以指定具体的版本。

4. 在准备好清单文件后：   
```bash
vcpkg install  # 自动读取当前目录的 vcpkg.json 文件
```
下载的二进制文件位于当前目录下的：vcpkg_installed目录。

### 4.0.1. 手动安装每个依赖

1. 安装软件包
```bash
vcpkg install zlib # 安装zlib
```

2. 卸载软件包
```bash
vcpkg remove zlib
```
3. 查找可用包
```bash
vcpkg remove zlib
```

## 4.1. CMake构建
```bash
cmake -B build -S . \ 
  -DCMAKE_TOOLCHAIN_FILE=/home/zcli/.APP/vcpkg/scripts/buildsystems/vcpkg.cmake \ 
  -DVCPKG_TARGET_TRIPLET=x64-linux \ 
  -DCMAKE_BUILD_TYPE=Release \ 
  -DCMAKE_INSTALL_PREFIX=/usr/local
```