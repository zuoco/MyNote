---
title: "容器适配器之 queue"
description: 
date: 2023-05-17
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
    - "C++ STL"
    - "C++容器"
---


- [1. std::queue](#1-stdqueue)
- [2. 创建 queue 容器对象](#2-创建-queue-容器对象)
- [3. 成员函数](#3-成员函数)


# 1. std::queue
底层可以使用deque或者list，默认使用deque。  
```cpp
// Defined in header <stack>
template<
    class T,
    class Container = std::deque<T>
> class stack;
```


# 2. 创建 queue 容器对象
```cpp
// 一个空的 queue 容器对象
std::queue<int> que;
```
------
```cpp
// 指定底层容器
std::queue<int, std::list<int>> que;
```
------
```cpp
// 使用底层容器实例初始化queue容器实例
std::deque<int> sq{1,2,3};
std::queue<int> que(values); // 拷贝sq的内存
```
------
```cpp
std::deque<int> sq{1,2,3};
std::queue<int> que1(values);
std::queue<int> que2(my_queue1);  // 拷贝构造
```

# 3. 成员函数

|成员函数	| 功能 |
|----------|-----|
|empty()	| queue为空返回 true。|
|size()	|返回 queue 中元素的个数。|
|front() |	返回 queue 中第一个元素的引用。如果 queue 是常量，就返回一个常引用；如果 queue 为空，返回值是未定义的。|
|back()	| 返回 queue 中最后一个元素的引用。如果 queue 是常量，就返回一个常引用；如果 queue 为空，返回值是未定义的。|
|push(const T& obj)	| 在 queue 的尾部添加一个元素的副本。调用底层容器的成员函数 push_back()。|
|emplace()|	在 queue 的尾部原地构造一个元素。|
|push(T&& obj)|	以移动的方式在 queue 的尾部添加元素。调用底层容器的具有右值引用参数的成员函数 push_back() 完成。|
|pop()	|删除 queue 中的第一个元素。|
|swap(queue<T> &other_queue)|	两个 queue 容器中的元素互换，2 个 容器中存储的元素类型以及底层采用的基础容器类型都必须相同。|



