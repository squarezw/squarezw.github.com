---
layout: post
title: "ARC 与内存管理"
date: 2013-04-24 13:15
comments: true
categories: 
---

ARC: Automatic Reference Counting (自动引用计数)

ARC 是 iOS 5 后推出的一项为Objective - C程序在编译时提供自动内存管理的功能。ARC可以让你把注意力集中在你感兴趣的代码，减少开发中的内存管理步骤，简化开发。

它通过指定的语法，让编译器(LLVM 3.0)在编译代码时，自动生成实例的引用计数管理部分代码。有一点，ARC并不是GC，它只是一种代码静态分析（Static Analyzer）工具。

在过往我们通常使用的是MRC: Manual Reference Counting(手动内存管理)。这些规则将逐渐变为本能，你会发现少一个release的代码怎么看怎么别扭，从而减少或者杜绝内存管理的错误。可以说MRC的规则非常简单，但是同时也非常容易出错。往往很小的错误就将引起crash或者leak之类问题。

很多人担心内存管理不受自己控制，其实这是对于ARC机制了解不足从而不自信，所导致的对新事物的恐惧。

下面我们从几个方面来详细介绍ARC到底如何实现，如何使用，它的好处，注意事项等。

<!-- more -->

## 需要的基本环境:
ARC is supported in Xcode 4.2 for OS X v10.6 and v10.7 (64-bit applications) and for iOS 4 and iOS 5. Weak references are not supported in OS X v10.6 and iOS 4.

**注意：iOS4 不支持 weak 引用**

## 原理

![ARC Illustration](http://developer.apple.com/library/ios/releasenotes/ObjectiveC/RN-TransitioningToARC/Art/ARC_Illustration.jpg)

	ARC的一个基本原则: 只要某个对象被任一strong指针引用，那么它将不会被销毁。当对象没有被任何strong指针引用时，那么就将被销毁。

## 默认行为

对象默认为声明为 strong 类型, ARC 确保对象在函数体内是不会被 dealloc。比如

``` objc
- (void)takeLastNameFrom:(Person *)person {
    NSString *oldLastname = [self lastName];
    [self setLastName:[person lastName]];
    NSLog(@"Lastname changed from %@ to %@", oldLastname, [self lastName]);
}

```

## 强制规则

* 你不能再调用 dealloc 或者实现、调用 retain, release, retainCount, autorelease, 同样@selector(retain), @selector(release)也是不允许的，
  当然你可以实现 dealloc 方法，来管理你的实例变量，或者你会调用 [systemClassInstance setDelegate:nil]等.定制的 dealloc 方法不需要写 [super dealloc],这个动作会默认调用。 

  你仍然可以用 CFRetain, CFRelease 和相关Core Foundation方法。 如果要管理这类对象可以参考：(Managing Toll-Free Bridging 管理自由桥接) 

* 你不能使用 NSAllocateObject 或 NSDeallocateObject
  
  创建对象用 alloc

* 你不能在 C 结构体中使用对象指针
 
  因此下面代码是不可用的
  
``` objc
typedef struct {
    UIImage *selectedImage;
    UIImage *disabledImage;
} ButtonImages;

```
  建议是用 OC类来管理它们

* 不能随意在 id 与 void * 之间随意转换

  编译器同样是无法管理 void * 这类 Core Foundation类型的东东，都要用 Managing Toll-Free Bridging 进行生命同期的管理。

* 你不能再使用 NSAutoreleasePool 对象

  ARC 提供了性能更好的 @autoreleasepool block 替换原来这类使用方式。

* 你不能使用内存区

  你不需要再使用 NSZone 等这类对象，因为现在Objective-C运行时已经忽略NSZone了，所以没必要再使用NSZone了。

* 你不能使用 new 开头的属性名，但你可以手动指定 getter 方法名

  例如:

``` objc
// 非法:
@property NSString *newTitle;

// OK:
@property (getter=theNewTitle) NSString *newTitle;
```  

# ARC 新的对象生命周期声明

* 属性

``` objc
// 用 strong 代替 retain, @property(retain) MyClass *myObject;
@property(strong) MyClass *myObject;
 
// 用 weak 代替 assign "@property(assign) MyClass *myObject;"
// 实例变量被释放后，会自动赋予 nil 指针，省得我们自己在手动赋nil操作。
@property(weak) MyClass *myObject;

```

在 ARC 中 strong 将是默认的类型.

* 变量

变量同样有以下几种管理生命周期的声明

``` objc
__strong
__weak
__unsafe_unretained
__autoreleasing
```

__unsafe_unretained 类似原来的 assign

所以你可以这样声明这些对象

``` objc
MyClass * __weak myWeakReference;
MyClass * __unsafe_unretained myUnsafeReference;
```

需要注意的是 __weak 变量在栈中的情况，例如:

``` objc
NSString * __weak string = [[NSString alloc] initWithFormat:@"First Name: %@", [self firstName]];
NSLog(@"string: %@", string);
```

尽管 string 被实例化，但由于 string 声明为 weak 类型，它没有 strong 这个引用，所以他在赋值后立即就被释放了,在Log它时，它已经被释放了。

## 同样你也要注意对象值传递. 比如下面的代码

``` objc
NSError *error;
BOOL OK = [myObject performOperationWithError:&error];
if (!OK) {
    // Report the error.
    // ...
```

实际上这个代码是这样隐示声明的
``` objc
NSError * __strong e;
```

而函数是这样被声明了的

``` objc
-(BOOL)performOperationWithError:(NSError * __autoreleasing *)error;
```

所以最后编译的结果就是:

``` objc
NSError * __strong error;
NSError * __autoreleasing tmp = error;
BOOL OK = [myObject performOperationWithError:&tmp];
error = tmp;
if (!OK) {
    // Report the error.
    // ...
```

当本地变量声明(__strong *error)和函数的参数((NSError * __autoreleasing *)error)不匹配的时候，编译器会创建一个临时变量。当你获得一个__strong变量的地址时，你可以初始化一个id __strong *的指针来声明 ，这样你就可以获得指针的原型，或者你可以声明一个变量为 __autoreleasing。

# 避免循环引用

你可以使用生命周期修饰符来避免Strong引用周期。例如，当你制作了一组父子结构的对象，而且父类要引用子类，则会出现Strong引用周期；反之，当 你将一个父类指向子类为strong引用，子类指向父类为weak引用，就可以避免出现Strong引用周期。当对象包含block objects时，这样的情况会变的更加隐性。

在手动内存管理模式下， ``` __block id x ```; x不会被 retaining
在ARC模式下，``` __block id x ``` , 默认被retaining

为了使手动内存管理模式代码可以在ARC模式下正常工作， 你可以用 ``` __unsafe_unretained ``` 来修饰 ``` __block id ``` x;。就和"__unsafe_unretained"字面上的意思一样, 不过,这样一个non-retained变量是危险的(因为它会变成一个野指 针) 会带来不良后果。有两种更好一点的方法来处理，一是使用__weak (当你不需要支持iOS 4或OS X v10.6), 二是设__block值为nil，结束他的生命周期。

* 这是MRC时代处理 __block 里对象释放问题:

``` objc
MyViewController *myController = [[MyViewController alloc] init…];
// ...
myController.completionHandler =  ^(NSInteger result) {
   [myController dismissViewControllerAnimated:YES completion:nil];
};
[self presentViewController:myController animated:YES completion:^{
   [myController release];
}];
```

你可以使用 __block修饰符然后设置myController的值为nil 替代上面的方式:

``` objc
MyViewController * __block myController = [[MyViewController alloc] init…];
// ...
myController.completionHandler =  ^(NSInteger result) {
    [myController dismissViewControllerAnimated:YES completion:nil];
    myController = nil;
};
```

无伦哪种形式，你都可以使用一个 weak 引用对象避免循环引用:

``` objc
MyViewController *myController = [[MyViewController alloc] init…];
// ...
MyViewController * __weak weakMyViewController = myController;
myController.completionHandler =  ^(NSInteger result) {
    [weakMyViewController dismissViewControllerAnimated:YES completion:nil];
};
```

在某个时候这个对象，如果放在异步执行时，对象可能已经被释放，所以需要一个 strong 的对象把它 hold 住。

``` objc
MyViewController *myController = [[MyViewController alloc] init…];
// ...
MyViewController * __weak weakMyController = myController;
myController.completionHandler =  ^(NSInteger result) {
    MyViewController *strongMyController = weakMyController;
    if (strongMyController) {
        // ...
        [strongMyController dismissViewControllerAnimated:YES completion:nil];
        // ...
    }
    else {
        // Probably nothing...
    }
};
```

## 栈里的变量初始化即为 nil
使用ARC后， strong, weak, autoreleasing 栈里的变量默认初始为nil
``` objc
- (void)myMethod {
    NSString *name; // 这里 name 已经被赋予了nil指针， 所以下面的代码不会出错。
    NSLog(@"name: %@", name);
}
```


## 修改编译的Flag 打开和关闭 ARC

如果有遇到第三方插件，或有一些文件你不想用 ARC 来控制，可以在 Build Phases > Compile Sources > 某个文件上 > Compiler Flags: -fno-objc-arc

相互如果想在部分文件中用到 arc 则标记上: -fobjc-arc

![Compiler flags for arc](/assets/compiler_flags_arc.png)


## Managing Toll-Free Bridging

由于ARC不能管理Core Foundation Object的生命周期，所以在Core Foundation和ARC之间，我们需要使用到__bridge,__bridge_retained和__bridge_transfer三个转换关键字。

__bridge只做类型转换，但是不修改对象（内存）管理权；

__bridge_retained（也可以使用CFBridgingRetain）将Objective-C的对象转换为Core Foundation的对象，同时将对象（内存）的管理权交给我们，后续需要使用CFRelease或者相关方法来释放对象；

__bridge_transfer（也可以使用CFBridgingRelease）将Core Foundation的对象转换为Objective-C的对象，同时将对象（内存）的管理权交给ARC。


## 使用weak property声明Outlet

在被ARC处理过的iOS和OS X中，声明的outlets将会趋于统一。

一般来说outlets变量被修饰为weak，但是如果outlets变量的所有者是nib文件中的top-level对象(或者是storyboard scene)时，应被修饰为strong。

详细参考Resource Programming Guide中的“Nib Files”。

当我们使用 Interface Builder 生成Outlet对象的时候，一般都是作为 subview 来使用的。比如 UIViewController 的view。所以说Outlet的持有者就是superview对象，即有“父子”关系。我们知道，当对象间有“父子”关系时，需要使用弱参照，以避免“循环参照”。

ViewController 本身是不会作为Outlet的所有者的，所以使用weak property声明。

![arc outlet weak property](/assets/arc_outlet_weak_property.png)

简化viewDidUnload

Outlet都使用weak property声明的时候，还有一个好处，就是简化viewDidUnload的处理。

iOS在系统内存不足的时候，UIViewController会将没有表示的所有view做unload处理，即调用viewDidUnload接口。

所以，如果是强参照的情况下，需要释放所有权，

``` objc
@property (nonatomic, strong) IBOutlet UILabel *label;

-(void) viewDidUnload {
    self.label = nil; // 取消强参照，释放所有权
    [super viewDidUnload];
}
```

如果没有 self.label = nil 的处理，那么 UIViewController 将不会释放 label 的所有权；结果，系统是调用了unload，但是subview对象始终留在内存中。随着界面上控件的增多，内存泄露会越来越大。

如果使用的是weak property声明的话，会是怎样的呢？

``` objc
@property (nonatomic, weak) IBOutlet UILabel *label;
```

这时，系统在unload时，由于label没有被强参照，更加ARC的规则，这时，label的对象即被释放。并在释放的同时，变量自动指向nil。

``` objc
- (void)viewDidUnload {
    // 这里什么也不用管
    [super viewDidUnload];
}
```

其实，如果我们的viewDidUnload只是用来释放Outlet用的话，那么该函数也可以不被重载的。

什么时候要用strong property

由上我们也可以看到，并不是所有的Outlet都用weak来声明都是正确的；当使用Interface Builder生成的第一层的view或者windows被作为Outlet来使用的话，那么不是不能声明为weak property的。（比如，Storyboard的各个scene）

# 转化原MRC项目到ARC

* 用Xcode自带工具转换MRC项目到ARC:
  Edit > Refactor > Convert to Objective-C ARC)

  ![Provides a tool that convert to ARC](/assets/tool_for_convert_to_oc_arc.png)

  在这个选项下，还有一个 Convert to Modern Objective-C Syntax.. 转化成更现代的写法, 有兴趣的可以试试。:)

  在转化的过程中，编译器会先对代码进行检查，如果遇到错误警告，可以根据提示进行处理后，再进行转化 (如果你要无视这些错误可以在Preferences 里设定 Continue building after errors)

* 将项目用ARC方式编译
  Build Settings -> LLVM compiler 将 Objective-C Automatic Reference Counting 设置为 Yes

  ![Convert to ARC in LLVM](/assets/convert_to_arc_in_llvm.png)

# 常见问题

* 通常遇到的错误有这样一些：

  ``` Receiver type ‘X’ for instance message is a forward declaration ```

  这往往是引用的问题。ARC要求完整的前向引用，也就是说在MRC时代可能只需要在.h中申明@class就可以，但是在ARC中如果调用某个子类中未覆盖的父类中的方法的话，必须对父类.h引用，否则无法编译。

  ``` Switch case is in protected scope ```

  现在switch语句必须加上{}了，ARC需要知道局部变量的作用域，加上{}后switch语法更加严格，否则遇到没有break的分支的话内存管理会出现问题。

  ``` A name is referenced outside the NSAutoreleasePool scope that it was declared in ```

  这是由于写了自己的 autoreleasepool，而在转换时在原来的pool中申明的变量在新的@autoreleasepool中作用域将被局限。解决方法是把变量申明拿到pool的申请之前。

  ``` ARC forbids Objective-C objects in structs or unions ```
  
* ARC 需要你指定 super init 的结果到 self
   [super init]; // 这将是无效的

   推荐用 
``` objc
   self = [super init];
   if (self) {
	...
```

实例变量会变成 strong 类型

在用 ARC 之前, thing 这个变量是一个 weak 类型

``` objc
@interface MyClass : Superclass {
    id thing; // Weak reference.
}
// ...
@end
 
@implementation MyClass
- (id)thing {
    return thing;
}
- (void)setThing:(id)newThing {
    thing = newThing;
}
// ...
@end
```

使用 ARC 后，thing 变量实际上是默认用了 strong 类型，所以如果你要想继续使用 weak 类型，必须显示声明

``` objc
@interface MyClass : Superclass {
    id __weak thing;
}
// ...
@end
 
@implementation MyClass
- (id)thing {
    return thing;
}
- (void)setThing:(id)newThing {
    thing = newThing;
}
// ...
@end
```

或

``` objc
@interface MyClass : Superclass
@property (weak) id thing;
// ...
@end
 
@implementation MyClass
@synthesize thing;
// ...
@end
```

* 首先，我们需要转变一下观念, 对于在.h中申明的实例变量：

``` objc
@interface MainViewController : UIViewController  
{ 
	NSOperationQueue *queue;
}
```

我们不妨仔细考虑一下，为什么在interface里出现了实例变量的申明？通常来说，实例变量只是在类的实例中被使用，而你所写的类的使用者并没有太多必要了解你的类中有哪些实例变量。而对于绝大部分的实例变量，应该都是protected或者private的，对它们的操作只应该用setter和getter，而这正是property所要做的工作。可以说，将实例变量写在头文件中是一种遗留的陋习。更好的写实例变量名字的地方应当与类实现关系更为密切，为了隐藏细节，我们应该考虑将它们写在@implementation里。好消息是，在LLVM3.0中，不论是否开启ARC，编译器是支持将实例变量写到实现文件中的。甚至如果没有特殊需要又用了property，我们都不应该写无意义的实例变量申明，因为在@synthesize中进行绑定时，我们就可以设置变量名字了，这样写的话可以让代码更加简洁。

在这里我们对实例变量申明移到.m里中。修改后的.h是这样的，十分简洁

``` objc
@implementation MainViewController 
{ 
    NSOperationQueue *queue;  
}
```

这样的写法让代码相当灵活，而且不得不承认.m确实是这些实例变量的应该在的地方


参考资料:

* [About Memory Management](http://developer.apple.com/library/ios/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html#//apple_ref/doc/uid/10000011i)

![memory managment](http://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/MemoryMgmt/Art/memory_management_2x.png)

* [Transitioning to ARC](http://developer.apple.com/library/ios/#releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)

* [Objective-C Automatic Reference Counting](http://clang.llvm.org/docs/AutomaticReferenceCounting.html#blocks)