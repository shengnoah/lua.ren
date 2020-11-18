---
layout: post
title: Lua基本类型及Basic Functions 
tags: [lua文章]
categories: [lua文章]
---
## Lua基本类型及Basic Functions

#### 概述

#### Lua的基本类型

###### 基本类型

![lua data type](https://moorle.github.io//image/lua_data_type.png)

###### e.g.

    
    
    function testType()
    	print (string.format("the type of _G = %s ", type(_G)))
    	print (string.format("the type of _VERSION = %s ", type(_VERSION)))
    	print (string.format("the type of X = %s ", X))
    	print (string.format("the type of nil = %s ", type(nil)))
    	print (string.format("the type of 1 + 1 = %s ", type(1 + 1)))
    	print (string.format("the type of {a='a'} = %s ", type({a='a'})))
    	print (string.format("the type of true = %s ", type(true)))
    	print (string.format("the type of Hello world = %s ", type("Hello world")))
    	print (string.format("the type of testType = %s ", type(testType)))
    end
    
    output:
    the type of _G = table 
    the type of _VERSION = string 
    the type of X = nil 
    the type of nil = nil 
    the type of 1 + 1 = number 
    the type of {a='a'} = table 
    the type of true = boolean 
    the type of Hello world = string 
    the type of testType = function

###### 注意事项

  * 在Lua中，false和nil会被逻辑运算符当成false，其他都为true(0也是true)

#### Basic Functions

Lua语言built-in方法其实并不多，最基本的也就二十几个，比起其他语言可谓是小巫见大巫。现在罗列一下以便以后查阅。

sequence | function or variable | des  
---|---|---  
1 | assert (v [, message]) |
assert函数检查其第一个参数是否为true。若为true，则简单地返回该参数；否则(为false或nil)就会引发一个错误  
2 | collectgarbage ([limit]) | Sets the garbage-collection threshold to the
given limit (in Kbytes) and checks it against the byte counter  
3 | dofile (filename) | 读入文件编译并执行, 本质上位辅助函数，真正实现其功能的是loadfile()  
4 | error (message [, level]) | Terminates the last protected function called
and returns message as the error message. Function error never returns  
5 | _G | holds the global environment (that is, _G._G = _G)  
6 | getfenv (f) | 返回当前函数的运行环境，如果f为0，则返回全局环境变量；只有setfenv了环境，getfenv才能生效。  
7 | getmetatable (object) | 返回对象的元表(metatable)
[如果元表(metatable)中存在__metatable键值，当返回__metatable的值]  
8 | gcinfo () | Returns two results: the number of Kbytes of dynamic memory
that Lua is using and the current garbage collector threshold (also in Kbytes)  
9 | ipairs (t) | Returns an iterator function  
10 | loadfile (filename) | Loads a file as a Lua chunk (without running it)  
11 | loadlib (libname, funcname) | Links the program with the dynamic C
library libname  
12 | loadstring (string [, chunkname]) | Loads a string as a Lua chunk
(without running it)  
13 | next (table [, index]) | Allows a program to traverse all fields of a
table  
14 | pairs (t) | Returns the next function and the table t (plus a nil), so
that the construction for k,v in pairs(t) do … end will iterate over all
key–value pairs of table t.  
15 | pcall (f, arg1, arg2, …) | Calls function f with the given arguments in
protected mode  
16 | print (e1, e2, …) | Receives any number of arguments, and prints their
values in stdout, using the tostring function to convert them to strings  
17 | rawequal (v1, v2) | Checks whether v1 is equal to v2, without invoking
any metamethod. Returns a boolean.  
18 | rawget (table, index) | Gets the real value of table[index], without
invoking any metamethod  
19 | rawset (table, index, value) | Sets the real value of table[index] to
value, without invoking any metamethod  
20 | require (packagename) | Loads the given package  
21 | setfenv (f, table) | Sets the current environment to be used by the given
function  
22 | setmetatable (table, metatable) | Sets the metatable for the given table  
23 | tonumber (e [, base]) | Tries to convert its argument to a number  
24 | tostring (e) | Receives an argument of any type and converts it to a
string in a reasonable format  
25 | type (v) | Returns the type of its only argument, coded as a string. The
possible results of this function are “nil” (a string, not the value nil),
“number”, “string”, “boolean, “table”, “function”, “thread”, and “userdata”.  
27 | unpack (list) | Returns all elements from the given list  
28 | _VERSION | A global variable (not a function) that holds a string
containing the current interpreter version.  
29 | xpcall (f, err) | A global variable (not a function) that holds a string
containing the current interpreter version.  
  
  * __
  * [Lua 2](/posts.html#Lua)

* * *