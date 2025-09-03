---
title: "Python的C++扩展开发（03） —  PyCxx的可调用对象类型(Py::Callable)"
description: 
date: 2025-08-13
image: 
math: 
license: 
hidden: false
comments: true
draft: true
categories:
    - Python
---





```cpp
class Callable: public Object
{
public:
    explicit Callable(): Object() {}

    explicit Callable( PyObject *pyob, bool owned = false ): Object( pyob, owned )
    {
        validate();
    }

    Callable( const Object &ob ): Object( ob )
    {
        // 从Object继承的函数，判断ob是否是一个可调用对象。 
        validate();   
        // 如果ob不是可调用对象，
    }


    /* 执行可调用对象 */
    Object apply( const Tuple &args ) const;
    Object apply( const Tuple &args, const Dict &kw ) const;
    Object apply() const;
    Object apply( PyObject *pargs ) const;
    Object apply( PyObject *pargs = 0 ) const;

}
```

部分成员函数：  
```cpp
Object apply( const Tuple &args ) const
{
    PyObject *result = PyObject_CallObject( ptr(), args.ptr() );
    if( result == NULL )
    {
        ifPyErrorThrowCxxException();
    }
    return asObject( result );
}
```
`PyObject_CallObject`，PyObject_CallObject是Python C API中的一个关键函数，用于调用Python可调用对象，PyObject_CallObject函数接受两个参数： 
```cpp
PyObject * PyObject_CallObject( /* 指向可调用Python对象的指针 */, /* 一个包含调用参数的元组 */ );
```
`ptr()`函数继承自Object，返回一个Object对象。
