---
layout: post
title:  "Redis的keys命令被禁用的思考"
date: 2019-02-21 16:43:26 +0800
categories: node.js
---
&#160; &#160; &#160; &#160; 为了删除某个特定前缀的数据，我在项目里用keys模糊匹配来获取需要删除的key，测试服跑的好好的，可是部署正式服的时候就报错了。后来追踪报错信息原来时腾讯云Redis集群禁用了keys这个命令。

<!--description-->  
&#160; &#160; &#160; &#160; 搜索之后才发现原来使用keys命令有这么严重的后果。虽然redis以快著称，但是架不住keys命令的遍历所有检查还需要模糊匹配的需求啊。之前做的小项目没有发觉，但是如果数据量够大整个redis会被keys命令阻塞，CPU飙升，妥妥的事故。
  
&#160; &#160; &#160; &#160; 不用keys有替代方法吗？1. 优化代码改用Set集合。2. 使用SCAN命令代替。

&#160; &#160; &#160; &#160; redis2.8之后就支持了SCAN命令，下面是代替keys命令的实现。
```javascript
exports.searchKeys = function( keyword ){
    return new Promise( async function( resolve, reject ){
        let keyArr = [];
        let cursor = 0;          //游标
        try {
            do{
                //redis客户端执行命令的实现 具体代码就不放了
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

