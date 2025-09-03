---
title: "placement New"
description: 
date: 2024-09-10
image: 
math: 
license: 
hidden: false
comments: true
draft: true
categories:
    - "C++"
---



在已经分配的内存上构造对象，也就是说要先分配一块内存，然后使用 placement new 方法初始化内存。   

```cpp
    alignas(T) char buffer[sizeof(T)];   // 确保内存对齐
    T* obj = new (buffer) T(42);         // 调用T的构造函数，在 buffer 上构造对象
```

**使用场景**：  
内存池管理：  在内存池中预先分配一大块内存，按需使用 placement new 构造对象，避免频繁的 new/delete 调用。
STL容器：  STL中的部分容器提供了 placement new 的方法，如 vector、list、map 等。   

**注意事项**：  
placement new 仅构造对象，不分配内存，因此不能使用 delete 释放对象， 所以要手动调用析构函数释放对象。  





