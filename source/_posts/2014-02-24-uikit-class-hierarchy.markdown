---
layout: post
title: "UIKit Class Hierarchy"
date: 2014-02-24 16:17
comments: true
categories: 
tags: uikit class hierarchy
---

是时候复习一下基础内容了，先从最常用到的 UIKit 开始吧。

[UIKit reference introduction](https://developer.apple.com/library/ios/documentation/uikit/reference/UIKit_Framework/Introduction/Introduction.html)

先看看这个层级结构图吧：
![image](https://developer.apple.com/library/ios/documentation/uikit/reference/UIKit_Framework/Art/uikit_classes.jpg)

---

## [UIAcceleration](https://developer.apple.com/library/ios/documentation/uikit/reference/UIAcceleration_Class/Reference/UIAcceleration.html):

加速计类: 加速度实为UIAcceleration对象实例，又被称为加速事件，它代表即时的三维空间上，三个不同轴上的加速度数据。

![加速计三维空间](https://developer.apple.com/library/ios/documentation/uikit/reference/UIAcceleration_Class/Art/device_axes.jpg)

 实用的场景中可以用于“摇晃”，游戏中用于控制对象移动等。

*iOS 5.0之后，它被放在了 CoreMotion 框架里*

* ## [UIAccelerometer](https://developer.apple.com/library/ios/documentation/uikit/reference/UIAccelerometer_Class/Reference/UIAccelerometer.html)

 获得当前设备的加速计单例，里面的delegate 可以获得 UIAcceleration 的实例

*iOS 5.0之后，它被放在了 CoreMotion 框架里*

* ## [UIAccessibilityElement](https://developer.apple.com/library/ios/documentation/uikit/reference/UIAccessibilityElement_Class/Reference/Reference.html)
 
 这个类封装的是可便捷的访问信息，主要是针对一些特殊人群，比如颜色识别有困难，残障人士等。 默认情况下你的设备设定中是不开启的，如果你要开启时，可以在设置里的Accessibility中打开相应的辅助功能。

*原生的UI组件的`isAccessibilityElement`默认是YES的。自定义的UI组件的isAccessibilityElement属性是NO，当`isAccessibilityElement`为NO时，instruments将无法捕获。所以这种情况，我们需要将自定义UI的的isAccessibilityElement属性置为YES，instruments就能获取到。*

* ## [UIBarItem](https://developer.apple.com/library/ios/documentation/uikit/reference/UIBarItem_Class/Reference/Reference.html)

UIBarItem 是一个抽象的超类，用来在Bar上添加道具。 类似于一个按钮。有标题，有图片，动作和 目标。

####  [UIBarButtonItem](https://developer.apple.com/library/ios/documentation/uikit/reference/UIBarButtonItem_Class/Reference/Reference.html)
		
这个类的实例是用在 UIToolbar 或 UINavigationBar 上的专用对象按钮。它从它的抽象父类 UIBarItem 继承的基本按钮的行为。比如 NavigationBar 的返回，关闭按钮。

#### [UITabBarItem](https://developer.apple.com/library/ios/documentation/uikit/reference/UITabBarItem_Class/Reference/Reference.html)

这个类的实现，用于 Tabbar 上的按钮对象。

* ## [UIBezierPath](https://developer.apple.com/library/ios/documentation/uikit/reference/UIBezierPath_class/Reference/Reference.html)

UIBezierPath类可以创建基于矢量的路径。
	
此类是Core Graphics框架关于path的一个封装。使用此类可以定义简单的形状，如椭圆或者矩形，或者有多个直线和曲线段组成的形状。

它还提供了添加二次贝塞尔曲线和三次贝塞尔曲线的支持。

<!-- more -->

* ## [UIColor](https://developer.apple.com/library/ios/documentation/uikit/reference/UIColor_Class/Reference/Reference.html)

绘制UI中用的最多的一个类了吧，一个 UIColor 对象代表颜色，alpha 值。可以使用 UIColor 对象来存储颜色数据，并在绘画过程中，你可以用它们来设置当前填充和笔触颜色。

* ## [UIDecive](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDevice_Class/Reference/UIDevice.html)

UIDevice类提供了一个单例代表当前设备。从这个实例中，可以获取有关设备的唯一的ID，分配名称，设备型号，和操作系统名称和版本等信息。也可以使用的UIDevice实例，检测设备的特点， 如物理方向的变化。

*常用在判断 iOS 版本。*

* ## [UIDocumentInteractionController](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIDocumentInteractionController_class/Reference/Reference.html)

文件交互控制器委托对象，提供应用程序管理与本地系统中的文件的用户交互的支持。例如，一个电子邮件程序可能使用这个类，允许用户预览附件和其他应用程序中打开它们。使用这个类， 目前预览相应的用户界面，打开，复制或打印指定的文件。

一般在程序间共享文档可以通过UIDocumentInteractionController（该类经常被开发者忽略），用其它APP预览PDF文档等。

第三方程序只需要通过在info.plist 中注册支持的相关格式，并安装到ios设备中，便可以自由打开，无需你程序中自己检测第三方程序是否安装，而且文件之间的传输也实现了跨出沙盒的功能。

* ## [UIEvent](https://developer.apple.com/library/ios/documentation/uikit/reference/UIEvent_Class/Reference/Reference.html)

一个 UIEvent 对象（或者简单地说，一个事件对象）在 IOS 中有三种类型的事件：触摸事件(Touch)，运动事件(Motion)和远程控制(RemoteControl)事件。远程控制的事件使一个 Responder 对象来接收来自外部的 附件或耳机的命令，以便它可以管理管理音频和视频，例如，播放视频或跳过到下一音轨。

![event handling](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/events_to_app_2x.png)

触摸事件，会经过一系列的响应链，最后被 UIResponder 接受到。并回调：touchesBegan:withEvent: 方法。

* ## [UIFont](https://developer.apple.com/library/ios/documentation/uikit/reference/UIFont_Class/Reference/Reference.html)

同样常用的字体设定类，提供了用于获取和设置字体信息的接口。

除了使用系统指定的字体外，还可以使用自定义的字体，只需要将自定义字体加到你的工程资源文件中，并在 info.plist 文件中增加一名为 UIAppFonts 的key。将这个key修改成array
将你用到的所有字体的名字，作为这个array的值，一项一项填进去（包括扩展名），然后就可以在代码中直接用[UIFont fontWithName:@”CustomFontName” size:12]取得自定义的字体了。

* ## [UIGestureRecognizer](https://developer.apple.com/library/ios/documentation/uikit/reference/UIGestureRecognizer_Class/Reference/Reference.html)

	UIGestureRecognizer 是手势识别的抽象基类。它有以下具体的子类：

```
	UITapGestureRecognizer: 点击手势，可以识别单次与多次点击
	UIPinchGestureRecognizer: 缩放手势
	UIRotationGestureRecognizer: 旋转手势
	UISwipeGestureRecognizer: 滑动手势
	UIPanGestureRecognizer: 拖动手势
	UILongPressGestureRecognizer: 长按手势
```
* ## [UIImage](https://developer.apple.com/library/ios/documentation/uikit/reference/UIImage_Class/Reference/Reference.html)

UIImage 封装了显示图片的高层级方法，你可以从 文件 ，Quartz ，原始图片数据对象（相机，扫描仪等）中创建。

支持的文件格式：

`.tiff, .tif, .jpg, .jpeg, .gif, .png, .bmp, .BMPf, .ico, .cur, .xbm`

*可通过 UIImagePickerController 类从iPhone照片库或照相机获取图像*

* ## [UILocalizedIndexedCollation](https://developer.apple.com/library/ios/documentation/iPhone/Reference/UILocalizedIndexedCollation_Class/UILocalizedIndexedCollation.html)

	提供索引标题的类，常用在TableView的右方索引列。

![localized index](/assets/localized_index.png)

* ## [UIMenuController](https://developer.apple.com/library/ios/documentation/iPhone/Reference/UIMenuController_Class/UIMenuController.html)

	菜单控制器，默认的单例方法，提供了剪切，复制，粘贴， 选择，选择，和删除功能。你也可以创建自定义的 UIMenuController，

![UIMenuController](/assets/UIMenuController.jpeg)

	
* ## [UIMenuItem](https://developer.apple.com/library/ios/documentation/uikit/reference/UIMenuItem_Class/Reference/MenuItem.html)

	是UIMenuController 的 menuItems 数组里的每个实例。你可以自行定义他们，它只需要两个属性，title 与 action (SEL)

* ## [UINavigationItem](https://developer.apple.com/library/ios/documentation/uikit/reference/UINavigationItem_Class/Reference/UINavigationItem.html)

	一个UINavigationItem 管理显示在 UINavigationBar 上的按钮和视图对象。每个 View Controller push 到 navigation 栈上，必须有一个在 NavigationBar 上包含按钮与被显示的视图 UINavigationItem 对象。

* ## [UINib](https://developer.apple.com/library/ios/documentation/uikit/reference/UINib_Ref/Reference/Reference.html)

	UINib 可以从 Interface Builder nib 文件中将数据进行封装，并将它进行实例化缓存起来。当你需要用到 nib 时，不必去读 nib 文件了。

	比如：

```
	// Load the hoverView from HoverView.xib
	UINib *hoverViewXib = [UINib nibWithNibName:@"HoverView" bundle:nil];
	[hoverViewXib instantiateWithOwner:self options:nil];
```

* ## [UIPasteboard](https://developer.apple.com/library/ios/documentation/uikit/reference/UIPasteboard_Class/Reference.html)

	读写系统里的剪贴版内容

* ## [UIPopoverController](https://developer.apple.com/library/ios/documentation/uikit/reference/UIPopoverController_class/Reference/Reference.html)

	iPad 里的专有 Controller, 效果如下：

![UIPopoverController](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/splitview_portrait_popover.jpg)

* ## [UIPrintFormatter](https://developer.apple.com/library/ios/documentation/uikit/reference/UIPrintFormatter_Class/Reference/Reference.html)

打印格式的抽象基类，可定制打印的页边距等。具体实现类有如下三种：

```
	UISimpleTextPrintFormatter
	UIMarkupTextPrintFormatter
	UIViewPrintFormatter
```

* ## [UIPrintInfo](https://developer.apple.com/library/ios/documentation/uikit/reference/UIPrintInfo_Class/Reference/Reference.html)

	封装了关于打印工作的信息，包括打印的id,名称,输出类型，方向，双面打印模式。当它需要打印时，这些信息会被打印系统使用。

* ## [UIPrintInteractionController](https://developer.apple.com/library/ios/documentation/uikit/reference/UIPrintInteractionController_Class/Reference/Reference.html#//apple_ref/occ/cl/UIPrintInteractionController)
	它包括了打印的UI与，打印文稿，图片和其它打印内容之间的典型交互情景。你可以使用 UIPrintInteractionController 单例方法，将整个打印过程进行控制。

![UIPrint Center](https://developer.apple.com/library/ios/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/Art/print_options_ipad_2x.png)


* ## [UIResponder](https://developer.apple.com/library/ios/documentation/uikit/reference/UIResponder_Class/Reference/Reference.html)

该UIResponder类定义了响应和处理事件的对象接口。它是UIApplication，UIView的和它的子类（包括UIWindow）的父类。这些类的实例通常被作为应答对象，或者简单地说，应答者。

通常有两种类型的事件：触摸与运动事件。主要的触摸事件处理方法有  touchesBegan:withEvent:,touchesMoved:withEvent:, touchesEnded:withEvent:, and touchesCancelled:withEvent:. 这些方法的参数和他们新的，或有改变的特别触摸事件有关。这样响应对象可以跟踪和处理这些事件。任何时候当手指触摸屏幕，拖动，抬起时，UIEvent 对象都会被创建，这个事件对象包括了所有刚从屏幕抬起时的手指的 UITouch 对象。

```
-touchesBegan:withEvent: // 当用户触摸到屏幕时调用方法
-touchesEnded:withEvent: // 当用户触摸到屏幕并移动时调用此方法
-touchesMoved:withEvent: // 当触摸离开屏幕时调用此方法
-touchesCancled:withEvent: // 当触摸被取消时调用此方法
```

通俗一点说，一个 UIResponder 对象表示一个可以接收触摸屏上的触摸事件的对象, iOS 中，所有显示在界面上的对象都是从 UIResponder 直接或间接继承的。

iOS 3 之后开始支持运动事件：

```
-motionBegan:withEvent: // 运动开始时执行
-motionEnded:withEvent: // 运动结束时执行
-motionCancled:withEvent: // 运动被取消时执行
```

iOS 4 之后开始支持远程控制事件：

```
-remoteControlReceivedWithEvent
```

接收事件之后，使用到的响应链函数
`-nextResponder` 下一个响应者，在通常的UIView 实现中，一般会返回父级对象

*所以你可以自行改变响应链的响应路径*

[响应链](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html#//apple_ref/doc/uid/TP40009541-CH4-SW2)

```
-isFirstResponder  指示对象是否为第一响应者，这里的第一响应者就是当前焦点
-canBecomeFirstResponder 布尔值，指定对象是否可以变为第一响应者
-becomeFirstResponder   把对象设置为 firstResponder
-canResignFirstResponder   对象是否可以取消 firstResponder
-resignFirstResponder 注销 firstResponder
```

* #### [UIApplication](https://developer.apple.com/library/ios/documentation/uikit/reference/UIApplication_Class/Reference/Reference.html#//apple_ref/doc/uid/TP40006728-CH3-DontLinkElementID_1)

这个UIApplication类提供运行在 iOS 上的 app 的集中协调与控制中心，每个 app 只有一个它的实例。当 app 启动时，UIApplicationMain 函数被调用，他会创建一个 UIApplication的单例对象。之后，你就可以通过 sharedApplication 类方法访问到它。

如我们在 main.c 文件中看到的：

```
return UIApplicationMain(argc, argv, nil, NSStringFromClass([SomeAppDelegate class]));
```

UIApplication 的主要任务是处理传入的用户事件的初始路由。此外，他还维护所有当前打开的窗口列表（UIWindow 对象），所以通过它你可以检索到应用程序的UIView对象。

这个app对象通常分配一个 delegate,接受一些运行时的标致性的通知，比如：app 启动，低内存，app 关闭，然后作出响应。

UIApplication 允许你管理设备的以下几种行为：

```
控制界面方向

挂起传入的触摸事件

接近感应（用户面部）开和关

注册远程通知

触发撤销重做 UI

判断是打开app 还是 URL

退到后台时继续执行任务

注册或取消本地通知

执行 app 级别的恢复任务
```

UIApplication 定义的 delegate 必须遵循 UIApplicationDelegate 协议并且实现一些协议的方法

---

- [UIView](https://developer.apple.com/library/ios/documentation/uikit/reference/uiview_class/uiview/uiview.html)

	UIView 类定义了一个矩形区域和管理该区域的接口。在运行时，一个 view 对象在它的区域内进行绘制和处理这些内容的交互。UIView 类本身也提供了一些带背景色的基本填充行为。更多精妙的内容，可以通过你自定义的子类实现一些所须的绘制和交互处理达成。

	另外UIKit 框架也自带了一些标准子类，从简单的按钮到复杂的表格都能被使用。例如: UILabel 可以绘制文本内容，UIImageView 绘制一个图片.

	参考：
	[View 与 Window 架构](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html#//apple_ref/doc/uid/TP40009503-CH2-SW1)

	[View 动画](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/AnimatingViews/AnimatingViews.html#//apple_ref/doc/uid/TP40009503-CH6-SW1)


	- [UIWindow](https://developer.apple.com/library/ios/documentation/uikit/reference/UIWindow_Class/UIWindowClassReference/UIWindowClassReference.html)

		UIWindow 类定义了被称为 window 的对象，它用来管理和协调在设备屏幕上出现的view。除非app有其它外部屏幕上，否则一个app只有一个 window

		一个 window的主要两个功能是提供view的显示区域和发配事件到view上，要想改变你的显示内容，你可以改变 window的 root view ,但你不能创建一个新的 window. 一个 window有一个明确的等级结构, UIWindowLevelNormal--表示它相对其它 windows的z轴位置. 例如一个系统提示window出现在 normal window 上面。

	- [UILabel](https://developer.apple.com/library/ios/documentation/uikit/reference/UILabel_Class/Reference/UILabel.html)

		UILabel 类实现了只读文本 view. 你能使用这个类画一行或多行静态文本。基本的 UILabel 支持简单和复杂的文本样式。你也可以控制它上面的样式，比如使用shadow 绘制阴影或高亮。如果需要，你可以通过子类来实现更加个性化的样式。

		参考:
		[在iOS上显示文本内容](https://developer.apple.com/library/ios/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/UsingTextClasses/UsingTextClasses.html#//apple_ref/doc/uid/TP40009542-CH2-SW1)

	- [UIPickerView](https://developer.apple.com/library/ios/documentation/uikit/reference/UIPickerView_Class/Reference/UIPickerView.html)

		UIPickerView 的实现对象叫做 picker view, 它使用类似角子机（赌场里经常见的777机器）来显示一个或多个值的设置。用户通过滚动轮子来选择对应的行的值。

		你也可以通过子类来定制显示内容。这个UI由包括(component)元件和行构成。元件是一个滚轮，在上面是一连串一行行的索引内容。在 picker view 上不同的元件从左至右顺虚排列。每一行的内容可以是一个字符串或view 对象。比如 UILabel 或 UIImageView.

		![Picker View](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uipickerview_intro.png)


	- [UIProgressView](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIProgressView_Class/Reference/Reference.html)

		UIProgressView 类表现了一个进度条描述进展的时间。它也支持管理样式的属性。

		![UI progress view](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uiprogressview_intro_2x.png)

	- [UIActivityIndicatorView](https://developer.apple.com/library/ios/documentation/uikit/reference/UIActivityIndicatorView_Class/Reference/UIActivityIndicatorView.html)

		使用一个动态的指示器显示内容正在执行中，它就象一个活动的齿轮。

		![UI activity indicator](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uiactivityindicator_intro.png)

	- [UIImageView](https://developer.apple.com/library/ios/documentation/uikit/reference/UIImageView_Class/Reference/Reference.html)

		一个 image view 对象支持基本的视图容器，可以显示单张图片或几个动画图片。如果是动画图片，UIImageView 类支持控制动画的持续时间和频率，你也可以自由的开始和停止动画。

		* iOS 6 之后，支持 “状态保留” 机制，同样也可以用于这里.

	- [UITabBar](https://developer.apple.com/library/ios/documentation/uikit/reference/UITabBar_Class/Reference/Reference.html)

		一个 tabbar 是一个控制器，通常在tab bar controller上下文中，它显示屏幕的底部。在 tab bar 上的每一个按钮是一个叫 UITabBarItem 的类的实例。如果你想替换Bar上的按钮执行不同的 action ，可以使用 UIToolbar 对象。

		![UI tab bar](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uitabbar_intro_2x.png)

	- [UIToolbar](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIToolbar_Class/Reference/Reference.html)

		一个 toolbar 可以控制显示一个或更多的按钮，叫 toolbar items. 要创建 toolbar items, 使用 UIBarButtonItem 类。 要添加 toolbar items 到 toolbar 上，使用 setItems:animated: 方法

		![UI tool bar](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uitoolbar_intro_2x.png)

	- [UINavigationBar](https://developer.apple.com/library/ios/documentation/uikit/reference/UINavigationBar_Class/Reference/UINavigationBar.html)

		这个 UINavigationBar 类用于控制导航栏内容。它是一个bar,通常显示上屏幕的最上方。主要的属性有一个左（返回）按钮，一个中心标题，和可选择的右侧按钮。你可以使用navigation bar 作为一个标准对象或结合 navigation controller 使用.

		在iOS5之后，你也可以自行定制它的样式.

		![UI nav bar](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uinavbar_intro_2x.png)

	- [UITableViewCell](https://developer.apple.com/library/ios/documentation/uikit/reference/UITableViewCell_Class/Reference/Reference.html)

		UITableViewCell 定义了在 UITableView 对象里的每cell的属性和行为。这个类包括了为设置和管理 cell 内容和背景的属性和方法（包括文本，图片，自定义的 view），管理 cell的选择，高亮状态，管理附属 view和初始编辑中 cell 内容。

		参考：
		[TableViewCell 详解](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/TableView_iPhone/TableViewCells/TableViewCells.html#//apple_ref/doc/uid/TP40007451-CH7)

		![cell part](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/TableView_iPhone/Art/tv_cell_parts.jpg)

	- [UIActionSheet](https://developer.apple.com/library/ios/documentation/uikit/reference/UIActionSheet_Class/Reference/Reference.html)

		UIActionSheet 提供了一个单选器，你也可以通过它来提示用户哪些是危险的操作。这个Action sheet 包括一个可选择的标题和一个或多个按钮，每个按钮都代表一个action

		![ui action sheet](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uiactionsheet_intro_2x.png)

	- [UIAlertView](https://developer.apple.com/library/ios/documentation/uikit/reference/UIAlertView_Class/UIAlertView/UIAlertView.html)

		使用 UIAlertView 用于提示用户一些信息，这个 alert View 类似但和 action sheet 表现上不一样。

		使用类提供的属性和方法设定 title, message, 和代理来配置按钮。如果你要添加一个自定义的按钮，你必须设定一个delegate. 这个代理一定要遵循 UIAlertViewDelegate 协议。使用 show 方法来显示 alert view.

		![ui alert view](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uialertview_intro_2x.png)

	- [UIScrollView](https://developer.apple.com/library/ios/documentation/uikit/reference/UIScrollView_Class/Reference/UIScrollView.html)

		UIScrollView 类提供了为显示超过一屏的超长内容的支持。它让用户可以支持滑动手势和收缩手势浏览内容。

		UIScrollView 也是一些 UIKit 类，包括 UITableView 和 UITextView 的超类。

		![UI scroll view](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uiscrollview_intro.png)


		- [UITableView](https://developer.apple.com/library/ios/documentation/uikit/reference/UITableView_Class/Reference/Reference.html)

			UITableView 实例是为显示和编辑列表信息。

			![UI table view](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/TableView_iPhone/Art/tv_plain_style.jpg)

			非常详细的文档：
			[Table View Programming Guide for iOS](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/TableView_iPhone/TableViewStyles/TableViewCharacteristics.html#//apple_ref/doc/uid/TP40007451)

		- [UITextView](https://developer.apple.com/library/ios/documentation/uikit/reference/uitextview_class/Reference/UITextView.html)

			UITextView 类实现了滚动的行为，多行文本区域. 这个类支持定制的文本样式，也支持文本编辑。通常用来显示多行文本，比如一个比较长的文档时。

			![UI text view](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uitextview_intro_2x.png)

		- [UISearchBar](https://developer.apple.com/library/ios/documentation/uikit/reference/UISearchBar_Class/Reference.html)

			这个 UISearchBar 类实现了一个文本控制器为搜索提供基本的文字输入，这个控制器提供了一个文本输入区域作为文本输入，一个搜索按钮，一个书签按钮，一个取消按钮。这个 UISearchBar 对象实现上不执行搜索操作，你需要使用 遵循 UISearchBarDelegate 协议的delegate，当文本输入和按钮点击时实现它的action。

			![UI search bar](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uisearchbar_intro_2x.png)

		- [UIWebView](https://developer.apple.com/library/ios/documentation/uikit/reference/UIWebView_Class/Reference/Reference.html)

			你可以在应用使用 UIWebView 类嵌入 web 内容。你可以非常简单的创建 UIWebView对象附加了 window上，并发送请求 web 内容。你也可以使用这个类进行返回前进到历史页面，并且你甚至可以以编程方式设定 web 内容属性。

			参考：
			[Safari Web Content Guide](https://developer.apple.com/library/ios/documentation/AppleApplications/Reference/SafariWebContent/Introduction/Introduction.html#//apple_ref/doc/uid/TP40002051)

		- [UIControl](https://developer.apple.com/library/ios/documentation/uikit/reference/uicontrol_class/reference/reference.html)

			UIControl 是所有 control 对象的基础类，比如 UIButton 和 滚动条 等，他将用户的意图传递给应用。你不能直接实例化 UIControl 类, 代替它的是一些常用的UI子类.

			UIControl 主要的任务是定义了一个接口，并且有一些基本实现，当一些事件调用时，会首先调用它们到它们的对象。

			为了说明 target-action 原理，可以在Cocoa Fundamentals Guide 看"Target-Action in UIKit" 。如果要了解 Multi-Touch 事件信息,可以看 [Event Handling Guide for iOS](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009541)

			UIControl 类也包括了 getting 和 setting control 状态的方法，如，为了确定control是否打开或者高亮。并且它还在内部定义一些触摸的跟踪方法，这些跟踪方法可以被子类覆盖。

			![about UI Control](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uicontrol_intro_2x.png)

			参考：[About Controls](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/UIControl.html#//apple_ref/doc/uid/TP40012857-UIControl)

			- [UIButton](https://developer.apple.com/library/ios/documentation/uikit/reference/UIButton_Class/UIButton/UIButton.html)

				一个 UIButton的实例实现了在屏幕上的按钮。当点击它时button截获触摸事件并发送 action 消息到 target 对象. 这个类支持设置title,image和其它外观属性方法。你还可以为不同的状态设定不同的表现形式。

				![UI button](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uibutton_intro.png)

			- [UIDatePicker](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDatePicker_Class/Reference/UIDatePicker.html)

				这个UIDatePicker类实现了一个使用多个齿轮对象，让用户选择日期和时间。iPhone 时间picker例子是一个时间闹钟设定应用，你也可以用它做为一个倒计时器。

				![UI date picker](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uidatepicker_intro_2x.png)

			- [UIPageControl](https://developer.apple.com/library/ios/documentation/uikit/reference/UIPageControl_Class/Reference/Reference.html)

				你可以用UIPageControl 类来创建和管理页码 control. 一个页码 control 显示一个水平的连续小点，每个点代表了文档的一部分。当前显示的页用白色的点表示。

				![page control](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uipagecontrol_intro_2x.png)

			- [UISegmentedControl](https://developer.apple.com/library/ios/documentation/uikit/reference/UISegmentedControl_Class/Reference/UISegmentedControl.html)

				一个 UISegmentedControl 对象是一个由多个部分组成的水平control ，每个部分作为一独立的按钮。

				一个 segmented control 可以显示一个title (一个 NSString 对象)或一个图片（UIImage 对象）.这个 UISegmentedControl 对象自动调整他们的大小比例在他们的父视图下，除非他们有一个指定的宽度设置。当你添加和删除部分片段时，你可以请求这个action ，他会有滑动和消退效果。

				![UI Segmented control](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uisegmentedcontrol_intro_2x.png)

			- [UITextField](https://developer.apple.com/library/ios/documentation/uikit/reference/UITextField_Class/Reference/UITextField.html)

				一个 UITextField 也是一个 control 对象，它是一个显示可编辑文本，并且当用户按下return按钮时发送 action 消息到对象。你典型的使用场景是，搜集用户一定数量的文本，并且马上执行一些action ,比如基于文本的搜索操作。

				![UI text field](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uitextfield_intro_2x.png)

			- [UISilder](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UISlider_Class/Reference/Reference.html)

				一个 UISilder 对象是一个可视化的 control , 它使用一个连续的值范围进行选择， Slider 通常显示为水平的滑动条。

				![UI slider](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uislider_intro_2x.png)

			- [UISwitch](https://developer.apple.com/library/ios/documentation/uikit/reference/UISwitch_Class/Reference/Reference.html)

				你可以使用 UISwitch 类来创建和管理 On / Off 按钮。例如，在设置飞行模式和蓝牙设置时。

				![UI switch](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/Art/uiswitch_on_2x.png)

- [UIViewController](https://developer.apple.com/library/ios/documentation/uikit/reference/UIViewController_Class/Reference/Reference.html)

	这个 UIViewController 类提供了基本的 view 管理模型。你很少直接实例化UIViewController对象。相反,你实例化UIViewController类的子类执行特定的任务。

	视图控制器管理一组视图组成应用程序的用户界面的一部分.作为应用程序的控制器层的一部分, 视图控制器和其它模型对象，其它视图控制器一起构成了一个一致的用户界面。

	必要时，一个视图控制器可以：

	- 调整和排版他们的 view
	- 调整 view 里的内容
	- 当用户与他们交互时，反应他们与视图的效果。

	参考：
	[View Controller Programming Guide for iOS](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007457)

	- [UISplitViewController](https://developer.apple.com/library/ios/documentation/uikit/reference/UISplitViewController_class/Reference/Reference.html)

		UISplitViewController是一个管理包含了两边ViewController的视图控制器，你使用这个类去实现主－从界面, 在左边显示的是列表视图控制器，右边是左边点击后的详细视图控制器。它只能适用于 iPad 设备。尝试去用到其它设备都会出现异常。

		它没有特定的界面，他的主要工作是管理两个视图控制器在切换方向时的表现形式。

		![ui split view controller](/assets/ui_split_view_controller.jpg)

	- [UITabBarController](https://developer.apple.com/library/ios/documentation/uikit/reference/UITabBarController_Class/Reference/Reference.html)

		UITabBarController 类实现了一个专门的类似radio-style的视图控制器管理界面。这个标签栏界面的标签显示在窗口的底部。它通常按它原来的样式显示，不过在 iOS6 之后可以用子类显示。

		每个标签栏控制器的标签对应一个定制的视图控制器。当用户选择一个指定的标签地，这个标签栏控制器替换之前的主视图为相关的视图控制器.

		![Tab bar controller views](https://developer.apple.com/library/ios/documentation/uikit/reference/UITabBarController_Class/Art/tabbar_controllerviews.jpg)
		
		你不能直接访问标签控制器上的标签视图。要配置这个标签，你可以分配这个 viewControllers 属性来提供根视图。它的显示顺序也是由你分配viewControllers时的顺序来确定的。

		标签栏内容是通过他们相应的视图控制器进行配置，通过创建一个UITabBarItem 类的实例来生成标签内容，配置合适的属性，然后赋给视图控制器的tabBarItem属性.如果你没有提供定制的标签栏内容，那它默认只会显示一个这个视图控制器的标题。

	- [UITableViewController](https://developer.apple.com/library/ios/documentation/uikit/reference/UITableViewController_Class/Reference/Reference.html)

		![Table View](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/TableView_iPhone/Art/tv_plain_style.jpg)

		UITableViewController 类创建了一个控制器管理表格视图. 它实现了下列行为:

		- 如果nib 文件通过 initWithNibName:bundle: 方法指定(它是父类 UIViewController 声明的方法), UITableViewController 会从这个nib文件中读取这个表格视图结构。否则，它是不能正确配置UITableView对象和计算大小的。你可以通过tableview属性来设置它。

		- 如果包含表视图的nib文件加载后，datasource(数据源)和delegate(委托)为nib文件中定义的对象(如果有的话).如果没有指定nib文件或者nib文件定义没有datasource或delegate, UITableViewController 设置表视图的datasource 与 delegate为它自己(self)

		- 当表视图第一次出现时，table-view controller重新加载表视图数据。它也在每次显示时清理了选择过的行。这个 UITableViewController 类的方法是在基于父类方法的viewWillAppear:时实现的。你也可以关闭这个默认行为，通过clearsSelectionOnViewWillAppear属性。

		- 当表视图被显示后，控制器刷新表视图的滚动条指示器。这个 UITableViewController 类的方法是实现在 viewDidAppear: 里的。

		- 它实现了父类方法 setEditing:animated:, 所以当用户点击导航栏上的编辑|完成按钮时，会控制表格的编辑模式。

		你可以创建它的子类来管理每个表视图。当你在控制器里初始化了 initWithStyle:，你必须指定表视图的样式（plain 或 grouped）。

		参考：
		[Table View Programming Guide for iOS](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/TableView_iPhone/TableViewStyles/TableViewCharacteristics.html#//apple_ref/doc/uid/TP40007451)

	- [UINavigationController](https://developer.apple.com/library/ios/Documentation/UIKit/Reference/UINavigationController_Class/Reference/Reference.html)

		UINavigationController 类实现了一专门的视图控制器去管理分层内容的导航。它的导航界面让数据更有效，更简单的呈现。通常你直接使用它即可。iOS 6之后你可以定制它的行为。

		![A sample navigation interface](https://developer.apple.com/library/ios/Documentation/UIKit/Reference/UINavigationController_Class/Art/navigation_interface_2x.png)

		导航控制器使用导航栈管理屏幕上的显示内容，它是一个视图控制器数据，数据第一个控制器对应的是根视图控制器，最后一个视图控制器对应的是当前屏幕上显示的内容。你可以使用导航控制器类代的方法编辑这个栈。比如你添加一个视图控制器到栈顶用 pushViewController:animated: 方法。

		![The views of a navigation controller](https://developer.apple.com/library/ios/Documentation/UIKit/Reference/UINavigationController_Class/Art/NavigationViews_2x.png)

	- [UIImagePickerController](https://developer.apple.com/library/ios/documentation/uikit/reference/UIImagePickerController_Class/UIImagePickerController/UIImagePickerController.html)

		UIImagePickerController类管理可定制的，系统提供的用户界面，用于拍照和电影支持的设备上，并选择保存的图像和电影用于您的应用程序.一个图像选择控制器管理用户交互,并通过delegate对象传递交互的结果.

		它的交互外观由 source type 决定：

		- UIImagePickerControllerSourceTypeCamera 类型提供一个获得新照片或电影的交互界面（设备必须支持媒体捕获）
		- UIImagePickerControllerSourceTypePhotoLibrary 或 UIImagePickerControllerSourceTypeSavedPhotosAlbum 这两个类型提供选择已保存在设备上的照片或电影

		![Taking Pictures and Movies](https://developer.apple.com/library/ios/documentation/AudioVideo/Conceptual/CameraAndPhotoLib_TopicsForIOS/Art/UIImagePickerController.jpg)

		参考：
		[Camera Programming Topics for iOS](https://developer.apple.com/library/ios/documentation/AudioVideo/Conceptual/CameraAndPhotoLib_TopicsForIOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40010400)

	- [UIVideoEditorController](https://developer.apple.com/library/ios/documentation/uikit/reference/UIVideoEditorController_ClassReference/Reference/Reference.html)

		一个UIVideoEditorController对象，或视频编辑器，管理系统提供的视频微调界面，从视频的开头到结尾帧都可以进行管理。并可以重新调整编码质量。这个对象管理用户交互，并提供电影编辑的文件路径给到你的delegate 对象 ([UIVideoEditorControllerDelegate Protocol Reference](https://developer.apple.com/library/ios/documentation/uikit/reference/UIVideoEditorControllerDelegate_ProtocolReference/Reference/Reference.html#//apple_ref/doc/uid/TP40009028))。这个特性只支持有视频录制的设备上。

		![editing video controller](/assets/editing_video_controller.jpg)

	- [UIScreen](https://developer.apple.com/library/ios/documentation/uikit/reference/UIScreen_Class/Reference/UIScreen.html)

		UIScreen对象包含设备的整个屏幕矩形边界。设置应用程序的用户界面时, 你应该使用该对象的推荐矩形属性设置应用程序的窗口。

		你可以通过它，获得整个设备的屏幕的宽，高。

		通常我们在didFinishLaunchingWithOptions: 方法里经常看到这样一句话：

			self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];

	- [UIScreenMode](https://developer.apple.com/library/ios/documentation/uikit/reference/UIScreenMode_class/Reference/Reference.html)

		UIScreenMode对象代表一组可以应用于一个UIScreen对象的属性.对象封装的信息是屏幕的基本显示缓冲区的大小和长宽比，及它使用的独立像素。

		开发者可以不必关心这个属性，屏幕和窗口对象会根据底层硬件自动确定像素和长宽比等。不过你要是想用到像素级信息，可以考虑用它来设置。

	- [UISearchDisplayController](https://developer.apple.com/library/ios/documentation/uikit/reference/UISearchDisplayController_Class/Reference/Reference.html)

		一个搜索控制器用于管理搜索状态栏和伴随搜索结果的视图表。

		你初始化一个带搜索栏的搜索显示控制器和视图控制器负责管理数据搜索.当用户开始搜索时， 搜索控制器添加搜索界面在原视图控制器上，当搜索后，显示一个搜索结果的表视图显示搜索结果。

		当你添加这个搜索控制器，并添加和管理搜索结果时，你需要遵守以下四点:

		1. 为搜索结果表视图提供数据源(searchResultsDataSource).
		2. 为搜索结果表视图提供代理(searchResultsDelegate).响应用户在结果表中的选择。
		3. 为搜索控制器提供代理(delegate).响应用户在搜索栏上的输入操作。
		4. 为搜索栏(UISearchBar)上提供代理(delegate UISearchBarDelegate), 响应搜索条件的改变。

		典型的初始化实例方法如下：

			searchController = [[UISearchDisplayController alloc] initWithSearchBar:searchBar contentsController:self];
			searchController.delegate = self;
			searchController.searchResultsDataSource = self;
			searchController.searchResultsDelegate = self;

	![UISearchController](/assets/UISearchController.jpg)

	- [UITextChecker](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITextChecker_Class/Reference/Reference.html)

		使用UITextChecker类的实例来检查一个字符串(通常是文档的文本)拼错的单词。

	- [UITextInputStringTokenizer](https://developer.apple.com/library/ios/documentation/uikit/reference/UITextInputStringTokenizer_Class/Reference/Reference.html)

		这个类实现了UIKit 框架里的UITextInputTokenizer 协议。实现分词效果.

		PS: 没有用过

	- [UITextPosition](https://developer.apple.com/library/ios/documentation/uikit/reference/UITextPosition_Class/Reference/Reference.html)

		这个类表示在一个文本内容的位置。换句话说，它是一个支持字符串在text-displaying视图的索引.

		UITextView 类的实例方法 positionFromPosition:offset: 返回的就是这样的对象

	- [UITextRange](https://developer.apple.com/library/ios/documentation/uikit/reference/UITextRange_Class/Reference/Reference.html)

		这个类表示在一个文本内容中的区域。换句话说，它表示从哪儿开始到哪儿结束的索引。

		UITextView 类的实例方法 selectedRange 返回的就是这样的对象 [textView selectedTextRange];

	- [UITouch](https://developer.apple.com/library/ios/documentation/uikit/reference/UITouch_Class/Reference/Reference.html)

		UITouch对象表示手指在屏幕上点击或移动的特定事件.你可以通过UIEvent传递过来的UITouch对象访问到。

		UITouch对象包括访问的方法,和在特定的视图或窗口触摸的位置.它还允许您找到接触发生时,用户是否不止一次点击,是否是滑动.从哪儿开始，哪儿结束等等。

		gestureRecognizers 属性在 iOS 3.2 后被引入.返回的是手势识别器类。

		![Discrete and continuous gestures](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/discrete_vs_continuous_2x.png)