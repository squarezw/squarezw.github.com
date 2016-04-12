---
layout: post
title: "App 启动优化之一：main.m 方法之前的优化"
description: "App 启动优化系列"
category: 
tags: [App launch 启动优化]
---
{% include JB/setup %}

说到APP的启动优化，网上应该也有不少经验总结，不过就我所之相对有些零散，缺乏一些系统性的梳理 。为了让我们有一个完整的认识，也为了能和大家一起学习提高，这里我先抛砖引玉，引入第一篇文章从头梳理APP启动的各个环节，希望对大家有所帮助。

![Launch Seq](/assets/launch_seq.png)

我们首先要谈的是在开启一个 APP 的生命周期前，也就是进入 `main.m()` 方法之前，系统要经历怎样的一个加载过程呢？

如果从这里开始分析，就不得不谈到 iOS 操作系统内核的工作流程，它是如何解析我们的二进制包，如何分配内存，然后找到入口，进入 **main()** 方法中的呢？在这个过程中我们还有没有可优化的空间，或者要注意的事呢？


## 关于iOS系统加载APP过程

1. App启动流程的关键节点

	![Mach-O-Execution](/assets/mach-o_execution.png)

	引用自：`Jamin's Blog`

2. 可执行文件的内核流程如下图

	![flow of process execution](/assets/flow_of_process_execution.png)

	引用自《Mac OS X and iOS Internals : To the Apple's Core》

3. 对应到源代码里：

	由于源代码较多，只引用关键性的代码.

```
// /bsd/kern/ker_exec.c
execve(proc_t p, struct execve_args *uap, int32_t *retval) 
{
    __mac_execve(proc_t p, struct __mac_execve_args *uap, int32_t *retval)
    {
        exec_activate_image(struct image_params *imgp)
        {
            for(i = 0; error == -1 && execsw[i].ex_imgact != NULL; i++) {
                // 这里只分析Mach-O这种执行格式，所以exec_fat_imgact和exec_shell_imgact最终都会调到exec_mach_imgact                
                error = (*execsw[i].ex_imgact)(imgp); 
                
               (*execsw[i].ex_imgact)(imgp) = exec_mach_imgact(imgp)
                exec_mach_imgact(struct image_params *imgp)
                {
                    
                    load_machfile(struct image_params *imgp, ...) 
                    {	                    
                        
                        // 设置内存映射，关键点1
                        if (create_map) {
                            vm_map_create();
                        }	                       	                       
                        // 解析 Mach-O 包核心，关键点2
                   		parse_machfile(struct vnode *vp, ..., load_result_t *result)
                   		{
                       		...
                  		}
                        
                    }
                    
                    ...
                                        
                    // 设置入口点，关键点3
                    /* Set the entry point */
                    thread_setentrypoint(thread, load_result.entry_point);	                    	                    	  
                    ...
                }
                
            }
        }
    }
    
}
```

- 关键点1： vm_map_create() 设置内存映射，这和你的APP分配的初始内存大小有关
- 关键点2： 加载解析Mach-O文件，并得到APP入口,重点在这个函数: `parse_machfile()`，下面会细聊到。
- 关键点3：进入入口点（也就是main()函数的入口地址）

如想更深入研究，请下载[源代码](http://opensource.apple.com/tarballs/xnu/xnu-2782.1.97.tar.gz)

### 关于 `parse_machfile()` 函数做了什么:

- `parse_machfile()` 是递归解析的，最初的递归深度为0，最高深度到6，防止无限递归。使用递归解析，主要是将不同Mach-O文件类型按照依赖关系，分前后进行解析。如解析可执行二进制文件类型(MH_EXECUTABLE)的Mach-O文件需要调用load_dylinker来处理加载命令LC_LOAD_DYLINKER，而动态链接器也是Mach-O文件，所以就需要递归到不同的深度进行解析；
	
- 其次，`parse_machfile()` 的每一次递归，在解析加载命令时，会将内核需要解析的加载命令按照加载循序划分为三组进行解析，在代码的体现上就是通过三次循环，每趟循环只关注当前趟需要解析的命令： (1)：解析线程状态，UUID和代码签名。相关命令为LC_UNIXTHREAD、LC_MAIN、LC_UUID、LC_CODE_SIGNATURE (2)：解析代码段Segment。相关命令为LC_SEGMENT、LC_SEGMENT_64； (3)：解析动态链接库、加密信息。相关命令为：LC_ENCRYPTION_INFO、LC_ENCRYPTION_INFO_64、LC_LOAD_DYLINKER

- 解析完可执行二进制文件类型的Mach-O文件(假设为A)之后，我们会得到A的入口点；但线程并不立刻进入到这个入口点。这是由于我们还会加载动态链接器(dyld)，在`load_dylinker()`中，dyld会保存A的入口点，递归调用`parse_machfile()`之后，将线程的入口点设为dyld的入口点；动态链接器dyld完成加载库的工作之后，再将入口点设回A的入口点，程序启动完成；	

## 关于要解析的 APP 包

我们最终上线后的 APP是一组文件所组成，Apple 称它们为 **Bundle**

``` 	
xxx.app/
Documents/
Library/
tmp/
...
```     

**xxx.app** 就是我们的 app 应用程序，主要包含了执行文件，和一堆资源文件（图片，xib文件）等。其中的执行文件是 Mach-O(Mach-Object) 格式的二进制文件。
   
Mach-O文件可能有多种架构组成，因为不同的iOS设备对应的CPU架构也是不同的，所以为了得到最优的执行效率，这个二进制包会有多个不同的架构所组成的包，比如 armv7, armv7s。
   
Mach-O 文件是一个独有的二进制可执行文件格式。虽然iOS/OS X采用了类UNIX的Darwin操作系统核心，完全符合UNIX标准系统，但在执行文件上，却没有支持UNIX的[ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)。
   
![mach-o](/assets/mach-o_format.png)
   
   
## 深入分析 Load Commends

这些加载命令在Mach-O文件加载解析时，被内核加载器或者动态链接器调用，指导如何设置加载对应的二进制数据段。
   
接下来我们来对比一下，一个最简单的APP（XCode模板创建的Signle View Application）与之后又多添加了几个动态链库的不同，观察一下Load Commends：
   
![图1](/assets/mach-o_different.jpg)
 	
可以看到即使最简单的APP，也链接了一个 APP 所必需的动态链接库(UIKit等)。如果我们在XCode 里这样引入了更多的动态链接库，像这样：
 	 	 	 	 	 	
![图3](/assets/more_dylb.png)
 	
就会多出这几项的链接库。

*(ps: 一些黑客就是利用这一点来嵌入自己的动态链接库以达到修改APP的目的)*

**LC_MAIN** 对应的就是APP的入口。


## 总结可能存在的优化点

有了上述的分析我们可以总结一下可能存在的优化点。
	
1. **减少动态链接库**
	
	内核在到达入口之前，会先装载所有的动态链接库。所以尽可能用静态链接库。
	
2. **减少不必要的 ObjectC 对象**
	
	OC创建的类，会在DATA SEGMENT 中占用较多的 VM Size，即使你创建了一个空的Class。因为本身这个对象就会留下一空较大的内存区域来创建与管理，虽然现在的内存分配已经够快，但也会有一定的开销。
			
3. **特别注意 load() 方法里的内容**
	
	在 main 入口被解析执行到之前，所有的OC实现类都会先执行`load()`方法。如果你有定制这个方法，特别是在方法里有太多外部依赖时，也会额外增加解析的时间。所以如果有特别耗时的操作在这里请谨慎!


### 参考资料


- [Jamin's Blog](http://oncenote.com/2015/06/01/How-App-Launch/)
- [Mac OS iOS Internals Apples](http://www.amazon.com/Mac-OS-iOS-Internals-Apples/dp/1118057651)

---

以上观点仅代表个人，由于水平有限，可能有遗漏和不对的地方，欢迎一起来纠正，讨论！