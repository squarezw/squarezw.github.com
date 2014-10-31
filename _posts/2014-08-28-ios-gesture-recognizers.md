---
layout: post
title: "iOS 手势识别"
date: 2014-08-28 20:01
comments: true
categories: Gesture
---

## 手势识别

手势识别器将抽像度低的事件操作转化为更容易理解的动作，它们是附加在视图上的对象，并允许对这些动作进行回应。手势识别器解释这些触摸事件是否是一个特定的手势，比如轻扫，缩放，旋转。如果它们能正确识别，它们会发送一个动作给目标对象。这个对象是你指定的视图控制器，如下图所示。这样的设计模式强大简单，你可以动态指定响应动作，而且你可以添加一个手势识别给一个视图，而不用子类化视图。



![一个附加在视图上的手势识别](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/gestureRecognizer_2x.png)


---

## 使用手势识别简化事件操作

UIKit 框架提供了预先定义的一些常用手势定义。它提供的预定义手势，大大简化了你自己去写识别代码,而且用标准的识别可以确保你的APP可以正确识别用户所期望的内容。

如果你想自己定义一个不一样的手势，比如对勾或圈圈，你可以创建属于你自己的手势识别。如果设计和实现这些专属的手势识别，可以看这里 [创建一个定制的手势识别](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/GestureRecognizer_basics/GestureRecognizer_basics.html#//apple_ref/doc/uid/TP40009541-CH2-SW44)

### 内嵌的常用手势识别

下表是UIKit 框架自带的手势类别

	手势 (Gesture)	UIKit class
	点击 (Tapping (any number of taps)) 	UITapGestureRecognizer
	捏合 (Pinching in and out (for zooming a view))		UIPinchGestureRecognizer
	滑动或拖动 (Panning or dragging)					UIPanGestureRecognizer
	轻扫 (Swiping (in any direction))			UISwipeGestureRecognizer
	旋转 (Rotating (fingers moving in opposite directions))	UIRotationGestureRecognizer
	长按 (Long press (also known as “touch and hold”))		UILongPressGestureRecognizer

你必须遵循苹果的规范去正确响应用户的手势操作 , 见 [iOS Human Interface Guidelines](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/index.html#//apple_ref/doc/uid/TP40006556)

### 手势附加在一个视图上

任何手势都和一个视图相关联，不过，一个视图可以有多个手势，因为一个视图可能会响应多个不同的手势。为了一个视图能识别特殊的手势，你必须附加手势识别到这个视图上。当用户触摸这个视图时，手势识别会在视图动作前收到触摸的消息，然后手势可以应答这个消息。

### 手势触发动作信息

当手势识别器识别它特定的手势，它会发送一个动作消息给它的对象。要创建一个手势识别，我要实例化它并附加一个对象和动作

### 离散和连续的手势

手势是离散或者连续的。一个离散的手势，比如轻击，只发生一次。一个连续的手势，比如捏合，是在一段时间内发生。对于离散的手势，手势识别器只发送一个单一的动作消息给对象。而一个连续的手势会持续的发送动作消息给它的对象直到多点触摸结束，正如下图所示：

![离散和连续的手势](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/discrete_vs_continuous_2x.png)

## 响应手势识别事件

<!-- more -->

要添加一个手势给你的APP有三件事情是你要去做的

1. 创建和配置一个手势识别实例
	这一步包括附值一个目标对象，动作，有时还会要指定特定的手势属性(比如 numberOfTapsRequired)
2. 附加这个手势识别器给视图
3. 实现响应这个动作的方法

### 使用 Interface Builder 附加手势

	PS: 用的不多，大家有空自己看苹果官方文档吧

### 编码方式添加手势识别

你可以通过allocating 和 initializing 来实例化 UIGestureRecognizer 的子类，比如UIPinchGestureRecognizer. 

如果你创建这个实例化对象后，你需要通过  addGestureRecognizer:  方法附加给一个 View. 下面是一个示例

```
- (void)viewDidLoad {
     [super viewDidLoad];
 
     // Create and initialize a tap gesture
     UITapGestureRecognizer *tapRecognizer = [[UITapGestureRecognizer alloc]
          initWithTarget:self action:@selector(respondToTapGesture:)];
 
     // Specify that the gesture must be a single tap
     tapRecognizer.numberOfTapsRequired = 1;
 
     // Add the tap gesture recognizer to the view
     [self.view addGestureRecognizer:tapRecognizer];
 
     // Do any additional setup after loading the view, typically from a nib
}
```

### 响应离散的手势

当你创建了手势识别并添加了 action 方法后，你就可以用此方法去响应手势操作。下面的代码是一个响应视图点击并通过 locationInView: 方法定位显示图片的例子

> 原码在这里: [Simple Gesture Recognizers](https://developer.apple.com/library/ios/samplecode/SimpleGestureRecognizers/Introduction/Intro.html#//apple_ref/doc/uid/DTS40009460)

```
- (IBAction)showGestureForTapRecognizer:(UITapGestureRecognizer *)recognizer {
       // Get the location of the gesture
      CGPoint location = [recognizer locationInView:self.view];
 
       // Display an image view at that location
      [self drawImageForGestureRecognizer:recognizer atPoint:location];
 
       // Animate the image view so that it fades out
      [UIView animateWithDuration:0.5 animations:^{
           self.imageView.alpha = 0.0;
      }];
}
```

每个手势都有它特定的一些属性，比如下面的例子， showGestureForSwipeRecognizer: 方法使用 swipe 手势识别的 direction属性来判断用户滑动的方向是左还是右。

```
// Respond to a swipe gesture
- (IBAction)showGestureForSwipeRecognizer:(UISwipeGestureRecognizer *)recognizer {
       // Get the location of the gesture
       CGPoint location = [recognizer locationInView:self.view];
 
       // Display an image view at that location
       [self drawImageForGestureRecognizer:recognizer atPoint:location];
 
       // If gesture is a left swipe, specify an end location
       // to the left of the current location
       if (recognizer.direction == UISwipeGestureRecognizerDirectionLeft) {
            location.x -= 220.0;
       } else {
            location.x += 220.0;
       }
 
       // Animate the image view in the direction of the swipe as it fades out
       [UIView animateWithDuration:0.5 animations:^{
            self.imageView.alpha = 0.0;
            self.imageView.center = location;
       }];
}
```


### 响应一个连续的手势

连续手势允许你的app响应手势正在发生的过程，例如，你可以在用户捏合时放大或缩小，或允许用户在屏幕在拖动对象

下面的例子显示当用户旋转手势时，将显示一个旋转后的图片。当用户手势放开时，将淡出并将图片回到水平方向。

```
// Respond to a rotation gesture
- (IBAction)showGestureForRotationRecognizer:(UIRotationGestureRecognizer *)recognizer {
       // Get the location of the gesture
       CGPoint location = [recognizer locationInView:self.view];
 
       // Set the rotation angle of the image view to
       // match the rotation of the gesture
       CGAffineTransform transform = CGAffineTransformMakeRotation([recognizer rotation]);
       self.imageView.transform = transform;
 
       // Display an image view at that location
       [self drawImageForGestureRecognizer:recognizer atPoint:location];
 
      // If the gesture has ended or is canceled, begin the animation
      // back to horizontal and fade out
      if (([recognizer state] == UIGestureRecognizerStateEnded) || ([recognizer state] == UIGestureRecognizerStateCancelled)) {
           [UIView animateWithDuration:0.5 animations:^{
                self.imageView.alpha = 0.0;
                self.imageView.transform = CGAffineTransformIdentity;
           }];
      }
 
}
```

每次方法调用，在 drawImageForGestureRecognizer: 方法里都会被设置为不透明。当用户手势完成时，图片又会在 animateWithDuration: 方法里设置为透明。这个 showGestureForRotationRecognizer: 方法会持续检测手势状态。关于这个状态解释可以查看下面的 "手势状态机"。


## 如何定义手势识别交互转换

常常当你添加手势识别到你的app 后，如果你想要在与其它触摸事件之间做特定的交互时，你首先要理解一下更多关于手势识别工作的细节。

### 有限状态机中的手势识别 

下图中，所有的手势开始于 Possible 状态 (UIGestureRecognizerStatePossible)，它们分析收到的多次触摸序列，然后决定转化为 fail 状态或其它状态.

![手势识别状态机](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/gr_state_transitions_2x.png)

当一个离散的手势识别器识别它是这样的手势后，它会将它的状态从 Possible 变为 Recognized  (UIGestureRecognizerStateRecognized) ，然后识别完成。

对于连续的手势第一次识别时，将状态从 Possible 变为 Began (UIGestureRecognizerStateBegan). 然后，它再将状态从 Began 变为 Changed (UIGestureRecognizerStateChanged), 然后在手势发生时持续的将状态从 Changed 变为 Changed。直到用户的手指从视图离开，手势状态转变为 Ended (UIGestureRecognizerStateEnded)，然后手势识别完成. 

如果你不想长时间适配手势，也只可以将一个连续的手势也可以从 Changed 变为 Canceled (UIGestureRecognizerStateCancelled)。

所有的时候，每次手势状态的改变，手势识别器都会发送一个 action message 给它的目标，除非它的状态是 Failed 或 Canceled.所以一个离散的手势，当从 Possible 到 Recognized 时 只发送一次 action message . 而一个连续的手势在状态变化时会发送很多 action message。

当手势识别状态变为 Recognized (或 Ended)状态时，它会重置经的状态为 Possible. 这个转化不会触发 action message.

## 和其它手势交互

一个视图可以有多个手势识别附加给它，你可以使用视图的 gestureRecognizers 属性来看有多少手势识别器，你也可以动态的添加或删除手势识别器，通过 addGestureRecognizer: 和 removeGestureRecognizer: 方法。

当一个视图有多个手势识别附加给它时，你可能想去解决他们产生的识别冲突。默认的情况下接收到触摸后，它们是没办法设定定顺序的，不过你可以覆盖这些默认的行为:

* 指定一个手势在识别时应该分析其它的手势识别
* 允许两个手势识别同时操作
* 通过触摸分析避免手势识别

使用 UIGestureRecognizer 类方法，delegate methods 或 子类化覆盖方法来达到这样的效果

## 声明指定两个手势的识别顺序

想象一下你想识别两种手势 swipe 与 pan 。你想区别这两种手势所触发的 action. 默认情况时，你的 swipe 手势会被识别为 pan . 这是因为它的手势必要条件被解释为了 pan 手势。

所以你可以显示的指明它们之间的关系，通过调用 requireGestureRecognizerToFail: 方法如下面所示

```
- (void)viewDidLoad {
       [super viewDidLoad];
       // Do any additional setup after loading the view, typically from a nib
       [self.panRecognizer requireGestureRecognizerToFail:self.swipeRecognizer];
}
```

这个方法指明只有其它的手势失败时才会让它自己开始识别。

> 注意如果你有单击与双击手势识别时，不能设定单击需要在双击失败时进行，因为这不符合苹果设计规范，也许用户就只是想单击而已

## 通过分析触摸来避免手势识别

你可以修改手势行为，通过添加一个 delegate 对象去体质手势识别。这个 UIGestureRecognizerDelegate 协议支持多种办法分析触摸避免识别。你可以识别 gestureRecognizer:shouldReceiveTouch: 方法或 gestureRecognizerShouldBegin: 方法，这两个方法都是 UIGestureRecognizerDelegate 协议里可选的

当手势触摸开始，如果你想立即决定是否识别本次触摸，使用 gestureRecognizer:shouldReceiveTouch: 方法，这个方法每次触摸都发被调用。返回 NO 避免识别，返回 YES 不会修改手势识别状态。

下面的例子使用 gestureRecognizer:shouldReceiveTouch: 代理方法避免收到触摸手势在定制的 subview 后识别

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Add the delegate to the tap gesture recognizer
    self.tapGestureRecognizer.delegate = self;
}
 
// Implement the UIGestureRecognizerDelegate method
-(BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch {
    // Determine if the touch is inside the custom subview
    if ([touch view] == self.customSubview){
        // If it is, prevent all of the delegate's gesture recognizers
        // from receiving the touch
        return NO;
    }
    return YES;
}
```
gestureRecognizer:shouldReceiveTouch: 与 gestureRecognizerShouldBegin: 的区别是，shouldReceiveTouch 会拿到 touch 事件，决定是否开始识别，而 gestureRecognizerShouldBegin 是准备改变手势状态时，你可以通过识别的情况来决定是否转化。
shouldReceiveTouch 会最先回调。

## 允许同时响应手势识别

默认的情况时，同一时间两个手势不能同时被识别。但有一种情况下，比如你想用户在缩放视图时同时旋转同时进行。你需要改变他的默认行为了。 可以用 gestureRecognizer:shouldRecognizeSimultaneouslyWithGestureRecognizer: 这个  UIGestureRecognizerDelegate 协议里的方法。这个方法会在判定是否阻止一个方法在识别时，同时阻止另一个的识别。默认它会返回 NO. 返回 YES 的会你可以让两个手势同时识别。

> 注意：虽然返回 YES 可以让两个手势同时被识别，但设置为 NO 不一定它不会被同时识别，因为也许另一个手势的 delegate 里会设置为 YES

## 指定两个手势之间的单向关系

如果你想控制两个手势的交互的单向关系，你可以覆盖 canPreventGestureRecognizer: 或 canBePreventedByGestureRecognizer: 子类方法，返回 NO (默认为 YES). 例如，你想旋转时防止收缩手势，但你不想收缩时防止旋转，你可以这样指定:

    [rotationGestureRecognizer canPreventGestureRecognizer:pinchGestureRecognizer];

然后覆盖这个手势子类方法返回 NO. 更多的子类化手势定制可以看下面的: [创建自定义手势]()

## 与其它界面控件的交互

在 iOS 6 之后，默认的控件避免了重叠的手势行为。例如，你有一个按钮并也附加了单击动作，然后你又有一个单击的手势附加在了这个按钮的父视图上，然后用户点击了这个按钮，这个按钮方法会首先拦截到这个触摸事件而不去响应手势识别。这样的规则只在默认的控件行为上，包括:

* 在 UIBUutton, UISwitch, UIStepper, UISegmentedControl, 和 UIPageControl 上单击
* 在 UISlider 上滑动
* 在 UISwitch 上平移。

如果你想定制这些控件的默认行为，请尽量参照 iOS 人机交互指南.

# 解释手势识别触摸原理

到目前为止，你已经了解到了怎样识别手势并如何响应他们，然而，如果你要创建一个自定义的手势识别，你还需要了解更多关于它的细节原理。

## 一个事件包括所有的点击序列

在 iOS 里，触摸表示手指出现在屏幕上或在上面运动。一个手势有一次或多次触摸，他们都被 UITouch 对象所表示。例如，一次捏合手势里有两个手指在屏幕上向相反的方向运动。

事件包括了一段时间里所有的触摸序列，触摸序列开始于一个手指的触摸，直到最后一个手指离开屏幕。作为一个手指移动，iOS 发送触摸对象给事件。多触摸事件被表示为 UIEventTypeTouches 类型的 UIEvent 对象.

每个触摸对象只跟踪一个手指，在一个多触摸序列中。这序列期间， UIKit 跟踪手指并更新它的触摸属性，这些属性包括触摸阶段，在视图的位置，上一次的位置，时间。

这个触摸阶段指明触摸开始，是否移动或停止等。看下图:

![触摸序列与阶段](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/event_touch_time_2x.png)

## APP 收到触摸时调用的操作方法

在多点触摸序列中，当有新的或改变的触摸阶段时都会发送信息给这些回调方法: 

touchesBegan:withEvent: 当一个或多个手指触摸到屏幕时
touchesMoved:withEvent: 当手指开始移动时
touchesEnded:withEvent: 当一个或多个手指离开屏幕时
touchesCancelled:withEvent: 当触摸被系统取消时，比如来电话时

> 注意这些方法和手势识别状态无关，比如 UIGestureRecognizerStateBegan 和 UIGestureRecognizerStateEnded ，手势识别状态只表示它自己的识别状态，不是触摸的状态。

# 将触摸发送到视图

当你想修改触摸的交付路径时，你需要理解它的默认行为。在一般情况下，当触摸发生，这个触摸对象会从 UIApplication 对象传递给 UIWindow 对象。在传递触摸对象给视图之前，它会先发送触摸对象到附加在触摸到的视图（或他的父视图）上的手势识别对象，

![默认触摸的交付路径](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/path_of_touches_2x.png)

## 手势识别首先得到识别触摸的机会

一个 window 延迟交付触摸对象给这个视图，所以是手势识别首先识别手势，在延误期间，如果手势能识别这个触摸手势，接下来这个 window 永远不会将触摸事件传递给视图.

例如你有一个离散的手势识别需要两个手指，下图显示了两个手指传入时将会的变化:

![触摸信息序列](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/recognize_touch_2x.png)

注意最后当触摸被识别后，之前在视图上的触摸就会被取消，发送了 touchesCancelled:withEvent: 方法。

## 影响交付触摸到视图

你可以通过一些 UIGestureRecognizer 属性修改默认的将会行为，如果你改变了它们默认的属性，你将得到下列的行为

* delaysTouchesBegan (默认为 NO) ，默认 window 在 Began 和 Moved 阶段会发送触摸对象给视图和手势. 如果设置为 YES 的时候，window 会延迟交付给视图，这确保当手势识别成功后不再将交付给视图. 注意设置这个属性的时候要谨慎，他可能会让你的界面感觉无反应。

这样的设置提供了一个类似的行为在 UIScrollView 的 delaysContentTouches 属性中，在这样的场景下，当滚动开始不久后，scroll-view 里的子对象不会收到 touch 信息，这样不会让他看直起来闪烁。

* delaysTouchesEnded (默认为YES) , 当它为 YES 时，如果手势想取消它时，确保视图不完成 action 。当手势识别在分析手势时， window 不会在 Ended 阶段交付触摸对象给视图。当手势识别它是手势时，触摸对象被取消。如果 识别失败，window 会交付它到视图，发送 touchesEnded:withEvent: 信息。设置它为 NO 时， 在发析时，也同时发送信息给视图。

考虑这样的情部，一个视图需要双击识别，用户双击这个视图，如果 属性设置为 YES, 这个视图会得到 touchesBegan:withEvent:, touchesBegan:withEvent:, touchesCancelled:withEvent:, 和 touchesCancelled:withEvent:。如果这个属性设置为 NO, 这个视图收到这样的序列 : touchesBegan:withEvent:, touchesEnded:withEvent:, touchesBegan:withEvent:, 和 touchesCancelled:withEvent:, which means that in touchesBegan:withEvent:，这个视图可以识别一个双击

如果手势识别判定一个触摸不是它手势的一部分，它可以直接传递给它的视图，要做到这一点，可以调用手势识别方法 ignoreTouch:forEvent: 。

# 定制手势识别

实现一个定制的手势，首先要创建 UIGestureRecognizer 的子类，然后 import 这个子类的头文件

  #import <UIKit/UIGestureRecognizerSubclass.h> 

然后从 UIGestureRecognizerSubclass.h 头文件中拷贝下面的方法声明到你的子类里:

```
- (void)reset;
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
```

这些方法和触摸操作一样有明确的行为，在你覆盖它们前，你要先调用父类的实现，即使它是一个空的实现。

注意 state 这个属性在你引入 UIGestureRecognizerSubclass.h 头文件后，就从只读变为可读写的了。

## 实现自定义的触摸实别操作

核心是在于通过下面四个方法 touchesBegan:withEvent:, touchesMoved:withEvent:, touchesEnded:withEvent:, and touchesCancelled:withEvent: 去将抽像低的触摸行为变为可识别的高级行为。

下面这个例子中，仅有一个 view ，但很多 app 中有多个 view ， 通常，你应该转换为屏幕坐标，以正确识别手势。

```
#import <UIKit/UIGestureRecognizerSubclass.h>

// Implemented in your custom subclass
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    [super touchesBegan:touches withEvent:event];
    if ([touches count] != 1) {
        self.state = UIGestureRecognizerStateFailed;
        return;
    }
}
 
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event {
    [super touchesMoved:touches withEvent:event];
    if (self.state == UIGestureRecognizerStateFailed) return;
    UIWindow *win = [self.view window];
    CGPoint nowPoint = [touches.anyObject locationInView:win];
    CGPoint nowPoint = [touches.anyObject locationInView:self.view];
    CGPoint prevPoint = [touches.anyObject previousLocationInView:self.view];
 
    // strokeUp is a property
    if (!self.strokeUp) {
        // On downstroke, both x and y increase in positive direction
        if (nowPoint.x >= prevPoint.x && nowPoint.y >= prevPoint.y) {
            self.midPoint = nowPoint;
            // Upstroke has increasing x value but decreasing y value
        } else if (nowPoint.x >= prevPoint.x && nowPoint.y <= prevPoint.y) {
            self.strokeUp = YES;
        } else {
            self.state = UIGestureRecognizerStateFailed;
        }
    }
}
 
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event {
    [super touchesEnded:touches withEvent:event];
    if ((self.state == UIGestureRecognizerStatePossible) && self.strokeUp) {
        self.state = UIGestureRecognizerStateRecognized;
    }
}
 
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event {
    [super touchesCancelled:touches withEvent:event];
    self.midPoint = CGPointZero;
    self.strokeUp = NO;
    self.state = UIGestureRecognizerStateFailed;
}
```

对于状态转化离散和持续手势是不一样的，正如上面的 "有限状态机中的手势识别" 所述一样，当你创建一个自定义手势识别，你必需明确指明它的相关状态是否为离散或连续。上面的那个例子，它的状态永远不会设置为 Began 或 Changed ，因为它是离散的。

更多的自定义手势资料可以查看 [WWDC 2012: Building Advanced Gesture Recognizers](https://developer.apple.com/videos/wwdc/2012/?id=233)

### 重置手势状态

如果你的手势识别转变为 Recognized/Ended, Canceled, 或 Failed, 这个 UIGestureRecognizer 类会在状态变为 Possible 之前调用 reset 方法

实现 reset 方法以便重置你的一些内部状态，以便你可以为下一次的识别做准备。下面是一个例子:

```
- (void)reset {
    [super reset];
    self.midPoint = CGPointZero;
    self.strokeUp = NO;
}
```


