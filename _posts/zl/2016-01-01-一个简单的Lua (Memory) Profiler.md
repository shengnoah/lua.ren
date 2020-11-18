---
layout: post
title: 一个简单的Lua (Memory) Profiler 
tags: [lua文章]
categories: [lua文章]
---
`Lua`没有内置的`Profiler`，但是提供了一些相关的接口，可以用来实现一个简单的[Lua
Profiler](https://github.com/qq410029478/luaprofiler)。

一个`Profiler`至少需要统计以下信息, 用函数名+调用位置(保留一层堆栈信息)作为`key`:

  * 执行次数
  * 总时间
  * 单次最大时间
  * 尚未gc的内存数量
  * 分配内存的最大值

# 二、基础

出发点是[LuaProfiler](https://github.com/LuaDist/luaprofiler)，结构比较合理，但是有一些小问题：

  * 统计数据应该驻留在内存中，不能写`log`，太卡。
  * `time()`的精度太低，换成`PerformanceCounter`(windows)。
  * `lua5.3`和`5.1`的`lua_Hook`处理`tail call`的接口不一样，需要转换一下。
  * `coroutine`相关的处理。

# 三、Memory

通过以下方式可以获取每个函数分配内存的数据：

  1. 启动`Profiler`时执行一次`full gc`
  2. 重载[`lua_Alloc`](http://www.lua.org/manual/5.3/manual.html#lua_Alloc)，按下列三种情况统计数据： 
    1. 分配新的内存：建立内存指针与当前函数数据的对应关系，如果新内存大小为`size`, 当前函数的内存数量`+=size`，同时检查更新内存最大值。
    2. 释放内存：获得对应的函数数据，如果释放的内存大小为`size`, 当前函数的内存数量`-=size`
    3. `realloc`: 按照释放旧内存，分配新内存处理，但是这样可能会出现一些问题。如果旧内存是在函数A中分配，新内存在另一个函数B中分配，旧内存对应的数据会被计入B的统计数据中。不过这个问题应该影响不大。
  3. 停止`Profiler`时执行一次`full gc`

# 四、应用

这个`Profiler`统计的数据虽然简单，但是已经足以用来进行一些精细的优化，其中值得一提的有：

## 4.1 内存泄漏

以下面这段代码为例：

    
    
    local function alloc()
        return {}
    end
    
    --分配
    local Cache = {}
    for i = 1, 100 do
        Cache[i] = alloc()
    end
    -- 释放
    for i = 1, 100 do
        Cache[i] = nil
    end

下面是函数`alloc`在内存方面的数据，可以看到`alloc`分配的内存都释放掉了：

尚未gc的内存数量(Byte) | 分配内存的最大值(Byte)  
---|---  
0 | 5600  
  
把释放内存的代码注释掉，函数`alloc`在内存方面的数据变成：

尚未gc的内存数量(Byte) | 分配内存的最大值(Byte)  
---|---  
5600 | 5600  
  
如果代码中存在(持续地)内存泄漏，表现在`profile`数据中，是相关函数的`尚未gc的内存数量(Byte)`项不但不为0，还可能持续的变大。

## 4.2 不必要的临时内存

用`..`拼接字符串是最典型的例子，下面的函数`ConcatStrings`将`SubStrList`中的字符串拼接成一个字符串：

    
    
    local SubStrList = {}
    for i = 1, 10000 do
        table.insert(SubStrList, tostring(i))
    end
    
    local function ConcatStrings(SubStrList)
        local Result = ""
        for _, SubStr in ipairs(SubStrList) do
            Result = Result .. SubStr
        end
        return Result
    end

调用一次`ConcatStrings`的`profile`数据如下，从中可以看出产生了大量临时的内存，虽然可以`gc`掉：

函数名 | 尚未gc的内存数量(Byte) | 分配内存的最大值(Byte)  
---|---|---  
ConcatStrings | 38919 | 1641422  
  
拼接字符串的正确姿势应该是：

    
    
    local function ConcatStrings(SubStrList)
        return table.concat(SubStrList)
    end

调用这个版本`ConcatStrings`的数据如下：

函数名 | 尚未gc的内存数量(Byte) | 分配内存的最大值(Byte)  
---|---|---  
ConcatStrings | 0 | 0  
table.concat | 39582 | 137942  
  
比较两个版本的数据可以看出，最终拼接好的字符串占用的内存是相似的，但是拼接过程中产生的临时内存差别非常大。