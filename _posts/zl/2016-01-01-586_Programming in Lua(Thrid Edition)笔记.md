---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 14 The Environment

这章有点难理解，有些段落反复看了好多遍才感觉好像是看懂了。  

  * Lua将所有全局变量存储在一个叫做global environment的table中
    
        1  
    

|

    
        for n in pairs(_G) do print(n) end  
      
  
---|---  
  * 对于动态变量名，可动态创建chunk并编译：
    
        1  
    2  
    3  
    4  
    

|

    
        x = 1  
    varname = "x"  
    value = loadstring("return " .. varname)  
    print(value())   
      
  
---|---  

这样的消耗较多，可用global environment：  

    
    
    1  
    2  
    

|

    
    
    val = _G[varname]  
    print(val)   
      
  
---|---  
  
  * `_G["a"] = _G["var1"]`写法繁琐，应简写为`a = var1`

  * `_G["io.read"]`无法获得`io`table的`read`域，可用以下`getfield()`实现：
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
        function (f)  
    	local v = _G -- start with the table of globals  
    	for w in string.gmatch(f, "[%w_]+") do  
    		v = v[w]  
    	end  
    	return v  
    end  
    getfield("io.write")("1n")  
      
  
---|---  
  * 通过`_G`设置多级全局变量：
    
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
    

|

    
        function setfield(f, v)  
    	local t = _G -- start with the table of globals  
    	for w, d in string.gmatch(f, "([%w_]+)(%.?)") do  
    		if d == "." then -- not last name?  
    			t[w] = t[w] or {} -- create table if absent  
    			t = t[w] -- get the table  
    		else -- last name  
    			t[w] = v -- do the assignment  
    		end  
    	end  
    end  
    setfield("t.x.y", 1)  
    print(t.x.y)   
    print(getfield("t.x.y"))   
      
  
---|---  
  * 全局变量声明，因为全局变量存储在table`_G`中，所以可以通过metatable处理未定义元素的操作：
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    

|

    
        setmetatable(_G, {  
    	__newindex = function (t, n, v)  
    		error("attempt to write to undeclared variable " .. n, 2)  
    	end,  
    	__index = function (_, n)  
    		error("attemp to read undeclared variable " .. n, 2)  
    	end,  
    })  
    print(a) --> stdin:1: attempt to read undeclared variable a  
      
  
---|---  

用`rawset()`声明新变量，`initval or false`是为了使新的全局变量总会得到一个不同与nil的值：  

    
    
    1  
    2  
    3  
    

|

    
    
    function declare(name, initval)  
    	rawset(_G, name, initval or false)  
    end  
      
  
---|---  
  
用debug库限制新的全局变量的声明只能在main chunk中，`debug.getinfo(2,
"S")`返回一个table，其`what`域说明metamethd调用是在main chunk、普通Lua函数和C函数其中之一  

    
    
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
    

|

    
    
    __newindex = function (t, n, v)  
    	local w = debug.getinfo(2, "S").what  
    	if w ~= "main" and w ~= "C" then  
    		error("attemp to write to undeclared variable " .. n, 2)  
    	end  
    	rawset(t, n, v)  
    end  
    a = 1  
    print(a)   
    function b()  
    	c = 2  
    end  
    b() --> stdin:1: attemp to write to undeclared variable c  
      
  
---|---  
  
判断一个变量是否存在，不能通过与nil比较，因为`__index`会报错，取而代之，我们可以用`rawget()`  

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    if rawget(_G, var) == nil then  
    	-- 'var' is undeclared  
    	...  
    end  
      
  
---|---  
  
为了使声明为nil的变量不被Lua认为是未声明的，可以用一个table保存声明的变量的名字，每次调用metamethod时都检查一下变量是否在该table中  

    
    
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
    18  
    19  
    20  
    21  
    22  
    23  
    

|

    
    
    local declaredNames = {}  
    setmetatable(_G, {  
    	__newindex = function (t, n, v)  
    		if not declaredNames[n] then  
    			local w = debug.getinfo(2, "S").what  
    			if w ~= "main" and w ~= "C" then  
    				error("attempt to write to undeclared variable " .. n, 2)  
    			end  
    			declaredNames[n] = true  
    		end  
    		rawset(t, n, v) -- do the actual set  
    	end,  
    	__index = function (_, n)  
    		if not declaredNames[n] then  
    			error("attempt to read undeclared variable " .. n, 2)  
    		else  
    			return nil  
    		end  
    	end,  
    })  
    a = nil  
    print(a) --> nil  
    print(b) --> stdin:1 attempt to read undeclared variable b  
      
  
---|---  
  
  * 非全局环境，所谓非全局环境，就是指之前所说的全局环境并非真正的全局环境，是Lua实现的伪全局环境，是为了操作全局变量更方便。

  * free name是并非绑定在一个特定声明的名字，即并非出现在有这些名字的局部变量所在的域（可以简单理解为全局变量的名字），例如`var1 = var2 + 3`这个chunk里的`var1`和`var2`就是free name，Lua会将该chunk编译为：
    
        1  
    2  
    3  
    4  
    

|

    
        local _ENV = <some code>  
    return funtion (...)  
    	_ENV.var1 = _ENV.var2 + 3  
    end  
      
  
---|---  

也就是说Lua会把chunk编译在一个预定义的upvalue`_ENV`的域中。`load()`则会把`_ENV`初始化global environment  

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    local _ENV = <the global environment>  
    return function (...)  
    	_ENV.var1 = _ENV.var2 + 3  
    end  
      
  
---|---  
  
  * 总结一下Lua处理全局变量的方式：

    * 将任何chunk编译在一个upvalue`_ENV`的域中
    * 将任何free name`var`变为`_ENV.var`（简单来说，就是`_ENV`只会记录全局变量，而不记录局部变量）
    * `load()`和`loadfile()`将一个chunk的第一个upvalue初始化为global environment
  * 通过`_ENV = nil`阻止在之后的chunk中获取全局变量，可以用来控制代码可以使用什么变量
    
        1  
    2  
    3  
    4  
    5  
    

|

    
        local print, sin = print, math.sin  
    _ENV = nil  
    print(13) --> 13  
    print(sin(13)) --> 0.42016703682664  
    print(math.cos(13)) -- eror!  
      
  
---|---  
  * 用`_ENV`绕过局部变量
    
        1  
    2  
    3  
    4  
    

|

    
        a = 13 -- global  
    local a = 12  
    print(a) --> 12 (local)  
    print(_ENV.a) --> 13 (global)  
      
  
---|---  
  * 用`_ENV`改变环境
    
        1  
    2  
    3  
    4  
    

|

    
        -- change current environment to a new empty table  
    _ENV = {}  
    a = 1 -- create a field in _ENV  
    print(a) --> stdin:4: attempt to call global 'print' (a nil value)  
      
  
---|---  

用旧环境填充新环境：  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    a = 15 -- create a global variable  
    _ENV = {g = _G} -- change current environment  
    a = 1 -- create a field in _ENV  
    g.print(a)   
    g.print(g.a) --> 15  
      
  
---|---  
  
用`_G`代替`g`：  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    a = 15 -- create a global variable  
    _ENV = {_G = _G} -- change current environment  
    a = 1 -- create a field in _ENV  
    _G.print(a)   
    _G.print(_G.a) --> 15  
      
  
---|---  
  
用继承来填充新环境：  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    a = 1  
    local newgt = {} -- create new environment  
    setmetatable(newgt, {__index = _G})  
    _ENV = newgt -- set it  
    print(a)   
      
  
---|---  
  
这里只设置了`__index`，所以任何赋值操作都发生在新环境中，所以错误地改变了全局环境的一个变量没有危险，仍可以通过`_G`来活取原值  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    -- continuing previous code  
    a = 10  
    print(a) --> 10  
    print(_G.a)   
    _G.a = 20  
    print(_G.a) --> 20  
      
  
---|---  
  
  * `_ENV`符合通常的域规则，一个chunk中的函数可以像获取其他外部变量那样活取`_ENV`
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
        _ENV = {_G = _G}  
    local function foo()  
    	_G.print(a) -- compiled as '_ENV._G.print(_ENV.a)'  
    end  
    a = 10 -- _ENV.a  
    foo() --> 10  
    _ENV = {_G = _G, a = 20}  
    foo() --> 20  
      
  
---|---  
  * 用局部的`_ENV`生成一个局部环境：
    
        1  
    2  
    3  
    4  
    5  
    6  
    

|

    
        a = 2  
    do  
    	local _ENV = {print = print, a = 14}  
    	print(a) --> 14  
    end  
    print(a) --> 2 (back to the original _ENV)  
      
  
---|---  
  * 定义一个有私有环境的函数：
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    

|

    
        function factory(_ENV)  
    	return function ()  
    		return a -- "global" a  
    	end  
    end  
    f1 = factory{a = 6}  
    f2 = factory{a = 7}  
    print(f1()) --> 6  
    print(f2()) --> 7  
      
  
---|---  

这里`factory()`返回一个闭包，其upvalue为`_ENV`。用这种方法可以让一些函数共享一个公共的环境，或者让一个函数专门用于改变这些函数共享的这个环境

  * `load()`的可选的第四个参数用来设定`_ENV`的值，`loadfile()`则为第三个参数

  * 使用配置文件：
    
        1  
    2  
    3  
    4  
    

|

    
        -- file 'config.lua'  
    width = 200  
    height = 300  
    ...  
      
  
---|---  

可用一下代码加载：  

    
    
    1  
    2  
    3  
    

|

    
    
    env = {}  
    f = loadfile("config.lua", "t", env)  
    f() -- only after executing will env be populated  
      
  
---|---  
  
配置文件中的代码会运行在一个空环境中，并且所有配置都会存储在该环境中，对其他代码无影响，有一定安全性

  * `load()`和`loadfile()`返回的函数可多次调用，为了使每次调用时的环境不同，可用以下两种方法：

  * 法一：`debug.setupvalue()`可以改变一个函数的任何upvalue
    
        1  
    2  
    3  
    4  
    

|

    
        f = loadfile(filename)  
    ...  
    env = {}  
    debug.setupvalue(f, 1, env)  
      
  
---|---  

`debug.setupvalue()`的第一个参数是一个函数，第二个参数是upvalue索引，第三个参数是新的upvalue，此处第二个参数总为1，因为Lua保证`load()`和`loadfile()`返回的函数只有一个upvalue，即`_ENV`。此法破坏了Lua的可见性规则：一个局部变量不能被外部获取

  * 法二：用可变参数，用`Exercise 8.1`的`loadwithprefix()`在加载的chunk之前加上`local _ENV = ...;`
    
        1  
    2  
    3  
    4  
    

|

    
        f = loadwithprefix("local _ENV = ...;", io.lines(filename, "*L"))  
    ...  
    env = {}  
    f(env)  
      
  
---|---