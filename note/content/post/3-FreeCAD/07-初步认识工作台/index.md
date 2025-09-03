---
title: "FreeCAD（07）— 工作台"
description: 
date: 2025-08-11
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - FreeCAD
---


Workbench是FreeCAD模块提供的工作台实现，工作台就是一组功能的集合，例如“零件建模工作台”、“CAM工作台”、“装配体工作台”。它定义了哪些GUI元素（如工具栏、菜单）会被添加到主窗口中，以及哪些会被移除或隐藏。当一个workbench对象首次被激活时，它所代表的模块会被加载到RAM中。    


- [1. 工作台基类 — Gui::Workbench](#1-工作台基类--guiworkbench)
- [2. 工作台类](#2-工作台类)
- [3. 零件设计工作台 — PartDesignWorkbench](#3-零件设计工作台--partdesignworkbench)


##  1. 工作台基类 — Gui::Workbench
```cpp
namespace Gui {

// GUI元素体系，支持4类界面元素，实际上还有一个命令栏（CommandBars）
class MenuItem;        // 菜单栏
class ToolBarItem;     // 工具栏
class DockWindowItems; // 停靠窗口

class WorkbenchManager;

class GuiExport Workbench : public Base::BaseClass
{
    // ...

    // 核心生命周期管理
    /**
     * Activates the workbench and adds/removes GUI elements.
     */
    bool activate();
    /** Run some actions when the workbench gets activated. */
    virtual void activated();
    /** Run some actions when the workbench gets deactivated. */
    virtual void deactivated();

    // Python绑定支持
    PyObject* getPyObject() override;

    // ...

protected:
    // 设置各种小窗口
    /** Returns a MenuItem tree structure of menus for this workbench. */
    virtual MenuItem* setupMenuBar() const=0;                                        // 设置菜单栏
    /** Returns a ToolBarItem tree structure of toolbars for this workbench. */      
    virtual ToolBarItem* setupToolBars() const=0;                                    // 设置工具栏
    /** Returns a ToolBarItem tree structure of command bars for this workbench. */
    virtual ToolBarItem* setupCommandBars() const=0;                                 // 设置命令栏
    /** Returns a DockWindowItems structure of dock windows this workbench. */
    virtual DockWindowItems* setupDockWindows() const=0;                             // 设置停靠窗口

    // ... 
};
}
```
管理不同模块的GUI元素展示与激活逻辑。

##  2. 工作台类
FreeCAD提供了几种不同类型的workbench基类，最终的工作台继承自这些基类：
|基类型|  功能  |派生类型|
|------------|------------|------------|
|StdWorkbench| 标准工作台类，定义了标准的菜单、工具栏等元素 |
|BlankWorkbench| 完全空白的工作台 | 
|PythonBaseWorkbench| 支持Python操作的工作台 |
|其他|...|


以StdWorkbench为例，看看都重写了什么：  
```cpp
class GuiExport StdWorkbench : public Workbench
{
    TYPESYSTEM_HEADER_WITH_OVERRIDE();

public:
    StdWorkbench();
    ~StdWorkbench() override;

public:
    // 上下文菜单，可能是右键菜单
    /** Defines the standard context menu. */
    void setupContextMenu(const char* recipient, MenuItem*) const override;
    void createMainWindowPopupMenu(MenuItem*) const override;

protected:
    // 设置工作台的各个组件
    /** Defines the standard menus. */
    MenuItem* setupMenuBar() const override;
    /** Defines the standard toolbars. */
    ToolBarItem* setupToolBars() const override;
    /** Defines the standard command bars. */
    ToolBarItem* setupCommandBars() const override;
    /** Returns a DockWindowItems structure of dock windows this workbench. */
    DockWindowItems* setupDockWindows() const override;

    friend class PythonWorkbench;
};
```


## 3. 零件设计工作台 — PartDesignWorkbench
零件设计工作台提供参数化零件设计功能，分为App(核心逻辑)和Gui(界面)两个部分。       
源码目录如下：   
```bash
./Mod/PartDesign/
├── __init__.py
├── CMakeLists.txt
├── App/           # 核心逻辑实现，包含特征类（Extrude/Revolution/Pipe等）
├── Gui/           # 用户界面组件，包含参数面板（TaskXXXParameters）和视图提供者（ViewProvider）
├
├── fcgear/        # 用于生成渐开线齿轮的FreeCAD插件（通过贝塞尔曲线近似渐开线齿廓，并支持生成外部齿轮 和 内部齿轮）。
├── fcsprocket/    # 用于生成链齿轮的FreeCAD插件。
├── WizardShaft/   # 轴类零件，包括参数化建模（ShaftFeature）、力学计算（Shaft类）和可视化分析（Diagram类）
├── Resources/     # 提供符合国标的孔特征参数数据库，包含多个JSON文件，每一个代表一个标准，包含沉头孔、埋头孔、标准螺纹孔等类型。
├
├── Scripts/       # 建模功能扩展，每个文件实现特定的机械设计功能
├
├── Init.py
├── InitGui.py     # 零件设计工作台
├
├── InvoluteGearFeature.py    # 渐开线齿轮的参数化建模功能，基于fcgear中的Python模块
├── InvoluteGearFeature.ui    # 渐开线齿轮参数设计界面
├
├── SprocketFeature.py        # 链轮齿的参数化建模功能，基于fcsprocket中的Python模块
├── SprocketFeature.ui        # 链轮齿参数设计界面
├
├── PartDesign_Model.xml      # 这是Python扩展代码生成机制的配置文件
└── PartDesignGlobal.h        # 全局头文件 
```
工作台当然是基于FreeCAD、FreeCADGui模块，这两个模块是通过C++实现的Python扩展，是FreeCAD的公共基础，而对于每一个模块，模块本身的功能是使用Python实现的。   
在PartDesignGui命名空间下，定义的零件工作台类PartDesignGui::Workbench，实现核心功能，
```cpp
// Mod/PartDesign/Gui/Workbench.h
namespace PartDesignGui {

class PartDesignGuiExport Workbench : public Gui::StdWorkbench
{
    // ...
};

} 
```

Mod/PartDesign/InitGui.py中定义了PartDesignWorkbench类型，该类继承了 PartDesignGui::Workbench。当GUI运行起来时，会运行这个InitGui.py，从而加载零件工作台，包括PartDesignGui和PartDesign。       
```py
# Mod/PartDesign/InitGui.py
class PartDesignWorkbench ( Workbench ):   # 继承 PartDesignGui::Workbench
    "PartDesign workbench object"
    def __init__(self):
        self.__class__.Icon = FreeCAD.getResourceDir() + "Mod/PartDesign/Resources/icons/PartDesignWorkbench.svg"
        self.__class__.MenuText = "Part Design"
        self.__class__.ToolTip = "Part Design workbench"

    def Initialize(self):
        # load the module
        try:
            import traceback
            from PartDesign.WizardShaft import WizardShaft        # 加载 shaftwizard 模块
        except RuntimeError:
            print ("{}".format(traceback.format_exc()))
        except ImportError:
            print("Wizard shaft module cannot be loaded")
            try:
                from FeatureHole import HoleGui
            except Exception:
                pass

        import PartDesignGui
        import PartDesign
        try:
            from PartDesign import InvoluteGearFeature    # 加载 InvoluteGearFeature 模块
            from PartDesign import SprocketFeature        # 加载 SprocketFeature 模块
        except ImportError:
            print("Involute gear module cannot be loaded")
            #try:
            #    from FeatureHole import HoleGui
            #except:
            #    pass

    def GetClassName(self):
        return "PartDesignGui::Workbench"

Gui.addWorkbench(PartDesignWorkbench())
```

