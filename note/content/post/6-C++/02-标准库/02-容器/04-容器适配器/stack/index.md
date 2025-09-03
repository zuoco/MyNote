---
title: "容器适配器之 stack"
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


- [1. std::stack](#1-stdstack)
- [2. 创建Stack容器对象](#2-创建stack容器对象)
- [3. 成员函数](#3-成员函数)


# 1. std::stack
底层可以使用vector、deque或者list。   
```cpp
// Defined in header <stack>
template<
    class T,
    class Container = std::deque<T>  // 底层容器默认是deque
> class stack;
```

# 2. 创建Stack容器对象
```cpp
// 一个空的stack
std::stack<int> st;
```
------
```cpp
// 指定底层容器为list
std::stack<int, std::list<int>> st;  
```
------
```cpp
// 使用已有的基础容器实例初始化栈对象。
// 但要求基础容器类型和当前栈的底层类型相同。
std::list<int> lst = {4, 5, 6};
std::stack<int, std::list<int>> stack_list(lst);  // 拷贝lst的内存
```
------
```cpp
// 拷贝构造
// 要求基础栈数据类型和栈的底层类型相同。
#include <iostream>
#include <stack>

int main() {
    std::stack<int> original;
    original.push(10);
    original.push(20);
    original.push(30);

    std::stack<int> copied(original);

    while (!copied.empty()) {
        std::cout << copied.top() << " ";
        copied.pop();
    }

    return 0;
}
```

# 3. 成员函数

|成员函数	| 功能 |
|----------|------|
|empty()|	栈为空时返回true。|
|size()	|返回 stack 栈中存储元素的个数。|
|top()	|返回一个栈顶元素的引用，类型为 T&。如果栈为空，程序会报错。|
|push(const T& val)	|先复制 val，再将 val 副本压入栈顶。这是通过调用底层容器的 push_back() 函数完成的。|
|push(T&& obj)	|以移动元素的方式将其压入栈顶。这是通过调用底层容器的有右值引用参数的 push_back() 函数完成的。|
|pop()	|弹出栈顶元素。|
|emplace(arg...)	| 在栈顶位置原地构造一个元素。|
|swap(stack<T> & other_stack)|	将两个stack中的元素进行交换，2 个 stack 中的元素类型以及底层采用的基础容器类型都必须相同。|
