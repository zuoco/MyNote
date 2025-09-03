---
title: "泛型算法"
description: "了解C++标准库提供的一些泛型算法"  
date: 2023-05-27
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
    - "C++ STL"
---

1. [1. 泛型算法](#1-泛型算法)
2. [2. 典型泛型算法](#2-典型泛型算法)
  1. [2.1. 读算法](#21-读算法)
    1. [2.1.1. accumulate](#211-accumulate)
    2. [2.1.2. find](#212-find)
    3. [2.1.3. count](#213-count)
    4. [2.1.4. distance](#214-distance)
  2. [2.2. 写算法](#22-写算法)
    1. [2.2.1. fill](#221-fill)
    2. [2.2.2. fill\_n](#222-fill_n)
    3. [2.2.3. transform](#223-transform)
    4. [2.2.4. copy](#224-copy)
  3. [2.3. 排序算法](#23-排序算法)
    1. [2.3.1. sort](#231-sort)
    2. [2.3.2. unique](#232-unique)
3. [3. 其他相关](#3-其他相关)
  1. [3.1. 并行计算](#31-并行计算)
  2. [3.2. 谓词](#32-谓词)


# 1. 泛型算法
C++标准库提供了一些泛型算法，使用时需要包含对应的头文件：     
```cpp
#include <algorithm>  
#include <numeric>  
#include <ranges> 
```

**1.**&emsp;`为什么引入泛型算法而不是使用方法？`   
&emsp;&emsp;一方面，内建的类型是不支持方法的，如int、char、数组、指针等，这些类型不支持方法，只能使用运算符。另一方面，一些方法的计算逻辑是相似的，没有必要重复定义。      

**2.**&emsp;`泛型算法的简单举例`？    
&emsp;&emsp;泛型算法使用模板来实现，例如std::sort， 这是最常用的泛型算法：  
```cpp
// (constexpr since C++20)
template< class RandomIt >
void sort( RandomIt first, RandomIt last );                                               
```
&emsp;&emsp;编译器在编译期间会进行`模板特化`，这就是泛型算法实现的泛型的背后逻辑。  


**3.**&emsp;`迭代器与泛型算法`？  
&emsp;&emsp;迭代器是算法与数据结构之间的桥梁，泛型算法使用迭代器访问元素。   

**4.**&emsp;`泛型算法与类方法名称相同时`？   
&emsp;&emsp;此时使用类方法，因为，为了实现“泛型”带来了性能损失，例如： `std::find`和`std::map::find`，std::find是线性查找，而std::map::find是二分查找。   


# 2. 典型泛型算法

## 2.1. 读算法
&emsp;&emsp;给定迭代区间，读取其中的元素并进行计算： `accumulate`/`find`/`count`。        
### 2.1.1. accumulate  
&emsp;&emsp;对元素进行累加计算，需要包含头文件`#include <numeric>`，函数声明如下：     
```cpp
// (constexpr since C++20)
template< class InputIt, class T >
T accumulate( InputIt first, InputIt last, T init );                  

// (constexpr since C++20)
template< class InputIt, class T, class BinaryOp >
T accumulate( InputIt first, InputIt last, T init, BinaryOp op );
```
- first, last:&emsp;输入区间的起始迭代器，[first, last)。     
- init:       &emsp;进行累加的初始值。   
- op:         &emsp;二元操作函数对象。它定义了如何将当前累积结果 (result) 与序列中的当前元素 (*first) 组合成新的累积结果。   
&emsp;&emsp;如果字符串类型使用accumulate，那就是拼接字符串。   

### 2.1.2. find   
&emsp;&emsp;包含头文件`#include <algorithm>`，函数声明如下： 
```cpp
// (constexpr since C++20)  (until C++26)
template< class InputIt, class T >
InputIt find( InputIt first, InputIt last, const T& value );
```
- first, last:&emsp;输入区间的迭代器，[first, last) 。     
- value:&emsp;要查找的值。   
- 返回值：&emsp;返回一个迭代器，指向第一个等于value的元素，如果没有元素等于value，则返回last。   


### 2.1.3. count
&emsp;&emsp;包含头文件`#include <algorithm>`，函数声明如下： 
```cpp
// 计算范围 [first, last) 中等于 value 的元素数量。 
// (constexpr 自 C++20 起)  (直到 C++26)。
template< class InputIt, class T >
typename std::iterator_traits<InputIt>::difference_type
    count( InputIt first, InputIt last, const T& value );

// 计算范围 [first, last) 中满足谓词条件 p 的元素数量。
// (constexpr 自 C++20 起)
template< class InputIt, class UnaryPred >
typename std::iterator_traits<InputIt>::difference_type
    count_if( InputIt first, InputIt last, UnaryPred p );
```
- first、last: &emsp;输入迭代器范围，在输入迭代器范围 [first, last) 中查找满足条件的元素的数量。        
    - value: &emsp;统计范围 [first, last) 内等于 value 的元素数量。
    - p: &emsp;一元谓词，用于判断元素是否满足条件（true：满足条件，false：不满足条件）。  
- 关于UnaryPred（一元谓词）：&emsp;一个接受单个参数并返回布尔值的可调用对象，可以是Lambda、函数指针、函数对象。   
- 返回值：&emsp;typename std::iterator_traits<InputIt>::difference_type，其中typename告诉编译器这是类型而不是值，difference_type：表示两个迭代器间距离的有符号整数类型。    

### 2.1.4. distance
&emsp;&emsp;求取迭代器间距离。               
```cpp
template< class InputIt >
typename std::iterator_traits<InputIt>::difference_type
    distance( InputIt first, InputIt last );
```
实现案例如下：   
```cpp
template<class It>
constexpr typename std::iterator_traits<It>::difference_type  
    distance(It first, It last)
{
    using category = typename std::iterator_traits<It>::iterator_category;
    static_assert(std::is_base_of_v<std::input_iterator_tag, category>);
 
    // 根据迭代器类型进行优化
    if constexpr (std::is_base_of_v<std::random_access_iterator_tag, category>)
        return last - first; //随机访问迭代器支持迭代器距离 
    else
    {
        // 普通迭代器，只能通过++first来计算距离
        typename std::iterator_traits<It>::difference_type result = 0;
        while (first != last)
        {
            ++first;
            ++result;
        }
        return result;
    }
}
```

## 2.2. 写算法
&emsp;&emsp;给定迭代区间，将元素写入到指定位置: `fill`/`fill_n`/`transpose`/`copy`。    

### 2.2.1. fill 
```cpp
// (constexpr since C++20)  (un til C++26)  
template< class ForwardIt, class T >
void fill( ForwardIt first, ForwardIt last, const T& value );  
```
- 将范围 [first, last) 内的所有元素赋值为指定的 value。

### 2.2.2. fill_n   
```cpp
// (constexpr since C++20)   (until C++26)
template< class OutputIt, class Size, class T >
OutputIt fill_n( OutputIt first, Size count, const T& value );   
```
- 将指定值 value 赋给从 first 开始的连续 count 个元素。  

### 2.2.3. transform   
```cpp
// (constexpr since C++20)
template< class InputIt, class OutputIt, class UnaryOp >
OutputIt transform( InputIt first1, InputIt last1, OutputIt d_first, UnaryOp unary_op );
```
- 将输入范围 [first1, last1) 中的每个元素应用一元操作 unary_op，并将结果写入从 d_first 开始的输出范围。
```cpp
// 	(constexpr since C++20)
template< class InputIt1, class InputIt2, class OutputIt, class BinaryOp >
OutputIt transform( InputIt1 first1, InputIt1 last1, InputIt2 first2, OutputIt d_first, BinaryOp binary_op );
```
- 将两个输入范围 [first1, last1) 和 [first2, first2 + N)（其中 N = last1 - first1) 中的对应元素应用二元操作 binary_op，并将结果写入从 d_first 开始的输出范围。  
- 这个版本的transform用在什么地方呢？例如向量/矩阵的运算，如下：  
```cpp
// 向量相加
std::vector<double> a{1.0, 2.0, 3.0}, b{4.0, 5.0, 6.0};
std::vector<double> result(a.size());
std::transform(a.begin(), a.end(), b.begin(), result.begin(), [](double x, double y) { return x + y; });
```

### 2.2.4. copy
```cpp
template< class InputIt, class OutputIt >
OutputIt copy( InputIt first, InputIt last, OutputIt d_first );
```
- 将范围 [first, last) 内的元素按顺序复制到从 d_first 开始的目标位置


## 2.3. 排序算法
### 2.3.1. sort
```cpp

template< class RandomIt >
void sort( RandomIt first, RandomIt last );
```
- 对范围 [first, last) 内的元素进行升序排序（默认使用 operator<）。  

```cpp
template< class RandomIt, class Compare >
void sort( RandomIt first, RandomIt last, Compare comp );
```
- 对范围 [first, last) 内的元素进行升序排序（使用 comp 比较）。
- Compare类型： &emsp;比较函数，签名形式如下：  
```cpp
bool cmp(const Type1& a, const Type2& b);
// a < b 时返回true。
```

### 2.3.2. unique   
```cpp
// (constexpr since C++20)
template< class ForwardIt >
ForwardIt unique( ForwardIt first, ForwardIt last );  

// (constexpr since C++20)
template< class ForwardIt, class BinaryPred >
ForwardIt unique( ForwardIt first, ForwardIt last, BinaryPred p );
```
- 在范围[first， last）中，如果有多个重复且连续的元素，就保留第一个，删除其他几个重复的元素。
- unique操作后，容器有效数据范围缩小，返回一个迭代器，指向新的END，但是注意了，容器本身大小不变，可以理解容器范围为： [first, new_end),[new_end, last)。  


# 3. 其他相关

## 3.1. 并行计算
&emsp;&emsp;在C++标准库中，每个函数除了上面列举的重载类型，还是存在其他重载类型，如：    
```cpp
template< class RandomIt >
void sort( RandomIt first, RandomIt last );

// ExecutionPolicy
template< class ExecutionPolicy, class RandomIt >
void sort( ExecutionPolicy&& policy, RandomIt first, RandomIt last );
```
&emsp;&emsp;函数声明中的参数为涉及到 ExecutionPolicy 类型用于指定算法执行策略的类型，它允许开发者控制算法是顺序执行还是并行执行。通过传递不同的策略对象，可以控制算法的执行模式（如顺序、并行或向量化），这在处理大规模数据时可以显著提升性能。    

&emsp;&emsp;C++ 标准定义了多种执行策略，每种都有对应的全局对象：    
|            策略类型                           |          全局对象           |  版本  |          描述                 |  
|----------------------------------------------|---------------------------|-------|-------------------------------|  
| std::execution::sequenced_policy             | std::execution::seq       | C++17 |  顺序执行（无并行）              |  
| std::execution::parallel_policy              | std::execution::par       | C++17 |  并行执行（多线程）              |  
| std::execution::parallel_unsequenced_policy  | std::execution::par_unseq | C++17 |  并行+向量化执行（多线程+SIMD）   |  
| std::execution::unsequenced_policy           | std::execution::unseq     | C++20 |  向量化执行（单线程 SIMD）       |  


&emsp;&emsp;**食用方法**：  
```cpp
std::sort(std::execution::seq, first, last);
std::sort(std::execution::par, first, last);
std::sort(std::execution::par_unseq, first, last);
std::sort(std::execution::unseq, first, last);  
```

&emsp;&emsp;不同的计算任务对于这几个策略的效率不同，例如排序算法，多线程比单线程有较大提升，但是使用SIMD就没有什么提升了。 
&emsp;&emsp;Single Instruction, Multiple Data（单指令多数据流），现代CPU具有多PU单元，可以同时计算多条数据流，适用于向量、矩阵运算。   


## 3.2. 谓词
谓词，一种可调用对象：   
- 返回bool类型的函数对象。   
- 函数接受一个参数，称为一元谓词。
- 函数接受两个参数，称为二元谓词。
- 一般使用lambda表达式来定义谓词。    




