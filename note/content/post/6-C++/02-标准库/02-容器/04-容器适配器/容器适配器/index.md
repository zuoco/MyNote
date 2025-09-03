---
title: "容器适配器"
description: 
date: 2023-05-18
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




# 啥是容器适配器
`容器适配器`还是`容器`，具体地说就是对标准库的基础容器进行包装，把容器的成员函数组合一下，如此一来就搞出来一个新的容器，这个新容器可以满足某些特殊需求。   


|容器适配器	| 对于基础容器的要求 |满足条件的基础容器 | 默认使用的基础容器 |
|----------|----------------|-----------------|------------------|
|<br><br> stack | empty() <br> size() <br> back() <br> push_back() <br> pop_back()| vector <br> deque <br> list | deque |
|<br><br> queue|	empty() <br> size() <br> front() <br> back() <br> push_back() <br> pop_front()| deque <br> list | deque|
|<br><br> priority_queue| empty() <br> size() <br> front() <br> push_back() <br> pop_back() |   vector <br> deque | vector|  




