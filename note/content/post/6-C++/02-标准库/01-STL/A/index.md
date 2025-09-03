---
title: "标准库基本组成"
description: 
date: 2023-06-10
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
    - "C++ STL"
---


1. [1. 基本组成](#1-基本组成)
2. [2. 头文件](#2-头文件)


# 1. 基本组成
| 组成  |  用途 |
|------|------|
|容器|一些封装数据结构的模板类，例如 vector 向量容器、list 列表容器等。|
|算法|STL 提供了非常多（大约 100 个）的数据结构算法，它们都被设计成一个个的模板函数，<br> 这些算法在 std 命名空间中定义，其中大部分算法都包含在头文件 <algorithm> 中，少部分位于头文件 <numeric> 中。|
|迭代器|在 C++ STL 中，对容器中数据的读和写，是通过迭代器完成的，扮演着容器和算法之间的胶合剂。|  
|函数对象|如果一个类将 () 运算符重载为成员函数，这个类就称为函数对象类，这个类的对象就是函数对象（又称仿函数）。|  
|适配器|可以使一个类的接口（模板的参数）适配成用户指定的形式，从而让原本不能在一起工作的两个类工作在一起。值得一提的是，容器、迭代器和函数都有适配器。| 
|内存分配器|为容器类模板提供自定义的内存申请和释放功能，由于往往只有高级用户才有改变内存分配策略的需求，因此内存分配器对于一般用户来说，并不常用。|  


# 2. 头文件
13个标准库头文件。      
```cpp
#include <iterator>	   
#include <functional>	   
#include <vector>    	
#include <deque>    
#include <list>	    
#include <queue>	    
#include <stack>	   
#include <set>    
#include <map>	    
#include <algorithm>	   
#include <numeric>	    
#include <memory>    
#include <utility>    
```
