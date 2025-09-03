---
title: "枚举"
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

1. [1. 枚举](#1-枚举)
    1. [1.1. 无作用域枚举（C版枚举）](#11-无作用域枚举c版枚举)
    2. [1.2. 有作用域枚举](#12-有作用域枚举)
2. [2. 枚举的底层类型](#2-枚举的底层类型)



# 1. 枚举
枚举中的成员默认使用0初始化，从第一个枚举项开始，依次递增，可以使用常量表达式指定。可以将枚举项理解为`整型值的别名`，枚举可以分为`无作用域枚举`和`作有用域枚举`。  

## 1.1. 无作用域枚举（C版枚举）
```cpp
enum Color 
{
    Red,
    Green,
    Blue,
};

int main()
{
    Color color = Red;
}
```
在包含了这个枚举的作用域（Red，Green，Blue属于这个作用域，而不是enum Color）中，所以可以直接使用Red，Green，Blue，而不需要使用Color::Red这种形式。  


## 1.2. 有作用域枚举
```cpp
enum class Color 
{
    Red,
    Green,
    Blue,
};

int main()
{
    Color color = Color::Red;  // 需要使用Color::Red形式
}
```

# 2. 枚举的底层类型
我们可以指定枚举的底层类型，比如：  
```cpp
enum class Color : char
{
    Red, 
    Green,
    Blue,
};
```
此时，枚举项占用char大小的空间，这个枚举的成员的取值范围也限定了。   

