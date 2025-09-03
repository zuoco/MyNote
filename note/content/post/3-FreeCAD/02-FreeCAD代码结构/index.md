---
title: "FreeCAD（02） — 代码结构"
description: 
date: 2025-06-01
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - FreeCAD
---


# 代码目录结构
```bash
.
└── freecad
    ├── src               # 源码
    ├── tools
    ├── tests
    ├── cMake               
    ├── conda
    ├── contrib
    ├── data              # 示例
    ├── CMakeLists.txt
    └── 其他
```
---

src源码目录，其中App、Base、Main组成提个无UI运行程序：  
```bash
├── src
│   ├── 3rdParty             # 三方库，K维树、网格等等。
│   ├── App                  # 非界面代码，Document、Property、DocumentObject，支持Python。
│   ├── Base                 # 底层基础类型。
│   ├── Build                # 编译时的版本信息
│   ├── CXX                  # 对于PyCXX的修改,便于Python脚本调用C++代码。
│   ├── Doc                  # 生成手册文档
│   ├── Gui                  # 界面
│   ├── MacAppBundle         # Mac安装包配置文件
│   ├── Main                 # 程序入口，包括CLI和GUI两个入口程序
│   ├── Mod                  # 模块，CAM、工程图、有限元等等模块
│   ├── Tools
│   ├── XDGData              # Linux桌面相关文件
│   ├── zipios++             # 压缩文件读写
│   ├── CMakeLists.txt
│   └── 其他
```
Base目录下是FreeCAD的类型系统，接口类，抽象类等等。  
App目录下是属性系统，文档对象相关的代码。   
Main目录下就是不同模式的main函数了。  

---

源码中Mod目录下的内容：   
```bash
src/Mod/
├── Part            # 创建基础的3D元素，圆柱、立方体等等
├── OpenSCAD        # 建模
├── PartDesign      # 零件设计 
├── CAM             # 工艺，数控
├── Draft           # 2D草图 
├── Drawing         # 工程图 
├── Sketcher        # 
├── Assembly        # 装配图 
├── Fem             # 有限元
├── Material        # 材料属性
├── Mesh            # 网格化
├── MeshPart
├── Points             # 点云
├── ReverseEngineering # 从点云创建实体
├── Robot              # 机器人
└── 其他
```




