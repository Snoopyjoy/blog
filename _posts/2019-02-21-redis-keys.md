---
layout: post
title:  "禁止使用Redis的keys命令"
date: 2019-02-21 16:43:26 +0800
categories: 技巧
keywords: redis,keys命令,redis性能
description: keys命令很强大，强大到Redis容不下它，因为它的消耗太大了。
---

&#160; &#160; &#160; &#160; keys命令很强大，强大到Redis容不下它，因为它的消耗太大了。
&#160; &#160; &#160; &#160; 为了管理某个特定前缀的数据，我在项目里用keys命令模糊匹配来获取需要key，测试环境没问题，可是部署正式服的时候就报错了。

<!--description-->  
&#160; &#160; &#160; &#160; 原来是腾讯云Redis集群禁用了keys这个命令。虽然redis很快，但是keys命令的会遍历所有键并且可以模糊匹配，
这个开销随着数据量级的增长是越来越难以承受的。之前做的小项目没有发觉，但仔细想想如果数据量增长到一定量级，
必定会导致redis被keys命令阻塞，CPU飙升，在依赖Redis。
  
&#160; &#160; &#160; &#160; 不用keys有替代方法吗？  
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

