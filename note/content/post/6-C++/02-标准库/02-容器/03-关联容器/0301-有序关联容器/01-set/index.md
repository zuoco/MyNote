---
title: "有序关联容器之 Set"
description: 
date: 2023-04-22
hidden: false
comments: true
draft: false
categories:
    - "C++"
    - "C++ STL"
    - "C++容器"
---

- [1. 初步认识](#1-初步认识)
- [2. 自定义比较函数](#2-自定义比较函数)
- [3. 迭代器](#3-迭代器)
- [4. 插入元素](#4-插入元素)
  - [4.1. insert、emplace](#41-insertemplace)
  - [4.2. emplace\_hint](#42-emplace_hint)
- [5. 删除元素](#5-删除元素)
- [6. 访问元素](#6-访问元素)
  - [6.1. contains](#61-contains)
  - [6.2. find](#62-find)
- [7. 修改元素](#7-修改元素)


# 1. 初步认识
set底层是红黑树，因此set中的元素必须支持比较大小，对于自定义类型需要重载“<”，或者自定义一个比较函数并在创建set对象时指定。     
set被放到关联容器中范畴里面，是从功能上来说的，它本身不是基于键值对形式的，如果非要理解为键值对形式的，那么键就是值，值就是键。      
```cpp
template<
    class Key,
    class Compare = std::less<Key>,
    class Allocator = std::allocator<Key>
> class set;
```
上面从类模板中，Compare是用于元素比较的函数，Allocator是分配器，这两个都有默认的参数，所以我们可以如下创建set对象：   
```cpp
# include <iostream>
# include <set>

int main()
{
    /**
     * 初始化时，元素顺序不重要，set会进行排序；
     * 元素必须支持比较大小，set会去除重复元素；
     */
    std::set<int> s{66, 99, 88};
    for (auto ptr = s.begin(); ptr != s.end(); ++ptr)
    {
        std::cout << *ptr << std::endl;
    }
}
```
代码输出:  
```
66
88
99
```
代码输出结果为升序，考虑到set底层使用是红黑树，因此推测set遍历方式为`中序遍历`。   


# 2. 自定义比较函数
仿函数。   
```cpp
#include <iostream>
#include <set>

// 自定义比较仿函数：按降序排序
struct CompareDesc {
    bool operator()(int a, int b) const {
        return a > b;  // 降序排列
    }
};

int main() {
    std::set<int, CompareDesc> mySet = {3, 1, 4, 1, 5};
    for (const auto& val : mySet) {
        std::cout << val << " ";  // 输出：5 4 3 1
    }
    return 0;
}
```   
Lambda 表达式。   
```cpp
#include <iostream>
#include <set>
#include <functional>

int main() {
    // 使用 Lambda 表达式定义比较规则（降序）
    auto compareDesc = [](int a, int b) { return a > b; };
    std::set<int, decltype(compareDesc)> mySet(compareDesc);

    mySet.insert(3);
    mySet.insert(1);
    mySet.insert(4);

    for (const auto& val : mySet) {
        std::cout << val << " ";  // 输出：4 3 1
    }
    return 0;
}
```

# 3. 迭代器
支持双向迭代器，但是set迭代器所指向的对象是const的，因此不能通过迭代器修改元素。   


# 4. 插入元素
## 4.1. insert、emplace
```cpp
int main()
{
    std::set<Str, decltype(&MyCompare)> s{Str{Str{66}, Str{99}}, Compare};
    s.insert(Str{88}});
    s.emplace(77);
}
```
根据参数来看，insert是先构造一个Str对象，然后拷贝或者移动到s中，而emplace直接在s中构造一个Str对象。

## 4.2. emplace_hint
```cpp
template< class... Args >
iterator emplace_hint( const_iterator hint, Args&&... args );
```
通过hint参数，告诉系统，新元素大约要插入到那里，参数"Args&&... args"用来构造新元素，这样能够减少比较次数，但是要求hint准确，如果hint不准确，可能导致效率更低。


# 5. 删除元素  
```cpp
size_type erase( const Key& key );          
iterator erase( const_iterator pos );                        
iterator erase( const_iterator first, const_iterator last );   
```
# 6. 访问元素
## 6.1. contains
```cpp
bool contains( const Key& key ) const;    

template< class K >
bool contains( const K& x ) const;    
```
如果包含就返回true，否则返回false。


## 6.2. find
```cpp
iterator find( const Key& key );                  
const_iterator find( const Key& key ) const;      

template< class K >
iterator find( const K& x );                      

template< class K >
const_iterator find( const K& x ) const;     
```
如果找到元素就返回该元素的迭代器，否则返回end()。


# 7. 修改元素
由于set迭代器是const的，因此不能通过迭代器修改元素，可以使用`extract`()完成。   
```cpp
node_type extract( const_iterator pos );    

node_type extract( const Key& k );          

template< class K >
node_type extract( K&& x );                 
```
使用方法：  
```cpp
#include <algorithm>
#include <iostream>
#include <string_view>
#include <set>
 
void print(std::string_view comment, const auto& data)
{
    std::cout << comment;
    for (auto datum : data)
        std::cout << ' ' << datum;
 
    std::cout << '\n';
}
 
int main()
{
    std::set<int> cont{1, 2, 3};
 
    print("Start:", cont);
 
    // Extract node handle and change key
    auto nh = cont.extract(1);
    nh.value() = 4;
 
    print("After extract and before insert:", cont);
 
    // Insert node handle back
    cont.insert(std::move(nh));
 
    print("End:", cont);
}
```
程序输出：   
```
Start: 1 2 3
After extract and before insert: 2 3
End: 2 3 4
```
可以看出来，extract的使用流程是先将这个元素从set中取出来，然后我们修改元素的值，修改完成后再重新放到set中，但不是原先的位置了，因为值变了。
