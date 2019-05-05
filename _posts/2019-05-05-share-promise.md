---
layout: post
title: 使用Promise加速你的api响应
category: Share 
description: 在没有Node.js没有Promise之前，异步方法只能通过回调的方式来处理，回调的嵌套导致了代码结构的混乱和代码的可读性变差，在复杂的回调嵌套里代码简直就像毛线球一样。在Node的v7.6版本后官方支持了await/async和Promise这几个重要的ES6特性。await/async和Promise的引入允许我们使用类似同步的写法写出异步的功能，使异步的调用也可以简洁和优雅了。正确的使用await/async和Promise可以使我们的异步调用更加简洁和高效，加快我们服务的响应速度。
keywords: Node.js Promise
---
在没有Node.js没有Promise之前，异步方法只能通过回调的方式来处理，回调的嵌套导致了代码结构的混乱和代码的可读性变差，在复杂的回调嵌套里代码简直就像毛线球一样。在Node的v7.6版本后官方支持了await/async和Promise这几个重要的ES6特性。await/async和Promise的引入允许我们使用类似同步的写法写出异步的功能，使异步的调用也可以简洁和优雅了。正确的使用await/async和Promise可以使我们的异步调用更加简洁和高效，加快我们服务的响应速度。
    
## 回调和Promise的对比
回到只有回调的时代，我们实现异步功能的实现方式是这样的：
```javascript
function asyncMethod1( callback ){
  setTimeout( function(){
    callback(1);
  }, 1000)
}

function asyncMethod2( callback ){
  setTimeout( function(){
    callback(2);
  }, 2000)
}

function asyncMethod3( callback ){
  setTimeout( function(){
    callback(3);
  }, 3000)
}

function test1(callback){
  asyncMethod1( function( result1 ){
    asyncMethod2( function( result2 ){
      asyncMethod3( function( result3 ){
        callback( {result1, result2, result3} );
      } )
    } )
  } )
}
console.time('callback test');
test1( function( {result1, result2, result3} ){
  console.log( 'final result: ', result1, result2, result3 );
  console.timeEnd('callback test');
} );
```
我们的代码像洋葱一样一层一层的嵌套着。在await/async和Promise引入之后代码就可以变成这样：
```javascript
function asyncMethod1(){
  return new Promise( (resolve, reject)=>{
      setTimeout( function(){
        resolve(1);
      }, 1000)
  } );
}

function asyncMethod2(){
  return new Promise( (resolve, reject)=>{
      setTimeout( function(){
        resolve(2);
      }, 2000)
  } );
}

function asyncMethod3(){
  return new Promise( (resolve, reject)=>{
      setTimeout( function(){
        resolve(3);
      }, 3000)
  } );
}

async function test2(){
  const result1 = await asyncMethod1();
  const result2 = await asyncMethod2();
  const result3 = await asyncMethod3();
  return {result1, result2, result3}
}

console.time('async test');
test2().then(({result1, result2, result3})=>{
  console.log( 'final result: ', result1, result2, result3 )
  console.timeEnd('async test');
});
```
## 加速运行——并行的异步调用
以上代码test1和test2中的asyncMethod都是模拟的异步调用第三方服务的情况，如果用以上写法，那么得到最终结果需要6秒。如何提高调用服务的响应速度呢？如果我们并行的调用这几个第三方服务，我们的响应速度理论上就可以达到3秒，那么用传统的回调方法该怎么实现呢？以下是回调的实现。
```javascript
function asyncMethod1( callback ){
  setTimeout( function(){
    callback(1);
  }, 1000)
}

function asyncMethod2( callback ){
  setTimeout( function(){
    callback(2);
  }, 2000)
}

function asyncMethod3( callback ){
  setTimeout( function(){
    callback(3);
  }, 3000)
}

function test3(callback){
  let flag1,flag2,flag3;
  let result1, result2, result3;
  asyncMethod1( function( result ){
    flag1 = true;
    result1 = result;
    if( flag1 && flag2 && flag3 ) callback( {result1, result2, result3} );
  } );
  asyncMethod2( function( result ){
      flag2 = true;
      result2 = result;
      if( flag1 && flag2 && flag3 ) callback( {result1, result2, result3} );
    } );
  asyncMethod3( function( result ){
      flag3 = true;
      result3 = result;
      if( flag1 && flag2 && flag3 ) callback( {result1, result2, result3} );
    } );
  
}
console.time('callback test1');
test3( function( {result1, result2, result3} ){
  console.log( 'final result: ', result1, result2, result3 );
  console.timeEnd('callback test1');
} );
```
上面代码的复杂程度看上去还可以接受，但是如果需要异步调用的方法更多的时候就太复杂了。下面是await/async和Promise的实现：
```javascript
function asyncMethod1(){
  return new Promise( (resolve, reject)=>{
      setTimeout( function(){
        resolve(1);
      }, 1000)
  } );
}

function asyncMethod2(){
  return new Promise( (resolve, reject)=>{
      setTimeout( function(){
        resolve(2);
      }, 2000)
  } );
}

function asyncMethod3(){
  return new Promise( (resolve, reject)=>{
      setTimeout( function(){
        resolve(3);
      }, 3000)
  } );
}

async function test3(){
  [result1, result2, result3] = await Promise.all([
    asyncMethod1(),asyncMethod2(),asyncMethod3()
  ]);
  return {result1, result2, result3}
}

console.time('async test1');
test3().then(({result1, result2, result3})=>{
  console.log( 'final result: ', result1, result2, result3 )
  console.timeEnd('async test1');
});
```
这里引入了Promise.all的用法，允许我们将promise对象数组作为参数传递给Promise.all方法，包装为一个Promise对象。最后Promise对象的返回结果为一个数组，且数组中的数据顺序和传入的promise对象数组的顺序是一一对应的。很显然，async/await+Promise可以是我们用更为简洁的代码实现并行调用异步方法。
## 异常处理
在使用awiat/async时异常处理需要通过try/catch代码块包裹处理。在Promise.all中，如果一个异步调用出错，那么后续代码就不过继续执行了。
```javascript
function asyncMethod1(){
  return new Promise( (resolve, reject)=>{
      setTimeout( function(){
        console.log('method1 handled!')
        resolve(1);
      }, 1000)
  } );
}

function asyncMethod2(){
  return new Promise( (resolve, reject)=>{
      setTimeout( function(){
        console.log('method2 err!')
        reject(new Error('2'));
      }, 2000)
  } );
}

function asyncMethod3(){
  return new Promise( (resolve, reject)=>{
      setTimeout( function(){
        console.log('method3 handled!')
        resolve(3);
      }, 3000)
  } );
}

async function test4(){
  [result1, result2, result3] = await Promise.all([
    asyncMethod1(),asyncMethod2(),asyncMethod3()
  ]);
  return {result1, result2, result3}
}

console.time('async test2');
test4().then(({result1, result2, result3})=>{
  console.log( 'final result: ', result1, result2, result3 )
  console.timeEnd('async test2');
}).catch((err)=>{
  console.error(err);
  console.timeEnd('async test2');
});
```
通过以上例子的log可以很清楚的看到，Promise.all中的第异步方法都已经调用过了，但是如果其中一个报错，那么后续代码就不会继续执行，而异常退出了。
## 使用情景及注意事项
通过以上示例我们可以看到Promise.all可以并发调用且处理异步方法，但是我们仍然需要注意什么情况下可以使用。当有上下文依赖的异步调用时，比如asyncMethod2需要用到asyncMethod1的返回结果，这种情况下Promise.all不适用。如果不希望异步方法的结果因为其中一个异步方法的异常导致所有结果都没有返回，请在异步方法里做好异常处理（用try/catch捕获单个异步方法异常，并友好的返回错误信息）。请使用最新的Node.js版本，至少8.x，早期版本的Node.js的Promise的运行效率并不高，如果因为版本问题不能升级可以使用第三方bluebird库代替默认的Promise。








