---
title: "DeskFlow 跨平台抽象层（07） — 字符编码"
description: 
date: 2024-09-22
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "DeskFlow"
    - "设计模式"
---



# **功能概述**
ArchString 主要负责提供跨平台的字符编码功能，ArchString的核心功能包括：  
- `字符编码转换`： 提供`多字节字符串`编码和`宽字节字符串`编码之间的双向转换。  
- `多种宽字符编码支持`：支持UCS2、UCS4、UTF16、UTF32等多种宽字符编码格式。  
- `平台相关的编码识别`：每个平台可以报告其原生的宽字符编码格式。    

ArchString将抽象接口和具体实现分离，由ArchStringUnix和ArchStringWindows分别负责Linux和Windows上的实现。  
```cpp
class ArchString : public IInterface
{
public:
    ArchString() = default;
    ArchString(const ArchString &) = delete;
    ArchString(ArchString &&) = delete;
    ~ArchString() override = default;

    ArchString &operator=(const ArchString &) = delete;
    ArchString &operator=(ArchString &&) = delete

    enum class EWideCharEncoding : uint8_t
    {
        kUCS2,  //!< The UCS-2 encoding
        kUCS4,  //!< The UCS-4 encoding
        kUTF16, //!< The UTF-16 encoding
        kUTF32, //!< The UTF-32 encoding
        kPlatformDetermined
    };

    /*
     * dst：    目标字符缓冲区，如果为NULL则只计算长度
     * src：    源宽字符字符串
     * n：      要转换的字符数量
     * errors： 错误标志指针，用于指示转换过程中是否发生错误
     *
     *  该函数将宽字符（wchar_t*）字符串转换为多字节字符串（char*），函数返回转换后的字节数。 
     *  函数会检测转换过程中的错误，当遇到无法转换的字符时，会设置错误标志并用问号（'?'）替代无效字符。
     */
    int convStringMBToWC(wchar_t *, const char *, uint32_t n, bool *errors) const;

    int convStringWCToMB(char *, const wchar_t *, uint32_t n, bool *errors) const;

    // 查看当前平台的宽字符编码方式，Unix平台使用UCS4编码，Windows平台使用UTF16编码
    EWideCharEncoding getWideCharEncoding() const;
};
```


