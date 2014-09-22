---
layout: post
title: "MKNetworkKit Auto Caching"
date: 2014-05-28 16:45
comments: true
categories: MKNetworkKit Third-libs Plugins Cache
---

MKNetworkKit 网上已有不少介绍它的文章了，不过对于它提供的众多特性的实现机制，还是值得研究研究的。其中 Auto caching 是其中之一。

官方文档是这样写的：

## High performance background caching (based on HTTP 1.1 caching specs) built in

MKNetworkKit can automatically cache all your “GET” requests. When you make the same request again, MKNetworkKit calls your completion handler with the cached version of the response (if it’s available) almost immediately. It also makes a call to the remote server again. After the server data is fetched, your completion handler is called again with the new response data. This means, you don’t have to handle caching manually on your side. All you need to do is call one method

	[[MKNetworkEngine sharedEngine] useCache];

Optionally, you can override methods in your MKNetworkEngine subclass to customize your cache directory and in-memory cache cost.

* 它是基于 [HTTP1.1](http://blog.toright.com/posts/3414/%E5%88%9D%E6%8E%A2-http-1-1-cache-%E6%A9%9F%E5%88%B6) 协议设计的Cache模式，刚好我们请求的后端服务也是基于这个协议，Cache-Control 用的是 `max-age=0, private, must-revalidate` , ETag 标记缓存  

 ![HTTP1.1 Cache Control](/assets/http1_1_cache_control.png)

* 如果你启用了 useCache ，它可以将所有的GET请求进行Cache，它根据请求的 url 来判断是否是同一个请求，然后调用缓存数据

* 它在返回缓存数据时，同时也向服务器请求最新数据，如果有最新内容，会返回新的 ETag，然后MK会更新缓存中的ETag

* 当 APP 退到后端时，缓存数据会持久化到Caches目录中

* 它生成的缓存数据文件根据 unique Identifier 命名，放在 MKNetworkKitCache 目录中，生成规则可以查看 MKNetworkKitOperation uniqueIdentifier 方法。

* 用 MKNetworkEngine emptyCache 实例方法清理缓存