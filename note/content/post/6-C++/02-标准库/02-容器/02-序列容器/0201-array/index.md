---
title: "序列容器之 Array"
description: 
date: 2023-04-15
comments: true
draft: false
categories:
    - "C++"
    - "C++ STL"
    - "C++容器"
---

- [1. Array](#1-array)
- [2. 用法](#2-用法)
  - [2.1. 迭代器](#21-迭代器)
  - [2.2. 成员函数](#22-成员函数)
    - [2.2.1. 数组填充](#221-数组填充)
    - [2.2.2. 元素交换](#222-元素交换)
    - [2.2.3. 比较](#223-比较)
    - [2.2.4. 容量方法](#224-容量方法)


# 1. Array
[std::array](https://en.cppreference.com/w/cpp/container/array.html)，基于数组封装，大小固定，不能动态改变。    
```cpp
// Defined in header <array>
template<
    class T,
    std::size_t N
> struct array;
```

# 2. 用法
思路和数组差不多。    
```cpp
#include <array>

int main() {
    // 1. 声明一个包含5个整数的数组
    std::array<int, 5> arr1 = {1, 2, 3, 4, 5};
    std::array<int, 5> arr2 = {};     // 全部元素初始化为0
    std::array<int, 5> arr3 = {1};    // 第一个元素初始化为1

    // 2. 访问数组元素
    std::cout << "First element: " << arr[0] << std::endl;
    std::cout << "arr.at(10) = " << arr.at(10) << '\n';       // 越界访问会崩溃

    // 3. 遍历数组
    std::ranges::sort(arr);         // 使用 C++20 的 std::ranges::sort 对 std::array 排序
    for (const auto& elem : arr) {
        std::cout << elem << ' ';
    }
    std::cout << "\n";

    // 4. 访问数组的第一个和最后一个元素
    std::cout << "First element: " << arr.front() << std::endl;
    std::cout << "Last element: " << arr.back() << std::endl;
}
```


## 2.1. 迭代器
支持最强大的随机访问迭代器。
```cpp
    std::cout << "Using forward iterator:\n";
    for (auto it = arr.begin(); it != arr.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << "\n";
```

## 2.2. 成员函数
### 2.2.1. 数组填充
用于将容器中的所有元素都设置为指定的值。
```cpp
void fill( const T& value );    // (since C++11)
                                // (constexpr since C++20)
```

```cpp
#include <array>
#include <cstddef>
#include <iostream>
 
int main()
{
    constexpr std::size_t xy = 4;
 
    using Cell = std::array<unsigned char, 8>;
 
    std::array<Cell, xy * xy> board;
 
    board.fill({0xE2, 0x96, 0x84, 0xE2, 0x96, 0x80, 0, 0}); // "▄▀";
 
    for (std::size_t count{}; Cell c : board)
        std::cout << c.data() << ((++count % xy) ? "" : "\n");
}
```
输出：    
```
▄▀▄▀▄▀▄▀
▄▀▄▀▄▀▄▀
▄▀▄▀▄▀▄▀
▄▀▄▀▄▀▄▀
```
这个比较有趣，是一个棋盘，讲解一下：   
这个棋盘由16个“▄▀”组成，每个“▄▀”在代码中用一个Cell表示，对于每个Cell:   
```cpp
Cell c = {0xE2, 0x96, 0x84, 0xE2, 0x96, 0x80, 0, 0};
```
**0xE2, 0x96, 0x84**打印出来就是“▄”，而**0xE2, 0x96, 0x80**打印出来就是“▀”， **0, 0**确保字符串以空字符结尾（因为代码中使用了data()方法）。   


### 2.2.2. 元素交换
```cpp
#include <array>
#include <iostream>
/* 
 * 重载“<<” 
 */
template<class Os, class V> 
    Os& operator<<(Os& os, const V& v)
{
    os << '{';
    for (auto i : v)
        os << ' ' << i;
    return os << " } ";
}
 

int main()
{
    std::array<int, 3> a1{1, 2, 3}, a2{4, 5, 6};
 
    auto it1 = a1.begin();
    auto it2 = a2.begin();
    int& ref1 = a1[1];
    int& ref2 = a2[1];
 
    std::cout << a1 << a2 << '\n';
    a1.swap(a2);
    std::cout << a1 << a2 << '\n';
}
```
这里简单介绍以下“<<”的工作原理：   
```cpp
template<class Os, class V> Os& operator<<(Os& os, const V& v)
```
函数模板，根据传递的参数进行推导并实例化模板函数。“<<”运算返回Os引用，以便于链式调用：    
```cpp
operator<<(operator<<(std::cout, a1), a2);
```

### 2.2.3. 比较
```cpp
operator==   (C++11)
operator<=>  (C++20)
```

1. **operator==**
```cpp
// (since C++11) (constexpr since C++20)
template< class T, std::size_t N >
bool operator==( const std::array<T, N>& lhs, const std::array<T, N>& rhs ); 
```
要求双方类型相同。
```cpp
std::array<int, 3> x;
std::array<int, 4> y;
```
x和y的类型不同，所以不能比较。  

2. **operator<=>**  
飞船运算符，以后单独章节介绍。   


### 2.2.4. 容量方法
对于std::array，**empty()** 、**size()** 、**max_size()**，这些在编译期就确定了，所以运行时调用这些方法没有意义。那么为什么还要提供这些方法呢？因为这些方法在其他容器中是有意义的，为了统一接口，这些方法在std::array中也提供了。