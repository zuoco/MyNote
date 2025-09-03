---
title: "Python的C++扩展开发（01） - PyCxx中的有那些类型"
description: 
date: 2025-08-11
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - Python
---


为了简化 C++ 与 Python 的互操作，PyCXX设计了 Object 类，使用 C++ 封装了 Python 对象（PyObject*）。   
整个库最核心的就是Object类，它的核心是封装了一个 PyObject* 指针，并管理其生命周期（引用计数）。   
通过 C++ 的 RAII 机制，确保 Python 对象在 C++ 作用域内安全地创建和销毁。     


**PyCXX中的所有类型位于 Py 命名空间下：**         
|Py命名空间中的类型|功能|
|--------------------------------------------|--------------------------------------------|
| `Py::Object`             | PyCXX框架中的基类，它封装了一个 PyObject* 指针（p），并管理其生命周期（引用计数）。<br> 通过 C++ 的 RAII（资源获取即初始化）机制，确保 Python 对象在 C++ 作用域内安全地创建和销毁。|
| `Py::PythonClass<T>`     | 用于将一个 C++ 类型 （T） 封装为 Python 可识别的类对象，暴露 C++ 成员函数和属性给 Python|   
| `Py::ExtensionModule<T>` | 用来定义Python模块，在构造时将模块注册到 Python 解释器中。|        
| `Py::List`               | C++实现的Python序列类，存储序列对象（如列表/元组）|   
| `Py::Tuple`              | C++实现的Python元组|  
| `Py::Dict`               | C++实现的Python字典|      
| `Py::Callable`           | 使用C++封装的Python可调用对象|  

**Py 命名空间下的函数：**    
|Py命名空间下的函数||
|------------------------------|----------------------------------|
|`void Py::_XINCREF( PyObject *op );`| 增加PyObject的引用计数。 |      
|`void Py::_XDECREF( PyObject *op );` |减少PyObject的引用计数，当引用计数减至0时，对象会被自动释放。这是CPython内部用于内存管理的核心机制。 |
|`bool operator==( const Object &o1, const Object &o2 );`<br> |Py::Object对象比较, 同时也实现了 !=, >=, <=, <, > |



## 1. Py::Object基类
在PyCXX中，所有类型的基类都是Py::Object派生类型，直接派生或者间接派生。   

```cpp
namespace Py
{

class Object
{
private:
       PyObject *p;  // 通过组合 PyObject* 实现对 Python 对象的封装
protected:
      void set( PyObject *pyob, bool owned = false )  // 设置 p
      {
          release();  // 先释放原先持有的 p
          p = pyob;
          if( !owned ) { Py::_XINCREF( p ); }
          validate();
      }

      void release()  // 释放 p
      {
          Py::_XDECREF( p );
          p = NULL;
      }

      void validate();  //  于验证当前对象的类型是否与预期匹配
      
public:  
      // 构造函数:  根据 owned 决定是否增加 pyob 的引用计数。  
      //  owned:  Object对象是否拥有 pyob。  
      explicit Object( PyObject *pyob=Py::_None(), bool owned = false );
      Object( const Object &ob );

      Object &operator=( const Object &rhs );  // 调用 set( rhs.p );
      Object &operator=( PyObject *rhsp )      // 防止自赋值

      // 析构函数: pyob 引用计数减1，并 p = NULL。
      virtual ~Object()
      {
          release();
      }

      PyObject *operator*() const;        // 重载解引用，也就是返回 p
      void increment_reference_count();   // 调用Py_INCREF宏增加对象引用计数
      void decrement_reference_count();   // 特殊处理引用计数减少，当引用计数为1时抛出异常（防止对象自我销毁），正常情况调用Py_DECREF宏减少引用计数

      PyObject *ptr() const;        // 直接返回 PyObject 对象指针

      /*
       *  判断给定的Python对象是否可用于当前类, 
       *  例如派生类型Py::Tuple重写了该函数，当Python对象为Py::Tuple时返回true
       */
      virtual bool accepts( PyObject * ) const;   
      Py_ssize_t reference_count() const;  // 返回当前对象的引用计数
      Type type() const;                   // 返回当前Python对象对应的类型对象，用于获取Python对象的类型信息等

      String str() const;                  // 获取Python对象的字符串表示(Py::string)，例如： 字典 {'a':1} -> "{'a': 1}"
      std::string as_string() const;       // 调用了str()，但是强制转换为std::string

      List dir() const;                             // 获取Python对象的属性名称列表并封装为C++的List对象返回

      bool hasAttr( const std::string &s ) const;   // 检查Python对象是否包含指定属性
      Object getAttr( const std::string &s ) const; // 获取属性的值
      void setAttr( const std::string &s, const Object &value );
      void delAttr( const std::string &s );

      // 函数调用，function_name就是函数名
      Object callMemberFunction( const std::string &function_name ) const; // 调用无参函数
      Object callMemberFunction( const std::string &function_name, const Tuple &args ) const; // 调用带有可变参的函数
      Object callMemberFunction( const std::string &function_name, const Tuple &args, const Dict &kw ) const;  // 调用带有可变参和关键字参数的函数


      Object getItem( const Object &key ) const; // 键值访问，用于Python容器类型（如字典/列表）
      Py_hash_t hashValue() const;               // 计算对象哈希值

      bool is( PyObject *pother ) const;        // return p == pother;
      bool is( const Object &other ) const;     // return p == other.p;

      // 检测 p 是否为某个类型
      bool isNull() const;    
      bool isNone() const;     
      bool isCallable() const;    
      bool isDict() const;
      bool isList() const;
      bool isMapping() const;
      bool isNumeric() const;
      bool isSequence() const;
      bool isTrue() const;
      bool isTuple() const;
      bool isString() const;
      bool isBytes() const;
      bool isBoolean() const;
      bool isType( const Type &t ) const;  // 当前对象的类型是否与参数t的类型完全相同

      void delItem( const Object &key );   // 删除 p 中的键值对
};

}
```