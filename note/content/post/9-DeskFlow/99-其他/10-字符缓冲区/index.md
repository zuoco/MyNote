---
title: "DeskFlow 字节缓冲区设计（零拷贝队列）"
description: 
date: 2024-10-27
image: 
math: 
license: 
hidden: false
comments: true
draft: false

---


这是一个先进先出的队列结构，队列中的元素是vector。  

```cpp
class StreamBuffer
{
public:
    StreamBuffer() = default;
    ~StreamBuffer() = default;

    // 无删除、无拷贝的高效数据读取地读取缓冲区数据，将数据合并到第一个块中并返回这个块的地址。  
    // peek方法实现了智能的块合并策略：当请求的数据跨越多个块时，会自动将多个块合并到第一个块中，确保返回连续的内存空间。
    const void *peek(uint32_t n);

    // 从m_chunks的头部丢弃指定数量的字节
    void pop(uint32_t n);

    // 向m_chunks中写入指定字节数
    void write(const void *data, uint32_t n);  

    uint32_t getSize() const;  // m_size

private:
    static const uint32_t kChunkSize;     // 用来限制单个Chunk的最大容量

    using Chunk = std::vector<uint8_t>; 
    using ChunkList = std::list<Chunk>;   // 队列中每一个Chunk最大只能是4096

    ChunkList m_chunks;
    uint32_t m_size = 0;                  // m_chunks包含的总字节数
    uint32_t m_headUsed = 0;              // m_chunks中第一个Chunk块中已弹出的字节数量
};
```

---

```cpp
const uint32_t StreamBuffer::kChunkSize = 4096;

```




