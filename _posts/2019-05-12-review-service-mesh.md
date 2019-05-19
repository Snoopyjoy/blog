---
layout: post
title: 读后点评《微服务的Service Mesh》
category: Review 
description: 原文 《Service Mesh for Microservices》文章对微服务中Service Mesh作出了详细介绍。
keywords: service mesh
---

原文 《[Service Mesh for Microservices](https://medium.com/microservices-in-practice/service-mesh-for-microservices-2953109a3c9a)》文章对微服务中Service Mesh作出了详细介绍。

## 为什么使用Service Mesh
当我们面对这样一个场景，我们需要调用多个下游的服务来组成一个新服务。对于ESB架构，我们需要用到ESC的熔断器、超时处理、服务发现等功能去实现。而使用微服务时就不需要那个集成虚拟服务的ESB层了，取而代之的是一些系列的微服务。  
可以互相通信的微服务由业务逻辑和网络功能组成。实现微服务架构的难点并不是服务本身的业务逻辑，如何实现微服务之间的通信才是最难的。
## 什么是Service Mesh
简而言之，微服务就是服务内部通信的基础设施。  
### Service Mesh的特性
1. 一个特定的微服务不会直接与其他微服务直接通信。
2. 取代所有服务对服务的通信的是组件顶部的service mesh(或称为边车代理)
3. service mesh 提供内置的网络功能如弹性伸缩、服务发现等。
4. 因此服务的开发者可以更专注于业务逻辑开发而大多数和网络通信相关的工作只需要交给service mesh。
5. 例如你在请求其它服务时不需要担心熔断处理，因为这已经是service mesh的一部分。
6. service mesh可以使用任意语言。由于微服务和service mesh的通信通常是基于标准通信协议如http1.x/2.x, gRPC等等。你可以使用其它任何技术来构建微服务，且仍然可以与service mesh通信。 
![]({{site.baseurl}}/assets/img/service-mesh.png)  
  
### 业务逻辑
### 原生网络
### 应用网络功能
### Service Mesh控制面板
服务注册，服务发现  
## Service Mesh的功能  
### 内部通信的弹性功能  
熔断处理，重试，超时处理，故障处理，均衡负载，故障转移。
### 服务发现
### 路由
### 监控
### 安全
### 连接控制
### 部署
### 通信协议
## Service Mesh的实现
[Linkerd](https://linkerd.io/) 和 [Istio](https://istio.io/) 
## Service Mesh的优缺点
### 优点
1. 特性代码在微服务代码外，有高复用性
2. 解决了微服务架构中的大多数问题：服务监控追踪，日志服务，安全性，连接限制
3. 微服务的实现语言选择更加自由，只需要支持微服务间的通信协议
  
### 缺点
1. 复杂
2. 服务通信走额外跳板
3. 仍有其它需要解决的问题
4. 不成熟
  
## 结论
Service Mesh 解决了微服务中的大部分问题，给了我们可以选择多个实现语言的自由，但是并不能解决业务相关和服务整合相关的问题。