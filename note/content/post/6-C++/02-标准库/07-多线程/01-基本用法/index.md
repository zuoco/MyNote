---
title: "std::thread"
description: 
date: 2024-02-03
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "C++"
    - "C++ STL"
---


- [1. **创建一个线程对象**](#1-创建一个线程对象)
- [2. **创建带参数thread对象**](#2-创建带参数thread对象)
- [3. **C++20新鲜玩意儿**](#3-c20新鲜玩意儿)
- [4. **线程管理**](#4-线程管理)
  - [4.1. **线程安全退出**](#41-线程安全退出)
  - [4.2. **异步任务**](#42-异步任务)
  - [4.3. **捕获线程内的异常**](#43-捕获线程内的异常)
- [5. **标准库容器的线程安全**](#5-标准库容器的线程安全)



&emsp;&emsp;C++11提供了一个std::thread，它通过构造函数启动一个新线程，任务可以是普通函数、Lambda 表达式、类的成员函数等。在Linux平台上是基于pthread的，在Win平台上则是基于Windows原生API（MSVC），是对平台上的线程接口进行面向对象的封装，所以对于熟悉pthread的，这个std::thread比较好理解。


## 1. **创建一个线程对象** 
使用普通函数：  
```cpp
#include <iostream>
#include <thread>

void hello() {
    std::cout << "std::thread" << std::endl;
}

int main() {
    std::thread t(hello);  // 创建线程
    t.join();              // 等待线程完成
    return 0;
}
```

---
使用lambda:  
```cpp
#include <iostream>
#include <thread>

int main() {
    std::thread t([]() { std::cout << "std::thread" << std::endl; });
    t.join();
    return 0;
}
```

---
使用类成员函数：  
```cpp
#include <iostream>
#include <thread>

class MyClass {
public:
    void print() {
        std::cout << "std::thread" << std::endl;
    }
};

int main() {
    MyClass obj;
    std::thread t(&MyClass::print, &obj);  // 传递成员函数指针和对象指针
    t.join();
    return 0;
}
```


## 2. **创建带参数thread对象**
参数使用引用方式时，要小心参数的生命周期，数据竞态。    

值拷贝（默认）：
```cpp
#include <iostream>
#include <thread>
#include <string>

void print_message(const std::string& message) {
    std::cout << message << std::endl;
}

int main() {
    std::string msg = "AAABBBCCC!";
    std::thread t(print_message, msg);  // 按值传递参数
    t.join();
    return 0;
}
```

---
引用传递，通过 std::ref/std::cref 包装参数，传递参数的引用:  
```cpp
#include <iostream>
#include <thread>
#include <functional>    // std::ref

void modify_value(int& value) {
    value += 10;
    std::cout << "----------: " << value << std::endl;
}

int main() {
    int data = 100;
    std::thread t(modify_value, std::ref(data)); // 按引用传递参数
    t.join();
    std::cout << "+++++++++++: " << data << std::endl;
    return 0;
}
```

---
使用移动语义，将临时对象的所有权转移给线程函数，避免拷贝开销。  
```cpp
#include <iostream>
#include <thread>
#include <string>
#include <utility>   // std::move

void process_string(std::string str) {
    std::cout << "Processing string: " << str << std::endl;
}

int main() {
    std::string data = "AAAAAA";
    std::thread t(process_string, std::move(data)); // 移动语义传递参数
    t.join();
    return 0;
}
```


## 3. **C++20新鲜玩意儿**
C++20引入了std::jthread，它在对象析构时自动调用join()。
```cpp
#include <iostream>
#include <thread>

void task() {
    std::cout << "Thread is running..." << std::endl;
}

int main() {
    std::jthread jt(task); // 自动 join，无需显式调用
    return 0;              // 程序正常退出
}
```

---

std::jthread 提供 request_stop() 和 std::stop_token，允许线程主动响应取消请求。
```cpp
#include <iostream>
#include <thread>
#include <chrono>

void cancellable_task(std::stop_token stoken) {
    while (!stoken.stop_requested()) {
        std::cout << "Working..." << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
    std::cout << "Thread stopped." << std::endl;
}

int main() {
    std::jthread jt(cancellable_task);
    std::this_thread::sleep_for(std::chrono::seconds(3));
    jt.request_stop(); // 请求线程停止
    return 0;
}
```



## 4. **线程管理**
### 4.1. **线程安全退出**
使用join() 与 detach()保证线程安全退出。  

- join()：阻塞主线程，等待子线程完成。   
```cpp
std::thread t(task);
t.join();  // 主线程等待 t 完成
```

- detach()：分离线程，让其独立运行。    
```cpp
std::thread t(task);
t.detach();  // t 在后台独立运行
```
分离后的线程就不能再join()了，所以要需确保线程在程序退出前完成。   
如果 std::thread 对象销毁时未调用 join() 或 detach()，会抛出 std::system_error 异常。     


### 4.2. **异步任务**   
std::async，在并发场景下，我们将耗时操作（如网络请求、文件读写、复杂计算）异步执行，避免阻塞主线程，提升程序响应性。但是使用异步任务时，要注意异步策略，如果使用默认策略，代码如下：  
```cpp
#include <future>
#include <iostream>

int main() {
    // 默认策略：由系统决定执行方式
    auto future = std::async([]() {
        std::cout << "异步任务运行在 ID: " << std::this_thread::get_id() << std::endl;
        return 42;
    });

    // 可能在 future.get() 时才真正执行任务，不符合并发预期。
    int result = future.get();                         // 阻塞等待任务完成
    std::cout << "结果: " << result << std::endl;
    return 0;
}
```
系统默认策略是： `std::launch::async` | `std::launch::deferred` 。系统可能以异步方式（std::launch::async）在新线程中执行任务，也可能以延迟方式（std::launch::deferred）在当前线程中执行任务，也就是说有可能不会创建线程，那就不是并发了，所以异步时要明确指定异步策略。   
```cpp
auto future = std::async(std::launch::async, task_function);
```

### 4.3. **捕获线程内的异常**
如果线程中抛出了异常，但是我们没有捕获它，C++ 会调用 std::terminate() 直接终止程序。所以需要在每个线程函数中使用 try-catch 将异常信息传出：   
```cpp
#include <iostream>
#include <thread>
#include <future>

void risky_function() {
    // 模拟可能抛出异常的函数
    if (rand() % 2 == 0) {
        throw std::runtime_error("Something went wrong!");
    }
    std::cout << "Task succeeded." << std::endl;
}

int main() {
    std::promise<void> prom;
    std::future<void> fut = prom.get_future();

    std::thread t([&] {
        try {
            risky_function(); // 可能抛出异常
            prom.set_value(); // 成功时设置值
        } catch (...) {
            prom.set_exception(std::current_exception()); // 捕获并传递异常
        }
    });

    t.join(); // 等待线程完成

    try {
        fut.get(); // 获取结果
    } catch (const std::exception& e) {
        std::cerr << "Exception in thread: " << e.what() << std::endl;
    }

    return 0;
}
```

搭配`std::promise`和`std::future`来处理子线程抛出的异常，两者成对存在的：
- std::promise 在子线程中：  
    - 任务成功执行，没有抛出异常，通过`set_value()`通知主线程: “我这里任务完成了”，该方法会标记 std::promise 的内部状态为“已就绪”（ready），并唤醒所有等待该 future 的线程。  
    - 任务抛出异常，通过`set_exception(std::current_exception())`捕获并传递异常，同样会唤醒所有等待该 future 的线程。  
- std::future 在主线程中：
    - 通过`fut.get();`获取子线程执行结果，如果有异常就抛出。  



## 5. **标准库容器的线程安全**  
stl标准库的容器本身不是线程安全的，多线程访问需要开发者自行加锁。   