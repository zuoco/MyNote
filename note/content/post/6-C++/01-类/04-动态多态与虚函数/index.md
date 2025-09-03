---
title: "C++面向对象（06） — 动态多态与虚函数"
description: 
date: 2023-03-26
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
---

- [1. **虚函数与纯虚函数**](#1-虚函数与纯虚函数)
  - [1.1. **虚函数**](#11-虚函数)
  - [1.2. **纯虚函数**](#12-纯虚函数)
- [2. **构造与析构**](#2-构造与析构)
  - [2.1. **构造**](#21-构造)
  - [2.2. **析构**](#22-析构)
- [3. **虚函数表**](#3-虚函数表)
  - [3.1. **什么是虚函数表**](#31-什么是虚函数表)
  - [3.2. **虚函数表结构**](#32-虚函数表结构)
- [4. **多态**](#4-多态)
  - [4.1. **类的向上转型**](#41-类的向上转型)
  - [4.2. **基类指针访问派生类**](#42-基类指针访问派生类)
    - [4.2.1. **访问成员函数**](#421-访问成员函数)
    - [4.2.2. **访问成员变量**](#422-访问成员变量)


# 1. **虚函数与纯虚函数**
- `抽象类`：  包含至少一个`纯虚函数`的类，这种类无法实例化，用于定义接口规范，然后让派生类去实现。     
- `强制实现`：派生类必须实现所有纯虚函数，否则仍为`抽象类`。  

## 1.1. **虚函数**
基类中声明为 virtual 的成员函数，允许派生类继承并重写该函数，从而实现运行时多态性。  
```cpp
#include <iostream>
using namespace std;

class A {
public:
    A() { cout << "A constructor" << endl; }
    virtual ~A() { cout << "A destructor" << endl; }    // 虚析构

    virtual void print() {
        cout << "A::print()" << endl;
    }
};

class B : public A {
public:
    B() { cout << "B constructor" << endl; }
    ~B() { cout << "B destructor" << endl; }

    // 覆盖虚函数
    void print() override {
        cout << "B::print()" << endl;
    }
};

int main() {
    A* ptr = new B();  // 基类指针指向派生类对象
    ptr->print();      // 实际调用的是B实现的print

    delete ptr;        // 多态析构，调用B的析构函数
    return 0;
}
```
代码输出：  
```bash
A constructor
B constructor
B::print()
B destructor
A destructor
```
`动态绑定`特性，通过基类指针或引用调用虚函数时，会根据所指向的对象的实际类型动态决定调用哪个实现。

## 1.2. **纯虚函数**   
声明为`virtual void func() = 0;`的虚函数，基类不会给出实现。      
```cpp
class AbstractBase {
public:
    virtual void pureFunc() = 0;  // 纯虚函数
};

class Concrete : public AbstractBase {
public:
    void pureFunc() override { std::cout << "Concrete::pureFunc" << std::endl; }
};
```
一般C++的项目的架构中，会有一个最底层的基类，只有默认构造函数和虚析构函数，项目的所有类型的基类都从这里派生。


# 2. **构造与析构**

## 2.1. **构造**
构造函数和普通类型的构造函数的规则一样。   

## 2.2. **析构**
抽象类的析构函数必须是虚函数。    
```cpp
class AbstractBase {
public:
    virtual ~AbstractBase() = default;
}
```

# 3. **虚函数表**
## 3.1. **什么是虚函数表**
&emsp;&emsp;是编译器在`编译期`为每个有虚函数的类自动生成虚函数的函数指针数组（`虚函数表vtable`），只读数据段，在程序加载到内存时已存在。在`运行期`，通过构造函数设置对象的隐藏指针（vptr）指向虚函数表来实现多态调用（这个 vptr 通常位于对象内存的最前面）。 在继承链中，构造函数的调用顺序决定了 vptr 的初始化过程。  
&emsp;&emsp;在vtable中，基类的虚函数的索引（下标）是固定的，不会随着继承层次的增加而改变，派生类新增（基类没有的）的虚函数放在后面。如果派生类继承并重写了基类虚函数，那么就使用派生类的实现的虚函数替换基类的虚函数，位置是不变的。这样具有遮蔽关系的虚函数在vtable中只会出现一次。在单继承情况下，当通过对象指针p调用虚函数时，编译器内部会发生类似下面的转换：
<div style="width: 80%; margin: 0 auto; background-color:#f5f9ff; padding:1em; border-radius:10px; box-shadow: 0 2px 6px rgba(0,0,0,0.1);">
<code style="color:#333; font-family:monospace; font-size:1.2em; white-space: pre;">
<mark style="background-color:#ffeb3b; padding:0.2em 0.4em; border-radius:4px;">*(p+vbi)</mark>               //虚函数表的地址
<mark style="background-color:#8bc34a; padding:0.2em 0.4em; border-radius:4px;">*(*(p+vbi)+vfi)</mark>        //该虚函数的在表中的地址
<mark style="background-color:#2196f3; padding:0.2em 0.4em; border-radius:4px;">(*(*(p+vbi)+vfi))(p);</mark>   //函数调用，p是参数
</code>
</div>


- `vbi`是`vfptr`(虚函数表指针)在对象中的偏移，`p+vbi`是`vfptr`的地址，虚函数表指针始终位于对象的起始位置，所以`vbi = 0`;
- `*(p+vbi)`是`vfptr`的值，而`vfptr`是指向`vtable`的指针，所以`*(p+vbi)`也就是vtable的地址;
- `vfi`是虚函数在`vtable`中的索引，所以`(*(p+vbi)+vfi)`也就是虚函数的地址;
- 知道了虚函数的地址，`(*(*(p+vbi)+vfi))(p)`也就是对虚函数的调用了，这里的p就是传递的实参，它会赋值给this指针。

可以看到，转换后的表达式是固定的，对于同一个虚函数，不管是哪个派生类的对象来调用，都会使用这个表达式，最终指向的是虚函数表中的同样位置。   


## 3.2. **虚函数表结构**
- 基类的虚函数在 vtable 中的索引（下标）是固定的，不会随着继承层次的增加而改变。  
- 派生类新增的虚函数放在基类虚函数的后面。   
- 如果派生重写了基类的虚函数，那么将使用派生类重写的虚函数替换基类的虚函数，这样vtable只有重写后的虚函数。     

# 4. **多态**
多态可以分为编译时多态(重载、模板)和运行时多态（虚函数），静态多态在编译时就绑定了，动态多态在运行时才绑定，需要虚函数表的支持，所以动态多态，多少还是会带来性能损失的。此处主要讲解动态多态。   

## 4.1. **类的向上转型**   
- 向上转型（Upcasting）是指，从派生类类型转换为基类类型。
- 在C++中，向上转型可以是隐式的，也可以是显式的，使用显式类型转换主要是为了代码的清晰性和可读性（ static_cast()、dynamic_cast() ）。  
- 向上转型后通过基类的对象、指针、引用只能访问从基类继承过去的成员（包括成员变量和成员函数），不能访问派生类新增的成员。  

**向上转型的三种情况：**     

1. **将派生类指针赋值给基类指针**   
2. **将派生类引用赋值给基类引用**: 类似于将派生类指针赋值给基类指针。  
3. **将派生类对象赋值给基类对象**: 对象的内存只包含了成员变量，所以对象之间的赋值是成员变量的赋值。将派生类对象赋值给基类对象时，会舍弃派生类新增的成员     

## 4.2. **基类指针访问派生类**
### 4.2.1. **访问成员函数**
编译器根据指针类型查找成员函数，然后成员函数根据this指针访问成员变量。如果没有虚函数机制，类型指针指向派生类对象时，使用的成员函数是基类的，但使用的成员变量却是派生类的（继承自基类的），如此一来，代码运行的结果必然会很离谱。C++通过`虚函数`机制来支持基类指针调用派生类成员函数。   

### 4.2.2. **访问成员变量**
看下面代码，A类型的指针指向C类型的对象，此时只能访问A子对象中的成员。当有名字遮蔽时，例如下面代码中变量**x**，实际上内存中存在两个**x**，只是内层作用域将外层作用域的同名变量遮蔽了，此时使用A类型的指针指向派生类对象，访问到的就是外层作用域的变量，使用C类型的指针访问到的就是内层作用域的变量。一般也不会去遮蔽基类成员变量，遮蔽基类成员函数的情况很多。  
```cpp
#include <iostream>

class A {
public:
    int a;
    int x = 100;
};

class B : public A {
public:
    int b;
};

class C : public B {
public:
    int c;
    int x;
};

int main()
{
    C m;
    m.a = 66;
    m.x = 88;

    C *c = &m;
    B *b = &m;
    A *a = &m;

    std::cout <<  a->a << std::endl;     // 只能访问A子对象的成员，A::a
    // std::cout << a->c << std::endl;   // 错误，A中没有成员c。
    std::cout <<  b->b << std::endl;     // 只能访问B子对象的成员，B::b, A::a

    std::cout << c->x << std::endl;      // 输出88
    std::cout << a->x << std::endl;      // 输出100，参考《c++对象内存模型》篇章
}静态多态在编译时就绑定
```