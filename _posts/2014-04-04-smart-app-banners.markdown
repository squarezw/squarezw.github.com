---
layout: post
title: "智能的App Banner - Smart App Banners"
date: 2014-04-04 09:50
comments: true
categories: 
---

Safari 在iOS6设备里有一个智能的App Banner特性，添加一行代码在你的网站里，如果用户在iPhone或iPad等iOS设备上，用Safari浏览器打开网站，他将在顶部看到如下的内容：

![Smart App Banner](https://developer.apple.com/library/ios/documentation/AppleApplications/Reference/SafariWebContent/Art/smartappbanner.png)

这行代码就是：

```
<meta name="apple-itunes-app" content="app-id=myAppStoreID, affiliate-data=myAffiliateData, app-argument=myURL">
```

app-id 就是你的AppStore上的id 号，affiliate-data 与 app-argument 是可选的两个参数

添加后，他会智能的判断你的手机上是否安装有此APP，如果有则为打开按钮，没有则出现一个AppStore下载链接。

_如果点过关闭显示[x]后，它将不再显示，除非你去设置中清除Safari Cookies_