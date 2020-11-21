---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 8 Compilation, Execution, and Errors

  * `dofile()`实现：
    
        1  
    2  
    3  
    4  
    

|

    
        function (filename)  
    	local f = assert(loadfile(filename))  
    	return f()  
    end  
      
  
---|---  

可简单一些：  

    
    
    1  
    2  
    3  
    

|

    
    
    function (filename)  
    	assert(loadfile(filename))()  
    end  
      
  
---|---  
  
`loadfile()`只编译不运行，将编译好的chunk以函数返回，在runtime中执行此函数后才可以使用chunk中的代码,因为Lua中的函数定义是赋值；`loadfile()`遇到错误时返回错误码代码而不提出错误  

    
    
    1  
    2  
    3  
    4  
    

|

    
    
      
    function foo(x)  
    	print(x)  
    end  
      
  
---|---  
      
    
    1  
    2  
    3  
    4  
    

|

    
    
    f = loadfile("foo.lua") -- 'foo.lua' defines function foo()  
    print(foo) --> nil  
    f()        -- defines 'foo'  
    foo("ok")  --> ok  
      
  
---|---  
  
  * load()从字符串中读取chunk
    
        1  
    2  
    3  
    4  
    

|

    
        f = load("i = i + 1")  
    i = 0  
    f(); print(i) --> 1  
    f(); print(i) --> 2  
      
  
---|---  

简单版：  

    
    
    1  
    

|

    
    
    assert(load(s))()  
      
  
---|---  
  
  * `f = function () i = i + 1 end`比`f = load("i = i + 1")`快，更常用；`load()`编译chunk时是在全局环境中
    
        1  
    2  
    3  
    4  
    5  
    6  
    

|

    
        i = 32  
    local i = 0  
    f = load("i = i + 1; print(i)")  
    g = function () i = i + 1; print(i) end  
    f() --> 33  
    g() --> 1  
      
  
---|---  
  * `load()`可以用来对表达式字符串求值：
    
        1  
    2  
    3  
    4  
    

|

    
        print "enter your expression"  
    local l = io.read()  
    local func = assert(load("return " .. l))  
    print("the value of your expression is " .. func())  
      
  
---|---  

也可以多次调用返回的函数，使其作用于全局变量上  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    print "enter function to be plotted (with variable 'x'):"  
    local l = io.read()  
    local f = assert(load("return " .. l))  
    for i = 1, 20 do  
    	x = i -- global 'x' (to be visible from the chunk)  
    	print(string.rep("*", f()))  
    end  
      
  
---|---  
  
  * `load()`的第一个参数可以为一个读函数，`load()`连续调用该读函数直到其返回nil
    
        1  
    

|

    
        f = load(io.lines(filename, "*L"))  
      
  
---|---  

`io.lines(filename."*L")`返回一个函数，该函数每次被调用时都会返回文件的新的一行  

    
    
    1  
    

|

    
    
    f = load(io.lines(filename, 1024))  
      
  
---|---  
  
`io.lines(filename, 1024)`返回1024 bytes的块

  * Lua将任何独立的chunk看做一个可变参数的匿名函数的函数体，`load("a = 1")`相当于`function (...) a = 1 end`

  * `load()`中的chunk可以有局部变量
    
        1  
    2  
    

|

    
        f = load("load a = 10; print(a + 20)")  
    f() --> 30  
      
  
---|---  
  * 避免`load()`中的chunk使用全局变量
    
        1  
    2  
    3  
    4  
    5  
    6  
    

|

    
        print "enter function to be plotted (with variable 'x'):"  
    local l = io.read()  
    local f = assert(load("local x = ...; return " .. l))  
    for i = 1, 20 do  
    	print(string.rep("*", f(i)))  
    end  
      
  
---|---  
  * `load()`不会提出错误，遇到错误时，其只返回nil和错误信息
    
        1  
    

|

    
        print(load("i i")) --> nil	[string "i i"]:1: syntax error near 'i'  
      
  
---|---  
  * 可以用`luac`产生预编译Lua源代码，`lua`可以执行预编译文件，`loadfile()`和`load()`也接受预编译文件
    
        1  
    2  
    

|

    
         luac -o prog.lc prog.lua  
     lua prog.lc  
      
  
---|---  
  * `luac`的实现：
    
        1  
    2  
    3  
    4  
    

|

    
        p = loadfile(arg[1])  
    f = io.open(arg[2], "wb")  
    f:write(string.dump(p))  
    f:close()  
      
  
---|---  

`string.dump()`接受一个Lua函数，返回其预编译代码，该代码可以被Lua重新载入

  * `load()`的第二个参数是chunk的名字，只用在错误信息中

  * `load()`的第三个参数控制加载何种chunk，参数是一个字符串，`"t"`表示textual chunks，`"b"`表示binary chunks，`"bt"`表示两者均可，该参数可以防止运行恶意binary chunks

  * `load()`的第四个参数是一个环境

  * 预编译形式源代码不一定比原始源代码短，但加载得块

  * `loadlib()`加载库并将Lua与其链接起来，该函数并不调用库中的函数，只是将C函数返回为Lua函数，出错时返回nil和错误信息
    
        1  
    2  
    

|

    
        path = "/usr/local/lib/lua/5.1/socket.so"  
    f = package.loadlib(path, "luaopen_socket")  
      
  
---|---  
  * 加载C函数库，用`require()`更方便，其会搜索函数库并用`loadlib()`加载链接，返回包含库中函数的table

  * `error()`可以明确提出一个错误，参数为一条错误信息
    
        1  
    2  
    3  
    

|

    
        print "enter a number"  
    n = io.read("*n")  
    if not n then error("invalid input") end  
      
  
---|---  

用`assert()`简化  

    
    
    1  
    2  
    

|

    
    
    print "enter a number"  
    n = assert(io.read("*n"), "invalid input")  
      
  
---|---  
  
当第一个参数为非false时返回之，否则返回第二个可选的参数，错误信息

  * `tonumber()`常用来检测参数是否为数字或者是否可以转换为数字

  * 文件打开的错误处理
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
        local file, msg  
    repeat  
    	print "enter a file name:"  
    	local name = io.read()  
    	if not name then return end -- no input  
    	file, msg = io.open(name, "r"）  
    	if not file then print(msg) end  
    until file  
      
  
---|---  
  * 错误处理和异常，用`pcall()`执行函数
    
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
    

|

    
        local ok, msg = pcall(function ()  
    	<some code>  
    	if unexpected_condition then error() end  
    	<some code>  
    	print(a[i]) -- potential error: 'a' may not be a table  
    	<some code>  
    end)  
    if ok then      -- no errors while running protected code  
    	<regular code>  
    else            -- protected code raised an error: take appropriate action  
    	<error-handling code>  
    end  
      
  
---|---  

如果无错误，则`pcall()`返回true和执行的函数的返回值，如果有错误，则返回false和错误信息，错误信息为Lua自动生成的错误信息(索引非table变量)或执行的函数中的`error()`的参数，该参数可以为任何类型  

    
    
    1  
    2  
    

|

    
    
    local status， err = pcall（function () error({code = 121}) end)  
    print(err.code) --> 121  
      
  
---|---  
  
  * `error()`的第二个参数level可用来指定错误的来源
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
        function foo(str)  
    	if type(str) ~= "string" then  
    		error("string expected", 1)  
    	end  
    	<regular code>  
    end  
    foo({x = 1})  
      
  
---|---  

以上为`level.lua`，用`lua`执行后错误信息如下：  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    lua: hello.lua:3: string expected  
    stack traceback:  
    	[C]: in function 'error'  
    	hello.lua:3: in function 'foo'  
    	hello.lua:7: in main chunk  
    	[C]: in ?  
      
  
---|---  
  
将`error()`的第二个参数level改为2后错误信息如下：  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    lua: hello.lua:7: string expected  
    stack traceback:  
    	[C]: in function 'error'  
    	hello.lua:3: in function 'foo'  
    	hello.lua:7: in main chunk  
    	[C]: in ?  
      
  
---|---  
  
错误信息`string expected`之前的代码行数表明不同的错误来源

  * `pcall()`在遇到错误时会摧毁从它到错误点的栈，为了跟踪错误，可以用`xpcall()`，其第一个参数是待跟踪的函数，第二个参数是一个消息处理函数，遇到错误时，在释放栈之前，Lua会调用此消息处理函数，从而可以用debug library搜集错误的任何额外信息