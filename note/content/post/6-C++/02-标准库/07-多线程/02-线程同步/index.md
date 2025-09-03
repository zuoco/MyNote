---
title: "C++ 多线程中的同步方法"
description: 
date: 2024-02-07
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


- [1. 线程同步](#1-线程同步)
- [2. 互斥锁 (std::mutex)](#2-互斥锁-stdmutex)
- [3. 避免死锁](#3-避免死锁)
- [4. 使用消息队列](#4-使用消息队列)
- [5. 读写锁（std::shared\_mutex）](#5-读写锁stdshared_mutex)
- [6. 原子操作（atomic）](#6-原子操作atomic)



## 1. 线程同步
标准库中的同步方法都是 RAII 的。  
|||
|-----------|---------------|
|std::mutex|互斥锁，防止多个线程同时访问共享资源。 <br> 在C代码中往往会直接使用pthread线程库中的mutex，但是在C++中一般使用lock_guard或者unique_lock。|
|std::lock_guard|RAII 风格的锁管理器，离开作用域自动释放锁。<br> 相对于std::unique_lock更加轻量级，用于简单场景。<br> 构造时立即加锁，析构时自动解锁。 <br> 不支持手动解锁或重新加锁。<br> 不可复制，不可移动。|
|std::unique_lock|功能类似lock_guard，功能更多。<br> 支持延迟加锁（通过 std::defer_lock）。<br> 支持手动调用 lock() 和 unlock()。 <br> 支持尝试加锁（try_lock()、try_lock_for()、try_lock_until()）。<br> 可移动（std::move），不可复制。|
|std::atomic|原子操作模板类型，基于硬件级别的原子指令，用于简单数据类型，比如说基本数据类型的自增，自减。|


## 2. 互斥锁 (std::mutex) 
其中最基础的就是互斥锁了， 用来保证同一时刻只有一个线程访问临界区代码，通常配合 `std::lock_guard`、`std::unique_lock` 使用。std::lock_guard的功能非常简单，就是创建一个 RAII 风格的锁管理器，在作用域结束后自动销毁锁。          
此处主要介绍std::unique_lock：         

1. **`延迟加锁`**（std::defer_lock），在构造 std::unique_lock 时不立即加锁，而是稍后手动加锁。  
```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx1, mtx2;

void task() {
    std::unique_lock<std::mutex> lock1(mtx1, std::defer_lock); // 不立即加锁
    std::unique_lock<std::mutex> lock2(mtx2, std::defer_lock); // 不立即加锁

    // 手动加锁，避免死锁（按固定顺序）
    std::lock(lock1, lock2); // 安全地同时加锁两个互斥量

    // 访问共享资源
    std::cout << "Thread acquired both locks." << std::endl;

    // 离开作用域，自动销毁锁
}
```

---

2. **`手动加锁/解锁`**，在函数内部临时执行共享数据的互斥读/写。  
```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx;
int shared_data = 0;

void process_data() {
    // ...
    std::unique_lock<std::mutex> lock(mtx);   // 上锁
    ++shared_data;                            // 读写共享数据
    lock.unlock();                            // 解锁
    // ...
}
```

---

3. **`尝试加锁（try_lock）`**，避免线程长时间等待锁，提高程序响应性。   
```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx;

void attempt_lock() {
    std::unique_lock<std::mutex> lock(mtx, std::try_to_lock); // 尝试加锁

    if (lock.owns_lock()) {
        std::cout << "Lock acquired successfully!" << std::endl;
        // 访问共享资源
    } else {
        std::cout << "Failed to acquire lock. Doing something else..." << std::endl;
    }
}
```

---

4. **`与条件变量配合`**，线程等待某个条件满足后被唤醒，实现线程间的同步（如生产者-消费者模型）。  
```cpp
#include <iostream>
#include <mutex>
#include <condition_variable>
#include <thread>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void wait_for_ready() {
    std::unique_lock<std::mutex> lock(mtx);
    /**
     * 1. 自动解锁，并等待信通知。   
     * 2. 线程被唤醒时，会自动重新获取锁，确保后续操作的原子性。  
      */
    cv.wait(lock, []{ return ready; });                         
    std::cout << "Condition met! Proceeding..." << std::endl;
}

void set_ready() {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one();   // 唤醒一个等待线程
}

int main() {
    std::thread t1(wait_for_ready);
    std::thread t2(set_ready);

    t1.join();
    t2.join();
    return 0;
}
```

---

5. **`转移锁所有权（移动语义）`**，在函数间传递锁的管理权。  
```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx;

void transfer_lock(std::unique_lock<std::mutex> lock) {
    std::cout << "Lock transferred to this function." << std::endl;
    // 函数结束时自动解锁
}

int main() {
    std::unique_lock<std::mutex> lock(mtx);
    transfer_lock(std::move(lock)); // 移动锁的所有权
    // 此时 lock 为空，不能再使用
    return 0;
}
```


## 3. 避免死锁  
std::scoped_lock 支持同时锁定多个互斥锁，并通过原子性加锁（内部调用 std::lock）避免死锁。
```cpp
#include <mutex>
#include <thread>

std::mutex mtx1, mtx2;

void update_shared_resources() {
    std::scoped_lock lock(mtx1, mtx2); // 同时锁定两个互斥锁
    // 安全地访问共享资源
}
```


## 4. 使用消息队列  
共享变量越多，锁的复杂度越高，所以尽可能使用消息队列进行线程间通信。   
```cpp
std::queue<int> tasks;
std::mutex mtx;
std::condition_variable cv;

void producer() {
    std::lock_guard<std::mutex> lock(mtx);
    tasks.push(42);                          //  入队
    cv.notify_one();                         //  通知消费者
}

void consumer() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return !tasks.empty(); });  //  等待队列非空
    int task = tasks.front();                      //  读取队头数据
    tasks.pop();  
}
```


## 5. 读写锁（std::shared_mutex）
现在有一个多线程场景，一个线程写数据，另外还有多个线程读数据，那么 `std::shared_mutex` 就派上了用场了， 这是 C++17 提供的特性，允许多个线程同时读，但写操作必须独占。搭配 `std::shared_lock` 和 `std::unique_lock` 使用。   
```cpp
#include <shared_mutex>
#include <thread>
#include <vector>
#include <iostream>

std::shared_mutex rw_mtx;

void reader(int id) {
    std::shared_lock<std::shared_mutex> lock(rw_mtx);
    std::cout << "Reader 线程读数据。" << std::endl;
}

void writer(int val) {
    std::unique_lock<std::shared_mutex> lock(rw_mtx);
    std::cout << "Writer 线程写数据 " << std::endl;
}
```




## 6. 原子操作（atomic）  
可以理解为一个非常简单的计数器。可以原子的递增、递减。     

```cpp
#include <iostream>
#include <thread>
#include <atomic>

std::atomic<int> counter{0};

void worker() {
    for (int i = 0; i < 10000; ++i) {
        counter.fetch_add(1);
    }
}

int main() {
    std::thread t1(worker);
    std::thread t2(worker);
    t1.join();
    t2.join();
    std::cout << "counter = " << counter << std::endl;
}
```


