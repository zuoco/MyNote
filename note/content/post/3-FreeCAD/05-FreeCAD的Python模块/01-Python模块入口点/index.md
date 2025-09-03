---
title: "FreeCAD（05） — Python模块初始化"
description: 
date: 2025-08-09
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - FreeCAD
---


|||
|------|------|
|src/Main/MainCmd.cpp| CLI版本的入口程序|
|src/Main/MainGui.cpp| GUI版本的入口程序|
|src/Main/MainPy.cpp | Python模块（FreeCAD）的初始化函数，负责在Python环境中加载FreeCAD核心功能 |
|src/Main/FreeCADGuiPy.cpp| Python模块（FreeCADGui）的初始化函数，负责在Python环境中加载FreeCADGui核心功能 |


---


- [1. Python模块初始化](#1-python模块初始化)
- [2. FreeCAD模块初始化](#2-freecad模块初始化)
- [3. FreeCADGui模块初始化](#3-freecadgui模块初始化)



## 1. Python模块初始化
FreeCAD定义了宏——PyMOD_INIT_FUNC来辅助实现Python模块的初始化函数:   
```cpp
#define PyMOD_INIT_FUNC(name) PyMODINIT_FUNC PyInit_##name(void)
```  
这里的`PyMODINIT_FUNC`其实是 Python/C API 库中的一个宏，在 Python 3 中，PyMODINIT_FUNC 就是 `PyObject*`，表示函数返回值。   
`PyInit_xxx(void)`，就是模块的初始化函数，它在import模块时执行，并返回一个Python模块对象。   



---


## 2. FreeCAD模块初始化
FreeCAD模块初始化函数定义在MainPy.cpp中：   
```cpp
PyMOD_INIT_FUNC(FreeCAD)
{
    // ......
    // 这个函数负责Python模块的初始化。  
    // ......

    return module;  // 返回模块对象
}
```
这个宏（PyMOD_INIT_FUNC）展开就是：  
```cpp
PyObject* PyInit_FreeCAD(void)
{
    // ......

    return module; // 返回一个 PyObject* 类型的对象
}
```
我们复制 “`PyMOD_INIT_FUNC(`” 到FreeCAD工程源码中搜索，就能看到都有那些模块。   


---

## 3. FreeCADGui模块初始化
加载FreeCADGui模块时会先加载FreeCAD模块。    
```cpp
// FreeCADGuiPy.cpp
PyMOD_INIT_FUNC(FreeCADGui)
{
    try {
        Base::Interpreter().loadModule("FreeCAD");           // 加载FreeCAD模块， Base::Interpreter()返回 Python 解释器单例
        App::Application::Config()["AppIcon"] = "freecad";
        App::Application::Config()["SplashScreen"] = "freecadsplash";
        App::Application::Config()["CopyrightInfo"] = "\xc2\xa9 Juergen Riegel, Werner Mayer, Yorik van Havre and others 2001-2024\n";
        App::Application::Config()["LicenseInfo"] = "FreeCAD is free and open-source software licensed under the terms of LGPL2+ license.\n";
        App::Application::Config()["CreditsInfo"] = "FreeCAD wouldn't be possible without FreeCAD community.\n";

        if (Base::Type::fromName("Gui::BaseView").isBad()) {
            Gui::Application::initApplication();
        }
        static struct PyModuleDef FreeCADGuiModuleDef = {PyModuleDef_HEAD_INIT,   // 这个参数是固定的。  
                                                         "FreeCADGui",            // 模块名称
                                                         "FreeCAD GUI module\n",  // 模块的描述
                                                         -1,                      // 模块实例的大小（-1 表示动态分配）
                                                         FreeCADGui_methods,      // 模块导出的方法表
                                                         nullptr,
                                                         nullptr,
                                                         nullptr,
                                                         nullptr};
        PyObject* module = PyModule_Create(&FreeCADGuiModuleDef);   //  创建模块对象
        return module;
    }
    catch (const Base::Exception& e) {
        PyErr_Format(PyExc_ImportError, "%s\n", e.what());
    }
    catch (...) {
        PyErr_SetString(PyExc_ImportError, "Unknown runtime error occurred");
    }
    return nullptr;
}
```
上面代码中涉及到一个`struct PyModuleDeff`结构体类型， 这是定义 Python 模块的核心结构体。它描述了模块的基本信息、方法、资源管理等。 
```c
struct PyModuleDef {
    PyModuleDef_Base m_base;          // 这个参数是固定的,就是PyModuleDef_HEAD_INIT
    const char* m_name;               // 模块名称
    const char* m_doc;                // 模块文档字符串
    Py_ssize_t m_size;                // 模块实例的大小（-1 表示动态分配）
    PyMethodDef *m_methods;           // 模块导出的方法表
    PyModuleDef_Slot *m_slots;        // 模块的插槽（slots）
    traverseproc m_traverse;          // 遍历函数（垃圾回收）
    inquiry m_clear;                  // 清理函数（垃圾回收）
    freefunc m_free;                  // 释放函数
};
```
这里面需要注意的是`PyMethodDef m_methods`成员，这是模块的方法列表，PyMethodDef结构的原型是：  
```c
struct PyMethodDef 
{
    const char  *ml_name;    // 定义该方法在 Python 中的名称（即用户调用时使用的函数名）。
    PyCFunction  ml_meth;    // 指向实现该方法的 C 函数指针。
    int          ml_flags;   // 标志位组合，描述该方法的参数规则和行为。
    const char  *ml_doc;     // 方法的描述
};
```
FreeCADGui的模块方法列表如下（m_methods参数），包含5个核心GUI操作方法：   
```cpp
struct PyMethodDef FreeCADGui_methods[] = {
    // 创建并显示主窗口
    {"showMainWindow",
     FreeCADGui_showMainWindow,
     METH_VARARGS,
     "showMainWindow() -- Show the main window\n"
     "If no main window does exist one gets created"},
    
    // 启动GUI事件循环
    {"exec_loop",
     FreeCADGui_exec_loop,
     METH_VARARGS,
     "exec_loop() -- Starts the event loop\n"
     "Note: this will block the call until the event loop has terminated"},
    
    // 设置非GUI模式
    {"setupWithoutGUI",
     FreeCADGui_setupWithoutGUI,
     METH_VARARGS,
     "setupWithoutGUI() -- Uses this module without starting\n"
     "an event loop or showing up any GUI\n"},
    
    {"embedToWindow",
     FreeCADGui_embedToWindow,
     METH_VARARGS,
     "embedToWindow() -- Embeds the main window into another window\n"},
    {nullptr, nullptr, 0, nullptr} /* sentinel */
};
```














