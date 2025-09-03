---
title: "有序关联容器之 multimap和multiset"
description: 
date: 2023-04-24
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


- [1. map和multimap的区别](#1-map和multimap的区别)
  - [1.1. 区别](#11-区别)
  - [1.2. 特殊成员函数](#12-特殊成员函数)
- [2. set和multiset的区别](#2-set和multiset的区别)
  - [2.1. 区别](#21-区别)
  - [2.2. 特殊成员](#22-特殊成员)



# 1. map和multimap的区别
## 1.1. 区别

- 相对于map，multimap允许存在重复的key。   
- 和 map 容器相比，multimap 未提供 at() 成员方法，也没有重载 [] 运算符。这意味着，map 容器中通过指定键获取指定指定键值对的方式，将不再适用于 multimap 容器。其实这很好理解，因为 multimap 容器中指定的键可能对应多个键值对，而不再是 1 个。

## 1.2. 特殊成员函数
|||
|------------|--------------|
|lower_bound(key)|	返回一个双向迭代器，指向当前 multimap 容器中第一个大于或等于 key 的键值对。<br> 如果 multimap 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。|
|upper_bound(key) |  返回一个双向迭代器，指向当前 multimap 容器中第一个大于 key 的键值对。<br> 如果 multimap 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。|
|equal_range(key)	| 返回一个 pair 对象（包含 2 个双向迭代器），其中 pair.first 和 lower_bound() 方法的返回值等价，pair.second 和 upper_bound() 方法的返回值等价。也就是说返回了一个范围，该范围中的元素就是键为 key 的键值对。|
|count(key)	| 在当前 multimap 容器中，查找键为 key 的键值对的个数并返回。|




# 2. set和multiset的区别
## 2.1. 区别
相对于set，multiset允许存在重复的元素。    


## 2.2. 特殊成员
和multimap一样。




