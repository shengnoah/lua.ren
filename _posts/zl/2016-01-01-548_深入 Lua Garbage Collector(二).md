---
layout: post
title: 深入 Lua Garbage Collector(二) 
tags: [lua文章]
categories: [topic]
---
这一篇我们主要介绍一下 **Lua** 的 **GC** 机制

## Lua垃圾回收器函数

 **Lua** 提供了以下函数 **collectgarbage ([opt [, arg]])** 用来控制自动内存管理:

  * collectgarbage(“collect”): 做一次完整的垃圾收集循环。通过参数 opt 它提供了一组不同的功能：

  * collectgarbage(“count”): 以 K 字节数为单位返回 Lua 使用的总内存数。 这个值有小数部分，所以只需要乘上 1024 就能得到 Lua 使用的准确字节数（除非溢出）。

  * collectgarbage(“restart”): 重启垃圾收集器的自动运行。

  * collectgarbage(“setpause”): 将 arg 设为收集器的 间歇率 。 返回 间歇率 的前一个值。

  * collectgarbage(“setstepmul”): 返回 步进倍率 的前一个值。

  * collectgarbage(“step”): 单步运行垃圾收集器。 步长”大小”由 arg 控制。 传入 0 时，收集器步进（不可分割的）一步。 传入非 0 值， 收集器收集相当于 Lua 分配这些多（K 字节）内存的工作。 如果收集器结束一个循环将返回 true 。

  * collectgarbage(“stop”): 停止垃圾收集器的运行。 在调用重启前，收集器只会因显式的调用运行。

以下演示了一个简单的垃圾回收实例:

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8
    
    9
    
    10
    
    11

|

    
    
    mytable = {"apple", "orange", "banana"}
    
    print(collectgarbage("count"))
    
    mytable = nil
    
    print(collectgarbage("count"))
    
    print(collectgarbage("collect"))
    
    print(collectgarbage("count"))  
  
---|---  
  
* * *

## 基本算法

基本算法就是我们之前的 **Mark & Sweep**

> 首先，系统管理着所有已经创建了的对象。每个对象都有对其他对象的引用。 **root** 集合代表着已知的系统级别的对象引用。我们从 **root**
> 集合出发，就可以访问到系统引用到的所有对象。而没有被访问到的对象就是垃圾对象，需要被销毁。

我们可以将所有对象分成三个状态：

①. White状态，也就是待访问状态。表示对象还没有被垃圾回收的标记过程访问到。

②. Gray状态，也就是待扫描状态。表示对象已经被垃圾回收访问到了，但是对象本身对于其他对象的引用还没有进行遍历访问。

③. Black状态，也就是已扫描状态。表示对象已经被访问到了，并且也已经遍历了对象本身对其他对象的引用

伪代码如下：

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8
    
    9
    
    10
    
    11
    
    12
    
    13
    
    14
    
    15
    
    16
    
    17

|

    
    
    当前所有对象都是White状态;  
    
    将root集合引用到的对象从White设置成Gray，并放到Gray集合中;  
    
    while  
    
    {  
    
        从Gray集合中移除一个对象O，并将O设置成Black状态;  
    
        for(O中每一个引用到的对象O1) {  
    
            if(O1在White状态) {  
    
                将从White设置成Gray，并放到到Gray集合中；  
    
            }  
    
        }  
    
    }  
    
    for(任意一个对象O){  
    
        if(O在White状态)  
    
            销毁对象O;  
    
        else  
    
            将O设置成White状态;  
    
    }  
  
---|---  
  
* * *

## Incremental Garbage Collection

>
> 上面的算法如果一次性执行，在对象很多的情况下，会执行很长时间，严重影响程序本身的响应速度。其中一个解决办法就是，可以将上面的算法分步执行，这样每个步骤所耗费的时间就比较小了。我们可以将上述算法改为以下下几个步骤:

  1. 首先标识所有的 **root** 对象

  2. 遍历访问所有的 **gray** 对象。如果超出了本次计算量上限，退出等待下一次遍历

  3. 销毁垃圾对象

>
> 在每个步骤之间，由于程序可以正常执行，所以会破坏当前对象之间的引用关系。black对象表示已经被扫描的对象，所以他应该不可能引用到一个white对象。当程序的改变使得一个black对象引用到一个white对象时，就会造成错误。解决这个问题的办法就是设置barrier。barrier在程序正常运行过程中，监控所有的引用改变。如果一个black对象需要引用一个white对象，存在两种处理办法：

①. 将white对象设置成gray，并添加到gray列表中等待扫描。这样等于帮助整个GC的标识过程向前推进了一步。  
②. 将black对象该回成gray，并添加到gray列表中等待扫描。这样等于使整个GC的标识过程后退了一步。

> 这种垃圾回收方式被称为 **Incremental Garbage Collection** (简称为 **IGC** ， **Lua**
> 所采用的就是这种方法。使用 **IGC** 并不是没有代价的。 **IGC** 所检测出来的垃圾对象集合比实际的集合要小，也就是说，有些在 **GC**
> 过程中变成垃圾的对象，有可能在本轮 **GC** 中检测不到。不过，这些残余的垃圾对象一定会在下一轮 **GC** 被检测出来，不会造成泄露。

在下一篇中我们将会具体的来看一看 **Lua** 的 **GC源码**