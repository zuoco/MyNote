---
title: "shared_ptr、unique_ptr、weake_ptr"
description: "shared\_ptr, unique\_ptr, weake\_ptr"
date: 2023-06-03
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

- [1. shared\_ptr](#1-shared_ptr)
    - [1.1. 基本方法](#11-基本方法)
    - [1.2. 自定义删除器](#12-自定义删除器)
    - [1.3. make\_shared](#13-make_shared)
    - [1.4. 对象数组](#14-对象数组)
- [2. unique\_ptr](#2-unique_ptr)
    - [2.1. make\_unique](#21-make_unique)
    - [2.2. 基本方法](#22-基本方法)
    - [2.3. 自定义删除器](#23-自定义删除器)
- [3. weake\_ptr](#3-weake_ptr)
    - [3.1. 解决循环引用](#31-解决循环引用)
    - [3.2. 安全访问 std::shared\_ptr 管理的对象](#32-安全访问-stdshared_ptr-管理的对象)



# 1. shared_ptr
共享智能指针，内部维护一块内存，允许多个shared_ptr对象共享这块内存。   
```cpp
// Defined in header <memory>
template< class T > class shared_ptr;
```
shared_ptr对象内部包含两个指针： `指向资源的指针`、`指向控制块的指针`，而控制块包含： `引用计数`、`弱引用计数`（用于统计weak_ptr）、自定义删除器等等。  
shared_ptr就是通过`引用计数`来管理对象的生命周期，引用计数的操作是原子的，当引用计数为0时，对象会被自动销毁。  
使用 std::make_shared 时，资源与控制块会分配在同一块内存，减少内存碎片并提升缓存效率。    

```cpp
// int* x = new int(10);
std::shared_ptr<int> x(new int(10));    // 申请一块内存来初始化x，引用计数为 1
std::cout << x.use_count() << std::endl;      // 输出1
{
    std::shared_ptr<int> y = x;         // 这块内存的引用计数 +1
    std::cout << x.use_count() << std::endl;  // 输出2
}

// 离开y的作用域，y被销毁，引用计数-1
std::cout << x.use_count() << std::endl;      // 输出1
```

## 1.1. 基本方法   
1. 重载了`*`，所以可以用`*x`来获取x指向的元素。  
2. 提供了get()方法获取原始指针，但是要注意这个操作不会增加引用计数，所以不要用它来初始化另一个shared_ptr, 或者delete它，因为shared_ptr并不知道你干了什么，所以会导致多次delete。   
3. 提供了reset()方法，先将引用计数减1，然后判断是否为0，如果为0，就则释放管理的原始内存。  
4. `template< class Y > void reset( Y* ptr );`，该方法先将引用计数减1，然后判断是否为0，如果为0，就释放管理的原始内存，并将ptr赋值给内部的指针。如果不为0，那就什么也不做。   


## 1.2. 自定义删除器
```cpp
// 构造函数
template< class Y, class Deleter >
    shared_ptr( Y* ptr, Deleter d );
```
删除器是一个可调用对象（函数、Lambda 表达式、仿函数等），在最后一个 shared_ptr 被销毁时调用。   
```cpp
#include <iostream>
#include <memory>


// 自定义删除器仿函数
struct FileDeleter {
    void operator()(FILE* file) const {
        if (file) fclose(file);
        std::cout << "File closed via functor." << std::endl;
    }
};


int main() {
    // 创建 shared_ptr，绑定文件句柄和仿函数
    std::shared_ptr<FILE> file(fopen("example.txt", "r"), FileDeleter());
    if (file) {
        std::cout << "File opened successfully." << std::endl;
    }
    return 0;
}
```

## 1.3. make_shared
```cpp
std::shared_ptr<int> x(new int(10));  // 引用计数和int 10的内存分别位于两个地方。
auto y = std::make_shared<int>(10);   // 引用计数和int 10的存储空间会尽挨得近一些，使得访问更快。  
```


## 1.4. 对象数组
```cpp
// C++ 17
std::shared_ptr<int[]> x(new int[10]); 
// C++20
auto y = std::make_shared<int[]>(10); 
```


# 2. unique_ptr
独享智能指针，内部维护一块内存，同一时刻只允许一个unique_ptr拥有这块内存，unique_ptr不支持拷贝和赋值，只能移动。   
```cpp
// Defined in header <memory>   
template<class T, class Deleter = std::default_delete<T>> 
class unique_ptr;
　　　　　　　
template <class T, class Deleter> 
class unique_ptr<T[], Deleter>;
```

代码示例：   
```cpp
std::unique_ptr<int> x(new int(10));
std::unique_ptr<int> y = x;              // ✘
std::unique_ptr<int> z = std::move(x);   // ✔
```
在函数中返回unique_ptr，会使用移动语义（如果返回类型支持移动语义）。  

## 2.1. make_unique
```cpp
auto x = std::make_unique<int>(10);
```


## 2.2. 基本方法
和shared_ptr一样，也有get、reset方法。


## 2.3. 自定义删除器
这一点和shared_ptr不一样了，unique_ptr的模板参数是两个。
```cpp
// 仿函数
struct FileDeleter {
    void operator()(FILE* fp) const {
        fclose(fp);
    }
};

int main() {
    auto file = std::unique_ptr<FILE, FileDeleter>(
        fopen("example.txt", "r"),
        FileDeleter{}
    );
    // 使用 file...
}
```

```cpp
// Lambda
int main() {
    auto file = std::unique_ptr<FILE, decltype([](FILE* fp) { fclose(fp); })>(
        fopen("example.txt", "r"),
        [](FILE* fp) { fclose(fp); }
    );
    // 使用 file...
}
```


```cpp
// 函数指针
void custom_deleter(FILE* fp) {
    fclose(fp);
}

int main() {
    auto file = std::unique_ptr<FILE, decltype(&custom_deleter)>(
        fopen("example.txt", "r"), 
        &custom_deleter
    );
    // 使用 file...
}
```

# 3. weake_ptr
weake_ptr提供一种轻量级引用，不占用引用计数。shared_ptr在使用中可能会发生循环引用，此时可以使用weak_ptr来打破 std::shared_ptr 的所有权闭环。   

## 3.1. 解决循环引用
当两个对象通过 std::shared_ptr 相互持有对方时，引用计数永远不会归零，导致内存泄漏。  
```cpp
class B;  // 前向声明
class A {
public:
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed\n"; }
};

class B {
public:
    std::shared_ptr<A> a_ptr;
    ~B() { std::cout << "B destroyed\n"; }
};

int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    a->b_ptr = b;
    b->a_ptr = a;  // 形成循环引用
    a = nullptr;   // 引用计数仍为 1，内存泄漏
    b = nullptr;   // 引用计数仍为 1，内存泄漏
    return 0;
}
```
**解决方案：**   
将其中一个 std::shared_ptr 替换为 std::weak_ptr，打破循环引用。
```cpp
class B;  // 前向声明
class A {
public:
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed\n"; }
};

class B {
public:
    std::weak_ptr<A> a_ptr;  // 使用 weak_ptr 避免循环引用
    ~B() { std::cout << "B destroyed\n"; }
};

int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    a->b_ptr = b;
    b->a_ptr = a;  // 不增加 a 的引用计数
    a = nullptr;   // a 的引用计数归零，对象被销毁
    b = nullptr;   // b 的引用计数归零，对象被销毁
    return 0;
}
```

## 3.2. 安全访问 std::shared_ptr 管理的对象
不要直接解引用 std::weak_ptr，必须通过 lock() 获取 std::shared_ptr，检查是否为空，不为空才能访问对象。   
```cpp
std::shared_ptr<int> shared = std::make_shared<int>(42);
std::weak_ptr<int> weak = shared;

if (auto locked = weak.lock()) {
    std::cout << *locked << std::endl;  // 对象存在时访问
} else {
    std::cout << "Object has been destroyed\n";  // 对象已销毁时处理
}
```


