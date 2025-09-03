---
title: "联合体"
description: 
date: 2023-02-19
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
---

1. [1. 联合体](#1-联合体)
2. [2. 匿名联合体](#2-匿名联合体)
3. [3. 联合中包含非内建类型](#3-联合中包含非内建类型)

# 1. 联合体
```cpp
#include <iostream>

// 联合体占用的空间等于最大成员的大小，成员共享（复用）这块空间。
union U
{
    int x;
    double y;
};

int main()
{
    std::cout << sizeof(U) << std::endl;

    U u;
    // 1. 可以使用int 或者 double 类型的值来初始化 u。 
    u.x = 66;
    u.y = 99.33;  // 覆盖之前的值。

    // 2. 可以使用int 或者 double 类型来读取 u。
    std::cout << u.x << std::endl;           // 以int的存储格式（定点数）来读取这块内存，注意了！ 这里不是类型转换！！！。
    std::cout << u.y << std::endl;           // 以浮点数的存储格式读取这块内存。

    return 0;
}
```
输出：  
```
8
-1202390843
99.33
```


# 2. 匿名联合体
道理类似于无作用域枚举。   

# 3. 联合中包含非内建类型
待续...