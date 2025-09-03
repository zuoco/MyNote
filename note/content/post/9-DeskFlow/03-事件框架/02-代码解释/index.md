---
title: "DeskFlow 事件框架（02） — 代码解释"
description: 
date: 2024-10-12
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "DeskFlow"
---


- [1. **事件的基本结构**](#1-事件的基本结构)
- [2. **事件队列的实现**](#2-事件队列的实现)
  - [2.1. **事件队列的上层实现**](#21-事件队列的上层实现)
  - [2.2. **事件队列的平台层实现**](#22-事件队列的平台层实现)


# 1. **事件的基本结构**
```cpp
class Event
{
public:
  using Flags = uint32_t;
  // 事件标志位，用于控制事件处理的行为模式
  struct EventFlags
  {
    inline static const Flags NoFlags = 0x00;            // 无任何特殊行为的标志位
    inline static const Flags DeliverImmediately = 0x01; // 立即投递：事件将被立即处理并释放,用于需要即时响应的事件（如输入设备状态更新）
    inline static const Flags DontFreeData = 0x02;       // 禁止释放数据：在事件销毁时不自动释放关联数据， 用于需要跨事件传递数据的场景（如剪贴板共享数据）
  };

  Event() = default;

  /*  
   *  type： 事件类型。
   *  target: 事件的接收者。 
   *  data: 事件的附加数据，必须是POD类型（不能包含构造函数/析构函数或依赖其他非POD类型的类型），若数据为复杂对象（如 std::string），需使用 setDataObject() 或其他构造函数。  
   *  flags: 事件标志。   
   */
  Event(EventTypes type, void *target = nullptr, void *data = nullptr, Flags flags = EventFlags::NoFlags); 
  Event(EventTypes type, void *target, EventData *dataObject);  // 在这里，EventData就是


  static void deleteData(const Event &);      // 释放m_data和m_dataObject
  void setDataObject(EventData *dataObject);  // m_dataObject = dataObject;

  EventTypes getType() const;        // m_type
  void *getTarget() const;           // m_target
  void *getData() const;             // m_data
  EventData *getDataObject() const;  // m_dataObject
  Flags getFlags() const;            // m_flags

private:
  EventTypes m_type = EventTypes::Unknown;  // 事件类型
  void *m_target = nullptr;                 // 事件接受者
  void *m_data = nullptr;                   // 附加数据，POD类型
  Flags m_flags = EventFlags::NoFlags;      // 事件处理策略
  EventData *m_dataObject = nullptr;        // 非POD类型附加数据
};
```

**76个事件类型**  
定义了 76 个事件类型，涵盖以下核心功能领域：    
- 连接管理（ClientConnected、ClientDisconnected、DataSocketConnected 等）   
- 输入设备控制（KeyStateKeyDown、PrimaryScreenMotionOnPrimary 等）       
- 剪贴板同步（ClipboardGrabbed、ClipboardChanged）         
- 安全与加密（ClientConnectionRefused、ClientConnectionFailed）           
- 系统状态（ScreenSuspend、ScreenResume、OsxScreenConfirmSleep）          
- 协议交互（ClientProxyReady、ClientProxyUnknownSuccess）        
- 用户界面反馈（ServerScreenSwitched、ServerLockCursorToScreen）           

```cpp
enum class EventTypes : uint32_t
{
    // ...
};
```

---

# 2. **事件队列的实现**  
## 2.1. **事件队列的上层实现**
```cpp
class IEventQueue : public IInterface
{
public:  
    // 事件处理逻辑
    using EventHandler = std::function<void(const Event &)>;  

    class TimerEvent
    {  
    public:
        EventQueueTimer *m_timer; 
        uint32_t m_count;       
    };  

    // 1. 事件循环核心逻辑，处理挂起的事件，并通过dispatchEvent()分发事件。
    virtual void loop() = 0;

    // 2. 事件订阅/移除，将handler添加到一个Hash表中。
    virtual void addHandler(EventTypes type, void *target, const EventHandler &handler) = 0;
    virtual void removeHandler(EventTypes type, void *target) = 0;
    virtual void removeHandlers(void *target) = 0;

    // 3. 发布事件： 添加事件到事件缓冲区
    virtual void addEvent(const Event &event) = 0;

    // 4. 事件分发，就是执行对应的handler
    virtual bool dispatchEvent(const Event &event) = 0;

    // 5. 动态切换底层事件缓冲区
    virtual void adoptBuffer(IEventQueueBuffer *) = 0;

    virtual bool getEvent(Event &event, double timeout = -1.0) = 0;
    virtual EventQueueTimer *newTimer(double duration, void *target) = 0;
    virtual EventQueueTimer *newOneShotTimer(double duration, void *target) = 0;
    virtual void deleteTimer(EventQueueTimer *) = 0;
    virtual void waitForReady() const = 0;
    virtual bool isEmpty() const = 0;
    virtual void *getSystemTarget() = 0;
};
```
----
```cpp
class EventQueue : public IEventQueue
{
public:
    EventQueue();
    EventQueue(EventQueue const &) = delete;
    EventQueue(EventQueue &&) = delete;
    ~EventQueue() override;
    EventQueue &operator=(EventQueue const &) = delete;
    EventQueue &operator=(EventQueue &&) = delete;

    // IEventQueue overrides
    void loop() override;   // 将事件队列设置为就绪状态，并唤醒等待线程， 把挂起的事件添加到事件缓冲区等待处理。 然后分发事件缓冲区中的事件。 
    void adoptBuffer(IEventQueueBuffer *) override;   // 采用一个新的事件队列缓冲区，替换当前正在使用的缓冲区。
    bool getEvent(Event &event, double timeout = -1.0) override;  
    bool dispatchEvent(const Event &event) override;  // 分发事件，就是运行对应的事件处理器。
    void addEvent(const Event &event) override;       // 将事件添加到事件缓冲区： 如果是立即事件就立即执行，如果缓冲区未就绪就添加到挂起队列中。
    EventQueueTimer *newTimer(double duration, void *target) override;        // 创建定时器，并将添加到定时器管理结构中。
    EventQueueTimer *newOneShotTimer(double duration, void *target) override; // 创建一次性定时器，并将添加到定时器管理结构中。
    void deleteTimer(EventQueueTimer *) override;     
    void addHandler(EventTypes type, void *target, const EventHandler &handler) override;  // 向处理器表中添加事件处理器。
    void removeHandler(EventTypes type, void *target) override;
    void removeHandlers(void *target) override;
    bool isEmpty() const override;       // 事件队列是否为空（事件缓冲区为空 & 定时器队列为空）。
    void *getSystemTarget() override;    // 返回 m_systemTarget。
    void waitForReady() const override;  // 等待事件队列就绪。

private:
    const EventHandler *getHandler(EventTypes type, void *target) const;
    uint32_t saveEvent(const Event &event);     // 将事件存储到m_events中。
    Event removeEvent(uint32_t eventID);        // 将事件从m_events中移除。
    bool hasTimerExpired(Event &event);         // 检测并处理到期的定时器事件，如果有到期的定时器就将信息填充到event。
    double getNextTimerTimeout() const;         // 查询定时器件超时时间。
    void addEventToBuffer(const Event &event);  // 将事件添加到事件缓冲区。
    bool processEvent(Event &event, double timeout, Stopwatch &timer);

private:
    class Timer  // EventQueueTimer 本身只作为类型标识符存在，实际的定时器功能完全由 EventQueue::Timer 类承担。
    {
    public:
        Timer(EventQueueTimer *, double timeout, double initialTime, void *target, bool oneShot);
        ~Timer() = default;

        void reset();                 // m_time = m_timeout; 重新开始计时
        Timer &operator-=(double);    // 定时器时间递减dt
        operator double() const;      // 查看当前时间，m_time
        bool isOneShot() const;       // 定时器是否为单次触发模式

        EventQueueTimer *getTimer() const;   // return m_timer;
        void *getTarget() const;             // return m_target;
        void fillEvent(TimerEvent &) const;  // 将当前定时器状态记录到记录到的定时器事件中，包括事件队列关联的定时器对象，以及触发次数。

    private:
        EventQueueTimer *m_timer;   // 事件队列关联的定时器对象
        double m_timeout;
        void *m_target;             // 事件的目标对象
        bool m_oneShot;             // 单次触发标志
        double m_time;              // 当前时间
    };

    using Timers = std::set<EventQueueTimer *>;
    using TimerQueue = PriorityQueue<Timer>;
    using EventTable = std::map<uint32_t, Event>;
    using EventIDList = std::vector<uint32_t>;
    using TypeHandlerTable = std::map<EventTypes, EventHandler>;
    using HandlerTable = std::map<void *, TypeHandlerTable>;

    int m_systemTarget = 0;       // 标识系统事件，将区分系统事件和用户事件。
    mutable std::mutex m_mutex;

    // 事件缓存区
    std::unique_ptr<IEventQueueBuffer> m_buffer;  // 事件队列缓冲区，负责事件队列不同平台上的实际操作

    // 事件的存储与索引
    EventTable m_events;
    EventIDList m_oldEventIDs;

    // 定时器
    Stopwatch m_time;          // 测试时间间隔，用于文件传输等等。
    Timers m_timers;           // 存储和管理当前活跃的计时器。
    TimerQueue m_timerQueue;   // 管理定时器（Timer）的优先队列，确保定时器事件能够按照正确的时间顺序被处理。
    TimerEvent m_timerEvent;   // 用来记录定时器事件。 

    // 管理事件处理器
    HandlerTable m_handlers;  

    Mutex *m_readyMutex = nullptr;             // m_readyCondVar的守护者。
    CondVar<bool> *m_readyCondVar = nullptr;   // 事件队列是否就绪。
    std::queue<Event> m_pending;               // 管理挂起的事件。
};
```

## 2.2. **事件队列的平台层实现**
```cpp
class IEventQueueBuffer : public IInterface
{
public:
    enum class Type : uint8_t
    {
        Unknown, 
        System,  // 系统事件
        User     // 用户事件
    };

    virtual void init() = 0;

    // 阻塞等待事件，最多等待 timeout 秒。
    virtual void waitForEvent(double timeout) = 0;

    // 从缓冲区取出事件，填充 event 和 dataID。
    virtual Type getEvent(Event &event, uint32_t &dataID) = 0;

    // 将用户事件添加到缓冲区，并通知等待线程。
    virtual bool addEvent(uint32_t dataID) = 0;                   

    virtual bool isEmpty() const = 0;
    virtual EventQueueTimer *newTimer(double duration, bool oneShot) const = 0;
    virtual void deleteTimer(EventQueueTimer *) const = 0;
};
```
针对Win、Linux(WayLand、X11)、Mac都有对应的派生类实现，以Linux(Wayland)平台为例，简单看一下代码：    
```cpp
class EiEventQueueBuffer : public IEventQueueBuffer
{
public:
    EiEventQueueBuffer(const EiScreen *screen, ei *ei, IEventQueue *events);
    ~EiEventQueueBuffer() override;

    // IEventQueueBuffer overrides
    void init() override { /* do nothing */ }
    void waitForEvent(double timeout_in_ms) override;
    Type getEvent(Event &event, uint32_t &dataID) override;
    bool addEvent(uint32_t dataID) override;
    bool isEmpty() const override;
    EventQueueTimer *newTimer(double duration, bool oneShot) const override;
    void deleteTimer(EventQueueTimer *) const override;

private:
    ei *m_ei;                                      //  EI事件，libei，WayLand环境下输入设备模拟。
    IEventQueue *m_events;                         //  指向关联的事件队列。
    std::queue<std::pair<bool, uint32_t>> m_queue; // 内部事件队列，存储事件对（第一个元素表示是否为系统事件，第二个元素为数据ID）。

    // 非阻塞管道，构造函数中创建。
    int m_pipeWrite; // 写端
    int m_pipeRead;  // 读端

    mutable std::mutex m_mutex;
};
```
这里需要关注的是m_pipeWrite和m_pipeRead两个成员变量，这两货分别是管道的写端和读端，但这个管道不是用来发送数据的，而是用来实现线程间同步机制的：   
- **创建管道用于线程通信**：在构造函数中，使用pipe2()系统调用创建一个非阻塞的管道，m_pipeRead作为读端，m_pipeWrite作为写端。   
- **事件等待监听**：在waitForEvent()方法中，使用poll()同时监听两个文件描述符：libei的事件文件描述符和管道的读端m_pipeRead。   
- **唤醒等待线程**：当有新事件通过addEvent()方法添加到队列时，会向管道的写端m_pipeWrite写入数据来唤醒可能正在waitForEvent()中阻塞等待的线程。  
- **清理管道数据**：当管道有数据可读时，会读取并丢弃这些数据，因为数据本身并不重要，重要的是唤醒动作。   

这是一个典型的生产者-消费者模式中的唤醒机制。管道的作用仅仅是作为信号传递工具。当有线程调用addEvent()添加事件时，除了将事件加入队列外，还会通过管道唤醒可能正在等待的事件处理线程，确保事件能够及时被处理。  


