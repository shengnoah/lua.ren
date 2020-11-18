---
layout: post
title: Lua中ipairs和pairs的区别与使用 
tags: [lua文章]
categories: [lua文章]
---
[ __

tolua++安装

](https://hulinhong.com/2015/11/11/lua_cpp_toluapp_tutorial/ "tolua++安装")

[

C++与Lua本质原始交互API

__](https://hulinhong.com/2015/11/11/lua_cpp_bind/ "C++与Lua本质原始交互API")

关于ipairs()和pairs(),Lua官方手册是这样说明的：

**pairs (t)**

If t has a metamethod __pairs, calls it with t as argument and returns the
first three results from the call.

Otherwise, returns three values: the next function, the table t, and nil, so
that the construction

    
    
    ` for k,v in pairs(t) do body end`
    

will iterate over all key–value pairs of table t.

See function next for the caveats of modifying the table during its traversal.

**ipairs (t)**

If t has a metamethod __ipairs, calls it with t as argument and returns the
first three results from the call.

Otherwise, returns three values: an iterator function, the table t, and 0, so
that the construction

    
    
    ` for i,v in ipairs(t) do body end`
    

will iterate over the pairs (1,t[1]), (2,t[2]), …, up to the first integer key
absent from the table.

根据官方手册的描述，pairs会遍历表中所有的key-
value值，而pairs会根据key的数值从1开始加1递增遍历对应的table[i]值，直到出现第一个不是按1递增的数值时候退出。

**. . .**

下面我们以例子说明一下吧  

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    stars = {[1] = "Sun", [2] = "Moon", [5] = 'Earth'}  
    for i, v in pairs(stars) do  
       print(i, v)  
    end  
      
  
---|---  
  
使用pairs()将会遍历表中所有的数据，输出结果是：

    
    
    1    Sun
    2    Moon
    5    Earth
    

如果使用ipairs（）的话，

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    for i, v in ipairs(stars) do  
      
       print(i, v)  
      
    end  
      
  
---|---  
  
当i的值遍历到第三个元素时，i的值为5，此时i并不是上一个次i值（2）的+1递增，所以遍历结束，结果则会是：

    
    
    1    Sun
    2    Moon
    

ipairs()和pairs()的区别就是这么简单。

还有一个要注意的是pairs()的一个问题，用pairs()遍历是[key]-[value]形式的表是随机的，跟key的哈希值有关系。看以下这个例子：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    stars = {[1] = "Sun", [2] = "Moon", [3] = "Earth", [4] = "Mars", [5] = "Venus"}  
      
    for i, v in pairs(stars) do  
      
       print(i, v)  
      
    end  
      
  
---|---  
  
结果是：

    
    
    2    Moon
    3    Earth
    1    Sun
    4    Mars
    5    Venus
    

并没有按照其在表中的顺序输出。

但是如果是这样定义表stars的话

`stars = {"Sun", "Moon", “Earth”, "Mars", "Venus"}`

结果则会是

    
    
    1    Sun
    2    Moon
    3    Earth
    4    Mars
    5    Venus
    

你清楚了吗？:)