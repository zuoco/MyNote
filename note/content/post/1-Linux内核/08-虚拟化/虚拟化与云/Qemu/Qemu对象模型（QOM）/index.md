---
title: "Qemu对象模型（QOM）"
description: 
date: 2024-11-12
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - Qemu
---

# 1. Qemu类型系统
Qemu目前主要是C语言编写（有少量Rust），C语言没有语言层面的面向对象支持，所以Qemu开发团队就为Qemu设计了一套对象系统，我们本专题要讲的就是Qemu的对象系统。   
在Qemu代码中，我们经常看到如下代码：   
```c
type_init(<class_name>_register_types);
```
- <class_name>_register_types: Qemu中定义一个类型后通过type_init宏函数来注册该类型。   

下面我们就来看一下type_init宏函数的定义：  
```c
// include/qemu/module.h
#define type_init(function) module_init(function, MODULE_INIT_QOM);  
```
- function: 类型注册函数;   
- MODULE_INIT_QOM: Qemu对象模型模块;  

module_init也是一个宏函数，它的定义如下:  
```c
#define module_init(function, type)                                         \
static void __attribute__((constructor)) do_qemu_init_ ## function(void)    \
{                                                                           \
    register_module_init(function, type);                                   \
}
```
首先我们分析一下这个宏函数:  
1. 使用了`##`，用于拼接，例如function是“cpu_register_types”，结果就是do_qemu_init_cpu_register_types，这样可以避免命名冲突，确保每个模块的注册函数独立存在。   
2. `__attribute__((constructor))`，GCC的一个函数属性，由该属性标记的函数，会在main函数前面执行。   
3. 实际上，这个宏函数就是将用户设计好的类型注册到一个Hash表中，也就是type(类型名称)和function(类型注册函数)。   


## 1.1. 类型注册
上面我们提到了`register_module_init`，这个函数定义如下:   
```c
// util/module.c  
void register_module_init(void (*fn)(void), module_init_type type)
{
    ModuleEntry *e;
    ModuleTypeList *l;

    e = g_malloc0(sizeof(*e));
    e->init = fn;
    e->type = type;

    l = find_type(type);

    QTAILQ_INSERT_TAIL(l, e, node);
}
```
这个函数在main函数前就会被执行，它包含了4个知识点，下面我们一一讲解:  
## 1.2. ModuleEntry类型
```c
typedef struct ModuleEntry
{
    void (*init)(void);     // 类型注册函数，就是type_init(<class_name>_register_types);中的<class_name>_register_types
    QTAILQ_ENTRY(ModuleEntry) node;
    module_init_type type;  // 所属模块类型，类型系统就是MODULE_INIT_QOM
} ModuleEntry;
```
关于