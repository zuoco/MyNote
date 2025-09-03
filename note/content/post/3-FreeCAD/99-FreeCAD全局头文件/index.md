---
title: "FreeCAD 全局头文件（FCGlobal.h）"
description: 
date: 2025-08-12
image: 
math: 
license: 
hidden: false
comments: true
draft: true

---


先了解一些基本概念：  
- 动态库开发中的“导入”、“导出”：  

|  特性  |  导出（Export） |  导入（Import） |   
|------------|------------|------------|   
|作用          |   对外暴露符号（本模块定义）	 |使用外部符号（其他模块定义）   | 
|Windows 标记  |   __declspec(dllexport)	|__declspec(dllimport)      |
|Linux 标记    |    默认可见（无需标记）	    |默认可见（无需标记）         |
|典型场景       |	模块源文件中定义类/函数	     |模块头文件中声明外部类/函数   |



# FreeCAD符号导入导出机制
FCGlobal.h 是 FreeCAD 项目中的全局头文件，其核心功能是为跨平台开发提供统一的符号导出/导入机制。   


