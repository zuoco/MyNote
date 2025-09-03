---
title: "C++面向对象（04）— C++运算符重载"
description:
date: 2023-03-18
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
---


- [1. **赋值运算符“operator=”**](#1-赋值运算符operator)
- [1.1. **拷贝赋值**](#11-拷贝赋值)
- [1.2. **移动赋值**](#12-移动赋值)


# 1. **赋值运算符“operator=”**
```cpp
int main()
{
    MyClass a;
    MyClass b;
    a = b; // 调用赋值运算符, b.operator=(a)
}
```

# 1.1. **拷贝赋值**
```cpp
struct MyClass {
    int a;
    std::string b; // std::string 已支持深拷贝
    MyClass& operator=(const MyClass&) = default; // 默认生成赋值运算符
};
```
需要深拷贝时要自行实现赋值运算符
```cpp
class MyClass {
public:
    int* data;
    size_t size;

    // 构造函数
    MyClass(size_t s = 0) : size(s), data(new int[s]()) {}

    // 拷贝赋值运算符
    MyClass& operator=(const MyClass& other) {
        if (this != &other) {   // 防止自赋值
            delete[] data;      // 释放已有资源

            size = other.size;
            data = new int[size]; // 分配新资源
            std::copy(other.data, other.data + size, data); // 深拷贝数据
        }
        return *this; // 返回赋值后的结果
    }


    ~MyClass() {
        delete[] data;
    }
};
```

# 1.2. **移动赋值**
```cpp
struct MyClass {
    std::vector<int> vec; // 支持移动语义
    MyClass& operator=(MyClass&& other) = default; // 默认生成移动赋值
};
```
 
```cpp
class MyClass {
public:
    int* data;
    size_t size;

    // 构造函数
    MyClass(size_t n = 0) : data(new int[n]), size(n) {}

    // 析构函数
    ~MyClass() {
        delete[] data;
    }

    // 移动赋值运算符重载
    MyClass& operator=(MyClass&& other) noexcept {
        if (this != &other) {
            delete[] data;             // 释放当前对象拥有的资源
            data = other.data;         // 转移 other 的资源
            size = other.size;

            other.data = nullptr;      // 防止 other 在析构时释放资源
            other.size = 0;
        }
        return *this;
    }
};
```


