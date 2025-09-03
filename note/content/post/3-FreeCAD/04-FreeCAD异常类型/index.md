---
title: "FreeCAD（04） — 异常类型"
description: 
date: 2025-06-14
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - FreeCAD
---


- [1. **异常类型基类**](#1-异常类型基类)
- [2. **抛出异常**](#2-抛出异常)
- [3. **派生的异常类型**](#3-派生的异常类型)


# 1. **异常类型基类**

```cpp
namespace Base
{

class BaseExport Exception: public BaseClass
{
    TYPESYSTEM_HEADER_WITH_OVERRIDE();  // 类型系统

public:
    ~Exception() noexcept override = default;

    virtual const char* what() const noexcept;  // 获取_sErrMsg成员

    virtual void ReportException() const;   // 报告异常信息，用于记录日志、显示错误提示或调试信息。

    /* 设置/获取异常信息 */
    inline void setMessage(const char* sMessage);         // 将sMessage赋值给内部成员_sErrMsg 
    inline void setMessage(const std::string& sMessage);
    inline std::string getMessage() const;

    inline std::string getFile() const;                  // _file
    inline int getLine() const;                          // _line
    inline std::string getFunction() const;              // _function 

    inline void setDebugInformation(const std::string& file, int line, const std::string& function); // 设置异常的调试上下文信息（_line、_file、_function）

    inline bool getReported() const
    {
        return _isReported;  // 返回异常是否已经被报告过（即是否已经通过某种方式输出或记录）
    }

    inline void setReported(bool reported)
    {
        _isReported = reported;  // 设置异常的报告状态，标记异常是否已处理。
    }

protected:
    explicit Exception(const char* sMessage);
    explicit Exception(std::string sMessage);
    Exception();
    Exception(const Exception& inst);
    Exception(Exception&& inst) noexcept;

protected:

    /* 异常相关的上下文信息 */
    std::string _sErrMsg;
    std::string _file;
    int _line;
    std::string _function;

    mutable bool _isReported; // 异常是否已报告
};

}
```


# 2. **抛出异常**
提供了三个版本的throw函数，不同平台上实现有细微的不同。   
```cpp
#define THROW(exception);
#define THROWM(exception, message)  
#define THROWMF_FILEEXCEPTION(message, filenameorfileinfo)  // 携带一些信息
```

# 3. **派生的异常类型**
|||
|------|-----|
|AbortException|中断异常，继承自Exception|  
|XMLBaseException|XML异常基础类型，继承自Exception|  
|XMLParseException|XML文档解析出错，继承自XMLBaseException|  
|XMLAttributeError|XML属性异常，请求的XML属性不存在时抛出异常，继承自XMLBaseException|
|FileException|文件IO异常类型，继承自Exception|   
|FileSystemError|文件系统操作错误的异常类，继承自Exception|
|MemoryException|内存异常|  
|BadGraphError|图异常，不是有向无环图|  
|UtilsMismatchError|单位异常|  
|CADKernelError|内核（OCC）异常|
|||   


异常类型非常多，就不一一列举了。  





