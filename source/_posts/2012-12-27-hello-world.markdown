---
layout: post
title: "强大的日志分析工具 -- NSLogger"
date: 2012-12-27 16:43
comments: true
categories: 
---
源码：[https://github.com/fpillet/NSLogger](https://github.com/fpillet/NSLogger)

**特点**

* 摆脱Xcode的小窗查看日志
* 不用再将iPhone连接到电脑上才能看日志
* 支持通过互联网传送日志
* 可以输出图片的日志
* 可自己定义日志等级

![nslogger mainwindow](https://github.com/fpillet/NSLogger/raw/master/Screenshots/mainwindow.png)

## 安装

<!-- more -->

NSLogger分为两部分，LoggerClient和NSLogger Viewer，
LoggerClient是置入你APP的客户端，NSLogger Viewer是一个mac端日志分析器，NSLogger的日志可以通过网络传输到这个日志分析器中。

NSLogger 支持Pod方式安装,在你的APP中配置Podfile
``` ruby
pod 'NSLogger'
pod install
```
*如果不支持Pod，可以直接将LoggerClient文件放入你的APP下.*

[NSLogger Viewer](/assets/NSLoggerViewer.zip) - *这是编译好的日志监控客户端 NSLogger Viewer*

## 使用

* ``` #import "LoggerClient.h" ```
* 设置客户端网络监控的配置
``` objc
LoggerSetViewerHost(NULL, (CFStringRef)@“127.0.0.1”, (UInt32)50000);
```
这一段代码可以加在main.m里

* 除了基本的日志可以打印图片的日志
``` objc
UIImage *img = ONEDefaultImageWithName(@"actionBar");
CGSize sz = img.size;
LogImageData(@"image", 0, sz.width, sz.height, UIImagePNGRepresentation(img));
```
* 为了不动原来的NSLog输出日志方式，可以重新定义NSLog
``` objc
define NSLog(...) LogMessageF( __FILE__,__LINE__,__FUNCTION__, NULL, 0, __VA_ARGS__)
``` 

* 打开NSLogger Viewer mac端，在Preferences的Network中，勾选 Listen for loggers on TCP port.端口默认

**友情提示**

- 如果Xcode编译后，没有发送数据到客户端，可以先 clean 一下。
- 客户端建议用 TCP 协议连接,这样监听端口可以固定
- 点窗口左下角的 f 可以看到对应的日志文件与行号
