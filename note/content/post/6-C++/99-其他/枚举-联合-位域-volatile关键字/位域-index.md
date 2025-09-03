---
title: "位域"
description: 
date: 2023-02-21
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
---


# 位域
结构体和类中使用，显式表明对象的成员占用的位数。  
```cpp
// 占用一个字节
struct
{
    bool a : 1;   // 占用1位
    bool b : 1;
}
```
这种操作，可以节省内存，但是访问成员的需要更多地运算。而且声明了位域的成员，不能取地址。  
