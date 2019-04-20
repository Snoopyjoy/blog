---
layout: post
title: 如何用JWT代理Session？
category: Share 
description: JSON web token简称JWT，是一种身份验证解决方案。除了JWT我们还有Session作为身份验证的解决方案。那么我们为什么要使用JWT呢？
keywords: JWT,json web token
---
## 为什么使用JWT
JSON web token简称JWT，是一种身份验证解决方案。除了JWT我们还有Session作为身份验证的解决方案。那么我们为什么要使用JWT呢？  
  
首先Session方案和JWT的数据管理方式不同，Session采用集中管理的方式来保存会话信息，客户端保存的是一个SessionId，而JWT采用的加密签名的方式，用户信息保存在客户端。
    
在单服务器应用应用中，两种方式差别不大，使用JWT的意义只在于用时间换空间，Session数据是保存在内存或者持久化保存在数据库中，客户端是通过SessionId来获取用户会话信息，而JWT的数据就在Token字符串中，需要通过服务端解码验证来获取用户信息。那么在单服务器应用中应用JWT可以通过消耗更多CPU资源的方式换取内存空间或者数据库资源消耗。使用两种方案的判断就在于CPU资源是服务器承载能力的瓶颈还是内存或者数据库是性能瓶颈。
    
在服务器集群应用场景中，Session方案就必须采用一个数据库来管理Session信息了，一般会采用Redis、MemCache之类的高性能缓存服务器来保存。JWT的使用方式则不需要任何调整。在Session方案中一旦保存Session数据的服务器出现故障那么整个需要验证身份的服务就不可用了。随着用户增多，需要增加集群中的服务器时必须要考虑到Session数据库的性能瓶颈。而JWT方案就单纯用户身份验证这一方面则无需考虑更多，想增加多少就增加多少服务器。
  
现在有这么一个应用场景，有许多第三方站点希望接入站点A，通过站点A授权登录后在其他站点就不需要登录了，直接使用站点A的用户信息。这时候如果使用Session就需要站点A提供用户验证服务，并且在通过验证后各个站点都需要维护自己的一套用户验证系统。那么如果使用JWT，只需要站点A在加密JWT信息时采用非对称加密的方式，使用私钥加密信息，将公钥提供给其他站点用来解密。  
  
## 怎么使用JWT
通过以上几个场景的例子，我们有足够的理由使用JWT了吧。那么怎么来应用JWT呢？首先我们来看看JWT的组成。
1. JWT头
头部是一个描述数据的对象，有加密算法和类型两个字段，结构为：
```code
{
    "alg": "HS256",
    "type": "JWT"
}
```
2. JWT的payload
也就是JWT的主体内容。JWT规范提供了7个默认字段 ss：发行人, exp：到期时间(Unix时间戳), sub：主题, aud：用户, nbf：在此之前不可用(Unix时间戳), iat：发布时间(Unix时间戳), jti：JWT ID用于标识该JWT。除了这7个字段还可以加入任何自定义的字段。不过数据了应当尽量精简以减小JWT的长度，而且这个内容仅仅是经过Base64 URL算法转换过，因此不可以把需要保密的信息加入进去。
3. JWT的签名
是对以上两部分通过私钥经过加密算法后得出的签名哈希字符串。
  
JWT的三个部分是通过”.“字符连接起来的。
当用户把登录用户名密码传给服务端后，服务端生成JWT数据，这个Token数据可以在服务端设置到cookie中，或者加到response header中，或者消息体中。客户端在获取到token后保存在本地。客户端在发送请求时，可以通过cookie传递JWT，或者在请求的Header中将Token加入authorization字段，或者加到POST请求的请求消息体中。

## 关于安全
关于加密安全，JWT的加密算法是可以自定义的，一帮情况下只要秘钥强度足够是很难被破解的。但是JWT的加密算法是可以设置为none的。这时候如果一个伪造的Token，它的加密算法是none，服务端也是可以通过验证的。所以服务端在解密时一定要限制JWT头部中所允许的加密算法。  
  
JWT是一个凭证，它几乎不能被破解，但是它可以被盗用。所以传递Token的方式越安全越好，这里推荐服务端返回Token放到respose header中，客户端发送请求带的token放到request header中，客户端保存token根据业务场景来，如果可以就放在代码内存中，要么就是用http-only为true的cookie来传递。存在localStorage、sessionStorage或者存到http-only为false的cookie中要么就需要客户端有一套防XSS方案。最重要的是使用https！  
  
Session存储在服务端，服务端有足够的权限来控制Session。那么JWT是发给客户端的凭证，如何作废一个Token就是一个问题了。有人说把Token保存在Redis服务中，然后操作Redis数据来控制Token是否有效。这样实际上是实现了一套Session-JWT。这样其实使JWT失去了存在的意义。那么我们该怎么做呢？  
  
首先在对于安全性要求较高的服务中，我们的JWT过期时间一定要设置的够短，在Token快过期时可以通过请求用仍未失效的Token请求一个新的Token凭证或者重新登录。这样即使一个Token被盗，在这个Token过期后没用了。  
  
关于如何作废一个未过期的Token，我的做法是在Redis中保存一个时间戳，这个时间戳之前颁发的Token全部作废，而且这个时间戳可以设置一个过期时间，最长为Token的有效时间。这个方案虽然用到了Redis，但是相对于Session，这个方案消耗的资源是很小的，它只是保存了一个有生命周期有限的的时间戳。  

参考:   
[https://baijiahao.baidu.com/s?id=1608021814182894637&wfr=spider&for=pc](https://baijiahao.baidu.com/s?id=1608021814182894637&wfr=spider&for=pc)  


