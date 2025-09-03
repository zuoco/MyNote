---
title: "DeskFlow 多线程框架（01） — 异步任务的封装"
description: 
date: 2024-09-28
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - "设计模式"
  - "DeskFlow"
---


- [1. **核心接口设计**](#1-核心接口设计)
- [2. **具体实现类型**](#2-具体实现类型)
    - [2.1. **函数包装器 (FunctionJob)**](#21-函数包装器-functionjob)
    - [2.2. **方法包装器 (TMethodJob)**](#22-方法包装器-tmethodjob)
- [3. **其他任务类型**](#3-其他任务类型)


# 1. **核心接口设计**
IJob是一个纯虚接口，继承自IInterface基类，该接口只定义了一个核心方法，用于执行任务逻辑。这种简洁的设计使得任何需要执行特定功能的代码都可以通过实现这个接口来统一处理。  
```cpp
class IJob : public IInterface
{
public:
    virtual void run() = 0;  //  执行任务
};
```

|类型|功能|
|------------|------------|
|IJob|所有任务类型的基础接口|
|FunctionJob|继承自IJob，函数包装器，用于将普通的C函数包装成任务|
|TMethodJob|继承自IJob，方法包装器，是一个模板类，用于将类的成员函数包装成任务|  
|||


这些任务类型主要用于线程执行，Thread类接受IJob接口的实现作为构造参数。当Thread类使用IJob来在新线程中执行任务。线程会接管Job的所有权并在执行完成后删除它。这种设计模式提供了一个统一的任务执行框架，允许以统一的方式处理不同类型的可执行任务（普通函数和类成员函数），适用于多线程环境中的任务调度和执行。  

# 2. **具体实现类型**


## 2.1. **函数包装器 (FunctionJob)**
FunctionJob类允许将C风格的函数指针包装成Job对象。它存储函数指针和参数，在run()方法中调用该函数。实际执行时，如果函数指针不为空，就会调用该函数并传递参数。  
```cpp
/* 类定义 */
class FunctionJob : public IJob
{
public:
    FunctionJob(void (*func)(void *), void *arg = nullptr);
    ~FunctionJob() override = default;

    void run() override;

private:
    void (*m_func)(void *);  // 任务函数
    void *m_arg;             // 任务函数的参数列表
};

/* 实现 */
void FunctionJob::run()
{
    if (m_func != nullptr) {
        m_func(m_arg);
    }
}
```


## 2.2. **方法包装器 (TMethodJob)**
TMethodJob是一个模板类，用于包装类的成员方法。它存储对象指针、方法指针和参数，并在run()方法中调用对象的成员方法。  
```cpp
/* 类定义 */
template <class T> 
class TMethodJob : public IJob
{
public:
    TMethodJob(T *object, void (T::*method)(void *), void *arg = nullptr);
    ~TMethodJob() override = default;

    void run() override;  

private:
    T *m_object;                  // 任务的来源，事件驱动框架下
    void (T::*m_method)(void *);  // 任务函数，T的成员函数
    void *m_arg;                  // 任务函数的参数
};


/* 实现 */
template <class T> 
inline void TMethodJob<T>::run()
{
  if (m_object != nullptr) {
    (m_object->*m_method)(m_arg);
  }
}
```   

# 3. **其他任务类型**
还有ISocketMultiplexerJob，我们放到网络IO部分讲解。   




