---
title: "DeskFlow 多线程框架（02） — 线程对象管理"
description: 
date: 2024-09-29
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

- [1. **总体架构设计**](#1-总体架构设计)
- [2. **核心设计原则**](#2-核心设计原则)
- [3. **任务执行模型**](#3-任务执行模型)
  - [3.1. **Job模式**](#31-job模式)
  - [3.2. **异常安全的任务执行**](#32-异常安全的任务执行)
- [4. **线程控制流设计**](#4-线程控制流设计)
  - [4.1. **异常驱动的控制流**](#41-异常驱动的控制流)
  - [4.2. **取消点机制**](#42-取消点机制)
- [5. **资源管理设计**](#5-资源管理设计)
- [6. **线程同步和通信**](#6-线程同步和通信)
- [7. **代码**](#7-代码)
  - [7.1. **Thread线程类设计**](#71-thread线程类设计)
  - [7.2. **不同平台上的线程实体**](#72-不同平台上的线程实体)



# 1. **总体架构设计**
Thread类将平台无关的接口与平台相关的实现分离。Thread类作为高层接口，本身不包含任何平台特定的代码，所有具体操作都通过ARCH宏委托给具体的平台实现（如POSIX或Windows）。这个类本身只是一个线程句柄，删除Thread对象并不会终止线程本身。这种设计允许多个Thread对象引用同一个线程，提高了灵活性。
```cpp
class Thread
{
    // ...
    // ...
private:
    ArchThread m_thread = nullptr;
};
```
Thread类的实际操作是通过ARCH宏委托给平台特定的实现，实现了跨平台兼容性：  
```cpp
Thread::Thread(IJob *job)
{
  m_thread = ARCH->newThread(&Thread::threadFunc, job);  // 委托给ARCH层（IArchMultithread）实现
  if (m_thread == nullptr) {
    delete job;
    throw XMTThreadUnavailable();
  }
}
``` 

# 2. **核心设计原则** 
**Handle语义设计：** 上面也提到了，Thread类被设计为一个**句柄（Handle）**而不是线程对象本身。 他们这种设计允许：
- 多个Thread对象引用同一个底层线程。
- 复制构造和赋值操作不会创建新线程。
- RAII风格的资源管理。

**禁止继承设计：** hread类的注释中明确标注不允继承（可以使用final关键字），这是一个精心设计的决定，确保Thread类作为一个轻量级句柄，不会被扩展而导致复杂性增加。



# 3. **任务执行模型**
## 3.1. **Job模式**
Thread采用Job模式来封装要执行的任务。通过IJob接口，Thread可以执行任意类型的任务，看前面展示的构造函数就知道了，Thread的构造函数接受一个IJob*指针，并获得其所有权，这种设计带来了一些好处：
- 解耦了线程管理和任务逻辑。  
- 支持灵活的任务封装（函数、成员函数等）。  
- 简化了线程的创建和使用。   

## 3.2. **异常安全的任务执行**
Thread类型通过threadFunc执行线程的“任务”，而这个threadFunc方法实现了完善的异常处理机制，能够正确处理各种异常情况并确保资源清理
```cpp
void *Thread::threadFunc(void *vjob)
{
  // get this thread's id for logging
  IArchMultithread::ThreadID id;
  {
    ArchThread thread = ARCH->newCurrentThread();
    id = ARCH->getIDOfThread(thread);
    ARCH->closeThread(thread);
  }

  auto *job = static_cast<IJob *>(vjob);

  // run job
  void *result = nullptr;
  try {
    // go
    LOG((CLOG_DEBUG1 "thread 0x%08x entry", id));
    job->run();
    LOG((CLOG_DEBUG1 "thread 0x%08x exit", id));
  } catch (XThreadCancel &) {
    // client called cancel()
    LOG((CLOG_DEBUG1 "caught cancel on thread 0x%08x", id));
    delete job;
    throw;
  } catch (XThreadExit &e) {
    // client called exit()
    result = e.m_result;
    LOG((CLOG_DEBUG1 "caught exit on thread 0x%08x, result %p", id, result));
  } catch (XBase &e) {
    LOG((CLOG_ERR "exception on thread 0x%08x: %s", id, e.what()));
    delete job;
    throw;
  } catch (std::exception &e) {
    LOG((CLOG_ERR "standard exception on thread 0x%08x: %s", id, e.what()));
    delete job;
    throw;
  } catch (...) {
    LOG((CLOG_ERR "non-exception throw on thread 0x%08x: <unknown>", id));
    delete job;
    throw;
  }

  delete job;
  return result;
}
```

# 4. **线程控制流设计**  

## 4.1. **异常驱动的控制流**  
Thread类使用异常来实现线程的控制流，而不是传统的返回值方式，有两种情况：  
- 线程退出：通过抛出XThreadExit异常。  
- 线程取消：通过XThreadCancel异常实现。  

这种设计确保了栈展开（stack unwinding）和自动对象的正确析构，是C++中实现线程安全退出的优雅方式。   


## 4.2. **取消点机制**
Thread实现了**取消点（Cancellation Point）**机制，允许线程在特定点检查并响应取消请求。Thread也是通过ARCH层实现多线程功能的，而ARCH代理了平台层实现，在平台层的实现类中会有m_cancelling和m_cancell两个成员变量，testCancel()就是根据这两个成员变量值决定是否取消线程：  
```cpp
static void testCancel();  // 通过抛出XThreadCancel异常
```
这种设计确保线程能够在安全的时机响应取消请求，避免资源泄漏。   


# 5. **资源管理设计（RAII）**
Thread类严格遵循RAII原则：
- 构造函数获取资源（创建底层线程）。  
- 析构函数释放资源。   
- 拷贝构造和赋值操作正确管理`引用计数`，实现了类似智能指针的引用计数管理。   


- **析构函数**，释放当前线程句柄资源，对应的线程实体引用计数-1，如果引用计数为0，就释放线程实体资源。  
```cpp
Thread::~Thread()
{
  ARCH->closeThread(m_thread); // 处理引用计数，销毁当前线程句柄资源（Thread类本身只是一个线程句柄）
}
```

- **拷贝构造**，当前引用了thread的线程实体，这个线程实体的因不用计数+1。
```cpp
Thread::Thread(const Thread &thread) : m_thread{ARCH->copyThread(thread.m_thread)}
{
  // do nothing
}
```

- **赋值操作**，和拷贝构造一样。  
```cpp
Thread &Thread::operator=(const Thread &thread)
{
  // copy given thread and release ours
  ArchThread copy = ARCH->copyThread(thread.m_thread); 
  ARCH->closeThread(m_thread);

  // cut over
  m_thread = copy;

  return *this;
}
```


# 6. **线程同步和通信**   
Thread类提供了基本的线程同步操作：   
- **等待线程终止**：&emsp;wait()方法等待线程退出，支持超时。   
```cpp
bool Thread::wait(double timeout) const
{
  return ARCH->wait(m_thread, timeout);
}
```

- **获取执行结果**：&emsp;getResult()方法获取线程退出值。   
```cpp
void *Thread::getResult() const
{
  if (wait())   // 阻塞等待线程退出
    return ARCH->getResultOfThread(m_thread);
  else
    return nullptr;
}
```


# 7. **代码**  

## 7.1. **Thread线程类设计**
```cpp
class Thread
{
public:
    explicit Thread(IJob *adoptedJob);       // 构造新线程，传入IJob任务对象
    ~Thread();

    Thread &operator=(const Thread &); 

    static void exit(void *);                // 安全退出线程
    void cancel();                           // 设置线程取消点

    void setPriority(int n);                 // 线程优先级控制，没有实现
    void unblockPollSocket();

    static Thread getCurrentThread();        // 获取当前线程的线程实体

    bool wait(double timeout = -1.0) const;  // 阻塞等待线程退出

    static void testCancel();                // 检查当前线程是否被通知取消

    void *getResult() const;                 // 阻塞获取线程执行结果

    IArchMultithread::ThreadID getID() const;
    bool operator==(const Thread &) const;
    bool operator!=(const Thread &) const;

private:
    static void *threadFunc(void *); // 任务函数，Arch层会在子线程中执行该函数，而该函数又负责执行“任务” — IJob

private:
    ArchThread m_thread = nullptr;   // 所管理的线程实体
};
```

## 7.2. **不同平台上的线程实体**
Windows平台：  
```cpp
/* 线程实体 */
class ArchThreadImpl
{
public:
    ArchThreadImpl();
    ~ArchThreadImpl();

public:
    int m_refCount;    // 引用计数管理，防止线程对象在使用过程中被意外销毁。
    HANDLE m_thread;   // Win平台的线程句柄，表示线程的内核对象
    DWORD m_id;        // 线程标识符
    IArchMultithread::ThreadFunc m_func;  // 线程任务
    void *m_userData;  // 用户参数

    HANDLE m_cancel;   // 线程取消事件，线程被取消时触发
    bool m_cancelling; // 线程是否正在取消
    HANDLE m_exit;     // exit事件，线程退出时触发

    void *m_result;          // 存储线程执行结果
    void *m_networkData;     // 异步任务中的网络数据
};
```

Linux平台：  
```cpp
/*  Unix平台线程实体，和Win的差不多 */
class ArchThreadImpl
{
public:
  ArchThreadImpl() = default;

public:
  int m_refCount = 1;
  IArchMultithread::ThreadID m_id = 0;
  pthread_t m_thread;                              // Posix标准下的线程句柄
  IArchMultithread::ThreadFunc m_func = nullptr;
  void *m_userData = nullptr;
  bool m_cancel = false;
  bool m_cancelling = false;
  bool m_exited = false;
  void *m_result = nullptr;
  void *m_networkData = nullptr;
};
```




