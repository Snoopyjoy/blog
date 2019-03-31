---
layout: post
title: 解决第三方资源的跨域问题
category: Share 
description: Canvas要使用GPU渲染图片就需要用到WebGL，这时浏览器对图片资源是有严格的跨域限制的，如果不同域名下的资源响应的header里的Access-Control-Allow-Origin字段的值不是"*"或者当前域名，那么就会报跨域错误。如我我们需要用WebGL的高性能渲染，又要用第三方的图片资源，这时该如何解决呢？
keywords: nginx,资源跨域
---
Canvas要使用GPU渲染图片就需要用到WebGL，这时图片资源是有严格的跨域限制的，如果不同域名下的资源响应的header里的Access-Control-Allow-Origin字段的值不是"*"或者当前域名，那么就会报跨域错误。如我我们需要用WebGL的高性能渲染，又要用第三方的图片资源，这时该如何解决呢？  
一个方式就是我们将资源下载到服务器上，直接从服务器访问资源，那如果我们的资源服务器是一个分布式的集群呢？难道每个服务器都要将所有资源都下载一遍？
这时使用服务器转发资源，并且配合CDN内容分发网络就是一个好主意。  
因为Nginx的高性能而且可以很方便的实现方向代理，所以Nginx成了我的首选。下面是一个通用的Nginx配置，可以解决各种第三方资源跨域问题，配合CDN内容分发网络，会有更好的效果。
```code
location ~ /proxy/(.*) {
    resolver 223.5.5.5 223.6.6.6 1.2.4.8 114.114.114.114 valid=3600s;
    add_header 'Access-Control-Allow-Origin' "*" always;
    add_header 'Access-Control-Allow-Credentials' 'true' always;
    add_header 'Access-Control-Allow-Methods' 'GET, OPTIONS' always;
    add_header 'Access-Control-Allow-Headers' 'Accept,Authorization,Cache-Control,Content-Type,DNT,If-Modified-Since,Keep-Alive,Origin,User-Agent,X-Mx-ReqToken,X-Requested-With' always;
    set $paramAp $arg_ap;
    set $paramAh $arg_ah;
    if ($arg_ap = ''){
        set $paramAp 'http';
    }
    if ($arg_ah = ''){
        return 404;
    }
    set $key ${paramAp}://${paramAh}/$1;
    expires      30d;
    proxy_pass $key;
}
```
  
上面这个配置是通用的做法通过访问nginx服务器的/proxy路径就可以实现动态的反向代理。如：https://_mydomain/proxy/_**third-part-path.jpg**?ap=https&ah=**third-part-domain** ap参数传递http还是https请求，ah参数传递第三方域名，thrid-part-path.jpg就是第三方链接的path，通过这样的方式就可以在我们自己的服务器上访问到https://third-part-domain/third-part-path.jpg 这个链接里的资源并且加上了允许跨域的响应头。  
这个配置不光可以用在微信头像这方面，任何第三方资源跨域问题都可以同个这个方法解决，例如字体文件在css中使用时也是有跨域限制的，这时也适用。配合CDN内容分发可以很好的减少我们自己服务器的请求次数同时可以就近选择节点加速我们的访问。
