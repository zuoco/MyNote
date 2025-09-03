---
title: "C++面向对象（05） — 类继承中的知识点"
description: 
date: 2023-03-25
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
---


- [3. **基类指针**](#3-基类指针)
- [4. **使用using提权/降权**](#4-使用using提权降权)
- [5. **继承中的构造与析构**](#5-继承中的构造与析构)
    - [5.1. **构造**](#51-构造)
    - [5.2. **析构**](#52-析构)
- [6. **虚继承**](#6-虚继承)


# 3. **基类指针**
```cpp
class UsbDevice {

};

class UsbHub : public UsbDevice {

}

int main() {

    UsbHub hub;
    UsbDevice& device1 = hub;   // 基类引用指向派生类对象
    UsbDevice* device2 = &hub;  // 基类指针指向派生类对象
}
```

# 4. **使用using提权/降权**
用于修饰类成员（从基类继承过来的成员），提升成员的权限，但是只能对基类的public和protected成员进行提权或降权操作，不能改变基类private成员的访问权限，因为它们在派生类中不可见。    
```cpp
class Base {
protected:
    void internal() {}
};

class Derived : public Base {
public:
    using Base::internal;  // 将internal成员权限提升为public，原本为protected
};

Derived d;
d.internal();
```

# 5. **继承中的构造与析构**
构造与析构不会被继承。  

## 5.1. **构造**
- 构造函数调用顺序：&emsp;基类 --> 派生类，与析构相反。   
- 如果有多个基类，基类的构造是从左到右调用。   
- 如果派生类没有指定调用基类的那个构造函数。就调用默认构造函数。   
- 派生类只能调用`直接基类`的构造函数，但是虚继承中不同。   
```cpp
class Derived : public Base {
public:
    // 如果不指定调用基类的那个构成函数，就会调用基类的默认构造函数
    Derived( <参数列表> ) : Base( <基类构造函数参数> ) {
        // 派生类构造函数体
    }
};
```

**一个基类**：   
```cpp
#include <iostream>
using namespace std;

class Base {
public:
    Base(int x) {
        cout << "Base constructor with x = " << x << endl;
    }
};

class Derived : public Base {
public:
    // 显式调用 Base(int)
    Derived(int x, int y) : Base(x) {  
        cout << "Derived constructor with y = " << y << endl;
    }
};

int main() {
    Derived d(10, 20);
    return 0;
}
```

**多个基类**：   
```cpp
#include <iostream>
using namespace std;

class A {
public:
    A(int x) { cout << "A constructor with x = " << x << endl; }

    int x;
};

class B {
public:
    B(double y) { cout << "B constructor with y = " << y << endl; }  

    int y;
};

class Derived : public A, public B {
public:
    // 按继承顺序调用 A 和 B
    Derived(int x, double y) : A(x), B(y) {  
        cout << "Derived constructor" << endl;
    }
};

int main() {
    Derived d(10, 3.14);
    return 0;
}
```

**使用初始化列表**：  
```cpp
#include <iostream>
using namespace std;

class A {
public:
    A(int x) { cout << "A constructor with x = " << x << endl; }
};

class B {
public:
    B(double y) { cout << "B constructor with y = " << y << endl; }
};

class Derived : public A, public B {
private:
    int z;
public:  
    // 按继承顺序调用基类构造函数，然后是初始化列表
    Derived(int x, double y, int z) : A(x), B(y), z(z) {            
        cout << "Derived constructor with z = " << z << endl;
    }
};

int main() {
    Derived d(10, 3.14, 42);
}
```

## 5.2. **析构**
- 调用顺序： 派生类 --> 基类， 与构造函数相反。     
- 无需显式调用基类析构函数， C++ 编译器会自动调用基类的析构函数。   
- 基类的析构函数必须声明为`虚函数`， 这样才能通过基类的指针或引用调用派生类的析构函数，以安全释放内存。   
- 将基类的析构函数声明为虚函数后，派生类的析构函数也会自动成为虚函数。    
```cpp
class Base {
public:
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    ~Derived() override { /* 释放派生类资源 */ }
};
```

# 6. **虚继承**
菱形继承如下：  
```bash
    A
   / \
  B   C
   \ /
    D
```
&emsp;&emsp;这种结构导致D通过B和C继承了两个独立的A实例，从而引发数据冗余和成员访问二义性。C++提供了`虚拟继承`机制，该机制下由派生类D来初始化虚基类A，也就说，由派生类D来调用A的构造函数，B和C调用A的构造函数是无效的，最终D中仅保留一份A的实例，B和C共享该实例。     
&emsp;&emsp;C++标准库中的iostream类就是一个虚继承的实际应用案例。iostream从istream和ostream直接继承而来，而istream和ostream又都继承自一个共同的名为base_ios的类，是典型的菱形继承。此时istream和ostream必须采用虚继承，否则将导致iostream类中保留两份base_ios类的成员。  
![](菱形继承.png)   
iostream是最终派生类，istream、ostream是直接基类，base_ios是间接基类（虚基类）。istream和ostream的公共成员声明在虚基类中。 
```cpp
#include <iostream> 

class A{
	public:
		A() { std::cout << "AAA"<< std::endl; }
};

class B : virtual public A{
	public:
		B() { std::cout << "BBB"<< std::endl; }
};

class C : virtual public A {
	public:
		C() { std::cout << "CCC" << std::endl; }
};

class D : public B, public C{
	public:
		D() { std::cout << "DDD"<<  std::endl; }
};

int main()
{
	D a;
}
```
上面代码输出：     
```bash
AAA
BBB
CCC
DDD
```
如果没有virtual修饰，A的构造函数会被调用两次：     
```bash
AAA   ##
BBB
AAA   ##
CCC
DDD
```
如果A没有成员（除了构造与析构），或者A有成员但是没有被访问，还是可以编译的，但是只要访问A的成员就会二义性。     


