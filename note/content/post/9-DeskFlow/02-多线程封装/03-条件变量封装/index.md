---
title: "DeskFlow 多线程框架（03） — 互斥量与条件变量"
description: 
date: 2024-10-05
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - "DeskFlow"
---

- [1. **互斥量实体**](#1-互斥量实体)
- [2. **条件变量实体**](#2-条件变量实体)
- [3. **封装互斥量**](#3-封装互斥量)
- [4. **封装条件变量**](#4-封装条件变量)


|类型|功能|    
|------------|------------|  
|ArchMutexImpl|互斥量实体，互斥量在不同平台上的表示|  
|ArchCondImpl|条件变量实体，条件变量在不同平台上的表示|  
|Mutex|封装互斥量操作|   
|CondVarBase|封装条件变量操作|   
|CondVar<T>|类模板，包含额外数据，继承自CondVarBase|   


# 1. **互斥量实体**
Linux平台：    
```cpp
class ArchMutexImpl
{
public:
    pthread_mutex_t m_mutex;
};
```
Windows平台：  
```cpp
class ArchMutexImpl
{
public:
    CRITICAL_SECTION m_mutex;
};
```

其他类型定义时使用类型别名： 
```cpp
using ArchMutex = ArchMutexImpl *;
```

# 2. **条件变量实体**  
Linux： 
```cpp
class ArchCondImpl
{
public:
    pthread_cond_t m_cond;
};

Windows: 
```cpp
class ArchCondImpl
{
public:
    enum
    {
        kSignal = 0, // 唤醒一个等待的线程
        kBroadcast   // 唤醒所有等待的线程
    };

    HANDLE m_events[2];
    mutable int m_waitCount;     // 记录当前等待的线程数量
    ArchMutex m_waitCountMutex;  // 保护 m_waitCount 的访问，确保多线程环境下读写 m_waitCount 的线程安全
};
```

其他类型定义时使用类型别名： 
```cpp
using ArchCond = ArchCondImpl *;
```


# 3. **封装互斥量**  
```cpp
class Mutex
{
public:
    Mutex();
    Mutex(const Mutex &); // 会创建一个新的互斥量并返回
    ~Mutex();

    Mutex &operator=(const Mutex &); // 没有赋值操作，只是直接返回*this

    // 互斥量操作
    void lock() const;
    void unlock() const;

private:
    friend class CondVarBase;  // 允许 CondVarBase 访问 Mutex 的私有成员
    ArchMutex m_mutex;
};
```

# 4. **封装条件变量**
条件变量基类型，基于ARCH层封装了条件变量的基础操作（锁、解锁、信号通知、等待），与互斥锁（Mutex）绑定，确保线程安全。
```cpp
class CondVarBase
{
public:
    explicit CondVarBase(Mutex *mutex);  // 初始化m_cond，绑定互斥量
    ~CondVarBase();                      // 析构条件变量

    /* 互斥量操作 */
    void lock() const;
    void unlock() const;

    /* 条件变量操作 */
    void signal();     // 唤醒一个等待线程    
    void broadcast();  // 广播唤醒所有等待m_cond的线程

    /*  */
    bool wait(double timeout = -1.0) const;            // 带超时机制的条件变量等待，内部调用wait()
    bool wait(Stopwatch &timer, double timeout) const; // 条件变量等待，内部设置了取消点

    Mutex *getMutex() const;

private:
    Mutex *m_mutex;       // 与条件变量关联的互斥锁
    ArchCond m_cond;      // 条件变量
};
```

**CondVar**   
携带额外数据的派生类型。
```cpp
template <class T> 
class CondVar : public CondVarBase
{
public:
  CondVar(Mutex *mutex, const T &value);
  CondVar(const CondVar &);
  ~CondVar();

  CondVar &operator=(const CondVar &cv);

  CondVar &operator=(const T &v);

  operator const volatile T &() const;

private:
  volatile T m_data;  // 额外数据
};
```

重载“**&**”运算符：  
```cpp
template <class T> 
inline CondVar<T>::operator const volatile T &() const
{
  return m_data;  // 将m_data转换为 const volatile T& 类型的引用。
}
```
重载“**&**”运算符用于支持隐式转换：   
```cpp
CondVar<int> cv;
int val = cv;  // 隐式调用 operator const volatile T &()
```
