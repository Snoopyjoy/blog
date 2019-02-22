---
layout: post
title:  "安卓微信内置浏览器跳转默认浏览器"
date: 2019-02-21 16:43:26 +0800
categories: web
keywords: 微信app唤起,微信跳转,微信宣传页
---
&#160; &#160; &#160; &#160;在进行APP推广时，我们通常需要发布一个推广页面，提供APP的下载或者唤起APP。不管这个页面的链接是被制作为二维码或是在微信中分享，都避免不了在微信中打开。  

<!--description-->
&#160; &#160; &#160; &#160;唤起APP可以通过浏览器中的scheme协议的URL来实现，但这种方式在微信浏览器中是无效的。  

&#160; &#160; &#160; &#160;在安卓中通过连接打开唤起APP是可以实现的。原理是利用微信对响应header中的Content-Disposition的逻辑处理。  
 
Node.js使用Express路由实现：
```javascript
    const express = require('express');
    const router = express.Router();
    
    /**
    * @description 根据user-agent判断是否为来自微信的请求
    * @param UA
    * @returns {boolean}
    */
    function isWeixin(UA){
      return /MicroMessenger/gi.test(UA);
    }
    
    router.get('/', function(req, res, next) {
      if( isWeixin( req.headers['user-agent'] ) ){
        //实测filename为baidu.apk时不会弹出应用宝弹窗
        res.writeHead(206, { "Content-Type":"text/plain; charset=utf-8",
        "Content-Disposition":"attachment;filename=baidu.apk" });
        res.end();
      }else{
        //非微信浏览器中正常显示
        res.writeHead(200);
        res.end("test");
      }
    });
    module.exports = router;
```
[测试链接](http://www.hxl2lgy.top/jump) `http://www.hxl2lgy.top/jump` 在微信中打开试试吧。

IOS中还没仔细研究，最无奈的解决方案就是加图层引导用户点击右上角使用默认浏览器打开了。