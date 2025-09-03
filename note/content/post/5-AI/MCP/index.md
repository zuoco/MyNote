---
title: "Function Calling 与 MCP协议"
description: 
date: 2025-03-08
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - AI
---

1. [1. Function Calling](#1-function-calling)
    1. [1.1. 核心流程](#11-核心流程)
    2. [1.2. 问题](#12-问题)
2. [2. MCP](#2-mcp)
    1. [2.1. 核心组件与架构](#21-核心组件与架构)
3. [3. MCP与Functional Calling的关系](#3-mcp与functional-calling的关系)



# 1. Function Calling
&emsp;&emsp;大模型的 Function Calling（函数调用） 是一种让大语言模型与外部工具或 API 交互的核心技术。它使模型能够根据用户请求，智能地生成`结构化`调用指令（如 JSON 格式），触发开发者预定义的函数，从而突破纯文本生成的限制，执行实际任务（如查天气、发邮件、查数据库等）。  


## 1.1. 核心流程

**1. `用户提问`**  
用户提出需要外部能力的请求，例如： “今天北京的温度是多少？”   

**2. `模型决策`**   
模型分析请求，判断是否需要调用函数（例如天气查询 API），并选择匹配的函数（如 get_current_weather）。   

**3. `生成结构化请求`**   
模型输出标准化的函数调用参数（非自然语言），例如：  
```json
{
  "function": "get_current_weather",
  "arguments": {"location": "北京", "unit": "celsius"}
}
```
**4. `开发者代码执行函数`**    
开发者代码解析该请求，调用真实 API 获取数据：    
```python
weather_data = get_current_weather("北京", "celsius")  # 返回：{"temp": 25, "condition": "晴"}
```

**5. `将结果返回给模型`**   
将 API 返回的数据重新传给模型：   
```json
{"temp": 25, "condition": "晴"}
```

**6. `生成最终回答`**   
模型将原始数据转化为自然语言回复：&emsp; “北京今天气温 25 摄氏度，天气晴朗。”

## 1.2. 问题
&emsp;&emsp;业务（用户提问）和平台接口调用是一体的，客户端直连平台API，需要管理函数的声明，参数传递，结果分析。而且每一次提问，就会进行一次Function Calling。不同的Calling之间是隔离的。      


# 2. MCP
&emsp;&emsp;MCP（Model Context Protocol）是由 Anthropic 推出的开源协议，旨在实现大语言模型（LLM）与外部数据源和工具的高效集成。这是一种通用接口协议，类似于 OpenAPI，但专为 AI 模型设计。它通过标准化通信协议、数据格式和规则，让 LLM 能够安全、灵活地连接本地和远程资源（如数据库、API、文件系统等）。    

## 2.1. 核心组件与架构
MCP 采用 **`客户端`** - **`服务器`** 架构，主要组件包括：   
- **MCP Hosts**   
角色：受控本地资源的入口层，发起上下文请求。   
示例：Claude Desktop、AI 开发 IDE。   
- **MCP Clients**   
角色：协议转换层，维护与服务端的持久连接。
示例：语言模型接口适配器。
- **MCP Servers**   
角色：封装数据/工具能力，提供标准化接口。   
示例：文档解析服务、API 网关服务。   
- **Local Data**   
角色：受控本地资源（如企业知识库、私有数据库）。   
- **Remote Services**
角色：云端扩展能力（如搜索引擎、支付接口）。   

# 3. MCP与Functional Calling的关系
&emsp;&emsp;可以简单理解为，MCP是对于Functional Programming的封装，业务层不在需要直接Function Calling，这些都由MCP Servers完成。业务层只需要对接标准的MCP协议接口即可，而且以此业务请求也可以触发多次Function Calling，并将Calling结果整合输出给用户。   