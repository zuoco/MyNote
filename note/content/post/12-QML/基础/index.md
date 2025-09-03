---
title: "认识 QML"
description: 
date: 2025-08-25
image: 
math: 
license: 
hidden: false
comments: true
draft: true
categories:
    - QML
---

QML可以编译成资源文件，或者也可以转换成C++代码，再编译成二进制文件。  

## QML 和 Qt Quick
类似于C++和STL的关系。  
```py
import QtQuick

# Window 是 Quick 库中定义好的控件。  
Window {
    width: 640            # 控件的宽度，大部分控件都有宽度和高度属性
    height: 480
    visible: false        # 控件是否显示
    title: qsTr("Hello World")  # 控件的标题
}
```





