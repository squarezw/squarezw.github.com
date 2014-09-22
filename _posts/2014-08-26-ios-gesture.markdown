---
layout: post
title: "iOS 手势操作详解"
date: 2014-08-26 14:38
comments: true
categories: iOS Gesture
---

## 为什么要有手势
  
  在我们拿起手机时，除了语音外，最重要的交互操作就是通过手指触摸了。就如游戏机的手柄，电脑的键盘一样。在iPhone设备里，因为它没有物理键盘，所以每一个手势动作，必须准确的覆盖到每一个用户想要表达的真实意图。手势的重要性也不言而喻。

## 苹果如何设计手势

1. 什么时候开始识别
    
    当我们手机里的App启动时，它就运行在一个无限循环里，这样使它不会在打开后立即退出(NSRunloop)，不过它也提供了打断这个循环的条件，比如触摸(UITrackingRunLoopMode)。当它发生时，会立即被App   接收到，可以让能响应手势事件的对象捕获到它，并根据用户的触摸动作作出回应。

2. 捕捉用户触摸流程

    当用户触摸屏幕或摇晃设备等操作时，iOS 会将这些操作信息，包括发生时间，动作等传递给你的App, 然后你可以根据这些信息、更自然的响应用户操作，并带来真实的体验。

3. 关于 iOS 上的事件分类 (event)

    事件被包装为一个对象发给你的APP, 在 iOS 设备里，事件有下面几类

    ![Event handling](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/events_to_app_2x.png)

        * 多点触摸事件 (Multi-Touch events)
        * 动作事件 		(Motion events)
        * 远程控制事件 (Remote control events)

4. UIKit 对手势的封装
    
    iOS APP 可以识别多种手势操作，比如放大，捏合，滚动等。实际上一些常用的手势操作被封装到了 UIKit 框架里，比如UIControl的子类，如 UIButton 或 UISlider, 他们可以响应点击或滚动。你可以通过苹果设计的 target-action 设计模式将这些手势操作绑定到这些对象上，比如

        [button addTarget:self action:@selector(pressedButton:) forControlEvents:UIControlEventTouchUpInside];

    你也可以使用这种方式在一个普通的视图里添加手势绑定，这样他就可以像一个控件器样响应你的手势操作。

    手势识别是对复杂的触摸动作，做出的一种抽像封装。你可以自己定制和识别特殊的手势动作，有不少第三方库提供了开源的代码，比如利用手势识别算法 [$1 Unistroke Recognizer](http://depts.washington.edu/aimgroup/proj/dollar/)实现复杂的手势识别操作。让它可以识别文字，手势密码等。

    [iOS 的手势识别文档](/ios/gesture/recognizer/2014/08/28/ios-gesture-recognizers/)

5. 一个事件按照特定的路径寻找处理它的对象
    当 iOS 识别一个事件后，它会将这个事件发送给和它最近的对象，比如一个被触摸的视图，如果这个对象不能处理它，它会传递给下一级的对象，走到有人可以处理它，这是一种响应链设计模式。

    [iOS 事件交付: 响应链](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html#//apple_ref/doc/uid/TP40009541-CH4-SW2)

6. 一个 UIEvent 包括了触摸(Touch), 摇晃(Shake-Motion), 或远程控制(Remote-Control)事件
    很多事件都是 UIEvent 类的实例。一个 UIEvent 对象包括了很多你如何响应用户的事件. 一个用户动作，比如手指触摸和在屏幕移动上移动，都会不断有事件对象发向App 去处理它们。每个事件都有一个触摸类型，比如摇晃，远程控制等。

    [多点触摸事件(Multitouch Events)](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/multitouch_background/multitouch_background.html#//apple_ref/doc/uid/TP40009541-CH5-SW9)

    [位移事件(Motion Events)](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/motion_event_basics/motion_event_basics.html#//apple_ref/doc/uid/TP40009541-CH6-SW14)

    [远程控制事件(Remote Control Events)](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Remote-ControlEvents/Remote-ControlEvents.html#//apple_ref/doc/uid/TP40009541-CH7-SW3)

7. 当用户触摸视图时，APP 会接收到 Multitouch 事件
    
    APP 本身的 UIKit 控制器和手势识别基本满足你所有的触摸事件操作，甚至你可以自定义 UIView 实现自己的手势识别规则

8. 当用户移动他们的设备时，APP 会接收到 Motion 事件

    Motion 事件支持的信息包括设备位置，方向，动作，他们都会被记录到。你还可以通过加速度和陀螺仪添加微妙强大的功能应用程序

    Motion 事件适用于不同的场景，我可以通过不同的框架操作它们。当你摇晃设备时，你可以通过 UIKit 框架收到 UIEvent 对象到你的 app. 如果你想app 可以接收，灵敏持续的加速计或陀螺仪数据 ，你可以使用 Core Motion Framework

9. APP 可接来自多媒体控件器发出的远程控制事件

    这些事件可以控制声音和视频


