---
title: "DeskFlow 异步网络接口（02） — 网络接口的工厂类"
description: 
date: 2024-10-19
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


- [1. **抽象工厂类**](#1-抽象工厂类)
- [2. **工厂类的实现**](#2-工厂类的实现)
- [3. **使用**](#3-使用)


# 1. **抽象工厂类**
ISocketFactory网络接口类将socket的创建过与使用解耦合，通过工厂方法，允许客户端代码通过统一的接口创建不同类型的套接字，而无需直接依赖具体的实现类，但这是一个抽象类型，实际由TCPSocketFactory实现，只提供了TCP的。  
```cpp
class ISocketFactory : public IInterface
{
public:

  /* 
   *  family : 协议族，IPv4或者IPv6
   *  securityLevel : 安全级别，明文还是加密
   */
  virtual IDataSocket *create(
      IArchNetwork::EAddressFamily family = IArchNetwork::kINET, SecurityLevel securityLevel = SecurityLevel::PlainText
  ) const = 0;

  virtual IListenSocket *createListen(
      IArchNetwork::EAddressFamily family = IArchNetwork::kINET, SecurityLevel securityLevel = SecurityLevel::PlainText
  ) const = 0;

};
```
该工厂提供两种网络接口的创建：   
- IDataSocket : 一般数据socket。      
- IListenSocket  : 监听套接字。    

# 2. **工厂类的实现**
TCPSocketFactory 是 ISocketFactory 的具体实现类，负责创建 TCP 协议的套接字（IDataSocket 和 IListenSocket）。它通过多路复用器（SocketMultiplexer）实现完整的异步网络IO，通过事件队列（IEventQueue）也业务层交互。   
```cpp
class TCPSocketFactory : public ISocketFactory
{
public:
    TCPSocketFactory(IEventQueue *events, SocketMultiplexer *socketMultiplexer);
    ~TCPSocketFactory() override = default;

    /* 重写抽象类的虚函数 */
    IDataSocket *create(
        IArchNetwork::EAddressFamily family = IArchNetwork::kINET, SecurityLevel securityLevel = SecurityLevel::PlainText
    ) const override;


    IListenSocket *createListen(
        IArchNetwork::EAddressFamily family = IArchNetwork::kINET, SecurityLevel securityLevel = SecurityLevel::PlainText
    ) const override;

private:
    IEventQueue *m_events;                     // 指向事件队列的指针，用于异步通知连接状态、数据就绪等事件。
    SocketMultiplexer *m_socketMultiplexer;    // 套接字多路复用器，用于高效管理多个套接字的并发操作（如监听连接请求、读取数据）。
};
```
我们稍微看一下实现：  
```cpp
// 创建一个普通数据套接字
IDataSocket *TCPSocketFactory::create(IArchNetwork::EAddressFamily family, SecurityLevel securityLevel) const
{
  if (securityLevel != SecurityLevel::PlainText) {
    // 加密套接字
    auto *secureSocket = new SecureSocket(m_events, m_socketMultiplexer, family, securityLevel);
    secureSocket->initSsl(false);
    return secureSocket;
  } else {
    // 普通套接字
    return new TCPSocket(m_events, m_socketMultiplexer, family);
  }
}

// 监听套接字
IListenSocket *TCPSocketFactory::createListen(IArchNetwork::EAddressFamily family, SecurityLevel securityLevel) const
{
  IListenSocket *socket = nullptr;
  if (securityLevel != SecurityLevel::PlainText) {
    socket = new SecureListenSocket(m_events, m_socketMultiplexer, family, securityLevel);
  } else {
    socket = new TCPListenSocket(m_events, m_socketMultiplexer, family);
  }

  return socket;
}
```

# 3. **使用**
以TCP服务端为例：&emsp;程序启动后首先创建一个ServerApp对象，ServerApp内部维护一个监听器，而监听器在启动时会通过工厂方法创建一个监听套接字用于监听客户端连接。对于ClientApp对象则是创建普通的数据要解字，并使用该套接字向服务端发起连接。     
- **业务层类型**
```cpp
class ServerApp : public App
{
public:  
    // ...
    // 省略一堆成员变量以及成员函数

    bool startServer();  // 启动服务端业务程序： 创建监听器，并等待客户端链接

private:
    // ...
    ISocketFactory *getSocketFactory() const;  // 创建一个网络接口工厂实例
    // ...

    // ...
    ClientListener *m_listener = nullptr;      // 监听器，内部包含一个工厂实例
};
```

- **网络监听器类型**  
```cpp
class ClientListener
{
    // ...
    // 省略一堆成员
    void start();  // 通过工厂方法创建一个监听套接字
  
private:
    // ...
    // ...
    IListenSocket  *m_listen;         // 监听Socket
    ISocketFactory *m_socketFactory;  // 工厂实例
};
```

- **实现**    
```cpp
/* 创建工厂实例 */
ISocketFactory *ServerApp::getSocketFactory() const
{
  return new TCPSocketFactory(m_events, getSocketMultiplexer());
}


/* 创建一个监听器 */
ClientListener *ServerApp::openClientListener(const NetworkAddress &address)
{
  auto securityLevel = args().m_enableCrypto ? args().m_chkPeerCert ? SecurityLevel::PeerAuth : SecurityLevel::Encrypted
                                             : SecurityLevel::PlainText;

  // 这个监听器中包含一个网络接口工厂实例
  auto *listen = new ClientListener(getAddress(address), getSocketFactory(), m_events, securityLevel);

  m_events->adoptHandler(
      EventTypes::ClientListenerAccepted, listen,
      new TMethodEventJob<ServerApp>(this, &ServerApp::handleClientConnected, listen)
  );

  return listen;  
}
```
