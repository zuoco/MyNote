---
title: "有序关联容器之 map"
description: 
date: 2023-04-23
hidden: false
comments: true
draft: false
categories:
    - "C++"
    - "C++ STL"
    - "C++容器"
---



- [1. std::pair](#1-stdpair)
- [2. map](#2-map)
- [3. 迭代器](#3-迭代器)
- [4. 插入元素](#4-插入元素)
- [5. 删除元素](#5-删除元素)
- [6. 查找](#6-查找)
- [7. 随机访问](#7-随机访问)
- [8. 其他](#8-其他)



# 1. std::pair
```cpp
template<
    class T1,
    class T2
> struct pair;
```
该类模板用于创建“键值对”，T1，T2可以内置类型，也可以是自定义类型。     

# 2. map
```cpp
Defined in header <map>
template<
    class Key,
    class T,
    class Compare = std::less<Key>,
    class Allocator = std::allocator<std::pair<const Key, T>>
> class map;
```
[std:map](https://en.cppreference.com/w/cpp/container/map.html)，基于红黑树，树的每一个节点都是一个std::pair，节点的`pair.first`的类型必须支持“<”运算，或者自定义比较逻辑，相对于set，map才是真正关联容器。    
![](map.svg)  
map的迭代器所指向的对象的“Key”是const的，“Key”本身是不能修改的，这样避免红黑树的平衡结构被改变，当然“value”是可以改变的。   


```cpp
#include <iostream>
#include <map>

int main()
{
    std::map<int,bool> m{{3, true}, {4, false}, {1, true}};
    for (auto p : m) 
    {
        std::cout << p.first << ' ' <<p.second << std::endl;
    }
}
```    
　　　　　　　
```cpp
#include <iostream>
#include <map>

int main()
{
    std::map<int,bool> m{{3, true}, {4, false}, {1, true}};
    for (auto [k, v] : m) 
    {
        std::cout << k << ' ' << v << std::endl;
    }
}
```

# 3. 迭代器
支持双向迭代器。

# 4. 插入元素
```cpp
int main()
{
    std::map<int, bool> m;
    m.insert(std::pair<const int, bool>(3,true));  // 插入一个std::pair
}
```

# 5. 删除元素
```cpp
iterator erase( iterator pos );                              
iterator erase( const_iterator pos );                         
iterator erase( const_iterator first, const_iterator last );  
size_type erase( const Key& key );                            

template< class K > 
    size_type erase( K&& x );                
```

# 6. 查找
```cpp
bool contains( const Key& key ) const;  

template< class K >
bool contains( const K& x ) const;      
```

# 7. 随机访问
使用[key]或者at()访问元素。  
对于`[key]`：   
```cpp
int main()
{
    std::map<int, bool> m;
    m.insert(std::pair<const int, bool>(6, true));
    std::cout << m[6] << std::endl;
}
```
对于`m[6]`，如果树的Key中没有6这个Key，map会向树中插入一个新元素，该节点的Key为6，value使用value类型的默认初始化方式。也就是说，“[]”操作可能会插入新节点，所以要注意const的map对象是不能使用“[]”的，请看如下代码：
```cpp
void fun(const std::map<int, int>& m)
{
    m[3];  //编译期报错，因为m是const的，是不能修改的。
}

int main()
{
    std::map<int, int> m;
    m.insert(std::pair<const int, int>(3, 100));
    fun(m);
}
```


对于`at()`：    
```cpp
int main()
{
    std::map<int, bool> m;
    m.insert(std::pair<const int, bool>(6, true));
    std::cout << m.at(6) << std::endl;
}
```
对于`m[6]`，树的Key中没有6这个Key，代码会报错，程序终止。


# 8. 其他
extract等等。