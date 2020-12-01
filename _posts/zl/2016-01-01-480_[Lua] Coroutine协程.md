---
layout: post
title: [Lua] Coroutine协程 
tags: [lua文章]
categories: [topic]
---
## Coroutine Manipulation

### coroutine.create (f)

Creates a new coroutine, with body f. f must be a function. Returns this new
coroutine, an object with type “thread”.

创建一个新的协程。f必须是一个函数。返回值是这个新的协程，返回值的类型是thread。

### coroutine.isyieldable ()

Returns true when the running coroutine can yield.

A running coroutine is yieldable if it is not the main thread and it is not
inside a non-yieldable C function.

返回true当正在运行的协程可以被yield。

### coroutine.resume (co [, val1, ···])

Starts or continues the execution of coroutine co. The first time you resume a
coroutine, it starts running its body. The values val1, … are passed as the
arguments to the body function. If the coroutine has yielded, resume restarts
it; the values val1, … are passed as the results from the yield.

If the coroutine runs without any errors, resume returns true plus any values
passed to yield (when the coroutine yields) or any values returned by the body
function (when the coroutine terminates). If there is any error, resume
returns false plus the error message.

### coroutine.running ()

Returns the running coroutine plus a boolean, true when the running coroutine
is the main one.

返回一个正在运行的协程和一个boolean值，这个协程是主线程的话boolean值为true。

### coroutine.status (co)

Returns the status of coroutine co, as a string: “running”, if the coroutine
is running (that is, it called status); “suspended”, if the coroutine is
suspended in a call to yield, or if it has not started running yet; “normal”
if the coroutine is active but not running (that is, it has resumed another
coroutine); and “dead” if the coroutine has finished its body function, or if
it has stopped with an error.

### coroutine.wrap (f)

Creates a new coroutine, with body f. f must be a function. Returns a function
that resumes the coroutine each time it is called. Any arguments passed to the
function behave as the extra arguments to resume. Returns the same values
returned by resume, except the first boolean. In case of error, propagates the
error.

### coroutine.yield (···)

Suspends the execution of the calling coroutine. Any arguments to yield are
passed as extra results to resume.

挂起调用这个函数的协程。参数会传递给resume，作为resume的返回值。

## 例子

### 例子1

    
    
    co = coroutine.create(
        function(i)
            print(i);
        end
    )
    
    print(coroutine.status( co ))
    coroutine.resume(co, 1)
    print(coroutine.status( co ))
    
    print("---------------")
    
    co = coroutine.wrap(
        function(i)
            print(i);
        end
    )
    
    co(2)
    
    print("---------------")
    
    co2 = coroutine.create(
        function()
            for i = 1, 10 do
                print(i)
                if i == 3 then
                    print(coroutine.status(co2))
                    print(coroutine.running( ))
                end
                coroutine.yield()
            end
        end
    )
    
    coroutine.resume(co2)
    coroutine.resume(co2)
    coroutine.resume(co2)
    
    print(coroutine.status(co2))
    print(coroutine.running( ))
    

输出：

    
    
    suspended
    1
    dead
    ---------------
    2
    ---------------
    1
    2
    3
    running
    thread: 0000000000459dd8        false
    suspended
    thread: 0000000000456638        true
    

### 例子2

    
    
    function foo(a)
        print("foo a = ", a)
        return coroutine.yield(2 * a)
    end
    
    co = coroutine.create(
        function(a, b)
            print("1==== ", a, b)
            local r = foo( a + 1 )
    
            print("2==== ", r)
            local r, s = coroutine.yield(a + b, a - b)
    
            print("3==== ", r, s)
            return b, "end of coroutine."
        end
    )
    
    print("main", coroutine.resume(co, 1, 10))
    print("--------------")
    
    print("main", coroutine.resume(co, "abc"))
    print("--------------")
    
    print("main", coroutine.resume(co, "xxx", "yyy"))
    print("--------------")
    
    print("main", coroutine.resume(co, "xxx", "yyy"))
    print("--------------")
    

输出：

    
    
    1====   1       10
    foo a =         2
    main    true    4
    --------------
    2====   abc
    main    true    11      -9
    --------------
    3====   xxx     yyy
    main    true    10      end of coroutine.
    --------------
    main    false   cannot resume dead coroutine
    --------------
    

### 例子3：生产者，消费者

    
    
    local newProductor
    
    function productor()
        local i = 0
        while true do
            i = i + 1
            if i > 10 then
                break;
            end
            send(i)
        end
    end
    
    function consumer()
        while true do
            local s, i = receive()
            if s == false then
                break;
            end
            print(i)
        end
    end
    
    function receive()
        local status, value = coroutine.resume(newProductor)
        return status, value
    end
    
    function send(x)
        coroutine.yield(x)
    end
    
    newProductor = coroutine.create( productor )
    consumer()
    
    

输出：

    
    
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
    nil