---
layout: post
title: 读后点评《一个你控制的api网关服务》
category: Review 
description: 原文《A Node.js API Gateway that you control》本文介绍了作为微服务架构中的重要一环，网关服务。
keywords: Node.js API Gateway
---
&#160; &#160; &#160; &#160; 原文 《[A Node.js API Gateway that you control](https://medium.com/sharenowtech/k-fastify-gateway-a-node-js-api-gateway-that-you-control-e7388c229b21)》。
本文介绍了作为微服务架构中的重要一环，网关服务。

<!--description-->   
## 一、什么是API网关服务
&#160; &#160; &#160; &#160; 大多数微服务的应用都会提供一个API网关服务作为服务的入口。API网关的职责包括提供路由、协议转换、组合请求等。它给客户端提供API服务，当后台服务宕机时可以返回缓存的结果或者默认结果。
   
## 二、API网关服务的职责
1. 整合内部服务、分发微服务应用的请求。通常会减少配置需求。
2. 在网关级别处理非功能性需求和跨域问题，使服务功能的实现尽可能的简单、轻量。这层服务常常提供的功能：
    1. SSL终端
    2. 日志服务
    3. 服务指标数据
    4. 均衡负载
    5. 授权
    6. 内容整合
    7. 缓存
    8. 内容压缩
    9. 请求频率限制
    10. 其它...
3. 简化客户端和微服务的交互，聚合的请求减少了网络传输的次数。
  
## 三、Node.js开原生态
&#160; &#160; &#160; &#160; node.js的npm开源生态很活跃，有许多优秀的REST的开源服务框架。但是作为API网关服务的框架就只有Express Gateway、Moleculer API Gateway。
## 四、介绍k-fastify-gateway
&#160; &#160; &#160; &#160; 作者发布了一款基于fastify网关服务框架 [k-fastify-gateway](https://www.npmjs.com/package/k-fastify-gateway) 有兴趣的可以了解一下。

