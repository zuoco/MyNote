---
title: "DeskFlow 异步网络接口（03） —  多路复用器设计"
description: 
date: 2024-10-20
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

- [1. **多路复用器**](#1-多路复用器)
- [2. **多路复用器任务**](#2-多路复用器任务)


对于多路复用，DeskFlow设计了两个类型 &emsp;—&emsp; `SocketMultiplexer`、`ISocketMultiplexerJob`：   
- SocketMultiplexer是套接字多路复用器的核心实现，借助poll来管理多个网络套接字（ISocket）的异步I/O。  
- ISocketMultiplexerJob是一个抽象类型，是套接字I/O任务的抽象。  

多路复用器在其主线程的任务循环中检测套接字的状态，当状态发生变化时（可读、可写、出错），多路复用器就会调用套接字任务处理这些IO状态，而套接字的I/O被封装为“多路复用器事件”。  

# 1. **多路复用器设计**  

```cpp
class SocketMultiplexer
{
public:
    SocketMultiplexer();
    SocketMultiplexer(SocketMultiplexer const &) = delete;
    SocketMultiplexer(SocketMultiplexer &&) = delete;
    ~SocketMultiplexer();

    SocketMultiplexer &operator=(SocketMultiplexer const &) = delete;
    SocketMultiplexer &operator=(SocketMultiplexer &&) = delete;

    /* 将ISocket对象与对应的IO任务对象添加到列表中（线程安全），更新m_update=true */
    void addSocket(ISocket *, ISocketMultiplexerJob *);     
    void removeSocket(ISocket *);                           // 
    static SocketMultiplexer *getInstance();                // 多路复用器单例

private:
    /* 核心数据结构类型 */
    using SocketJobs = std::list<ISocketMultiplexerJob *>;  // 多路复用器任务列表，“任务”用于处理I/O，这些“任务”是与ISocket绑定的  
    using JobCursor = SocketJobs::iterator;                 // 任务列表迭代器
    using SocketJobMap = std::map<ISocket *, JobCursor>;    // ISocket和SocketJobs中任务的映射关系，方便通过IScoket对象查找对应的Job


    [[noreturn]] void serviceThread(void *);                // 轮询所有注册的 socket，检测事件并调用回调（ISocketMultiplexerJob）


    /*  
     *  任务列表的迭代器游标，用于防止多线程并发时的迭代器失效的问题。  
     *  不再使用使用begin和end迭代器遍历列表，而是使用newCursor()和nextCursor()返回的迭代器来遍历。  
     */
    JobCursor newCursor();                // 将游标（m_cursorMark）插入列表开头，作为遍历的起点。
    JobCursor nextCursor(JobCursor);      // 在列表中找到一个有效的job，并返回。
    void deleteCursor(JobCursor);         // 移除游标。

    /* 读写共享资源时，先调用lockJobListLock()，再调用lockJobList()，然后开始读写共享资源 */
    void lockJobListLock();    // 获取 m_jobListLockLocked
    void lockJobList();        // 获取 m_jobListLock，获取成功后会释放m_jobListLockLocked
    void unlockJobList();      // 释放 m_jobListLock

private:
    Mutex *m_mutex = nullptr;  // 保护共享资源m_socketJobs、m_socketJobMap等

    /* 多路复用器主线程，构造函数中启动 */
    Thread *m_thread = nullptr;

    /* 任务列表更新标志 */
    bool m_update = false;

    /* 通知服务线程有新 job 需要处理，由unlockJobList()更新 */
    CondVar<bool> *m_jobsReady = nullptr;          

    /* 
     * 两个锁结合使用，防止死锁： 
     * 使用时先锁lockJobListLock，成功后再锁m_jobListLock，然后开始读写共享资源。  
     */
    CondVar<bool> *m_jobListLock = nullptr;        // 这是m_socketJobs的锁
    CondVar<bool> *m_jobListLockLocked = nullptr;  // 这是m_jobListLock的锁

    /* 两个锁的拥有者，用于检查拥有锁的是不是当前线程 */
    Thread *m_jobListLocker = nullptr;
    Thread *m_jobListLockLocker = nullptr;

    SocketJobs m_socketJobs = {};                    // 任务列表
    SocketJobMap m_socketJobMap = {};                // 套接字与任务的对应关系
    ISocketMultiplexerJob *m_cursorMark = nullptr;   // 
};
```
主线程实现：  
```cpp
[[noreturn]] void SocketMultiplexer::serviceThread(void *)
{
    std::vector<IArchNetwork::PollEntry> pfds;
    IArchNetwork::PollEntry pfd;
    
    for (;;) 
    {
        Thread::testCancel();  // 检查是否需要取消线程

        {
          // 等待，直到任务列表中有任务
          Lock lock(m_mutex);
          while (!(bool)*m_jobsReady) {
            m_jobsReady->wait();
          }
        }

        // 有任务了，先上锁
        lockJobListLock();
        lockJobList();

        // ...
        // 这里就是poll的逻辑代码，监控一组ISocket，有事件就执行对应的任务。
        // ...

        // 解锁
        unlockJobList();
    }
}
```

# 2. **多路复用器任务**  
这里重点关注一下`m_method`成员，这是多路复用器任务，他的返回值类型还是多路复用器任务类型，ISocketMultiplexerJob::run()函数会去执行这个任务，并且返回值类型也是ISocketMultiplexerJob， 也就是说，执行一个任务后返回一个任务，这种方式允许在运行时链式任务处理，例如在连接建立过程中从连接处理任务切换到数据传输任务。   
```cpp
/* 
 * 抽象基类: 定义了套接字多路复用器任务的核心功能。 
 * 该接口采用了事件驱动的设计模式，当套接字状态发生变化时（可读、可写、出错），多路复用器会调用 run() 方法处理这些事件。
 */
class ISocketMultiplexerJob : public IInterface
{
public:
  virtual ISocketMultiplexerJob *run(bool readable, bool writable, bool error) = 0;

  virtual ArchSocket getSocket() const = 0;

  virtual bool isReadable() const = 0;

  virtual bool isWritable() const = 0;
};


/* 派生类型 */
template <class T> 
class TSocketMultiplexerMethodJob : public ISocketMultiplexerJob
{
public:
    using Method = ISocketMultiplexerJob *(T::*)(ISocketMultiplexerJob *, bool, bool, bool);

    TSocketMultiplexerMethodJob(T *object, Method method, ArchSocket socket, bool readable, bool writeable);
    TSocketMultiplexerMethodJob(TSocketMultiplexerMethodJob const &) = delete;
    TSocketMultiplexerMethodJob(TSocketMultiplexerMethodJob &&) = delete;
    ~TSocketMultiplexerMethodJob() override;

    TSocketMultiplexerMethodJob &operator=(TSocketMultiplexerMethodJob const &) = delete;
    TSocketMultiplexerMethodJob &operator=(TSocketMultiplexerMethodJob &&) = delete;

    ISocketMultiplexerJob *run(bool readable, bool writable, bool error) override;
    ArchSocket getSocket() const override;
    bool isReadable() const override;  // m_readable
    bool isWritable() const override;  // m_writable
private:
    T *m_object;           // 任务的来源，事件驱动框架中
    Method m_method;       // 多路复用器任务函数
    ArchSocket m_socket;   // 任务对应的Socket
    bool m_readable;
    bool m_writable;
    void *m_arg;           
};
```

虚函数实现：   
```cpp
/* 
 * 运行多路复用器任务，允许在运行时动态改变套接字的处理逻辑，例如在连接建立过程中从连接处理任务切换到数据传输任务。
 * 返回自身：     继续使用当前任务处理后续事件
 * 返回新任务：   替换当前任务，改变处理方式
 * 返回 nullptr：停止处理该套接字事件
*/
template <class T> 
inline ISocketMultiplexerJob *TSocketMultiplexerMethodJob<T>::run(bool read, bool write, bool error)
{
  if (m_object != nullptr) {
    return (m_object->*m_method)(this, read, write, error);
  }
  return nullptr;
}
```





