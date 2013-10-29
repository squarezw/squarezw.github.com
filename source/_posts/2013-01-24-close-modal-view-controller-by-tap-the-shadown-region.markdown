---
layout: post
title: "点击阴影区关闭打开的Modal View Controller"
date: 2013-01-24 11:03
comments: true
categories: 
---

参考：[Close Modal View Controller by tap the shadow region](http://mengxiangping.com/?p=121)

如何实现用户点击阴影区域，将当前出现的ModalViewController消失
![modal view controller](/assets/modalViewC.png)


首先了解一下: UIApplicationMain

``` objc
int main(int argc, char *argv[]) 
{ 
    @autoreleasepool { 
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class])); 
    } 
}
```

argc 与 argv 是标准的c main 函数参数。 第三个参数是接收事件响应的主要对象(principalClassName)，如果存在的话必须是继承UIApplication, 第四个 delegateClassName, 实现 UIApplicationDelegate 中的协议方法.

任何时刻你点击屏幕，principalClassName都会监听, 并执行sendEvent方法, 所以我们只要拦截这个方法，然后做我们想做的事情就可以了。

## 实现

<!-- more -->

**测试环境：iOS5, iPad**

改变main的第三个参数对象为 我们自己定义的一个 MyAppplication 类。

main.m
``` objc
int main(int argc, char *argv[])
{
  @autoreleasepool {
    
    return UIApplicationMain(argc, argv, @"MyApplication", NSStringFromClass([AppDelegate class]));
    
  }
}
```

.h file
``` objc
#import <Foundation/Foundation.h>

@interface MyApplication : UIApplication

@end
```

.m file
``` objc
#import "MyApplication.h"

@implementation MyApplication

-(void)sendEvent:(UIEvent *)event{
 
  [super sendEvent:event];

  // 关键是在这里拿到点击事件后,如果判断点击的是阴影区, 阴影区的View 是一个私有类, 名字叫UIDimmingView, 所以如果响应的点击事件是在这个View上的，我们就可以关闭当前的ModalView
  UITouch* touch = [[[event allTouches] allObjects] lastObject];
  
  if ([NSStringFromClass([[touch view] class]) isEqualToString:@"UIDimmingView"]) {
      UIViewController * vc = [[[self keyWindow] rootViewController] presentedViewController]; // 找到正在显示的控制器
      [vc dismissModalViewControllerAnimated:YES];
  }
}

@end
```
