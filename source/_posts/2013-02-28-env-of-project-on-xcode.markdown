---
layout: post
title: "在Xcode里,为项目定义不同的环境配置"
date: 2013-02-28 09:30
comments: true
categories: 
---


有时项目经常需要配置不同的开发环境，比如Debug, QA, Release, Distribution等。每个环境下，会有各自不同的环境配置项，比如变量，常量，宏定义等。

我们最早的方法是先在Build Settings里先设定 Preprocessor Macros CONFIGURATION_$(CONFIGURATION)

这样我们就可以在代码根据该 Macro 来区分现在所处的环境。通过我们是定义在 Prefix.pch 文件里:

``` objc
#ifdef CONFIGURATION_Debug
#   import "ConfigDebug.h"
#else
#   ...
#endif
```

现在想到的更好的方法是直接修改Prefix Header的引用路径

在 Project 里的 Build Settings 里设定 Prefix Header 文件的导入位置.

<!-- more -->

![settings prefix header](/assets/settingsPrefixHeader.png)

比如: Test/Config/Test-Prefix-${CONFIGURATION}.pch

我们新建了一个Config目录，然后在里面添加了 Test-Prefix-Debug.pch , Test-Prefix-Release.pch 等不同的环境配置文件.

Test-Prefix-Debug.pch 

``` objc 
import "ConfigDebug.h"
...
```

Test-Prefix-Release.pch

``` objc
import "ConfigRelease.h"
...
```


## 关于各项环境配置应该放在哪儿比较好?

我们有时会定义一些自己的Macro, 通常我们也是象上面一样写在Preprocessor Macros 里。 实际上更好的方法是建立不同的 .xcconfig 文件, 然后将Target的配置指向它们

比如我们建立一个 Debug.xcconfig, Release.xcconfig

将 PROJECT Info => Configurations => target Based on Configuration File 指向不同环境下的config文件。

![target configurations](/assets/targetConfigurations.png)


我们就可以在这个文件上定义自己的Macro了，比如

Debug.xcconfig

``` objc 
TestMacroDef = 1
```

Release.xcconfig

``` objc 
TestMacroDef = 0
```

你可以在Build Settings中看到User-Defined一栏多了你自定义的Macro

我们还可以改变或添加系统默认的定义，比如上面在 Preprocessor Macros里定义的内容，可以这样写

``` objc 
GCC_PREPROCESSOR_DEFINITIONS = kShareKey=1 $(inherited) // inherited 是继承原有的定义
```

tips: 你可以选中settings的一栏copy， paste 到这个文件中，即可知道定义方法。


如果我们有一些公共的配置文件，可以创建一个 公共的 xcconfig 文件，然后 include 进来即可。

比如

Debug.xcconfig

``` objc
#include "Shared.xcconfig"
```






