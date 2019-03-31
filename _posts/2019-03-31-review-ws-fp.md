---
layout: post
title: 读后点评《驾驭JS面试：什么是函数式编程？》
category: Review 
description: 原文《Master the JavaScript Interview - What is Functional Programming?》在Javascript领域，最近几年函数式编程是一个很火的话题。那么什么是函数式编程呢？
keywords: 函数式编程
---
原文 《[Master the JavaScript Interview: What is Functional Programming?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-functional-programming-7f218c68b3a0)》。
在Javascript领域，最近几年函数式编程是一个很火的话题。那么什么是函数式编程呢？
<!--description-->   
函数式编程是一种编程范式。纯函数是一个重要概念，在编程时使用纯函数可以避免共享的状态、可变的数据和副作用。数据以流的方式经过一系列纯函数的处理，并且应用状态是在流中传递的而不是共享。
使用函数式编程可以是代码更加简洁，更好预测，并且比面向对象范式编程更好测试。  
学习函数式编程的曲线其实蛮陡峭，尤其是思维定势的改变是很难做的。首先理解函数式的基本概念就挺难的。什么函子(Functor)，柯里化(Curry)，函数组合（Composition），Monad等等。最难的还是应用，如何将函数式编程应用到项目中。
现在有不少优秀的函数式编程工具库使得我们构建函数式的项目更加容易：
1. [lodash](https://github.com/lodash/lodash)
2. [ramda](https://github.com/ramda/ramda)
3. [immutable-js](https://github.com/immutable-js/immutable-js)
4. [RxJS](https://github.com/Reactive-Extensions/RxJS)  
  
关于入门这里推荐一本书[《JS函数式编程指南》](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/)  
  
当然函数式编程不是一万金油，我发现在JS里执行效率是它的一个缺点。如果在比较耗性能的计算里使用函数式编程就需要权衡一下是否适用了。