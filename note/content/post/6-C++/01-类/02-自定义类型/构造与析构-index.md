---
title: "C++面向对象（03） — 构造与析构"
description:
date: 2023-03-12
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
---



- [1. **基本概念**](#1-基本概念)
- [2. **构造函数**](#2-构造函数)
    - [2.1. **默认构造**](#21-默认构造)
    - [2.2. **单一参数构造**](#22-单一参数构造)
    - [2.3. **委托构造**](#23-委托构造)
    - [2.4. **拷贝构成**](#24-拷贝构成)
    - [2.5. **移动构造**](#25-移动构造)
- [3. **初始化列表**](#3-初始化列表)
- [4. **析构函数**](#4-析构函数)


# 1. **基本概念**
- 构造函数名称与类的名称相同。  
- 创建对象，就是调用构造函数对对象进行初始化，确保成员包含有效值。   
- 如果类定义时没有提供构造函数，编译器会默认生成一个无参构造函数。  
- 如果构造函数没有初始化成员变量，那么这个成员变量就会包含一个垃圾值。   
- 一旦我们自定义了构造函数，那么编译器将不会生成默认构造函数，所以自定义构造函数，就必须提供一个默认构造函数。    


# 2. **构造函数**
构造函数一般是public的。  

## 2.1. **默认构造**
```cpp
class MyClass {
public:
    MyClass() = default;   // 默认构造函数
    // 其他构造函数
    // ...
private:
    // ...
}
```
或者： 
```cpp 
class MyClass {
public:
    MyClass(int x = 1, int y = 2, int z = 3);   // 所有参数都有默认值的构造函数也属于默认构造函数
    // 其他构造函数
    // ...
private:
    // ...
}
```

## 2.2. **单一参数构造**
仅接受一个参数的构造函数（会触发隐式类型转换），此种情况下，编译器会自动将参数类型转换为目标类型，无需显式调用构造函数。这种机制简化了代码，但也可能带来潜在风险。   
```cpp
class MyClass {
private:
    int m_age;
public:
    MyClass(int age) : m_age(age) {} // 单参数构造函数
};

void print(const MyClass& MyClass) {
    // ...
}

int main() {
    MyClass e1(10);      // 显式构造

    MyClass e2 = 20;     // 隐式转换：int → MyClass
    print(30);           // 隐式转换：int → MyClass
}
```
为避免意外的隐式转换，C++ 提供了 `explicit` 关键字。将构造函数标记为 explicit 后，只能显式调用构造函数，防止隐式转换。   
```cpp
class MyClass {
private:
    int m_age;
public:
    explicit MyClass(int age) : m_age(age) {} 
};

void print(const MyClass& MyClass) {
    // ...
}

int main() {
    MyClass e1(10);      // 显式构造
    MyClass e1 = 10;     // 错误

    print(e1(10));       // 显式构造
    print(30);           // 错误
}
```

## 2.3. **委托构造**
把构造工作委托给另一个构成函数完成。  

MyClass.h
```cpp
#ifndef MYCLMyClassSS_H
#define MYCLMyClassSS_H

class MyClass {
public:
    // 主构造函数声明
    MyClass(int a, double b);
    
    // 委托构造函数声明
    MyClass();
    MyClass(int a);

private:
    int x;
    double y;
};

#endif // MYCLMyClassSS_H
```
MyClass.cpp
```cpp
#include "MyClass.h"
#include <iostream>  // 包含cout

// 主构造函数定义（包含初始化列表和函数体）
MyClass::MyClass(int a, double b) : x(a), y(b) {
    std::cout << "Primary constructor\n";
}

// 委托构造函数定义（通过初始化列表委托）
MyClass::MyClass() : MyClass(0, 0.0) {
    std::cout << "Delegating constructor\n";
}

// 另一个委托构造函数定义
MyClass::MyClass(int a) : MyClass(a, 1.5) {
    // 可在此添加额外逻辑
}
```

## 2.4. **拷贝构成**
- 对于指针成员，要使用深拷贝。   
- 编译器会自动生成拷贝构造函数，但使用的是浅拷贝。   
- 如果不希望类具备拷贝构造的能力，则使用`delete`关键字，例如单例模式中。    
- 可以使用`MyClass(MyClass& other) = default;`来显式生成默认移动构造函数。   

```cpp
#include <iostream>
#include <memory> // 用于智能指针（非必须，但推荐）

class MyClass {
public:
    explicit MyClass(int value) 
        : data(new int(value)) {
        std::cout << "Constructor called\n";
    }

    // 拷贝构造函数（深拷贝），申请一块新内存，并将对方的数据拷贝过来。
    MyClass(const MyClass& other)
        : data(new int(*other.data)) {
        std::cout << "Copy constructor called\n";
    }

    ~MyClass() {
        delete data;
        std::cout << "Destructor called\n";
    }

    void display() const {
        std::cout << "Value: " << *data << " at " << data << "\n";
    }

private:
    int* data; // 指针成员
};
```

## 2.5. **移动构造**
将对象A所拥有的资源（new出来的资源）转移到对象B中，对象A将不再拥有原本资源的使用权。     
```cpp
#include <iostream>
#include <cstring> // 用于字符串操作

class MyClass {
public:
    explicit MyClass(const char* text) 
        : data(new char[std::strlen(text) + 1]) {
        std::strcpy(data, text);
        std::cout << "Constructed: " << data << "\n";
    }

    // 移动构造函数
    MyClass(MyClass&& other) noexcept
        : data(other.data) {      // 转移资源所有权
        other.data = nullptr;     // 置空原对象指针
        std::cout << "Moved: "  << "\n";
    }


    ~MyClass() {
        if(data) {
            std::cout << "Destroying: " << data << "\n";
            delete[] data;
        } else {
            std::cout << "Destroying null object\n";
        }
    }

    void display() const {
        std::cout << "Content: " << (data ? data : "null") << "\n";
    }

private:
    char* data; // 指针成员（动态分配字符串）
};
```
&emsp;&emsp;与拷贝构造不同，编译器不会自动生成移动构造函数，需要显式定义，可以使用`MyClass(MyClass&& other) noexcept = default;`，来显式生成默认移动构造函数，如果没有移动构造函数就会调用拷贝构造函数。  
**知识点**：      
&emsp;&emsp;`&&`表示右值引用，临时对象（如字面量、函数返回的临时对象），生命周期短，无法直接取地址，通过绑定到右值，将原本即将销毁的临时对象的地址“保存”下来，有了地址就可以像使用左值一样使用了。    
&emsp;&emsp;**noexcept** 用于声明一个函数不会抛出异常。例如std::vector，这个容器在空间不够时会自动扩容，也就是开辟一块新的空间，然后把原先的数据复制过去，然后释放原先的空间。这个拷贝的过程可能是`移动构造`，也可能是`拷贝构造`（如果移动构造不是noexcept的）。`开发人必须保证coexcept函数确实不会抛出异常**`，否则会带来严重的隐患，例如现在有一个`vector<MyClass> v`，容器中有6个成员，扩容的时候调用了noexcept的移动构造，但是扩容到一半抛出了异常，此时旧的空间只有后3个成员（前3个被移动到了新空间），新的空间只有前面3个成员，这样一来新的旧的都坏掉了，所以必须保证noexcept的函数真的不会抛出异常。   


# 3. **初始化列表**
**MyClass.h**  
```cpp
#ifndef MYCLASS_H
#define MYCLASS_H

#include <string>

class MyClass {
public:
    MyClass(int _id, std::string _name, double _value);
    
    void display() const;

private:
    int id;           
    std::string name; 
    double value;     
};

#endif // MYCLASS_H
```

**MyClass.cpp**     
```cpp
#include "MyClass.h"
#include <iostream>

// 构造函数定义（使用初始化列表）
MyClass::MyClass(int _id, std::string _name, double _value)
    : id(_id), name(_name), value(_value) {
    std::cout << "MyClass object created\n";
}

// 成员函数实现
void MyClass::display() const {
    std::cout << "ID: " << id << ", Name: " << name 
              << ", Value: " << value << "\n";
}
```
成员变量初始化顺序，由变量在类中声明的顺序决定。


# 4. **析构函数**
- 对于局部对象，离开作用域时，会自动调用析构函数。   
- 对于new出来的对象，需要手动调用delete。   
- 没有编写析构函数时，编译器会自动生成一个默认析构函数。   
- 析构函数不允许抛出异常。   
```cpp
class MyClass {
public:
    // 默认析构函数（由编译器隐式生成）
    ~MyClass() = default;

    // 显式定义析构函数
    ~MyClass() {
        // 清理资源（如释放内存、关闭文件等）
    }

    // 虚析构函数（用于多态基类）
    virtual ~MyClass() = default;
};
```