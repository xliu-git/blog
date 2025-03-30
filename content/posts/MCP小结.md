---
date: '2025-03-30T21:25:33+08:00'
draft: false
title: 'MCP小结'
tags: ["MCP", "AI"]
---

### 什么是MCP
MCP是一个标准化协议（protocol），规定了应用和大模型之间的通信格式，比如我们常说的USB-C就是一种协议，同一种协议，不同厂家的具体实现可能有很多种，比如USB2.0 Type-C, USB3.1 Gen1 Type-C, Thunderbolt3/4 Type-C等。
### MCP宏观架构
我根据[MCP introduction的图](https://modelcontextprotocol.io/introduction#general-architecture)，以及官方给出的MCP Server和Client的例子，结合自己的理解画了一张图。
![](./MCP_Architecture.png)
1. Host:一个应用，或者更抽象，应用中的一个connection pool，维护着各种MCP Client的connection instances。
2. MCP Client:我在这里把client理解成一个跟其他通信协议类似的client，比如http client、webSocket client，负责跟具体server的connection。
3. MCP Server:即提供不同服务的服务器，不同的外部Service Provider可以有若干个MCP Server负责容错冗余负载均衡，同时，内部的依赖也可以有自建的MCP Server。
4. Orchestration Service:负责组合各种MCP Client的tool，决定什么时候调用哪个MCP client的哪种tool，并通过返回的response进行下一步的工作。
### MCP Server例子
我最开始对于MCP Server的理解是Service Provider专门针对MCP协议所部属的server，那么所有的MCP Server的开发工作应该属于Service Provider，但查看了一下[github为Claude给出的MCP Server例子](https://github.com/modelcontextprotocol/servers/tree/main/src/github)后发现MCP Server其实更类似于一种本地对于service provider的封装，或者说专门针对MCP协议路由的controller，提供MCP协议要求的服务，至于服务的具体实现，则是通过service provider原有的API call实现的。

以github的MCP server为例：
1. 提供tool列表、描述以及schema：https://github.com/modelcontextprotocol/servers/blob/main/src/github/index.ts#L71
2. 提供tool路由：https://github.com/modelcontextprotocol/servers/blob/main/src/github/index.ts#L208
3. 运行tool（通过API实现）：https://github.com/modelcontextprotocol/servers/blob/main/src/github/operations/commits.ts#L12
### MCP Client例子



### Reference
1. https://modelcontextprotocol.io/introduction

