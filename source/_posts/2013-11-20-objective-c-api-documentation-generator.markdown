---
layout: post
title: "Xcode自动生成Objective C SDK文档"
date: 2013-11-23 14:26
comments: true
categories: Integration
tags: api objective c oc project xcode 自动生成 项目 文档
---

有的时候我们经常需要查看描述项目的API文档，传统的做法是手动写Markdown或Doc文档之类，现在我们可以利用标准的代码注释方法，然后用[appledoc](http://gentlebytes.com/appledoc/)工具来自动生成和苹果官方SDK文档类似的HTMl文档。


## 配置


下载源码并编译:

	git clone git://github.com/tomaz/appledoc.git 
	cd appledoc

用它自带的脚本编译

	sudo sh install-appledoc.sh (如果你需要默认HTML模板可以添加 '-t default'， 默认是存放在 ~/.appledoc下的)


你也可以自定义 bin 目录, 和模板存放地

	mkdir ~/Library/Application\ Support/appledoc (创建Doc存放目录)
	sudo sh install-appledoc.sh -b /usr/bin -t ~/Library/Application\ Support/appledoc


## 集成进Xcode5 脚本

1. 在你的Xcode项目中新建Target 
2. 选择 Other > Aggregate, 建议取名为Documentation
3. 选择 Documentation > Edit Scheme
4. 选择 Build > Post-actions
5. 点+ > New Run Script Action
6. Provide build settings from 选择 Documentation
7. 下面粘贴进脚本

*company 等内容修改为你自己的内容*

*outputPath 为输出html路径*
```
	#appledoc Xcode script 
    # Start constants 
    company="SQUARE"; 
    companyID="com.doruby";
    companyURL="http://doruby.com";
    target="iphoneos";
    #target="macosx";
    outputPath="~/help";
    # End constants
    /usr/local/bin/appledoc \
    --project-name "${PROJECT_NAME}" \
    --project-company "${company}" \
    --company-id "${companyID}" \
    --docset-atom-filename "${company}.atom" \
    --docset-feed-url "${companyURL}/${company}/%DOCSETATOMFILENAME" \
    --docset-package-url "${companyURL}/${company}/%DOCSETPACKAGEFILENAME" \
    --docset-fallback-url "${companyURL}/${company}" \
    --output "${outputPath}" \
    --publish-docset \
    --docset-platform-family "${target}" \
    --logformat xcode \
    --keep-intermediate-files \
    --no-repeat-first-par \
    --no-warn-invalid-crossref \
    --exit-threshold 2 \
    "${PROJECT_DIR}"
```

![objective-c api auto generator config](/assets/objectivec_api_documentation.png)

Build 后即可在 `~/help` 文件夹下看到多出了几个html 文件，那就是完整的项目API文档。

# 关于注释的写法

<!-- more -->

{% render_partial assets/CommentsFormattingStyle.markdown %}
