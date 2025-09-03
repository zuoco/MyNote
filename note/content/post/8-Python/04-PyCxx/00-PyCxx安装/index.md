---
title: "Python的C++扩展开发（00）—  PyCxx安装"
description: 
date: 2025-08-15
image: 
math: 
license: 
hidden: false
comments: true
draft: true
---




**下载：**    

PyCxx源码地址： https://sourceforge.net/projects/cxx/files/CXX/  

**安装：**  
```bash
sudo python3 setup.py install
```

**编译示例程序：**
```bash
python3 setup_makefile.py linux linux.mak
make -f linux.mak clean test
``` 

带调试的编译：  
```bash
python3 setup_makefile.py linux linux.mak --pycxx-debug
```


**编译Demo/Python3/下的模块**
cd 到 Demo/Python3/目录下

1. 构建Python模块
```bash
python3 setup.py build
```

2. 安装模块
```bash
sudo python3 setup.py install 
```



