---
title: "可调用对象之 bind"
description: 
date: 2023-05-29
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


1. [1. 可调用对象](#1-可调用对象)
2. [2. bind](#2-bind)
    1. [2.1. 使用方法](#21-使用方法)
    2. [2.2. bind的问题](#22-bind的问题)
3. [3. bind\_front与bind\_back](#3-bind_front与bind_back)
4. [4. 其他](#4-其他)



# 1. 可调用对象
&emsp;&emsp;很多算法可以通过`可调用对象`来提供自定义计算逻辑细节的能力，`可调用对象`可以是函数指针、函数对象(类)、bind、lambda表达式等。  


# 2. bind
&emsp;&emsp;std::bind通过绑定的方式，生成一个调用包装器（call wrapper），将可调用对象与部分参数绑定，形成一个新的可调用对象。

## 2.1. 使用方法
**1.** 使用案例：  

```cpp
bool MyCompare(int x, int y)
{
    return x > y;
}


int main()
{
    using namespace std::placeholders;     // 占位符定义在这里

    auto x = std::bind(MyCompare, _1, 3); 
    x(2);    // MyCompare(2, 3)
    x(6);    // MyCompare(6, 3)

    auto y = std::bind(MyCompare, _2, 3);
    y("hello", 7);   // MyCompare(7, 3)
}
```
&emsp;&emsp;代码解析：
```cpp
auto x = std::bind(MyCompare, _1, 3);
//   将3绑定到MyCompare的参数y， “_1”是参数x的占位符。
//   返回一个新的可调用对象f。  
//   但是要注意，占位符“_1”中的1指的是： “调用f()时，将f()中的第几个参数传给到MyCompare的参数x”，
//   请看如下代码：  
auto y = std::bind(MyCompare, _2, 3);
y("hello", 7);   // MyCompare(7, 3), 将f("hello", 7)中的第二个参数传给到MyCompare的参数x。
```
&emsp;&emsp;我们来看一下bind的声明：   
```cpp
// (since C++11) (constexpr since C++20)

template< class F, class... Args > 
    bind( F&& f, Args&&... args );


template< class R, class F, class... Args >
    bind( F&& f, Args&&... args );
```
&emsp;&emsp;在创建可调用对象时，也就是在调用std::bind时，参数列表Args&&中的参数顺序（包括占位符）对应着函数f的形参顺序，不在于占位符是“_1“还是”_2”。 而当我们调用`可调用对象`时，传给`可调用对象`几个参数，第一个传给“_1”, 第二个传给“_2”，以此类推。   
&emsp;&emsp;也就是说，当我们调用`可调用对象`时，传入的参数中，有些参数是无效的，因为实际使用的是绑定的参数。调用`可调用对象`时，参数的使用和原函数是一样的，至于那个参数会被采用，这是在调用std::bind时，通过`Args&&中的参数顺序`以及`占位符`设计好的。      


**2.** 使用案例：  
```cpp
bool MyCompare(int x, int y)
{
    return x > y;
}

bool MyAnd(bool x, bool y)
{
    return x && y;
}


int main()
{
    using namespace std::placeholders;

    auto x = std::bind(MyCompare, _1, 3); 
    auto y = std::bind(MyCompare, 6, _1);
    auto z = std::bind(MyAnd, x, y);
    z(4);    // MyAnd(x(4, 3), y(6 ,4)) 
}
```

**3.** 使用案例：
```cpp
bool MyCompare(int x, int y)
{
    return x > y;
}

int main()
{
    using namespace std::placeholders;
    auto x = std::bind(MyCompare, _1, _1);  // x 始终返回false
    x(6); 
}
```
## 2.2. bind的问题
&emsp;&emsp;调用std::bind时，传入的参数会被复制，这可能导致一些风险。   

**1.** 使用案例：   
```cpp
void MyFunc(int* x)
{

}

auto fun()
{
    int x;
    return std::bind(MyFunc, &x);  // 在fun()返回时，x会被销毁
}

int main()
{
    auto f = fun();  
    // 当调用f时，绑定的参数已经失效了，甚至会导致内存错误。   
}
```
&emsp;&emsp;若果有此类场景，可以使用智能指针来解决，在使用std::bind时，传入的参数应该是智能指针，保证参数有效。  


**2.** 使用案例：  
```cpp
void MyFunc(int& x)
{
    ++x;
}


int main()
{
    int x = 0;
    auto fun = std::bind(MyFunc, x);
    fun();
    std::cout << x << std::endl;
    // 输出为0， 因为std::bind，对于传入的参数采用了拷贝的方法， 压根不会改变x的值。
}
```
&emsp;&emsp;这种情况需要使用std::ref、std::cref来改变参数的绑定方式：   
```cpp
auto fun = std::bind(MyFunc, std::ref(x));   
```


# 3. bind_front与bind_back
&emsp;&emsp;std::bind_front, std::bind_back是C++20引入的新特性，用来给第一个参数绑定一个值，或者给最后一个参数绑定一个值。




# 4. 其他
&emsp;&emsp;在使用bind时，绑定那些参数，调用可调用对象时，那些参数有效，参数是如何对应的，非常的绕，所以后来引入了lambda表达式。   
