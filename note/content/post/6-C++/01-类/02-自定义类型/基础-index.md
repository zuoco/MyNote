---
title: "C++面向对象（02） —  自定义类型"
description: 
date: 2023-03-11
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
---



- [1. **对象的初始化**](#1-对象的初始化)
    - [1.1. **基本类型**](#11-基本类型)
    - [1.2. **自定义类**](#12-自定义类)
- [2. **自定义类型**](#2-自定义类型)
    - [2.1. **静态成员变量**](#21-静态成员变量)
    - [2.2. **this指针**](#22-this指针)
    - [2.3. **const成员函数**](#23-const成员函数)
    - [2.4. **静态成员函数**](#24-静态成员函数)
    - [2.5. **构造函数**](#25-构造函数)
    - [2.6. **析构函数**](#26-析构函数)
    - [2.7. **友元**](#27-友元)



# 1. **对象的初始化**
## 1.1. **基本类型**
对于基本类型，都可以使用零初始化。  
```cpp
int x {};   // 零初始化： 初始化为0

int y {10};  //
```

## 1.2. **自定义类**
创建对象就是调用类的构造函数。
```cpp
class MyClass {
public:
    MyClass() { /* 默认构造函数 */ }
    MyClass(int x, int y) : x_(x), y_(y) {}

private:
    int x_, y_;
};

int main() {
    MyClass obj;     // 默认构造函数
    MyClass p(3, 4); // 调用对应的构造函数
}
```

# 2. **自定义类型**   

## 2.1. **静态成员变量**   
所有实例共享，也遵循类的访问控制规则（public/private/protected），在单例模式中用的较多。
```cpp
// MyClass.h
class MyClass {
public:
    /* 
     * C++17 的 inline static 特性，允许在类内部定义并初始化静态成员变量
     * inline 允许在头文件中定义静态成员变量，并确保即使多个翻译单元包含该头文件，链接器也能正确处理，不会导致多重定义错误。
     */
    inline static int count = 0; 
};
```

----

```cpp
class MyClass {
public:
    /* 
     * C++20的constexpr特性, 编译时使用字面量初始化。  
     * 不可修改，默认内联。  
     */
    static constexpr int MaxValue = 100; 
};
```

## 2.2. **this指针**
在类的内部，成员函数可以使用`this`指针访问当前对象，实际上this就是成员函数的一个参数，由编译器隐式传入的，它的类型是`MyClass* const`，所以this本身是不能修改的。至于this指向的对象是否可写，取决于成员函数的声明是否为const。

## 2.3. **const成员函数**
参数列表后添加 const，用于修饰this指针。   
```cpp
class MyClass {
public:
    int getValue() const;    // 声明 const 成员函数，在函数实现时也必须重复 const 关键字。
    void setValue(int val);  // 普通成员函数
};

// 定义时必须保持 const 一致性
int MyClass::getValue() const {
    return value;  // 不能修改 value（除非 value 是 mutable）
}
```

需要注意两件事情：
- const 对象只能调用 const 成员函数，因为一般不允许const转换为非const：
```cpp
const MyClass obj;
obj.getValue();   // 合法（调用 const 函数）
obj.setValue(5);  // 错误：const 对象无法调用非 const 函数
```
- const 与非 const 版本可以构成重载，因为参数类型不同：  
```cpp
class class MyClass {
public:
    char get(int x, int y);        // 非 const 版本
    char get(int x, int y) const;  // const 版本
};

const MyClass cs;
cs.get(0, 0);  // 调用 const 版本

MyClass s;
s.get(0, 0);   // 调用非 const 版本 
```


## 2.4. **静态成员函数**
所有实例共享，也遵循类的访问控制规则（public/private/protected），但是这种成员函数没有this指针，无法访问非静态的成员。需要显式传递对象。  

## 2.5. **构造函数**
见《构造函数》篇章。  
## 2.6. **析构函数**
见《析构函数》篇章。  


## 2.7. **友元**
class中的成员默认是private的，但是有个别时候，需要访问private成员，此时就需要使用`友元`，但是友元会破坏封装，一般不用，等用到了再去研究。  
```cpp
class B;

class A {
    friend B;
    static int a;
    float b;
};

class B {
    void func() {
        // 作为A的友元，可以访问A的私有成员
        std::cout << A::a << std::endl;
    }
};
```

