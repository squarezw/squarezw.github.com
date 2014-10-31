---
layout: post
title: "事件交付: 响应链"
description: ""
category: Gesture
tags: ["Gesture", "Event Delivery"]
---
当你设计APP时，很可能要动态地响应事件。例如：触摸可能来自屏幕上不同的对象，你必须理解这些对象，知道如何正确的响应这些事件。

当用户生成的事件发生时，UIKit 会创建包含所需要处理的信息在一个事件对象里。然后将它放置在当前活动APP的事件队列中。对于触摸事件，它是将一组触摸封装在 UIEvent 对象里。对于运动事件，这个事件对象取决定于你使用的框架，并在里面有一些你所感兴趣的运动类型。

一个事件延特定的路径旅行，在它被交给下一个对象前，你都可以处理它。首先,单例的 UIApplication 对象从队列顶部拿到这个事件，并调度处理它。通常，它发送该事件给 **Key Window** 对象，然后它根据不同的事件类型传递给不同的实例对象。

- 触摸事件. 对于触摸事件，window 对象首先交给触摸发生的 view ，这个 view 必须是一个 hit-test view. 这个发现的过程是通过 hit-test view 调用 hit-testing 方法找到的。看下面的图例: *Hit-Testing Returns the View Where a Touch Occurred.*

- 运动与远程控制事件. 对于这些事件，window 对象发送摇晃或远程控制事件给第一个响应者(first responder)操作。关于第一响应者的描述看下面的 *The Responder Chain Is Made Up of Responder Objects*

这些事件的最终目的是找到一个对象可以处理和响应它。因此 UIKit 首先发送到最适合处理这些事件的对象. 对于触摸事件，该对象是 hit-test 视图, 对于其他的事件，是第一响应者。以下部分详细描述 hit-test视图与第一响应者。


### Hit-Testing 返回触摸发生的视图 (Hit-Testing Returns the View Where a Touch Occurred)

iOS 使用 hit-testing 来发现发生在触摸上的视图。Hit-testing 包括检查触摸是否发生在相关 view 的范围内。如果是的，它会递归检查视图下的和个子视图. 在视图层级中最底层的视图变成 hit-test view. 最后 iOS 向这个视图发送触摸事件.

下面的例子, 假设用户触摸在视图 E 在. iOS 通过检查各个子视图来找到 hit-test view

1. 触摸发生在 view A, 所以它检查 view B 与 C
2. 解摸不在 view B 范围，但它在 C 范围内，所以检查 D 和 E
3. 触摸不在 view D 范围，但它在 E 范围

视图 E 是最底层的子视图，所以它变成 hit-test view

![Hit-testing returns the subview that was touched](/assets/hit_testing_2x.png)

这个 `hitTest:withEvent:` 方法从给定的 CGPoint 与 UIEvent 返回 hit test view. 
这个 `hitTest:withEvent:` 方法最初从调用它自身 `pointInside:withEvent:` 方法开始。如果这个点在这个视图范围内，`pointInside:withEvent:` 返回 YES, 然后方法会递归调用每个子视图的 `hitTest:withEvent:` .

如果传入的这一点不在视图范围内，首先调用到 `pointInside:withEvent:` 方法时返回 NO, 这个点被点忽略，并且 `hitTest:withEvent:` 返回 nil. 如果一直子视图返回 NO ，那么它下面的所有子视图分支都会被忽略，因为，如果它不在发生这个 subview 上，它也不会不生在它下面的所有子视图中。这个意思是一些子视图可能在父视图的外面，如果触摸的区域在父视图的外面，它也不会接收到事件。这可能发生在了视图的 `clipsToBounds` 属性为 **NO** 时。

	一个触摸对象和它的生命周期有关，甚至它之后移出这个视图后。

这个 hit-test view 是被赋予最先处理触摸的对象，如果 hit-test view 不能处理这个事件，这个事件会延着视图响应链进行，走到发现能处理这个对象为止。见下面的详解。

### 由响应对象组成的响应链 (The Responder Chain Is Made Up of Responder Objects)

许多类型的事件，都依赖于一个响应者链的事件传递。响应链是一系列链接的响应对象，它开始于应用的开始与结束。如果第一个响应者不能响应这个事件，它会转发这个事件给响应链上的下一个响应者.

一个响应对象是一个可以响应和操作事件的对象，UIResponder 类是所有响应对象的基类，它定义的程序接口不仅是操作事件，同样也提供了通用的响应行为。UIApplication, UIViewController, UIView 类都是响应者，意味着所有的视图和大多数关键的控制器都是一个响应者。注意，Core Animation 层不是响应者。

第一响应者被设计为最先接收事件的开始，通常第一响应者是一个视图对象，一个对象要变为第一响应者可以做下面两件事:

- 覆盖 `canBecomeFirstResponder` 方法，并返回 YES
- 接收一个 `becomeFirstResponder` 消息，如果需要，对象可以给它自己发送这个消息

```
注意: 要确保分配第一响应者时要在你的APP创建对象的图形后。例如，通常你可以在 viewDidAppear: 方法里调用 becomeFirstResponder 方法。如果你试图在 viewWillAppear: 方法里分配，这个时候你的对象图形还没有被创建，所以它的 becomeFirstResponder 方法为 NO.
```
	
事件不仅仅是响应链上的唯一对象，响应链还有下面几种对象被使用:

- 触摸事件: 如果 hit-test view 不能操作一个触摸事件，这个事件会被传到 hit-test view 的响应链中
- 移动事件: 操作 UIKit shake-motion 事件的第一响应者必须实现 UIResponder 类里的 motionBegan:withEvent: 或 motionEnded:withEvent: 方法。正如这里的声明:  [Detecting Shake-Motion Events with UIEvent](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/motion_event_basics/motion_event_basics.html#//apple_ref/doc/uid/TP40009541-CH6-SW2)
- 远程控制事件: 要操作远程控制事件，第一响应者必须实现 remoteControlReceivedWithEvent: 方法.
- 控制消息. 当用户操作一个控制器，比如一个按钮，或切换，而且动作指定方法的对象是空，这个消息会延着控制器的视图响应链传递下去。
- 编辑菜单消息. 当用户在编辑菜单点击时，iOS 使用所需要的方法实现响应(比如剪切，拷贝，粘贴)具体看: [Displaying and Managing the Edit Menu](https://developer.apple.com/library/ios/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/AddingCustomEditMenuItems/AddingCustomEditMenuItems.html#//apple_ref/doc/uid/TP40009542-CH13)
- 文本编辑. 当用户在文本视图里点击文本输入框时也会自动变为第一响应者。默认，虚拟键盘出现，而且会聚焦在文本输入框中。你可以显示一个定制的输入视图代替键盘。你也可以添加一个自定义的输入视图给任何响应对象。更多信息看这里: [Custom Views for Data Input](https://developer.apple.com/library/ios/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/InputViews/InputViews.html#//apple_ref/doc/uid/TP40009542-CH12)

### 响应链遵循特定的响应路径

如果实例化的对象(hit-test view 或 第一响应者)没有操作这个事件，UIKit 将传递这个事件给响应链的下一个响应者。每个响应者决定是否操作这个事件或将它通过调用 nextResponder 方法传递给下一个响应者。这个过程持续到没有下一个响应者为止。

这个响应链最初是用 iOS 捕获事件后将它传递给一个实例化对象，通常是一个 view。这个实例化的视图有最初处理事件的机会。下面的图显示了两种不同的事件交付方法。一个APP的事件交付取决于它的特定结构，但所有的事件交付路径都采用同样的试控法则。

![The responder chain on iOS](/assets/iOS_responder_chain_2x.png)

对于左边的APP，事件遵循下面的路径

1. initial view 试图操作这个事件或消息. 如果它不能操作这个事件，它将这个事件传给它的 superview，因为这个 initial view 不是它的视图控制器的顶级视图
2. superview 试图操作这个事件. 如果 superview 不能操作这个事件，它将这个事件传给它的 superview, 因为它也不是顶级视图
3. 在视图控制器的 topmost view 试图操作这个事件. 如果这个顶级视图不能操作这个事件，它将这个事件传递给它的视图控制器
4. view controller 试图操作这个事件，如果它也不能操作，就传递给 window
5. 如果 window 对象不能操作这个事件，它就传递给 singleton app object
6. 如果 app object 不能操作这个事件，它就会被丢弃.

右边的的图在路径上稍有不同，但所有的交付路径都遵守下列的法则

1. 一个视图传递事件给它的上级视图，直到发现它的顶级视图
2. 顶级视图传递事件给视图控制器
3. 视图控制器传递这些事件给它的顶级视图的父视图.  
   重复 1-3 步，直到事件发现根视图控制器.
4. 根视图控制器传递事件给 window 对象.
5. window 对象传递事件给应用对象(app object)

```
重要: 如果你实现了一个自定义的视图去操作远程控制事件，动作消息，摇晃动作事件或编辑菜单消息，不要直接转发事件给下一个响应者(nextResponder)。要用调用父类实现来让UIKit 帮助你转发事件传递操作
```



{% include JB/setup %}
