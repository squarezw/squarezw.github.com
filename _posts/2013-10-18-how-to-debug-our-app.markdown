---
layout: post
title: "调试我们的iOS APP"
date: 2013-10-18 16:58
comments: true
categories: Debug
---
很早就想写一篇关于调试和测试的文章了。一直没写的原因一方面是工作太忙，一方面是现在新的调试工具和调试方法层出不穷，不知道什么时候现在用的方法就会过时，所以一直犹豫是不是等到有一个系统的方案出来在总结一下，不过我觉得还是现在先把能想到和用到写下来，让大家一起探讨。

**注：以下部分英文原文因本人翻译水平有限，不能给出准确的中文释义，所以将引用的原文进行了保留。**

关于如何来调试我们的APP，其实是可以从不同的维度来划分，比如应用调试，测试，错误分析等。我们先从最基本的调试工具入手一步步介绍。

## XCode Debug Tools

* **Debugger**
  
首先 Xcode 默认设置了你所用的 Debugger 

![Debugger Setting](/assets/setting_debugger.png)

通常我们这里用默认的[LLDB](http://lldb.llvm.org/),   XCode5之后默认就是用 LLDB了，XCode5以前还有 GDB。

使用LLDB之后，我们通常是在代码执行的地方，打上断点，如：

![Set BreakPoint](/assets/set_breakpoint.png)

这样程序运行遇到断点后会停下等待，这时我们可以在控制台，使用一些调试命令输出我们所需要的对象信息:

![Control NSLog](/assets/control_po.png)

还有很多命令:

```
  	[self.view recursiveDescription]  # 递归打印view
  	image lookup --address 0xffffff
```

更多调试命令可以参考上面的 LLDB 介绍，或用 help 命令显示帮助。

> 需要注意的是当Optimization Level不为None时，断点的定位会有差异，所以建议在Target 为 Debug 时调试更为准确，如果一定要在Release模式下调试，可以手动设置为Optimization Level 为 None

如果想看全部的信息，可以点这里选择:

![ALL Variables](/assets/all_variables.png)
  
---

* **Global Symbolic BreakPoints**

除了上面的指定断点设置，还可以设置Global Point, 作用域在当前用户的所有project，添加的Breakpoint 的方式有好几种，而且还可以以不同的方式呈现，比如日志输入，语音提醒等，可以在Action参数中设置。

BreakPoint 其中有几种设定类型，其中有：

1. Symbolic Breakpoint 

	我们可以添加指定的方法为断点。比如添加一个 viewDidLoad Symbol，会在运行到所有的viewDidLoad方法时停下. 如果你要添加某个特定类的实例方法，可以用 -[类名 实例方法名]。类方法是 +[类名 方法名]

	![Symbolic Breakpoint](/assets/symbolic_breakpoint.png)

 	如果你不知道这个方法格式应该如何书写，可以在你想要打断点的方法里先做断点，然后查看左边的Show the Debugger Navigator, 里的 Thread 指向的方法名:

	![Show The Debugger Navigator](/assets/show_the_debugger_navigator.png)

	看到 0 后面的方法调用了吗？


2. Exception Breakpoint

	另一个非常有用的断点设定, 

	在开发中除了用断点调试我们的应用分析问题外，还有一种情况是，我们向被释放的对象发送了消息，导致的crash (EXC_BAD_ACCESS)。

	关于Zombie:

	在 Cocoa 中，zombies 是一种即使生命终止了也会到惹麻烦的对象。我们可以做的是启动一个编译设置，使对象的引用计数降为0的时候不被释放，而是将它们转化为NSZombie对象。这个类的目的是记录任何对它的实例的调用，因为这意味着代码企图用一个已经消亡的对象调用方法。 

	通常我们的做法是通过设定 Exception Breakpoint 来统一排错，其中也包括了些类情况, 所以这个断点设定的方法来检查类似情况变得非常实用。

	![Exception Breakpoint Menu](/assets/exception_breakpoint_menu.png)

	如果添加这个类型的 Breakpoint , 好多隐藏很深的 Bug 都会被发现, 类似于 @try {} @catch {} 的 catch 部分

	![Exceptions Breakpoint](/assets/exceptions_breakpoint.png)


3. 如果使用GDB和更早版本的XCode的用户可以用下面的方式来检查：

	编辑Scheme, 将Diagnostics中的 Enable Zombie Objects 与 Malloc Stack 勾选上

	并选中 Enable Zombie Objects 与 Malloc Stack：

	当程序出现类似的问题而crash时，我们就可以找到被释放地址的真正原因

	![Enable Zombie Objects](/assets/enable_zombie_objects.png)


---

* **通过 Instrument 调试应用**

XCode 另外还自带了一个非常强大的APP调试工具： Instrument

关于 Instrument 的介绍，大家可以参考：
[Instrument Introduction](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/Introduction/Introduction.html)

![Instruments intro](/assets/Instruments_intro_ss.png)

*如果你在Schema Profile 中可以直接设置 Instrument 指定的分析工具，这样运行 Profile 时就可以直接启动它了*

Instrument 包括的小工具有好几种，我们这里先介绍几个: 

1. Leaks 内存泄露分析工具
  ![Instruments leaks](/assets/Instruments_leaks.png) [Leak Tutorial](http://www.mobileorchard.com/find-iphone-memory-leaks-a-leaks-tool-tutorial/)
2. Zombiles

---

* Analyze 分析代码

试一试通过选择 Menu => Product => Analyze。检测出可能会出现内存泄露的地方，重复引用，命名冲突等地方

---

* **自定义警告和错误提示**

我们还可以在代码中加入自定义的警告和错误提示，对于需要特别给某段代码加标注供日后处理时或出错判断时，可以在代码的上方加上

```#warning [message]```
在编译的时候会出现警告

```#error [message]```
在编译的时候会给出现错误提示


## Crash

由于使用Objective－c 和 c ，直接执行二进制指令，自己管理内存，会出现访问错误内存的情况出现。这时，系统会直接把你的进程干掉，iOS会给你生成一个Crash Log

* 关于crash时，如果显示的是堆栈信息，如何正确定位到程序部分
	
	```
	First throw call stack:
	(0x2f3a022 0x30cbcd6 0x2ee2a48 0x2ee29b9 0x2f392da 0x9cfd3 0x7f460 0x80a6e 0x103ba29 0x2f05855 0x2f05778 ...)
	```

  如何查看这些信息背后的真实情况，在main.m代码中加入以写部分：
  
```
  #ifdef DEBUGvoid eHandler(NSException*);

  void eHandler(NSException*exception){    
    NSLog(@"%@", exception);    
    NSLog(@"%@",[exception callStackSymbols]);
  }

  #endif


  int main(int argc,char*argv[]){
    #ifdef DEBUG    
    NSSetUncaughtExceptionHandler(&eHandler);
    #endif
    ...rest of your main function here...
  }
```

---

* 理解与分析 Crash Report

	在APP上线后，对 Crash Report 的监控是最为重要的环节了。itunes connect应用管理后台提供了部分的 Crash Reports，你可以在管理后台下载 .crash 文件，然后通过这个文件查找是哪儿引起的crash。

	但是这个文件中没有平时调试时候那样可以看到的函数名和函数具体调用行数。因为这里的这些信息都被转换成了16进制的地址，起到了一定的加密作用，别人拿到你的crash日志也不知道哪里崩溃了，需要利用你编译app的时候生成的dSYM文件然后将这些信息反转为可读模式。所以保留好你Archive后的dSYM文件是很有用的！

	如果你能找到 dSYM 文件，就可以利用symbolicatecrash工具查找具体的Bug发生地点了。

	```symbolicatecrash [CrashLog file] [dSYM file]```

*tips*
> 查找 symbolicatecrash 文件位置
> find /Applications/Xcode.app -name symbolicatecrash

* 设定 ```export DEVELOPER_DIR=/Application/Xcode.app/Contents/Developer/```

* 查看你的应用 uuid 与 dSYM的 uuid 是否能对应:

    dwarfdump --uuid yourapp.app/yourapp
    dwarfdump --uuid yourapp.app.dSYM

  * 搜索含有正确 uuid 的文件
    ```mdfind "com_apple_xcode_dsym_uuids == 5255A87A-B23C-3AE8-B367-14B49C21C1D6"```


## 分析

<!-- more -->

日志主要由几部分组成，其中:

* **Exception Type**

  * EXC_BAD_ACCESS (SIGSEGV) 

  	这个类型的Exception的意思是，你没有权限访问你所要访问的内存。一般都是由于访问了已经被release的object导致的，或者把一个object release了两次。甚至当你访问超出数组长度的内容时，也有可能出现这种类型的错误。它的意思应该是段错误。这个SIGSEGV不是objective－c的excption，而是更底层的C部分的信号。

  * EXC_CRASH (SIGKILL)或者(SIGABRT) 

  	这个类型的Exception比较特别，你需要认真查看后面所有Thread的BackTrace才能找到最终原因，因为有时候它所写的Crash Thread并不是真正引起崩溃的原因，在其中你也找不到什么有用的信息。(SIGABRT)一般是由于系统捕获到一个异常，然后把你的应用终结掉了，你可以在下面的栈信息中寻找有abort信息的那一个thread，能找到真正的原因。(SIGKILL)目前还没在自己的App中遇到过。

* **Exception codes**

  In the crash log is a line that starts with the text Exception Codes: followed by one or more hexadecimal values. These are processor-specific codes that may give you more information on the nature of the crash.


  1. The exception code **0xbaaaaaad** indicates that the log is a stackshot of the entire system, not a crash report. To take a stackshot, push the home button and any volume button. Often these logs are accidentally created by users, and do not indicate an error.

  2. The exception code **0xbad22222** indicates that a VoIP application has been terminated by iOS because it resumed too frequently.
  
  3. The exception code **0x8badf00d** indicates that an application has been terminated by iOS because a watchdog timeout occurred. The application took too long to launch, terminate, or respond to system events. One common cause of this is doing synchronous networking on the main thread. Whatever operation is on Thread 0: needs to be moved to a background thread, or processed differently, so that it does not block the main thread.

  4. The exception code **0xc00010ff** indicates the app was killed by the operating system in response to a thermal event. This may be due to an issue with the particular device that this crash occurred on, or the environment it was operated in. For tips on making your app run more efficiently, see iOS Performance and Power Optimization with InstrumentsWWDC session.
   
  5. The exception code **0xdead10cc** indicates that an application has been terminated by iOS because it held on to a system resource (like the address book database) while running in the background.
  
  6. The exception code **0xdeadfa11** indicated that an application has been force quit by the user. Force quits occur when the user first holds down the On/Off button until "slide to power off" appears, then holds down the Home button. It's reasonable to assume that the user has done this because the application has become unresponsive, but it's not guaranteed - force quit will work on any application.

* **一个单步分析 Crash Report 的方法**

  Steps to analyze crash report from apple:

  Copy the release .app file which was pushed to the appstore, the .dSYM file that was created at the time of release and the crash report receive from APPLE into a FOLDER.

  OPEN terminal application and go to the folder created above (using CD command)

    ```atos -arch armv7 -o YOURAPP.app/YOURAPP MEMORY_LOCATION_OF_CRASH.``` The memory location should be the one at which the app crashed as per the report.

  Ex: ```atos -arch armv7 -o 'app name.app'/'app name' 0x0003b508```

  This would show you the exact line, method name which resulted in crash.

  Ex: [classname functionName:]; -510

  Symbolicating IPA

  if we use IPA for symbolicating - just rename the extention .ipa with .zip , extract it then we can get a Payload Folder which contain app. In this case we don't need .dSYM file.


  或用 dwarfdump 命令也行

  ```dwarfdump –lookup 0x000036d2 –arch armv7 YOURAPP.app.dSYM```


参考文章:

- [Understanding and Analyzing iOS Application Crash Reports](https://developer.apple.com/library/ios/technotes/tn2151/_index.html)
- [理解Crash Log](http://www.whoslab.me/blog/?p=608)
- [KERN_INVALID_ADDRESS 与 KERN_PROTECTION_FAILURE 的区别](http://stackoverflow.com/questions/1282428/whats-the-difference-between-kern-invalid-address-and-kern-protection-failure)

一些第三方分析工具:

- [UMeng 日志监控](http://www.umeng.com/)
- [反汇编工具Hopper分析Crash Log](http://www.hopperapp.com/)
- [Crashlytics](http://blog.devtang.com/blog/2013/07/24/use-crashlytics/)


## 几种 Debug 输出方法

在Apple Tech Note TN2239：[iOS Debugging Magic](http://developer.apple.com/library/ios/#technotes/tn2010/tn2239.html) 中提到了程序开发中Debug output 方法：

    NSLog
    stderr
    system log

调试信息的输出主要有方式，一是通过输出到终端输出，二是输出到日志系统。下面讲介绍一下这几种输出调试信息的方式，首先从stderr说起。

* stderr （引用自TN2239）：

  Many programs, and indeed many system frameworks, print debugging messages to stderr. The destination for this output is ultimately controlled by the program: it can redirect stderr to whatever destination it chooses. However, in most cases a program does not redirect stderr, so the output goes to the default destination inherited by the program from its launch environment. This is typically one of the following:

  If you launch a GUI application as it would be launched by a normal user, the system redirects any messages printed on stderr to the system log. You    can view these messages using the techniques described earlier.
  If you run a program from within Xcode, you can see its stderr output in Xcode’s debugger Console window (choose the Console menu item from the Run menu to see this window).

  Attaching to a running program (using Xcode’s Attach to Process menu, or the attach command in GDB) does not automatically connect the program’s stderr to your GDB window. You can do this from within GDB using the trick described in the “Seeing stdout and stderr After Attaching” section of Technical Note TN2030, ‘GDB for MacsBug Veterans’.

  这样一段代码在真机上运行：

```
    NSLog(@"This is message from NSLog");
    fprintf(stderr, "This is message from stderr\n");
```
  
  如果是通过Xcode调试加载运行这个程序，那么

  在xcode的console中打印如下：

```
  2011-03-12 18:52:26.948 Test86[7891:307] This is message from NSLog```

  This is message from stderr
```

  在iPhone的system log中（通过Organizer的console查看）只打印

    Sat Mar 12 18:52:26 unknown Test86[7891] <Warning>: This is message from NSLog

  但是如果在iPhone上通过手指触摸启动这个程序，在iPhone的system log中会打印：

```
  Sat Mar 12 18:53:38 unknown Test86[7900] <Warning>: This is message from NSLog
  Sat Mar 12 18:53:38 unknown UIKitApplication:com.yourcompany.Test86[0x7d60][7900] <Notice>: This is message from stderr
```

  说明确实stderr在user 自己launch的app中被重定向为system log，而且log的等级为Notice；NSLog的等级为Warning。

 
* system log

  其实system log是unix系统都有采用syslog协议的一个日志系统（RFC详细讲解了这种协议http://tools.ietf.org/html/rfc5424）。每条日志是有等级的，主要分为如下等级：

```
    Level 0 – “Emergency”
    Level 1 – “Alert”
    Level 2 – “Critical”
    Level 3 – “Error”
    Level 4 – “Warning”
    Level 5 – “Notice”
    Level 6 – “Info”
    Level 7 – “Debug”
```

  在创建好日志之后，通过调用API发送日志信息给一个叫做syslogd的守护进程，然后syslogd根据自己的配置文件（位于/private/etc/syslog.conf, mac系统的在：/etc/asl.conf )

  在mac os和ios那么怎样调用API将日志发送给系统日志呢？有两种API：

- [syslog API](https://developer.apple.com/library/ios/#documentation/System/Conceptual/ManPages_iPhoneOS/man3/syslog.3.html#//apple_ref/doc/man/3/syslog) - *不要和之前syslog协议混淆*
- [ASL: Apple System Log facility](https://developer.apple.com/library/ios/#documentation/System/Conceptual/ManPages_iPhoneOS/man3/asl.3.html) - *是苹果自己实现的一种可以同syslogd服务器交互，用来替换syslog API的实现*

这里还有一些讲Syslog不错的文章：

- [Accesing the iOS system log](http://www.cocoanetics.com/2011/03/accessing-the-ios-system-log/)
- [Why ASL?](http://boredzo.org/blog/archives/2008-01-20/why-asl)

---
- [PonyDebugger: Remote Debugging Tools for Native iOS Apps](http://corner.squareup.com/2012/08/ponydebugger-remote-debugging.html) - *远程调试*
- Charles, Mitmproxy(免费) - *使用网络代理调试API Request*

#### 查看iOS 设备上的APP数据

![iOS Device Data](/assets/iOS_device_data.png)

