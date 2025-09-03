---
title: "FreeCAD（03） — 类型系统与根类型（Base模块）"
description: 
date: 2025-06-02
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - FreeCAD
---


- [1. **类型系统**](#1-类型系统)
- [2. **根类型 — BaseClass**](#2-根类型--baseclass)
- [3. **关键宏**](#3-关键宏)



# 1. **类型系统**
src/base/type.h文件中提供了类型信息、类型注册的功能，通过类名来创建对象，看起来是个工厂。   
```cpp
namespace Base
{

struct TypeData;


class BaseExport Type
{
public:
    Type(const Type& type) = default;
    Type(Type&& type) = default;
    Type() = default;
    ~Type() = default;

    // 一堆成员函数 ...
    void* createInstance();   // 创建对象，根据index查找typedata，使用typedata中的函数创建对象。
    static void* createInstanceByName(const char* TypeName, bool bLoadModule = false); // 根据提供的typename创建对象，内部调用了createInstance()

    // 创建一个类型，需要传入父类型
    static Type
    createType(const Type& parent, const char* name, instantiationMethod method = nullptr);


private:
    unsigned int index {0};  // typedata的索引

    static std::map<std::string, unsigned int> typemap;   // <类名称, 类型值>
    static std::vector<TypeData*> typedata;               // typedata 列表
    static std::set<std::string> loadModuleSet;           // 模块集合
};

}
```

TypeData类型： 
```cpp
struct Base::TypeData
{
    TypeData(const char* theName,
             const Type type = Type::badType(),
             const Type theParent = Type::badType(),
             Type::instantiationMethod method = nullptr)
        : name(theName)
        , parent(theParent)
        , type(type)
        , instMethod(method)
    {}

    std::string name;                      // 类型名称
    Type parent;                           // 父类的类型
    Type type;                             // 类型值，对于与name
    Type::instantiationMethod instMethod;  // 一个函数，用于创建对象
};
```

# 2. **根类型 — BaseClass**
src/Base/BaseClass.h
```cpp
namespace Base
{

namespace Base
{

class BaseExport BaseClass
{
public: 

    /* 获取类型的Type值 */
    static Type getClassTypeId();   
    virtual Type getTypeId() const; // 获取子类自己的类型Type，用于多态场景下
    
    /* 是否派生于某个类型 */
    bool isDerivedFrom(const Type type) const
    {
        return getTypeId().isDerivedFrom(type);
    }

    template<typename T>
    bool isDerivedFrom() const
    {
        return getTypeId().isDerivedFrom(T::getClassTypeId());
    }

    /* 注册根类型到类型系统中，里面调用了Type::createType（） */
    static void init();  

    /* 与Python的交互接口，用于支持脚本操作 */
    virtual PyObject* getPyObject(); 
    virtual void setPyObject(PyObject*); 

    /* 创建对象，根类型中，该函数返回空 */
    static void* create()
    {
        return nullptr;
    }
  
    /* 当前对象是否为T类型 */
    template<typename T>
    bool is() const
    {
        return getTypeId() == T::getClassTypeId();
    }

private:
    static Type classTypeId;   // getClassTypeId()获取到的就是这个玩意儿

protected:
    /* 注册新类型到类型系统，里面调用了Type::createType（） */
    static void initSubclass(Base::Type& toInit,             // 这是用于保存返回值，就是Type::createType（）的返回值
                             const char* ClassName,
                             const char* ParentName,
                             Type::instantiationMethod method = nullptr);

public:
    BaseClass();
    BaseClass(const BaseClass&) = default;
    BaseClass(BaseClass&&) = default;
    virtual ~BaseClass();
};

}
```

# 3. **关键宏**
src/Base/BaseClass.h   
我们在创建子类型时，需要添加一些宏函数，这些宏简化了子类对 Base::BaseClass 的继承，思路类似于Qt中的QObject宏，自动处理：
- 类型标识（getClassTypeId / getTypeId）
- 动态创建（create 方法）
- 类型注册（通过 initSubclass）

```cpp
// 类型定义时使用
#define TYPESYSTEM_HEADER_WITH_OVERRIDE()                                                          \
public:                                                                                            \
    static Base::Type getClassTypeId(void);                                                        \
    Base::Type getTypeId(void) const override;                                                     \
    static void init(void);                                                                        \
    static void* create(void);                                                                     \
                                                                                                   \
private:                                                                                           \
    static Base::Type classTypeId
```
**用途**：在子类头文件使用该宏来声明类型系统所需的方法和静态成员。   
**关键成员**：   
- getClassTypeId()：返回类的静态类型标识。   
- getTypeId()：虚函数，返回实例的运行时类型标识。   
- init()：初始化类型（注册到 Type 系统）。   
- create()：创建实例的工厂方法。   
- classTypeId：类的静态类型标识（由 Type 类管理）。   

还有一个TYPESYSTEM_HEADER宏，这是老的版本，功能是一样的，只是TYPESYSTEM_HEADER没有使用“override”来显式重写父类虚函数，可能是为了兼容98版本。  

---

上面是定义，当然也提供了用于实现的宏，并且给了多个版本：  
- `TYPESYSTEM_SOURCE_P`: 普通可实例化的类（非模板、非抽象）。   
- `TYPESYSTEM_SOURCE_TEMPLATE_P(_class_)`: 模板类（如 MyTemplateClass<T>）。   
- `TYPESYSTEM_SOURCE_ABSTRACT_P(_class_)`: 抽象类（不可实例化）。  

上面三个宏没有实现initSubclass方法，所以又提供了三个宏(基于上面的宏)，用于实现initSubclass方法：   
- `TYPESYSTEM_SOURCE(_class_, _parentclass_)`     
- `TYPESYSTEM_SOURCE_TEMPLATE_T(_class_, _parentclass_)`    
- `TYPESYSTEM_SOURCE_ABSTRACT(_class_, _parentclass_)`    




