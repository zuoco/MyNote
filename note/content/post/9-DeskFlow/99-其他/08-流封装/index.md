---
title: "DeskFlow 字节流"
description: 
date: 2024-10-26
image: 
math: 
license: 
hidden: false
comments: true
draft: true
categories:
  - "DeskFlow"
---



IStream 定义了 Deskflow 项目中所有流操作的通用接口规范。该接口类位于deskflow命名空间下，有两个派生类：  
- StreamFilter（流过滤器）。  
- IDataSocket（网络数据流）。  


```cpp
class IStream : public IInterface
{
public:
  IStream() = default;

  virtual void close() = 0;   // 关闭流并丢弃缓冲数据

  virtual uint32_t read(void *buffer, uint32_t n) = 0;     // 从流中读取数据
  virtual void write(const void *buffer, uint32_t n) = 0;  // 向流中写入数据

  virtual void flush() = 0;   // 强制刷新缓冲区确保数据传输

  // 分别控制输入/输出端的关闭
  virtual void shutdownInput() = 0;
  virtual void shutdownOutput() = 0;

  virtual void *getEventTarget() const = 0;  // 返回关联的事件处理对象

  virtual bool isReady() const = 0;          // 检查流是否可读
  virtual uint32_t getSize() const = 0;      // 获取可读数据的预估大小
};
```




