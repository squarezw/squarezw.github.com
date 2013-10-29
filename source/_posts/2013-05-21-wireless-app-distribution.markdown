---
layout: post
title: "无线分发应用--通过Safari安装App"
date: 2013-05-21 16:03
comments: true
categories: 
---

iOS 支持以无线方式安装企业级应用程序，这可让您在不使用 iTunes 的情况下将内部软件分发给用户。

## 简单几步：

1. 用户需要将设备的[UDID](http://www.innerfence.com/howto/find-iphone-unique-device-identifier-udid)加到 Apple Developer Center 中心的设备里，并更新 .mobileprovision文件

2. Scheme 里将 Archive 的 Build Configuration 换成 Debug 模式

3. Archive 后从 Organizer 找到app文件，生成 ipa 文件

4. 生成 plist 文件，将它与 ipa 文件放到服务器上，并可通过网址访问并可下载

5. 做一个网页供大家访问后点键接跳转下载此plist, 如果生成将网址生成一个二维码，就更方便了。
   
   比如: 

``` objc 
<a href="itms-services://?action=download-manifest&url=http://example.com/manifest.plist">Install App</a>

```


## plist 文件模板:

<!-- more -->

* 注: {}里的内容是要替换的

```objc
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>items</key>
	<array>
		<dict>
			<key>assets</key>
			<array>
				<dict>
					<key>kind</key>
					<string>software-package</string>
					<key>url</key>
					<string>{你的域名http://doruby.com/xxx.ipa}</string>
				</dict>
			</array>
			<key>metadata</key>
			<dict>
				<key>bundle-identifier</key>
				<string>{你的bundle identifier}</string>
				<key>bundle-version</key>
				<string>1.0</string>
				<key>kind</key>
				<string>software</string>
				<key>title</key>
				<string>{App名称}</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>

```

## 网页模板
* 注: {}里的内容是要替换的

```objc
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
<meta name="viewport" content="width=320, height=460, user-scalable=no,
initial-scale=1.0" />
<title>Install Dev App</title>
</head>
<br/><br/><br/>
<body>
<div align="center">
<a href="itms-services://?action=download-manifest&url={http://doruby.com/xxx.plist}"  style="color:orange; font-size:24px">Install the App</a>
</div>
</body>
</html>
```

## 注意:

可能需要配置你的 Web 服务器以便正确地传输清单文件和应用程序文件。

* 对于 OS X Server，将以下 MIME 类型添加到 Web 服务的“MIME Types”（MIME 类型）设置中：

application/octet-stream ipa

text/xml plist

* 对于 IIS，使用 IIS Manager 在服务器的“属性”页面中添加 MIME 类型：

.ipa application/octet-stream

.plist text/xml


## 参考:

[wireless enterprise app distribution](http://help.apple.com/iosdeployment-apps/mac/1.1/#app43ad871e)

[enterprise deployment of ios apps](http://thirteendaysaweek.com/2012/10/01/enterprise-deployment-of-ios-apps-with-monotouch/)

example:

[my app](http://www.doruby.com/assets/food.html)