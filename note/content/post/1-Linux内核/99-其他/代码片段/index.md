---
title: "代码片段"
description: 
date: 2025-07-19
image: 
math: 
license: 
hidden: false
comments: true
draft: true
---



```c
#define WRITE_ONCE(x, val)				\
({							\
	union { typeof(x) __val; char __c[1]; } __u = { .__val = (val) }; 			\
	__write_once_size(&(x), __u.__c, sizeof(x));	\
	__u.__val;					\
})
```
逐行解析：  
`union { typeof(x) __val; char __c[1]; } __u = { .__val = (val) };`   
- 作用：定义一个联合体 __u，用于将 val 转换为字节序列（__c），同时保留原始类型（__val）。
- 细节：
  - `typeof(x)`：获取变量 x 的类型，确保联合体 __val 与 x 类型一致。
  - `.__val = (val)`：初始化联合体的 __val 字段为 val。
  - `char __c[1]`：字节数组，用于按字节访问数据（实际大小应为 sizeof(x)，但此处为简化写法）。


`__write_once_size(&(x), __u.__c, sizeof(x));`  
- 作用：将 __u.__c 中的字节写入 x 的地址，抑制编译器优化。
- 细节：
  - `&(x)`：目标变量 x 的地址。
  - `__u.__c`：联合体中存储的字节数据。
  - `sizeof(x)`：目标变量 x 的大小。
  - `__write_once_size` 是一个内联函数或宏，负责按字节写入数据，通常通过内存屏障或编译器指令实现。  

`__u.__val;`
- 作用：返回写入的值 val，以支持赋值表达式（如 x = WRITE_ONCE(y, val);）。
