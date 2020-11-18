---
layout: post
title: cocos2dx lua 多线程之协成理解与实战 
tags: [lua文章]
categories: [lua文章]
---
lua是不支持多线程的，一般都是协同来调用的。但是lua却可以调用c函数。于是，我们通过lua调用C接口起一个线程，实现lua多线程的使用。子线程再调用lua中的function。就可以通过子线程获取一些数据。单纯的人儿，以为一切都是美好的。

> 问题就出现C调用lua中的function, 将数据传给lua。

### lua的运行

首先我们需要知道，lua是解释性语言。是在执行的时候才分配堆栈空间。通过查看lua的源码，我们可以知道，在main函数的开端，lua就创建了一个全局的L（状态机），这个状态机可以说是lua的核心所在。它保存了栈的地址。

当执行lua脚本时，lua会将全局的变量和function记录在堆中，当执行代码段是，就会将一些局部变量和参数压到栈中进行处理。这一切和c语言的解析是一样的。

> 我们知道C也是可以调用lua的function的，一般的操作是：
>

>>   1. 在lua中调用C函数，将需要注册的function，作为参数传给C函数

>>   2. C将获取到的function和L（状态机）进行保存。

>>   3. C通过向L压栈，将function和一些参数压入。通过lua_call函数进行调用。

>>

> 根据上述的解释，我们可以知道。其中C和lua通过通信的是L（状态机）。压入栈之后，通过lua_call,就会进入lua的状态中。lua会处理栈中的内容。

### 问题所在

核心问题就是C调用lua的L和lua的L是同一个L。这样就出现一个问题，当主线程的lua脚本才进行压栈操作，而子线程中也进行压栈操作，那岂不是乱了套？在一开始就不应该成功的，为什么会这样呢？通过查看代码，发现lua对进行堆操作的函数中，都加上了线程锁。当主线程进行栈操作时，子线程是不可以对栈进行操作的。

也就是说，子线程理论上是不会运行的，会卡在栈操作的函数那里。

> 但是为什么我们在运行的时候并没有出现这个现象呢？通过代码的查询，发现是主线程中有sleep函数，并且子线程中有阻塞，所以能够在几个线程中切换。
> 如果主线程的while循环中没有sleep，那么就会很快的出现问题。因此，lua从底层就是不支持多线程的。

### 为什么使用协同

如果你搜索lua多线程，大多数都会写搜索到协同程序。

    
    
    每一个协程有自己的堆栈，自己的局部变量，可以通过yield-resume实现在协程间的切换。不同之处是：Lua协程是非抢占式的多线程，必须手动在不同的协程间切换，且同一时刻只能有一个协程在运行。并且Lua中的协程无法在外部将其停止，而且有可能导致程序阻塞。
    

正如上诉，协同拥有自己的堆栈，那是用来避免和其他堆栈冲突的。但是两者之间想要通信，就不能通过栈了。因为栈的不同，压入的数据在另一端是无法接收到的。

但是，我们可以通过一个全局变量进行通信。比如，子线程通过协同的堆栈进行调用lua里面的function。在function中获取传入的值，将它赋值给一个全局变量。那么主线程也能够调用了。

### 线程与协同

协同程序与线程thread差不多，也就是一条执行序列，拥有自己独立的栈、局部变量和命令指针，同时又与其他协同程序共享全局变量和其他大部分东西。可以通过yield-
resume实现在协程间的切换。

>
> 从概念上讲线程与协同程序的主要区别在于，一个具有多个线程的程序可以同时运行几个线程，而协同程序却需要彼此协作的运行。也就是说多个协同程序在任意时刻只能运行一个协同程序，只有当正在运行的协同程序显式的要求挂起时，它的执行才会暂停。

  * 总结区别：

    * 不同之处是：Lua协程是非抢占式的多线程，必须手动在不同的协程间切换，且同一时刻只能有一个协程在运行。并且Lua中的协程无法在外部将其停止，而且有可能导致程序阻塞。
  * 关于更多区别和介绍，可以查看这里

    * <https://www.cnblogs.com/work115/p/5620272.html>
    * <https://www.cnblogs.com/lxmhhy/p/6041001.html>

### 协同程序coroutine

Lua将所有关于协同程序的函数放置在一个名为“coroutine”的table中。

  * 1、coroutine.create创建一个thread类型的值表示新的协同程序，返回一个协同程序。
  * 2、coroutine.status检查协同程序的状态（挂起suspended、运行running、死亡dead、正常normal）。
  * 3、coroutine.resume启动或再次启动一个协同程序，并将其状态由挂起改为运行。
  * 4、coroutine.yield让一个协同程序挂起。
  * 5、coroutine.wrap同样创建一个新的协同程序，返回一个函数。

> 注意coroutine的三个状态：  
> suspended（挂起，协同刚创建完成时或者yield之后）、  
> running（运行）、  
> dead（函数走完后的状态，这时候不能再重新resume）。

##### 创建协同程序：

create函数，接受一个函数值作为协同程序的执行内容，并返回一个协同程序。

    
    
    function func( ... )
     print("iCocos")
     -- thread: 0x79f721d4
     -- [Finished in 0.0s]
    end
    
    local  cor  = coroutine.create(func)
    print(cor)
    

##### 启动或再次启动一个协同程序：

resume函数，接受一个协同程序及一个或多个参数用于值传递给协同程序。

    
    
    function funcA(  _cor, ... )
     print("A: status_1",coroutine.status(_cor), ...)
     -- A: status_1  running 1   2   3
    end
    local corA = coroutine.create(funcA)
    coroutine.resume(corA, corA, 1,2,3)
    print("A: status_2", coroutine.status(corA))
    -- A: status_2   dead
    

##### resume-yield数据交换

Lua中协同的强大能力，还在于通过resume-yield来交换数据：

  * （1）resume把参数传给程序（相当于函数的参数调用）；
  * （2）数据由yield传递给resume;
  * （3）resume的参数传递给yield；
  * （4）协同代码结束时的返回值，也会传给resume

协同中的参数传递形势很灵活，一定要注意区分，在启动coroutine的时候，resume的参数是传给主程序的；在唤醒yield的时候，参数是传递给yield的。

> 挂起协同程序：yield函数，让一个协同程序挂起，并等待下次恢复它的运行。它可以接受resume函数传递进来的所有参数。
    
    
    -- resume  yield 参数传递
    function funcB( _cor )
    print("A: status_1", coroutine.status(_cor))
    ptint("A: status_2", coroutine.yield()) -- 挂起
    end
    
    local funcB = coroutine.create(funcB) -- wrap: wrap函数比create函数更易使用。它提供了一个对于协同程序编程实际所需的功能，即一个可以唤醒协同程序的函数。但也缺乏灵活性。无法检查wrap所创建的协同程序的状态，此外，也无法检测出运行时的错误。
    coroutine.resume(funcB, funcB) -- 启动，没有yield，参数属于主函数
    print("A: status_3", coroutine.status(funcB))
    coroutine.resume(funcB, 1,2,3) -- 从挂起出启动，并给yield传递参数
    print("A: status_4", coroutine.status(funcB))
    

Lua提供的是一种：”非对称的协同程序“。也就是说，Lua提供了两个函数来控制协同程序的执行，一个用于挂起执行，另一个用于恢复执行。而一些其他的语言则提供了”对称的协同程序“，其中只有一个函数用于转让协同程序之间的执行权。

### 管道与过滤器filter

关于协同程序的示例就是”生产者–消费者“的问题。其中涉及到两个函数，一个函数不断的产生值，另一个函数不断的消费这些值。

> 当消费者需要一个新的值时，它唤醒生产者。生产者返回一个新值后停止运行，等待消费者的再次唤醒。这种设计称为”消费者驱动“。通过resume—yield
> 函数之间的值交换可以轻易的实现程序。

过滤器filter，是一种位于生产者与消费者之间的处理功能，可以进行数据转换。它既是消费者又是生产者，它唤醒生产者促使其生产新值，然后又将变换后的值传递给消费者。

    
    
    --管道与过滤器filter
    --生产者与消费者通过过滤器进行值传递
    --这种模式通过消费者驱动生产者进行产生。
    
    --计数器函数
    function getCount( x )
    return function()
    x=x+1
    return x
    end
    end
    --创建闭合计数器
    local count = getCount(0)
    --发送新值
    function send(x)
    coroutine.yield(x)
    end
    --启动一个协同程序
    function receive( pro )
    local status,value = coroutine.resume( pro )
    return value
    end
    --生产者
    function producter()
    while true do
    send( count() )
    end
    end
    --过滤器，接受一个生产者
    function filter( pro )
    local x = 0
    return function()
    while true do
    x = receive( pro )
    send(x)
    end
    end
    end
    --消费者，接受一个生产者协同程序及控制条件，控制条件防止死循环
    --假设有100个消费者，驱动生产者来生产
    function consumer( pro,num )
    local x = 0
    while x < num do
    x = receive( pro )
    print( x )
    end
    end
    
    local pro = coroutine.create( producter )
    local fil = coroutine.create( filter( pro ) )
    consumer( fil,100 )
    
    print( "消费者协同程序状态：",coroutine.status(pro) )
    print( "生产者协同程序状态：",coroutine.status(fil) )
    

打印结果

    
    
    1
    2
    3
    ...
    -- 消费者协同程序状态：   suspended
    -- 生产者协同程序状态：   suspended
    

  * 推荐
    * <https://github.com/lichuang/Lua-Source-Internal/blob/master/doc/ch09-%E5%8D%8F%E7%A8%8B.md>