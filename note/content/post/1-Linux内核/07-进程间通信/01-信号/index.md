---
title: "Linux进程间通信（01） — 信号"
description: 
date: 2023-09-10
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - Linux内核
---

- [1. **注册信号处理器**](#1-注册信号处理器)
    - [1.1. **signal**](#11-signal)
    - [1.2. **sigaction**](#12-sigaction)
- [2. **发送信号**](#2-发送信号)
- [3. **使用**](#3-使用)


Linux信号机制是一种`异步通知机制`，用于通知进程某个事件的发生。常被看做是一种软中断，类似于硬中断，但作用对象是进程而非 CPU，这是Glib提供的功能，所以源码需要到Glib代码库中查看。  
```c
#define SIGHUP		 1
#define SIGINT		 2
#define SIGQUIT		 3
#define SIGILL		 4
#define SIGTRAP		 5
#define SIGABRT		 6
#define SIGIOT		 6
// ......
```

常见的有SIGINT、SIGKILL、SIGUSR1、SIGUSR2、SIGTERM等等，使用`kill -l`命令可以查看系统中的所有信号。   
|||
|------------|------------|
|SIGINT|                  中断信号，用户按下 Ctrl+C 触发，用于终止前台进程       |  
|SIGKILL|                 强制终止进程，无法被忽略或捕获                       |   
|SIGTERM|                 优雅终止信号，默认由 kill 命令发送，允许进程清理资源    |
|SIGUSR1 <br> SIGUSR2|    用户自定义信号 2，可由程序自定义用途                  |


# 1. **注册信号处理器**
## 1.1. **signal**
指定进程收到某个信号后要去做的事情。  
```c
// 若设置失败，返回 SIG_ERR，并设置 errno 表示错误原因。
extern __sighandler_t signal (int __sig, __sighandler_t __handler) __THROW;
```
其中 __sighandler_t： 
```c
typedef void (*__sighandler_t) (int);
```
所以__sighandler_t是一个函数指针类型。

## 1.2. **sigaction**
```c
/*
 *  __act:  内部包含了信号处理回调函数。
 *  __oact: 对于原来的信号处理回调函数的备份，一般为NULL，不备份。  
 */
extern int sigaction (int __sig, 
                      const struct sigaction *__restrict __act, 
                      struct sigaction *__restrict __oact
) __THROW;
```
struct sigaction的原型：   
```c
struct sigaction
  {
#if defined __USE_POSIX199309 || defined __USE_XOPEN_EXTENDED
    union
    {
        __sighandler_t sa_handler;                        // 普通信号处理函数。  
        void (*sa_sigaction) (int, siginfo_t *, void *);  // 带有额外数据的信号处理函数，需要将sa_flags成员设置为SA_SIGINFO。  
    } __sigaction_handler;


# define sa_handler	    __sigaction_handler.sa_handler
# define sa_sigaction	__sigaction_handler.sa_sigaction

# else
    __sighandler_t sa_handler;
#endif

    __sigset_t sa_mask;         // 信号处理期间阻塞的信号集，当前处理的信号会自动被阻塞（防止递归触发）。
    int sa_flags;               // 标志位，控制信号行为
    void (*sa_restorer) (void); // 已弃用，保留历史兼容性
  };
```

对于sa_mask：
```c
// sa是struct sigaction类型变量
sigemptyset(&sa.sa_mask);       // 清空信号集
sigaddset(&sa.sa_mask, SIGINT); // 添加 SIGINT 到阻塞列表
```
对于`siginfo_t`:  
siginfo_t 是 POSIX 信号处理中用于传递 信号详细信息 的结构体，通常与 SA_SIGINFO 标志一起使用。它提供了比传统 signal() 函数更丰富的上下文信息。   
```c
typedef struct
{
    int si_signo;        // 信号编号。
    int si_errno;        // 错误码，如果非零，表示与信号相关的系统错误码。
    int si_code;         // 指示信号产生的具体原因，分为系统生成（SI_KERNEL）和用户生成（SI_USER），需要的时候在进一步查询。 
    __pid_t si_pid;		 // 发送信号的进程 ID。  
    __uid_t si_uid;	     // 发送信号的用户 ID。  
    void *si_addr;       // 指向引发信号的内存地址（如段错误时的无效地址）。
    int si_status;       // 传递子进程的退出状态或信号值。  
    long int si_band;    // 与 SIGPOLL 或 SIGIO 信号一起使用，表示文件描述符的带事件（如 I/O 就绪）。
    __sigval_t si_value; // 通过 sigqueue() 发送的自定义信号值（sigval）。
} siginfo_t;
```

# 2. **发送信号**
发送信号有两个函数： kill、sigqueue，其中sigqueue可以携带额外的数据。
```c
/*
 *  __pid: 目标进程ID。
 *  __sig: 要发送的信号。
 */
extern int kill (__pid_t __pid, int __sig);

/*
 *  __val: 携带的额外数据。  
 */
extern int sigqueue (__pid_t __pid, int __sig, const union sigval __val); 
```
union sigval的结构如下：  
```c
union sigval
{
  int sival_int;
  void *sival_ptr;
};
```

# 3. **使用**
以带数据的信号为例：  
1. 信号接收方：  
```c
// 信号处理函数
void handler(int sig, siginfo_t *info, void *old)
{
    // 检查信号，sig。  
    // 从info中读取信号上下文。  
}

struct sigaction act;
act.sa_sigaction = handler;
act.sa_flags = SA_SIGINFO;
sigaction(SIGUSR1, &act, NULL);
```

2. 信号发送方：  
```c
union signal value;
value.sigval_int = 99;
sigqueue(getpid(), SIGUSR1, value);
```

