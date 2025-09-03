---
title: "Python编成中的 “__init__.py” 文件是干什么的？"
description: 
date: 2024-08-12
image: 
math: 
license: 
hidden: false
comments: true
draft: true
categories:
    - Python
---


当执行`import XXX`时，这个文件就会被执行，在 Python 3.3 之前，“\_\_init\_\_.py” 是 必需的，用于标识一个目录为 Python 包。Python 解释器通过检查该文件的存在来确认目录是否是一个包，从而允许通过 import 导入其中的模块或子包。后来到了Python 3.3+，引入了隐式命名空间包，即使没有“\_\_init\_\_.py”，目录也可以被当作包使用，但还是推荐显式的添加“\_\_init\_\_.py”。   


**这个文件的执行顺序：**  
```bash
pypackage/   
├── __init__.py   
├── mod01.py             
└── subpy/   
    ├── __init__.py   
    └── mod02.py   
```
当我们执行import pypackage.subpy.mod02时，会先执行外层的“\_\_init\_\_.py”（pypackage/），再执行内层的“\_\_init\_\_.py”（pypackage/subpy/），最后才会导入mod02.py。  


**这个文件的功能：**   
1. 包的初始化代码：设置全局变量或常量，加载配置文件等等。   
2. 控制导入行为： 定义 \_\_all\_\_，通过 __all__ 变量，可以控制 from package import * 时导入的内容，避免意外暴露内部模块。  
3. 简化导入路径：在 \_\_init\_\_.py 中提前导入子模块的函数或类，可以让用户通过更简洁的路径访问它们。  
4. 定义包级别的变量和函数: 可以在 __init__.py 中定义包级别的变量、函数或类，供包内所有模块共享。  





