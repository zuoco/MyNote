---
title: "Python的C++扩展开发（02） — 使用PyCxx开发Python模块"
description: 
date: 2025-08-12
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - Python
---



使用C++编写一个Python模块要做 4 件事情：      
- `1. C++类型定义`，将这个类型封装为 Python 类对象。     
- `2. 定义一个模块`，一个模块对应一个C++的类型。  
- `3. 实现模块初始化函数`，当import 模块时，Python会调用这个初始化函数。      
- `4. 构建模块`，使用 setup.py 构建模块。   


下面我们一 一讲解这 4 件事情:      
- [1. 使用C++封装一个 Python 对象](#1-使用c封装一个-python-对象)
- [2. 定义模块](#2-定义模块)
- [3. 模块初始化](#3-模块初始化)
- [4. 构建模块](#4-构建模块)


## 1. 使用C++封装一个 Python 对象
这一步涉及一些宏函数， 封装Python对象的C++类中需要实现一个静态成员函数`static void init_type(void)`，在这个函数中使用这些宏来注册那些要暴露给Python的成员函数，并且模块初始化中需要调用这个静态函数。   
这些宏分为两部分： `注册`、`实现`，先来了解一下：   

- **1. 用于注册方法的宏**：    
```cpp
/* 
 *  这些宏需要3个参数： 
 *  1. Python中的方法名称。      
 *  2. C++中的方法名称。   
 *  3. 函数描述信息。 
 */
PYCXX_ADD_NOARGS_METHOD     // 注册一个无参数方法
PYCXX_ADD_VARARGS_METHOD    // 注册一个接受可变参数的方法
PYCXX_ADD_KEYWORDS_METHOD   // 注册一个支持关键字参数的方法
```

- **2. 用于实现方法的宏**   

```cpp
/* 
 *  这些宏需要2个参数： 
 *  1. C++类名称。      
 *  2. C++函数名称。   
 */
PYCXX_NOARGS_METHOD_DECL     // 定义一个无参数方法
PYCXX_VARARGS_METHOD_DECL    // 定义一个接受可变参数的方法
PYCXX_KEYWORDS_METHOD_DECL   // 定义一个支持关键字参数的方法 
```

- **3. 示例**   

类定义：  
```cpp
class CppClass: public Py::PythonClass< CppClass >
{
public:
    // self: 指向 Python 实例的指针，用于将 C++ 对象与 Python 实例关联。在基类构造函数中，self 会绑定 C++ 对象到 Python 实例。
    CppClass( Py::PythonClassInstance *self, Py::Tuple &args, Py::Dict &kwds ); // 

    virtual ~CppClass();


    // 注册成要暴露给Python的成员函数
    static void init_type();

    Py::String m_value;
};
```

成员函数实现：   
```cpp
/* 构造函数 */
CppClass::CppClass(Py::PythonClassInstance* self, Py::Tuple& args, Py::Dict& kwds): 
        Py::PythonClass<CppClass>::PythonClass(self, args, kwds)，m_value("_default_value")
{
    // ... 
};


/* 注册成员函数 */
static void CppClass::init_type()
{
        behaviors().name( "simple.CppClass" );                  // 设置 Python 中该类的全限定名。simple 是模块名，new_style_class 是类名。
        behaviors().doc( "documentation for CppClass class" );  // 设置类的文档字符串（__doc__）， 在 Python 中，用户可以通过 help(simple.new_style_class) 查看此文档。
        behaviors().supportGetattro();  // 启用对 __getattr__ 的支持，允许 Python 通过属性名访问类的成员（如 obj.value）。
        behaviors().supportSetattro();  // 启用对 __setattr__ 的支持，允许 Python 修改属性值（如 obj.value = "new value"）。


        // 启用对 Python 数字运算的支持，允许 C++ 类对象参与数学运算, 需在类型中实现 number_add 和 number_inplace_add 方法。
        behaviors().supportNumberType( Py::PythonType::support_number_add, Py::PythonType::support_number_inplace_add );


        /* 注册C++成员函数到Pythonn对象 */ 
        /*
         *  python名称： func
         *  C++名称：    cppclass_func
         *  描述文档：    docs for cppfunc
         */             
        PYCXX_ADD_NOARGS_METHOD( func, cppclass_func, "docs for cppfunc" ); 
        
        /*
         *  Python 调用格式为 obj.call_member("func_name", arg1, arg2, ...)，其中 "func_name" 是目标成员函数名。
         */
        PYCXX_ADD_VARARGS_METHOD( func_call_member, cppclass_call_member, "docs for func_call_member" );


        // 完成类型注册，通知 PyCXX 类型定义完成，准备就绪。必须调用这个方法，否则类无法在 Python 中正常使用。
        behaviors().readyType();

};
```

被注册的函数定义时，要符合注册时函数签名要求。
```cpp
// 不带参数的成员函数
Py::Object cppclass_func_noargs( void )
{
    // ......
}
PYCXX_NOARGS_METHOD_DECL( CppClass, cppclass_func_noargs )


// 带有可变参数的函数，带有Py::Tuple &参数
Py::Object Py::PythonClass::cppclass_call_member( const Py::Tuple &args )  
{  
    // ......
}
PYCXX_VARARGS_METHOD_DECL( CppClass, cppclass_call_member ) // CppClass: 类名， cppclass_call_member：成员函数名。


// 带有字参数的函数，带有Py::Dict &参数
Py::Object cppclass_func_keyword( const Py::Tuple &args, const Py::Dict &kwds )
{
    // ......
}
PYCXX_KEYWORDS_METHOD_DECL( CppClass, cppclass_func_keyword )
```


## 2. 定义模块
使用`Py::ExtensionModule<T>`类来定义模块。
```cpp
class MyModule : public Py::ExtensionModule<MyModule>
{
public:
    MyModule(): Py::ExtensionModule<MyModule>( "mymodule" )  // 模块名称 "mymodule" 必须与生成的动态库文件名一致（如 simple.so 或 simple.pyd）。
    {
        /* 1. 初始化 CppClass 的 Python 类定义 */
        CppcClass::init_type();       


        /* 2. 注册模块级别的函数 */
        add_varargs_method("CppClass", &MyModule::factory_cppclass, "***");  // 向 Python 扩展模块注册接受可变参数的方法
        add_keyword_method("func", &MyModule::func, "***");  // 向 Python 扩展模块添加一个支持字典类型参数的方法
        // ... ...


        /* 3. 模块初始化 */
        initialize( "documentation for the mymodule" );


        /* 4. 模块的字典对象，字典包含了模块的所有属性（变量、函数、类等），这个字典对象是在initialize()中创建的 */
        Py::Dict d( moduleDictionary() );          // 获取模块的字典对象。 

        d["var"] = Py::String( "var value" );      // 在模块字典中创建一个名为 "var" 的字符串变量赋值为 "var value"。
    
        Py::Object x( CppClass::type() );          // 获取 CppClass 类的类型对象（PyTypeObject），将其包装为 PyCXX 的 Object 类型
        d["CppClass"] = x;     // 将 CppClass 类型对象添加到模块字典中，这样在 Python 中就可以通过 mymodule.CppClass 访问这个类并创建其实例


        /* 5. 初始化异常 */
        SimpleError::init( *this ); // 创建一个名为 "ModuleError" 的扩展异常类型，将这个异常类型添加到模块的字典中，使其在 Python 中可以通过 mymodule.ModuleError 访问。  
    }

    virtual ~MyModule(){}
};
```
add_varargs_method和add_keyword_method是 Py::ExtensionModule<T> 的两个静态方法，分别用于注册带可变参数的成员函数和带字典参数的成员函数。        
两种函数在定义时，也要符合注册时的规范：   
```cpp
// 使用add_varargs_method注册的函数，函数参数列表必须有Py::Tuple类型的参数，可以不用但是不能没有。
Py::Object MyModule::factory_cppclass( const Py::Tuple &/*args*/ )
{
    // ...
}

// 使用add_keyword_method注册的函数，函数参数列表必须有Py::Dict 类型的参数，可以不用但是不能没有。
Py::Object MyModule::func( const Py::Tuple &args, const Py::Dict &kwds )
{
    // ....
};
```


## 3. 模块初始化
现在需要定义一个模块初始化函数，该函数的形式为`PyObject *PyInit_xxx()`， “xxx”就是模块名称。    
当执行 import xxx 时，Python 解释器会加载 xxx 的动态库（如 xxx.so 或 xxx.pyd），并调用 PyInit_xxx 初始化模块。
```cpp
// EXPORT_SYMBOL是平台的动态库导出标记
extern "C" EXPORT_SYMBOL PyObject *PyInit_simple()
{
    // ...
    // ...
    // 返回模块之指针（一个 PyObject 对象）  
}
```


## 4. 构建模块
使用Python的setuptools编写构建脚本，一般命名为setup.py，该脚本文件包含三部分：    
1. 系统路径配置。
2. 配置必要的链接库。
3. 模块配置。 


```python
# 导入模块
import os, sys
import setuptools from setup, Extension

# 1. 系统路径配置： /usr/share/pythonx.xx/CXX/
support_dir = os.path.normpath(
                    os.path.join(
                    sys.prefix,             # sys.prefix在Linux系统上就是 /usr
                    'share',                
                    'python%d.%d' % (sys.version_info[0],sys.version_info[1]),
                    'CXX' ) )


# 2. 需要链接的库  
if os.name == 'posix':
    CXX_libraries = ['stdc++','m']
else:
    CXX_libraries = []


# 3. 模块配置
setup(
    name = "xxx",                                              # 模块名称
    version  = "%s.%s.%s" % (v_maj, v_min, v_pat),             # 模块版本
    maintainer = "xxx",                                        # 模块维护者 
    maintainer_email = "xxx@xxx.com",                          # 模块维护者邮箱
    description = "xxxxxx",                                    # 模块描述，简要说明项目用途
    url = "http://xxxx.com/",                                  # 模块地址


    packages = ['mxxx'],                                     # 要打包的Python包列表，也可以使用find_packages()查找                                           
    package_dir = {'mxxx': '.'},                             # 将mxxx包的源码目录（.表示setup.py所在目录）

    # 模块配置
    ext_modules = 
    [
        Extension( 'xxx.xxx',  # 模块全限定名
                    
                    # 源文件列表
                    sources = 
                    [
                       'src_0.cxx', 
                       'src_1.cxx', 
                       'src_n.cxx',   
                       os.path.join(support_dir,'cxxsupport.cxx'),     
                       os.path.join(support_dir,'cxx_extensions.cxx'),
                       os.path.join(support_dir,'cxx_exceptions.cxx'),
                       os.path.join(support_dir,'IndirectPythonInterface.cxx'),
                       os.path.join(support_dir,'cxxextensions.c')
                    ]  
                )
    ]
)
```
