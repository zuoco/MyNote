---
title: "Ollama部署"
description: "Ollama部署以及基础操作" 
date: 2025-05-01
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - AI
---


- [1. Ollama安装](#1-ollama安装)
- [2. Ollama基础命令](#2-ollama基础命令)


# 1. Ollama安装
登陆ollama官网，官网直接给出了一行命令，该命令可以自动下载并安装ollama：  
```bash
curl -fsSL https://ollama.com/install.sh | sh
```
该命令会下载一个脚本，该脚本使用curl命令下载ollama安装程序，并执行安装程序，但是这个curl始终不能连接网络，所以只能手动下载安装文件：       


**1. 第一步**： 下载安装脚本     
```bash
curl -fsSL https://ollama.com/install.sh -o ollama_install.sh # 下载安装脚本
```

**2. 第二步**： 在这个安装脚本中查找“下载链接”    
打开下载下来的脚本，搜索`https://ollama.com/download/ollama-linux-`，找到类似如下代码：  
```bash
status "Downloading Linux ${ARCH} bundle"
curl --fail --show-error --location --progress-bar \
    "https://ollama.com/download/ollama-linux-${ARCH}.tgz${VER_PARAM}" | \
    $SUDO tar -xzf - -C "$OLLAMA_INSTALL_DIR"
```
这段代码的功能就是下载ollama安装文件，从代码来看，安装文件会被解压到`$OLLAMA_INSTALL_DIR`目录下，其实就是`usr/local/`目录，我门先不管这些，在这段脚本前添加`echo "https://ollama.com/download/ollama-linux-${ARCH}.tgz${VER_PARAM}"`命令，然后运行脚本，打印出来的链接就是了。  


**3. 下载安装文件**       
复制前面打印出来的链接，直接到浏览器中下载，并将下载好的压缩包保存到和安装脚本相同目录下。    
```bash
zcli@fedora:~$ ls -lh | grep ollama
-rwxrwxrwx. 1 zcli zcli  13K  5月18日 21:11 ollama_install.sh      # 安装脚本
-rw-r--r--. 1 zcli zcli 1.6G  5月18日 21:04 ollama-linux-amd64.tgz # 安装文件
```


**4. 修改安装脚本**     
定位到第二步中给出的代码：   
```bash
status "Downloading Linux ${ARCH} bundle"
curl --fail --show-error --location --progress-bar \
    "https://ollama.com/download/ollama-linux-${ARCH}.tgz${VER_PARAM}" | \
    $SUDO tar -xzf - -C "$OLLAMA_INSTALL_DIR"
```
修改为：    
```bash
# 也就是不curl了，直接将已经下载好的安装文件解压到指定目录。
status "Downloading Linux ${ARCH} bundle"
$SUDO tar -xzf ollama-linux-amd64.tgz -C "$OLLAMA_INSTALL_DIR" # 解压到指定目录
```


**5. 执行安装脚本**     
```bash
zcli@fedora:~$ ./ollama_install.sh 
>>> Cleaning up old version at /usr/local/lib/ollama
[sudo] zcli 的密码：
>>> Installing ollama to /usr/local
>>> Downloading Linux amd64 bundle 
>>> Creating ollama user...
>>> Adding ollama user to render group...
>>> Adding ollama user to video group...
>>> Adding current user to ollama group...
>>> Creating ollama systemd service...
>>> Enabling and starting ollama service...
Created symlink '/etc/systemd/system/default.target.wants/ollama.service' → '/etc/systemd/system/ollama.service'.
>>> NVIDIA GPU installed.
```
OK，安装好了，在Linux上，这个ollama安装为一个Linux服务，并且安装后就自动运行起来了，如下：   
```bash
zcli@fedora:~$ sudo systemctl status ollama.service 
● ollama.service - Ollama Service
     Loaded: loaded (/etc/systemd/system/ollama.service; enabled; preset: disabled)
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf, 50-keep-warm.conf
     Active: active (running) since Sun 2025-05-18 23:03:56 CST; 1min 31s ago
 Invocation: 2dbc0a03db1c4a08a17829fa4039bf63
   Main PID: 43831 (ollama)
      Tasks: 13 (limit: 37474)
     Memory: 24.5M (peak: 40.9M)
        CPU: 248ms
     CGroup: /system.slice/ollama.service
             └─43831 /usr/local/bin/ollama serve
```



# 2. Ollama基础命令

1. 下载模型
```bash
# 但是ollama默认下载的可能是量化版本，且国内访问会很慢，可以从modelscope下载  
ollama pull <模型名>:<版本号> modelscope.cn/Qwen/<模型名称>:<版本号> # 国内下载
```

2. 运行模型
```bash
ollama run <模型名称>:<版本号>  
```

3. 查看推理过程
```bash
ollama run <模型名> --verbose   # 显示推理过程的耗时情况
```

4. 列出所有模型 
```bash
ollama list  
```

5. 查看模型参数信息
```bash
ollama show <模型名称>   
```

6. 删除模型
```bash
ollama rm <模型名称>   
```


7. 查看正在运行的模型
```bash
ollama ps   
```


8. 停止运行中的模型
```bash
ollama stop <模型名称>   
```
