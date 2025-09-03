---
title: "内存对齐"
description: 
date: 2024-08-21
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
---


- [1. alignas 与 alignof](#1-alignas-与-alignof)
- [2. #pragma pack](#2-pragma-pack)
- [3. 获取对齐的内存](#3-获取对齐的内存)
  - [3.1. std::aligned\_storage\_t](#31-stdaligned_storage_t)
  - [3.2. std::aligned\_alloc](#32-stdaligned_alloc)




## 1. alignas 与 alignof
alignof 用于在编译期查询类型或对象的对齐要求，返回一个 std::size_t 类型的值，也就是该类型在内存中对齐的字节数。   
alignas 用于控制变量或类型的内存对齐方式，例如：  
```c++
alignas(16) char buffer[256]; // buffer的起始地址是16的倍数   
```
如果alignas修饰一个类型，它指定的是这个类型整体的对齐，不干扰类中的成员变量对齐，类型或者结构体的成员变量默认按照自身对齐，例如：   
```c++
struct alignas(8) MyStruct {
    char a;      
    int b;       
    double c;   
};    
```
如果创建一个MyStruct对象，那么对象起始地址是8的倍数。但是其中的成员变量不一定是8的倍数，实际上上面的结构体占用的空间是16字节。   
`char a`（1字节）：对齐要求为1字节，无需填充。   
`int b`（4字节）：需要4字节对齐。由于a后紧跟b，a的地址是0，b的地址是1，但1不是4的倍数。因此，编译器会在a后插入3字节填充，使b的地址对齐到4字节（地址为4）。   
`double c`（8字节）：需要8字节对齐。b的地址是4，b之后是地址8。但c的地址需要是8的倍数，因此b后的地址8已经满足要求，无需额外填充。    


alignas 与 alignof通常一起使用：  `alignas( alignof(T) ) T num[256];`。而且 alignas 和 alignof 都是编译期运行的特性，它们的处理完全由编译器在编译阶段完成。
另外，声明和定义时必须使用相同的对齐值：  
```c++
struct alignas(8) MyStruct {
    char a;      
    int b;       
    double c;   
};

struct alignas(16) MyStruct my_struct;  // 错误  
```


## 2. #pragma pack
#pragma pack 是一个编译器指令，
```c++ 
#pragma pack(n)          // 设置对齐边界为 n 字节
#pragma pack(push, n)    // 保存当前对齐状态并设置为 n 字节
#pragma pack(pop)        // 恢复最近一次保存的对齐状态
```
`n`：    对齐边界，可以是 1, 2, 4, 8, 16 等。编译器会根据 n 和成员变量的大小，选择较小的值进行对齐。
`push`： 将当前对齐状态压入编译器的堆栈，以便后续恢复。
`pop`：  从堆栈中弹出最近保存的对齐状态，并恢复为该状态。

```c++
#pragma pack(1)  // 强制所有成员按 1 字节对齐
struct MyStruct {
    char a;   
    int b;    
    short c;  
};
#pragma pack()   // 恢复默认对齐
```
所有成员按 1 字节对齐，无需填充，总大小：1 + 4 + 2 = 7 bytes。  

使用 push 和 pop：   
```c++
#pragma pack(push, 1)  // 保存当前对齐状态并设置为 1 字节
struct PackedStruct {
    char a;
    int b;
};
#pragma pack(pop)      // 恢复之前的对齐状态
```
push 和 pop 可以避免影响其他代码的对齐设置，只作用于当前结构体。   



## 3. 获取对齐的内存

### 3.1. std::aligned_storage_t
提供一个未初始化的，满足特定对齐的要求内存块，常用于 placement new。   
```cpp
std::aligned_storage_t<sizeof(T), alignof(T)> storage[10]; 
````
storage中的每一个成员的地址都是 T 的整数倍。  


### 3.2. std::aligned_alloc
动态分配一个满足特定对齐要求的内存块。    
```cpp
void* std::aligned_alloc(std::size_t alignment, std::size_t size);
```
alignment: 内存对齐的边界（以字节为单位），必须是 2 的幂。      
size：     要分配的字节数，必须是 alignment 的整数倍，否则分配失败。   
成功时，返回指向对齐内存的指针。失败时，返回 nullptr。    




