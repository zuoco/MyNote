---
title: "关键字"
description: 
date: 2023-05-08
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
---

1. [1. explicit](#1-explicit)
2. [2. noexcept](#2-noexcept)
3. [3. constexpr](#3-constexpr)
4. [4. consteval](#4-consteval)
5. [5. final](#5-final)
6. [6. volatile 关键字](#6-volatile-关键字)
7. [7. pragma once](#7-pragma-once)
8. [8. 内存对齐](#8-内存对齐)
  1. [8.1. aligans](#81-aligans)






# 1. explicit
explicit 关键字用于修饰类的构造函数，表明该构造函数是显式的，不能用于隐式类型转换，也就是说用户给的参数类型必须与构造函数的参数类型一致，否则会编译错误。   

# 2. noexcept
声明函数不会抛出异常。  

# 3. constexpr
编译期常量，在编译器初始化。   

# 4. consteval
这个函数每次调用都返回一个编译期常量。

# 5. final
在类声明后添加 final，表示该类不可作为基类：
```cpp
class Base final {  // 此类不能被继承
    // ...
};

// 尝试继承将导致编译错误
class Derived : public Base {  // ❌ 错误：Base 是 final 的
    // ...
};
```

# 6. volatile 关键字
使用 volatile 告诉编译器：该变量的值可能在程序之外被意外修改，所以不要对关键代码进行优化，每次访问该变量都必须直接操作内存。如果没有volatile关键字，则编译器可能优化读取和存储，可能暂时使用寄存器中的值，如果这个变量由别的程序更新了的话，将出现不一致的现象。   
**核心作用：**   
- 防止编译器优化：编译器不会对 volatile 变量进行缓存（如寄存器）。
- 强制内存访问：每次读取 volatile 变量时，必须从内存中读取；每次写入后，必须立即刷新到内存。



# 7. pragma once
```cpp
// 用于传统头文件
 #pragma once // 告诉编译器，此文件只包含一次，防止重复包含
```


# 8. 内存对齐
## 8.1. aligans
alignas 是 C++11 引入的标准方法，用于显式指定变量或类型的对齐方式。通过调整结构体成员的顺序，可以减少填充字节（padding），从而优化内存布局。虽然这与标准无关，但仍是实际开发中常用的方法。
```cpp
// 指定结构体对齐
struct alignas(32) Data {  // 32 字节对齐   
    int x;      // 占用 [0, 3] 共 4 字节。  
    double y;   // 占用 [8, 15] 共 8 字节。   
    char z;     // 占用 [16] 共 1 字节。   
    // 为了对齐，尾部填充 15 字节（地址 [17, 31]）
    // 最终大小：32 字节。
};

// 指定变量对齐
alignas(64) char buffer[1024];  // 64 字节对齐

// 检查对齐
static_assert(alignof(Data) == 32);  // 编译期验证
```
C++17中又引入了std::aligned_union，std::aligned_alloc等等来支持分配对齐内存。C++20 引入了 `[[nodiscard]]` 特性，可以用于标记返回对齐内存的函数，防止开发者忽略对齐检查，也可以使用编译器特性，但是不同的编译器使用方法不同。