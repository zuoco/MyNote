---
title: "lock_guard、unique_lock、scoped_lock"
description: 
date: 2023-06-04
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


- [1. **std::lock\_guard**](#1-stdlock_guard)
- [2. **std::unique\_lock**](#2-stdunique_lock)
    - [2.1. **多种构造方式**](#21-多种构造方式)
    - [2.2. **使用方法**](#22-使用方法)
    - [2.3. **条件变量**](#23-条件变量)
- [3. **std::scoped\_lock**](#3-stdscoped_lock)




|智能锁|用途|
|------------|------------|
|std::lock_guard |简化的互斥锁管理模板类|
|std::unique_lock|增强版的lock_guard，支持条件变量|
|std::scoped_lock|用于理多个互斥锁|



# 1. **std::lock_guard**
std::lock_guard 是一个用于简化互斥锁（std::mutex）管理的RAII类模板。它的核心目标是确保在作用域结束时自动释放互斥锁，从而避免手动调用 lock() 和 unlock() 可能导致的资源泄漏或死锁问题。   
**主要特性包括：**    
- 自动加锁/解锁： 在构造时自动锁定互斥锁，析构时自动解锁。    
- 不可复制/不可移动： 禁止拷贝和移动操作，防止锁的意外传递或释放。    
- 非递归性： 不支持递归锁定同一互斥锁，多次锁定会导致未定义行为。   
- 异常安全性： 即使在临界区代码中抛出异常，锁也会被正确释放。    
```cpp
{
    std::lock_guard<std::mutex> lock(mtx);  // 构造时加锁
    // 临界区代码
}   // 析构时自动解锁
```


# 2. **std::unique_lock**
std::unique_lock 是 C++11 标准引入的互斥锁管理工具，用于管理 std::mutex 的生命周期。比 std::lock_guard 更强大，提供了更多控制选项，例如延迟锁定、尝试锁定、手动解锁等。但是std::unique_lock 的灵活性以性能为代价，简单场景推荐使用 std::lock_guard。  
**主要特性：**  
- RAII 机制： 自动加锁/解锁，在构造时自动获取锁，析构时自动释放锁，确保异常安全性。
- 手动控制：允许显式调用 lock() 和 unlock()，灵活控制锁的生命周期。
- 支持条件变量： 唯一支持条件变量（std::condition_variable）的锁类型，条件变量的 wait() 需要临时释放锁并重新获取。   
- 支持移动语义（move），允许将锁的所有权转移到其他 std::unique_lock 对象。   

## 2.1. **多种构造方式**
**std::unique_lock 提供了多种构造方式，通过不同的标签（tag）控制锁定行为：**   
- std::defer_lock：延迟锁定（构造时不锁定互斥锁）。   
- std::try_to_lock：尝试锁定（构造时尝试获取锁，失败则不阻塞）。   
- std::adopt_lock：接管已锁定的互斥锁（适用于手动锁定后交给 unique_lock 管理）。    

## 2.2. **使用方法**  
**1. 基本使用** 
```cpp
#include <mutex>
#include <thread>

std::mutex mtx;

void thread_func() {
    std::unique_lock<std::mutex> lock(mtx);  // 构造时自动锁定
    // 临界区代码
}   
// 析构时自动解锁
```
**2. 延迟锁定**
```cpp
std::mutex mtx;
std::unique_lock<std::mutex> lock(mtx, std::defer_lock);   // 构造时不锁定
// 执行其他操作
lock.lock();    // 手动锁定
// 临界区代码
lock.unlock();  // 手动解锁
```
**3. 尝试锁定**
```cpp
std::mutex mtx;
std::unique_lock<std::mutex> lock(mtx, std::try_to_lock);
if (lock.owns_lock()) {
    // 成功获取锁，执行临界区代码
} else {
    // 未获取锁，执行其他逻辑
}
```
**4. 接管已锁定的互斥锁**
```cpp
std::mutex mtx;
mtx.lock();  // 手动锁定
std::unique_lock<std::mutex> lock(mtx, std::adopt_lock);  // 接管已锁定的互斥锁
// 临界区代码
// 析构时自动解锁
```
**5. 超时锁定**
```cpp
std::mutex mtx;
std::unique_lock<std::mutex> lock(mtx);
if (lock.try_lock_for(std::chrono::seconds(1))) {
    // 最多等待1秒获取锁
    // 临界区代码
} else {
    // 超时未获取锁
}
```

## 2.3. **条件变量**
std::condition_variable，通过 wait()、notify_one() 和 notify_all() 实现线程间的协作。   
- wait()：释放锁并等待唤醒，唤醒后会重新获取锁并检查条件，还有wait_for()、wait_until()等超时等待方法。   
- notify_one()：唤醒一个等待线程。  
- notify_all()：唤醒所有等待线程。  
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> buffer;
const int MAX_BUFFER_SIZE = 5;

void producer() {
    for (int i = 0; i < 10; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        // 使用谓词防止虚假唤醒，被唤醒后会检查谓词，如果返回false会继续等待，如果返回true就获取锁并执行后续的代码（临界区）。
        cv.wait(lock, [ ] { return buffer.size() < MAX_BUFFER_SIZE; }); 
        buffer.push(i);
        std::cout << "Produced: " << i << std::endl;
        lock.unlock();
        cv.notify_one();  // 通知消费者
    }
}

void consumer() {
    for (int i = 0; i < 10; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [ ] { return !buffer.empty(); });
        int value = buffer.front();
        buffer.pop();
        std::cout << "Consumed: " << value << std::endl;
        lock.unlock();
        cv.notify_one();  // 通知生产者
    }
}

int main() {
    std::thread prod(producer);
    std::thread cons(consumer);

    prod.join();
    cons.join();
    return 0;
}
```


# 3. **std::scoped_lock**
在C++17中，std::scoped_lock 是一个用于管理多个互斥锁（std::mutex）的工具，它通过RAII机制确保锁的正确获取和释放。它的核心目标是简化多线程代码中对多个互斥锁的管理，避免死锁，并提高代码的异常安全性。

- 多锁管理：支持同时锁定多个互斥锁，避免手动管理多锁时的复杂性。
- 死锁预防：内部按照互斥锁的地址顺序统一锁定，确保所有线程以相同的顺序获取锁，从而防止死锁。
- 不可复制/可移动：禁止拷贝操作，但支持移动语义（如作为函数返回值）。

```cpp
{
    std::scoped_lock lock(mtx1, mtx2);  // 构造时同时锁定两个互斥锁
    // 临界区代码（安全地访问共享资源）
}  // 析构时自动解锁所有锁
```
与 std::unique_lock 不同，std::scoped_lock 不支持延迟锁定或尝试锁定（try_lock）。