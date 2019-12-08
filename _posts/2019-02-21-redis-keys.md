---
layout: post
title:  "使用Redis的教训"
date: 2019-02-21 16:43:26 +0800
categories: Tip
keywords: redis,keys命令,redis性能
description: 本地完美运行的项目部署到测试服竟然报错了。经查原来是Redis服务不支持keys命令。
---

&#160; &#160; &#160; &#160; 本地完美运行的项目部署到测试服竟然报错了。经查原来是Redis服务不支持keys命令。  
&#160; &#160; &#160; &#160; 为了管理某个特定功能的key，我在项目里使用了统一的前缀，然后用keys命令来模糊匹配获取相关key列表的做法。但是测试服部署时报错了。

<!--description-->  
&#160; &#160; &#160; &#160; 原来是腾讯云Redis集群禁用了keys这个命令。万幸是在测试服使用的云Redis服务禁用这个功能，否则一旦大规模部署，出现问题将是毁灭性的。  
  
&#160; &#160; &#160; &#160; 那么我为什么要这么设计这个功能呢？因为之前的项目是to B 的。因为用户量少，使用的自建Redis服务，所以在这个场景下是没有问题的。  
  
&#160; &#160; &#160; &#160; 不能使用keys有替代方法吗？  
1. 改用Set集合。  
2. 使用SCAN命令代替。

&#160; &#160; &#160; &#160; 用Redis2.8支持SCAN的命令来实现。
```javascript
exports.searchKeys = function( keyword ){
    return new Promise( async function( resolve, reject ){
        let keyArr = [];
        let cursor = 0;          //游标
        try {
            do{
                //redis客户端执行命令
                const result = await exports.do("SCAN" , [ cursor, "MATCH", keyword ] );
                cursor = result[0];
                const newKeys = result[1];
                keyArr = keyArr.concat( newKeys );
            }while ( Number(cursor) > 0  )
            resolve( keyArr );
        }catch (e) {
            reject(e);
        }
    } );
}
```


