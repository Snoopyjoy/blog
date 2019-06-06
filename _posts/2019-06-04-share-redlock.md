---
layout: post
title:  "重构Redis分布式锁"
categories: Share
keywords: redlock,redis,nodejs,分布式锁
description: 前几天看到了左耳听风ARTS微信群中小伙伴分享两篇文章《基于Redis的分布式锁是否安全的讨论（上）》《基于Redis的分布式锁是否安全的讨论（下）》，读完之后受益匪浅，对分布式锁的理解更深入了一些。抛开文中的一些争议点，结合自己的项目，我觉得有必要重构一下目前项目中的锁。  
---
## 前言  
&#160; &#160; &#160; &#160; 前几天看到了左耳听风ARTS微信群中小伙伴分享两篇文章[《基于Redis的分布式锁是否安全的讨论（上）》](http://lmwjohn.cn/2019/05/28/redis-txn-2/)[《基于Redis的分布式锁是否安全的讨论（下）》](http://lmwjohn.cn/2019/05/28/redis-txn-3/)，读完之后受益匪浅，对分布式锁的理解更深入了一些。抛开文中的一些争议点，结合自己的项目，我觉得有必要重构一下目前项目中的锁。  
  
## 当前设计缺陷
&#160; &#160; &#160; &#160; 由于目前的业务场景中出现资源竞争的情况并不是很频繁，所以在项目里用到的锁设计比较简陋。目前调用过程是这样的。
```code
1. A请求访问资源
2. A请求调用redis的SETNX命令获取锁
3. A请求调用EXPIRE命令设置锁过期时间
4. B请求访问资源
5. B请求调用redis的SETNX命令获取锁失败，直接返回错误码
6. A请求进行资源处理
7. A请求释放锁
8. B请求访问资源
...
```
#### 1) 获取锁的原子性操作问题
&#160; &#160; &#160; &#160; 这个实现正中了《基于Redis的分布式锁是否安全的讨论（上）》文章里提到的问题，获取锁的操作被分为了两步：  
```code
1. A请求调用redis的SETNX命令获取锁
2. A请求调用EXPIRE命令设置锁过期时间
```  

&#160; &#160; &#160; &#160; 这两个命令并不是原子性的操作，如果客户端在SETNX命令后崩溃，那么之后的EXPIRE命令就不会执行，此时这个资源就永远得不到释放了。
#### 2) 释放锁的权限问题
&#160; &#160; &#160; &#160; 获取锁时没有给客户端一个锁的唯一ID。这会导致B请求的锁被A请求释放。流程如下：
```code
1. A请求获取锁成功
2. A请求资源处理消耗时长超过锁过期时间
3. B请求获取锁成功
4. B请求处理资源中
5. A请求资源处理完毕，释放锁（导致B请求锁被释放）
6. C请求获取锁成功
7. C请求资源处理和B请求处理冲突
```
#### 3) redis故障发生时的备案
&#160; &#160; &#160; &#160; 当Redis故障时锁失效
#### 4) 自动重试机制问题
&#160; &#160; &#160; &#160; 没有自动重试机制，访问加锁资源时会直接挡住请求  
  
## 解决方案  

#### 1) 获取锁的原子性操作问题
&#160; &#160; &#160; &#160; 为了保证取锁操作的原子性，不可以使用SETNX再调用EXPIRE这种分成两步调用的形式，应该使用命令：
```code
SET resource_name my_random_value NX PX 30000
```
&#160; &#160; &#160; &#160; 这里保证了取锁操作的原子性。参考redlock里的实现，使用的是lua脚本：
```code
return redis.call("set", KEYS[1], ARGV[1], "NX", "PX", ARGV[2])
```
&#160; &#160; &#160; &#160; 这里因为只有一条命令，所以用lua脚本和直接调用redis命令的效果是一样的。  
#### 2) 释放锁的权限问题
&#160; &#160; &#160; &#160; 为了保证锁只可以被获得锁的客户端释放，在获取锁的时候，我们需要给锁加一个唯一标识。关于唯一标识的生成在当前项目环境中有如下方案：
1. 采用node-uuid库算法，生成随机uuid。
2. 在redis中记录一个自增键值，通过组合使用 INCR 和 EXPIRE 以保证一段时间内计数器的值唯一。INCR和EXPIR操作的原子性需要用lua脚本实现：
```code
local current
current = redis.call("incr",KEYS[1])
if tonumber(current) == 1 then
    redis.call("expire",KEYS[1],1)
end
```
3. 使用mongodb生成唯一键值
这几个方案中随机uuid的效率最高，但是有极小概率会重复。方案3会使锁功能产生对mongodb的依赖。方案2效率一般但是可靠。由于
在记录锁的唯一标识的问题时也带来了锁释放的问题。在释放前需要先对比锁的唯一标识，如果锁持有者的标识和redis中记录一致则可以释放锁。这是两个步骤，所以必须使用lua脚本来保证原子性操作。redlock里的实现：
```code
if redis.call("get", KEYS[1]) == ARGV[1] then return redis.call("del", KEYS[1]) else return 0 end
```
&#160; &#160; &#160; &#160; 关于锁id的实现考虑的项目需求和性能要求，我这边首选随机uuid的方案，生成的uuid在锁有效期内重复的几率是极低的。  
  
#### 3) redis故障发生时的备案
&#160; &#160; &#160; &#160; 在Redis故障时，采用主从架构的redis服务会从slave中重新选举出一个master，由于主从同步的时差性，有可能导致新的master数据和之前不一致。这是redlock的解决方案是用多个独立的redis服务，每次操作所有redis服务都记录，最后采用投票的方式来保证数据的正确性。这个方案虽然可以有效的保证数据的正确性，但是对于我目前的项目来讲代价有点大。目前项目用的时候腾讯云的集群版redis，如果采用redlock方案就需要增加另外两个独立的redis服务。关于腾讯云的集群版redis的架构，横向采用分片节点，垂直方向采用副本集。对于某一个分片节点故障时发生主从切换，此时可能会有数据不一致的情况发生。由于主从之间的数据同步是实时热备的，数据不同步的概率实际情况应该并不会很严重。关于是否采用多个独立节点来保证数据正确性，考虑到当前项目的应用场景和成本，我选择只采用单个redis服务。
#### 4) 自动重试机制问题
&#160; &#160; &#160; &#160; 在我之前的锁版本的实现方案中是有重试机制的。但是重试机制带来的问题就是服务中存在大量的timer，数据处理的上下文会被timer回调保持引用，在QPS高的情况下会导致内存暴增，大量timer回调需要处理而导致eventloop时间变长。但是如果不加重试机制则会导致访问一个有锁资源时直接返回错误码，这需要我们写额外的代码来实现重试机制。既然重试机制有这些问题，我们就需要在实现redis的锁服务重试机制时更加小心。  
&#160; &#160; &#160; &#160; 重试间隔的设定需要我们平衡一下，因为如果重试间隔长了会导致间隔时间内大量锁的堆积，内存激增，同时锁的时间长了会降低响应的速度。如果重试间隔短了，那么在短时间内会产生大量的重试回调需要处理，会造成cpu压力，eventloop时间变长。因此我们需要特殊情况特殊对待。
1. 对于已知操作时间会很长的，不使用重试机制。
2. 增加等待列表监控，当等待列表超过设定值时禁止重试。  
  
## 重构
&#160; &#160; &#160; &#160; 思考完这些问题，其实我得出的结论就是我要做一个去掉多个Redis客户端投票验证机制的redlock，区别在于锁id的生成和重试机制。  
所以需要实现的功能点有：
1. 获取锁，锁id唯一，原子性操作
2. 释放锁，根据锁id去释放，原子性操作
3. 获取锁失败重试机制，重试数量值维护，重试过多时停止重试
4. 对已成功获取的锁延时  
  
&#160; &#160; &#160; &#160; 具体实现已经集成到[ecoweb的Redis封装中](https://github.com/Snoopyjoy/ecoweb/blob/master/model/Redis.js)。
