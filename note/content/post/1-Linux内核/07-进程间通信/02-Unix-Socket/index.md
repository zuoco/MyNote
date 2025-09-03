---
title: "Linux进程间通信（02） — Unix-Socket"
description: 
date: 2024-08-10
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - Linux内核
---


Unix Socket（Unix域套接字）是一种用于 本地进程间通信的机制。它与网络 Socket 不同，不需要通过网络协议栈，而是通过文件系统实现进程间的通信。和网络Socket一样，也提供`流式套接字`和`数据报套接字`。API接口的网络套接字是一样的，

**1. 服务器**
```c
#include <sys/socket.h>
#include <sys/un.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

#define SOCKET_PATH "/tmp/mysocket"  // socket绑定到文件系统

int main() {
    int server_fd, client_fd;
    struct sockaddr_un addr;  // Unix-Socket地址
    socklen_t len;

    // 1. 创建 Socket
    server_fd = socket(AF_UNIX, SOCK_STREAM, 0); // AF_UNIX 或 PF_UNIX，表示使用本地文件系统作为通信地址
    if (server_fd == -1) {
        perror("socket");
        return 1;
    }

    // 2. 绑定地址
    addr.sun_family = AF_UNIX;
    strncpy(addr.sun_path, SOCKET_PATH, sizeof(addr.sun_path) - 1);
    unlink(SOCKET_PATH); // 如果文件已存在，删除它
    if (bind(server_fd, (struct sockaddr *)&addr, sizeof(addr)) == -1) {
        perror("bind");
        close(server_fd);
        return 1;
    }

    // 后面和网络套接字一样了。

    unlink(SOCKET_PATH); // 删除 Socket 文件
    return 0;
}
```

**2. 客户端**
```c
#include <sys/socket.h>
#include <sys/un.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

#define SOCKET_PATH "/tmp/mysocket"

int main() {
    int client_fd;
    struct sockaddr_un addr;

    // 1. 创建 Socket
    client_fd = socket(AF_UNIX, SOCK_STREAM, 0);
    if (client_fd == -1) {
        perror("socket");
        return 1;
    }

    // 2. 连接服务器
    addr.sun_family = AF_UNIX;
    strncpy(addr.sun_path, SOCKET_PATH, sizeof(addr.sun_path) - 1);
    if (connect(client_fd, (struct sockaddr *)&addr, sizeof(addr)) == -1) {
        perror("connect");
        close(client_fd);
        return 1;
    }

    // 3. 读写数据
    // ......


    // 4. 关闭 Socket
    close(client_fd);
    return 0;
}
```