---
title: "DeskFlow 跨平台抽象层（03） — 网络模块"
description: 
date: 2024-09-14
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "DeskFlow"
    - "设计模式"
---


- [1. **设计思路**](#1-设计思路)
- [2. **代码设计**](#2-代码设计)
  - [2.1. **纯虚接口设计**](#21-纯虚接口设计)
  - [2.2. **平台实现**](#22-平台实现)


# 1. **设计思路**  

&emsp;&emsp;`IArchNetwork` 采用纯虚接口设计，定义了所有网络操作的统一接口。该接口包含了完整的网络功能集合，包括socket创建、连接、数据传输、地址解析等操作。该设计采用了两个平台特定的实现：
- **ArchNetworkBSD**: &ensp;用于Unix/BSD系统的实现，ArchNetworkBSD.h。  
- **ArchNetworkWinsock**: &ensp;用于Windows系统的实现，ArchNetworkWinsock.h。  

**设计关键点：**   
- 每个实现都继承自 IArchNetwork 接口并重写所有虚函数，提供平台特定的网络功能。通过 Arch 单例类提供全局访问点。业务层代码通过 ARCH 宏来访问网络功能。   
- IArchNetwork 被高层组件如 `TCPSocketFactory` 使用，通过工厂模式创建不同类型的socket。例如，在创建TCP socket时，工厂会调用 `ARCH->newSocket()` 来创建底层的网络socket。  
- 设计中包含了完善的异常处理机制，所有网络操作都通过捕获 XArchNetwork 异常来处理错误情况。   

&emsp;&emsp;IArchNetwork 的设计体现了良好的软件架构原则：通过接口隔离了平台差异、使用工厂模式简化了对象创建、采用单例模式提供了全局访问点，并且通过异常机制确保了错误处理的一致性。这种设计使得 Deskflow 能够在不同平台上提供统一的网络功能，同时保持代码的可维护性和可扩展性。

---

# 2. **代码设计**
## 2.1. **纯虚接口设计**
```cpp
// 网络地址实体，就是不同平台上网络地址的实现
using ArchNetAddress = ArchNetAddressImpl *; 

// 
class IArchNetwork : public IInterface
{
public:

    // 地址族，需要在创建Socket时指定
    enum EAddressFamily
    {
        kUNKNOWN,
        kINET,     // IPV4
        kINET6,    // IPV6
    };

    // 协议类型
    enum ESocketType
    {
        kDGRAM,   // UDP
        kSTREAM   // TCP
    };


    // 异步I/O —— poll事件
    enum
    {
        kPOLLIN = 1,         // Socket 接收缓存区有数据可读。
        kPOLLOUT = 2,        // Socket 发送缓存区为空，可以发送数据。
        kPOLLERR = 4,        // 错误状态
        kPOLLNVAL = 8        // 无效socket
    };

    // 异步I/O ——　用于套接字事件轮询，监控多个套接字的状态变化。
    class PollEntry
    {
        public:
        ArchSocket m_socket;       // 要监听的套接字
        unsigned short m_events;   // 要监听的事件
        unsigned short m_revents;  // 实际监听到的事件
    };


    // 下面都是socket操作
    virtual ArchSocket newSocket(EAddressFamily, ESocketType) = 0; // 创建socket
    virtual ArchSocket copySocket(ArchSocket s) = 0;               
    virtual void closeSocket(ArchSocket s) = 0;                    // 关闭sokcet
    virtual void closeSocketForRead(ArchSocket s) = 0;             // 关闭读通道
    virtual void closeSocketForWrite(ArchSocket s) = 0;            // 关闭写通道

    // 连接网络
    virtual void bindSocket(ArchSocket s, ArchNetAddress addr) = 0;           // 绑定地址，服务端
    virtual void listenOnSocket(ArchSocket s) = 0;                            // 监听客户端连接，服务端
    virtual ArchSocket acceptSocket(ArchSocket s, ArchNetAddress *addr) = 0;  // 接收客户端连接，服务端
    virtual bool connectSocket(ArchSocket s, ArchNetAddress addr) = 0;        // 向服务端发起连接，客户端

    // 异步I/O
    virtual int pollSocket(PollEntry[], int num, double timeout) = 0;         // 使用poll轮训socket
    virtual void unblockPollSocket(ArchThread thread) = 0;                    // 解除轮询阻塞 

    virtual size_t readSocket(ArchSocket s, void *buf, size_t len) = 0;        // 读socket
    virtual size_t writeSocket(ArchSocket s, const void *buf, size_t len) = 0; // 写socket

    virtual void throwErrorOnSocket(ArchSocket s) = 0;
    virtual bool setNoDelayOnSocket(ArchSocket, bool noDelay) = 0;             // 启用/禁用Nagle算法
    virtual bool setReuseAddrOnSocket(ArchSocket, bool reuse) = 0;             // 地址重用选项

    virtual std::string getHostName() = 0;
    virtual ArchNetAddress newAnyAddr(EAddressFamily) = 0;                     // 创建通配地址
    virtual ArchNetAddress copyAddr(ArchNetAddress) = 0;                  
    virtual std::vector<ArchNetAddress> nameToAddr(const std::string &) = 0;   // 域名解析
    virtual void closeAddr(ArchNetAddress) = 0;                                
    virtual std::string addrToName(ArchNetAddress) = 0;
    virtual std::string addrToString(ArchNetAddress) = 0;
    virtual EAddressFamily getAddrFamily(ArchNetAddress) = 0;
    virtual void setAddrPort(ArchNetAddress, int port) = 0;            // 设置端口
    virtual int getAddrPort(ArchNetAddress) = 0;
    virtual bool isEqualAddr(ArchNetAddress, ArchNetAddress) = 0;      // 地址比较
    virtual bool isAnyAddr(ArchNetAddress addr) = 0;

    virtual void init() = 0;
};
```
---

## 2.2. **平台实现**
提供了Unix和Windows平台上的实现，就是将平台的系统调用封装了一下。  

**Windows平台**   
```cpp
#define ARCH_NETWORK ArchNetworkWinsock

/* 1. 网络地址实体 */
class ArchNetAddressImpl
{
public:
  static ArchNetAddressImpl *alloc(size_t);

public:
  int m_len;
  struct sockaddr_storage m_addr;   // 通用地址表示，IPv4、IPv6
};


/* 2. 网络设备句柄实现 */
class ArchSocketImpl
{
public:
  SOCKET m_socket;
  int m_refCount;
  WSAEVENT m_event;
  bool m_pollWrite;
};


/* 3. 网络接口实现 */
class ArchNetworkWinsock : public IArchNetwork
{
    // override 父类的纯虚函数
};
```

**Unix平台**
```cpp
#define ARCH_NETWORK ArchNetworkBSD

/* 1. 网络地址实现 */
class ArchNetAddressImpl      // 网络句柄
{
public:
  ArchNetAddressImpl() : m_len(sizeof(m_addr)) { /* do nothing */ }

public:
  struct sockaddr_storage m_addr;
  socklen_t m_len;
};

/* 2. 网络设备句柄实现 */
class ArchSocketImpl
{
public:
  int m_fd; 
  int m_refCount;
};


/* 3. 网络接口操作实现 */
class ArchNetworkBSD : public IArchNetwork
{
    // override 父类的纯虚函数
};
```