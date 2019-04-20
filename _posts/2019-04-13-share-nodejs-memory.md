---
layout: post
title: 谈谈Node.js的内存泄露
category: Share 
description: Node.js是基于Chrome V8引擎的。当JS在前端运行时内存管理相对没有那么重要，浏览器有足够的内存包容一些小的内存泄露问题，但是在服务端运行时是绝对不可以有内存泄露的。在有内存泄露时，浏览器刷新一下就可以重启程序了，但是在服务端如果没有监控报警，即使代码里有很小的内存泄露，在长时间的运行后也会导致程序响应越来越慢，最终会内存耗尽，服务没响应，进程崩溃。  
keywords: nodejs,nodejs memory
---
Node.js是基于Chrome V8引擎的。当JS在前端运行时内存管理相对没有那么重要，浏览器有足够的内存包容一些小的内存泄露问题，但是在服务端运行时是绝对不可以有内存泄露的。在有内存泄露时，浏览器刷新一下就可以重启程序了，但是在服务端如果没有监控报警，即使代码里有很小的内存泄露，在长时间的运行后也会导致程序响应越来越慢，最终会内存耗尽，服务没响应，进程崩溃。  
  
尽管有各种守护进程的工具，pm2,forever或者是kubernetes+docker，在进程彻底崩溃后会自动重启，但是在进程退出之前随着程序占用的内存越来越多，响应会越来越慢，严重影响了服务质量。这是为什么呢？  
  
首先我们来看看Node.js的底层事件驱动模型。  
![]({{site.baseurl}}/assets/img/nodejseventloop.png)     
当在一个事件循环的过程中需要处理各种回调，如果这是CPU处理过慢那么这个循环就会被阻塞直到这个回调处理结束。那么在内存占用越来越大时，Node.js执行垃圾回收的时间也就越长，从而导致响应过慢，这也是Node.js不胜任CPU密集型任务的原因。  

 为了保证服务的高并发性能和高可靠性，我们必须解决内存泄露的问题，那么如果在代码中很难发现内存泄露的代码块怎么办呢？这时我们就需要分析运行时的内存了。我们先来看看Node.js进程堆内内存的组成部分。
 1. New Space 新生代内存空间，对半分割，只是用其中一半，大部分对象创建和销毁的地方。
 2. Old Space 老生代内存空间，经过两个New Space的垃圾回收扫描后依然存活的对象。
 3. Code Space 代码空间，V8编译后的可执行代码。
 4. Map Space Object指向的隐藏类元对象。
 5. Large Object Space 大于507136字节的对象。
 
 通常的内存泄露是由于在V8的垃圾回收过程中不断有新对象一直存活，New Space不断被加入Old Space，而在Old Space中也同样得不到释放，最终导致Old Space超过最大内存限制，进程退出。在Node.js默认配置下，新生代空间的大小限制为32位系统16m、64位系统32m，老生代空间大小限制为32位系统0.7G、64位系统1.4G。这两个配置可以分别通过max-semi-space-size和max_old_space_size参数来改变。
 
 V8引擎的垃圾回收机制是怎样的呢？在New Space中垃圾回收采用Scavenge算法，这种算法采用宽度优先在From内存堆中从根节点开始遍历内存，将所有遍历到的内存复制到另一块同样大小的To堆中，遍历完毕From堆中的引用链断掉的对象就不会被复制到To堆中。这就是New Space需要对半分割的原因，这是一种以空间换时间的高效算法，其中一半内存的大小就是max-semi-space-size中配置的大小。  
   
 如果New Space中的对象经过两次垃圾回收依旧存活，就会从新生代晋升到老生代。在Old Space老生代中垃圾回收采用的是Mark-Sweep和Mark-Compact算法，这两个算法是用的一种三色标记的方式。没有扫描过的对象标记为白色，扫描过但是引用它的自己子节点没有被扫描过的对象标记为灰色，放入到一个标记栈中，被扫描过并且它的子节点也扫描过后标记为黑色，从标记栈中pop出来。其中Mar-Sweep算法采用的是标记清除，标记活的对象，之后清除没标记的对象。在老生代中主要采用Mar-Sweep算法，当空间不足时才会采用Mark-Compact算法，剩余栈空间不足时，先pop出子节点已扫描过的对象，以增量的形式继续扫描剩余的对象。Mark-Compact算法下，扫描是很频繁的，很耗性能。当堆达到一定大小时，会采用增量标记（incremental_marking）的形式降低老生代的全堆垃圾回收带来的时间停顿。它的原理是从标记阶段入手，拆分为许多小步进，与应用逻辑交替运行。  
   
 当我们在启动进程时加上--gc-trace参数，gc时消耗的时间是可以通过log查看到的。正常情况下New Space的回收时间应该在10ms以内，如果超过50ms那耗时就过长了，我们可以通过调整max-semi-space-size的大小减少触发gc的频率。  
 
 在上文中我们可以看出要定位内存泄露问题就是要看对象的引用关系。那么如何拿到内存的引用关系呢？Node.js有
 1. heapdump库
 2. v8-profiler库
 3. 阿里云Node.js性能平台  
 
 通过上面的方式可以拿到内存快照一个Heapsnapshot数据。然后我们可以用Chrome浏览器自带的开发者工具中导入Heapsnapshot数据，查看数据的引用关系。  
   
 造成内存泄露的原因有很多，大多数还是我们自己项目代码的问题，因此在我们写代码时严谨的态度对待。注意：
 1. 对象销毁时事件的监听是否移除
 2. 在不需要时Timer是否停止
 3. 对象初始化函数里是否有重复的对象生成
 
  
参考:  
唯块不破：高效定位线上Node.js应用内存泄露by黄一君  
[https://www.jianshu.com/p/8290715feec6](https://www.jianshu.com/p/8290715feec6)  
[https://www.jb51.net/article/81262.htm](https://www.jb51.net/article/81262.htm)

