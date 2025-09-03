---
title: "QML"
url: "docs/qml"
aliases:
- "/docs"

date: 2025-08-25
---


## 1. QML 基础概念

QML 和 Qt Quick 类似于 C++ 和 STL 的关系。  
```py
# 导入库
import QtQuick

# 使用库中的控件， Window 就是 Quick 库中定义好的控件。  
Window {
    width: 640                  # 控件的宽度，大部分控件都有宽度和高度属性
    height: 480
    visible: false              # 控件是否显示
    title: qsTr("Hello World")  # 控件的标题
}
```

## 2. 应用程序基础窗口
一个 QML 界面首先需要创建一个 `Window` 或者 `ApplicationWindow` 控件，然后使用 Rectangle 控件来填充这个界面，我们在 Rectangle 控件中绘制界面。      

**那么 Window 和 ApplicationWindow 的区别是什么呢？**   

| 特性        | ApplicationWindow                             | Window         |
| --------- | --------------------------------------------- | -------------- |
| **所属模块**  | QtQuick.Controls                              | QtQuick.Window |
| **功能定位**  | 完整的应用程序主窗口                                    | 轻量级的独立窗口       |
| **内置结构**  | 提供菜单栏（menuBar）、工具栏（header）、状态栏（footer）等应用窗口结构 | 仅提供一个空窗口，无内置结构 |
| **样式支持**  | 支持自动样式和主题切换                                   | 需手动设置样式        |
| **使用复杂度** | 封装度高，适合快速开发                                   | 灵活但需手动配置       |

实际上 ApplicationWindow 继承自 Window，两个控件都有 flags 属性，可以通过 flags 属性来控制窗口的外观和行为（如是否有标题栏、是否可以调整大小、是否置顶等）。该属性接受一个或多个 Qt 窗口标志（Qt::WindowFlags）的组合。      
| 标志                              | 说明               |
| ------------------------------- | ---------------- |
| **Qt.Window**                   | 普通窗口（默认）         |
| **Qt.Dialog**                   | 对话框窗口            |
| **Qt.Popup**                    | 弹出窗口（无边框，无任务栏图标） |
| **Qt.Tool**                     | 工具窗口（无任务栏图标）     |
| **Qt.SplashScreen**             | 启动画面             |
| **Qt.FramelessWindowHint**      | 无边框窗口（无标题栏），`需要手动实现拖动和缩放功能`。|  
| **Qt.WindowTitleHint**          | 显示标题栏            |
| **Qt.WindowSystemMenuHint**     | 显示系统菜单           |
| **Qt.WindowMinimizeButtonHint** | 显示最小化按钮          |
| **Qt.WindowMaximizeButtonHint** | 显示最大化按钮          |
| **Qt.WindowCloseButtonHint**    | 显示关闭按钮           |
| **Qt.WindowStaysOnTopHint**     | 置顶窗口，`窗口始终显示在其他窗口之上` |
| **Qt.WindowStaysOnBottomHint**  | 置底窗口             |

如果不指定 flags，默认使用 Qt.Window。   


## 3. 基础控件
控件的基础属性：   
|属性|描述|
|--------------------------|--------------------|
|width  | 控件的宽度|   
|height | 控件的高度|  
|id     | 控件的id，用于在其他处引用|  


### 3.1. Item
Item 控件的属性介绍：   

|属性|描述|
|-------------------------|------------------------|
|anchors | 控制矩形的锚点定位。 <br> 用于指定该元素与其他元素（父元素或同级兄弟元素）之间的相对位置关系。 <br> `使用方法`： <br> `1.` 通过锚线（top、bottom、left、right、horizontalCenter、verticalCenter等）将当前元素的边界与另一个元素的特定边界对齐。例如，anchors.left: parent.left（左对齐父元素）、anchors.centerIn: parent（在父元素中居中）、anchors.fill: parent（完全填充父容器的空间）。<br> `2.` 填充父元素：通过anchors.fill: parent使元素完全填充父容器的空间。<br> `3.` 偏移调整：可通过margins（如anchors.margins）或offsets（如anchors.horizontalCenterOffset）微调位置，实现类似“略微偏离中心”的效果。<br> `4.` 布局约束：锚点关系比直接设置x、y、width、height等几何属性优先级更高。|       
|||  
|||     

### 3.2. Rectangle
Rectangle 继承自 Item。  
|属性|描述|
|-----------------------------|------------------------|
|radius| 控制 Rectangle（矩形）元素的角半径，从而创建圆角矩形或圆形。<br> 创建圆形： 创建一个正方形，将 radius 设置为宽度或高度的一半 (radius: width / 2)，即可将该矩形变为一个完美的圆形。|   
|||   
|||   
|||   



## 4. 自定义控件的属性
在QML中使用 `property` 定义自定义的属性。   
```bash
    property int dragX: 0
    property int dragY: 0
    property bool dragging: false
```

## 5. 鼠标输入
### 5.1. MouseArea
`MouseArea` 是 Qt Quick 中一个用于处理鼠标输入的核心不可见元素。它充当一个敏感区域，能够检测并响应其范围内的鼠标操作（如点击、按下、释放、悬停等），从而为其他视觉元素赋予交互能力。`MouseArea 将输入处理逻辑与视觉呈现分离。   
一个 MouseArea 可以覆盖多个视觉元素，或者一个视觉元素可以由多个 MouseArea 处理不同部分的交互。我们可以将 MouseArea 做得比视觉元素更大，从而创建一个更大的“热区”，提升用户体验（例如，让一个小图标的可点击区域变大）。   


#### 5.1.1. MouseArea 的信号
MouseArea 通过发射预定义的信号来响应各种鼠标事件。我们可以通过实现对应的信号处理器来定义交互逻辑，常用的信号有：        
|信号| 描述|
|-----------------------|------------------------|
|onPressed  |  鼠标按下时立即触发。|
|onReleased |  鼠标释放时触发。|
|onDoubleClicked| 鼠标双击时触发。|  
|onPressAndHold|  鼠标长按（按住一段时间）时触发。|
|onWheel|  使用动鼠标滚轮时触发。|
|onEntered, onExited|   当鼠标进入或离开区域时触发（需要启用 hoverEnabled）。|   

**使用案例：**
```bash
MouseArea {
    onClicked: {
        console.log("Clicked!");
        // 在此添加点击后的逻辑，例如改变父元素的状态
    }
}
```

#### 5.1.2. MouseArea 属性
<span style="color:red;">**`1. anchors`**</span>     
anchors.fill: parent: 这是最经典的用法。此属性将 MouseArea 的尺寸锚定到其父元素，使其完全覆盖父元素，从而创建一个与视觉元素大小完全一致的可点击区域。   
```py
Rectangle {
    id: myRect
    width: 100; 
    height: 100
    color: "red"

    MouseArea {
        anchors.fill: parent                          // 此MouseArea的大小与myRect完全相同
        onClicked: console.log("Rectangle clicked")
    }
}
```


<span style="color:red;">**`2. hoverEnabled`**</span>    
hoverEnabled: 布尔值（默认为 false）。决定是否跟踪鼠标光标位置，而无需按下任何按钮。设置为 true 才能启用 onEntered 和 onExited 等悬停信号。
```py
MouseArea {
    hoverEnabled: true
    onEntered: { /* 鼠标进入时执行的操作 */ }
    onExited: { /* 鼠标离开时执行的操作 */ }
}
```

<span style="color:red;">**`3. acceptedButtons`**</span>       
acceptedButtons: 指定此 MouseArea 响应哪些鼠标按钮（例如 Qt.LeftButton, Qt.RightButton, Qt.MiddleButton）。这允许我们为左键和右键点击设置不同的操作。
```py
MouseArea {
    acceptedButtons: Qt.LeftButton | Qt.RightButton
    onClicked: (mouse) => {
        if (mouse.button === Qt.RightButton) {
            console.log("Right button clicked");
        }
    }
}
```

<span style="color:red;">**`4. enabled`**</span>     
enabled: 布尔值，用于启用或禁用整个 MouseArea。当设置为 false 时，它将不再响应任何鼠标事件。


<span style="color:red;">**`5. cursorShape`**</span>     
cursorShape: 允许在鼠标悬停在该区域时改变光标形状（例如 Qt.PointingHandCursor, Qt.CrossCursor）。


### 5.2. Input Handlers
与 MouseArea 不同，Input Handlers需要定义 `Handlers`， 每个 `Handler` 只处理一个鼠标事件，控制粒度更加精细。而对于 MouseArea，一个MouseArea元素集中处理鼠标的所有事件。    


## 6. 渐变色
使用 Gradient 和 GradientStop，`Gradient` 是一个渐变类型的容器，`GradientStop` 是一个渐变中的一个颜色停止点。    
GradientStop有两个属性： position 和 color，`position` 定义颜色停止点在渐变路径上的位置，取值范围通常是 0.0（渐变的起点）到 1.0（渐变的终点），例如，position: 0.5 代表在渐变的中间点。`color`定义在该位置的颜色。             

```qml
Rectangle {
    width: 100
    height: 100
    gradient: Gradient {
        // color也可以使用 “#c850c0” 的形式
        GradientStop { position: 0.0; color: "blue" }          // position: 0.0 表示控件最左边
        GradientStop { position: 1.0; color: "slategray" }     // position: 1.0 表示控件最右边
    }
}
```

## 7. 文本
###  7.1. 显示文本
```qml
  Text  
  {
      x: 530
      y: 130
      width: 120
      height: 30
      font.pixelSize: 24
      text: qsTr("登录系统")
      color: "#333333"
  }
```
### 7.2. 输入文本
```qml
    TextField
    {
        id:username
        x: 440
        y: 200
        width: 300
        height: 50
        font.pixelSize: 20
        color: "#666666"                     // 输入词颜色

        placeholderText: qsTr("用户名或邮箱")    // 提示文字
        placeholderTextColor: "#999999"      // 提示词颜色
        
        leftPadding: 60         // 提示/输入文本和输入框左边距的距离  

        // 背景填充，使用了 TextField 的 background 属性
        background: Rectangle
        {
            color: "#e6e6e6"            // 设置填充颜色
            border.color: "#e6e6e6"     // 设置边框颜色
            radius: 25                    // 矩形圆角效果
        }


        // 输入框小图标： 在输入框焦点状态变化时，切换图标样式来显示输入框状态
        Image
        {
            source: username.activeFocus ? "images/u2.png" : "images/u1.png"
            width:  20
            height: 20
            x: 30
            y: 15
        }

        NumberAnimation on y
        {
            from: username.y-50
            to: username.y
            duration: 1000
        }
    }
```
如果是用于输入密码的，可以使用 echoMode 属性， 例如`echoMode: TextInput.Password`，密码显示为星号。     


## 8. 按钮
|按钮|描述|
|----------------------------|-------------------------------|
|id||
|x，y||
|width，height||  
|text|按钮上显示的文本|
|font.pointSize|字体的大小|  
|background| 按钮的背景色|
|down| BOOL值，按钮按下时为true|
|hovered| BOOL值，按钮悬停时为true|  

```qml
    Button
    {
        id: submit
        x: username.x
        y: password.y + password.height + 10
        width: username.width
        height: username.height
        text: qsTr("登录")
        font.pixelSize: 20
        onClicked:
        {
            // 按钮按下时的逻辑
            print("登录：" + username.text + " : " + password.text)
        }

        // 按钮背景颜色
        background: Rectangle
        {
            radius: 25
            color:
            {
                if(submit.down)
                    return "#00b846"
                if(submit.hovered)
                    return "#333333"   // 鼠标悬停时的颜色
                return "#57b846"
            }

        }
    }
```

## 9. 加载和显示图片  
### 9.1. Image  

|属性 | 描述 |
|---------------------------|--------------------------------|
|source | 用于加载图片，指定图像资源的 URL（统一资源定位符），它告诉 Image 元素从哪里获取图像数据。 <br> **`1. `** source 的属性值是一个 url 类型，它可以指向两种主要的资源位置，本地文件路径和网络资源路径。 <br> **`1.`** source 属性可以被动态修改，可以在运行时根据程序的状态改变显示的图片，可以用于实现图标状态切换（如按钮的激活/未激活状态）或图片查看器时非常有用。<br> **`3. `** 在图像查看器中通过文件对话框或获取文件路径。   |  
|asynchronous| 默认为 false，所以图片加载是同步的，这意味着在图像完全加载之前，UI 线程可能会被阻塞。对于大型图像或网络图像，这会导致界面卡顿。<br> 我们将其设置为 true 来强制在后台线程中加载图像，避免阻塞主线程。|  
|x、y| 设置Image控件控件在父控件的位置。|       
|width、height| 空间的宽度和高度。|   
|||    




## 10. 动画
`NumberAnimation` 用于在两个数字值之间创建平滑的过渡动画，在控件中使用 `NumberAnimation` 控件定义简单动画。 

```qml
    NumberAnimation on y    // 沿控件的 y 轴移动
    {
        // 从 “y-50” 运动到 “y”
        from: username.y-50
        to: username.y
        duration: 3000     // 动画时间为3秒
    }
```


## 11. 自定义标题栏（关闭、最大化、最小化）  




## 12. ApplicationWindow 简单示例
创建一个登录窗口。   
```py
import QtQuick
import QtQuick.Controls

ApplicationWindow
{
    width: 640
    height: 480
    visible: true
    title: qsTr("Example Window")
    id: window

    Rectangle
    {
        Text
        {
            x: 300
            y: 50
            width: 120
            height: 100
            font.pixelSize: 24
            text: "登录"         //  文本为中文时使用 qsTr("中文")
        }

        TextField     // 文本输入框, 用户
        {
            id: user
            x: 360
            y: 100
            width: 200
            height: 50
        }

        TextField     // 文本输入框, 密码
        {
            id: passwd
            x: user.x
            y: user.y + 55
            width: user.width
            height: user.height
            font.pixelSize: user.font.pixelSize
            echoMode: TextInput.Password           // 密码模式，输入显示为**
        }

        Button
        {
            id: botton
            x: 230
            y: passwd.y + 200
            width: user.width
            height: user.height
            font.pixelSize: 24
            text: qsTr("登录")

            onClicked:
            {
                print("登录成功")  // 实际开发中，这里调用C++的函数
            }
        }
    }
}
```


