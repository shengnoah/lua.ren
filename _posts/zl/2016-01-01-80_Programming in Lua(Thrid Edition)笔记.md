---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 5 Functions

  * 当函数参数为literal string或table constructor时，可以不加括号
    
        1  
    2  
    3  
    4  
    5  
    

|

    
        print "Hello world"  
    dofile 'a.lua'  
    print [[a multi-line message]]  
    f{x=10, y=20}  
    type{}  
      
  
---|---  
  * `:`操作符可用于面向对象编程，`o:foo(x)`相当于`o.foo(o, x)`，需把o当做第一个额外的参数

  * 传给函数的参数个数与其所需的参数个数可以不同，Lua会按照多重赋值的规则对参数赋值：extra arguments are thrown away, extra parameters get nil.此可用于使用默认参数：
    
        1  
    2  
    3  
    4  
    

|

    
        function (n)  
    	n = n or 1  
    	count = count + n  
    end  
      
  
---|---  
  * 函数的多重返回值
    
        1  
    2  
    

|

    
        s, e = string.find("hello Lua users", "Lua")  
    print(s, e)   
      
  
---|---  

字符串的第一个字符索引为1

  * 函数返回多值只需把这些值以此写在return后即可

  * 函数调用只有在以下四个构造中作为唯一一个或最后一个表达式时才会返回多值，否则只返回一值：multiple assignments，arguments to function calls，tale constructors，return statements，多余的返回值丢弃，多余的变量赋nil
    
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

    
        function foo()  
    	return 1,2  
    end  
    a,b,c = 0,foo() -- a=0,b=1,c=2  
    x,y,z = foo(),0 -- x=1,y=0,z=nil  
    print(foo())    --> 1	2	3  
    t = {foo()}     --> t = {1, 2, 3}  
    t = {foo(), 0}  --> t = {1, 0}  
    return foo()    --> return 1,2,3  
      
  
---|---  
  * 返回值可以有多种形式：
    
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
    

|

    
        function foo(i)  
    	if i == 0 then return  
    	elseif i == 1 then return 1  
    	elseif i == 2 then return 1, 2  
    	end  
    end  
    print(foo(0)) -- (no results)  
    print(foo(1)) --> 1  
    print(foo(2)) --> 1	2  
    print(foo(3)) -- (no results)  
      
  
---|---  
  * 给函数表达式加上括号，可以强制其只返回一个值

  * table.unpack()可以返回一个sequence的所有元素，返回元素个数与`#`操作符结果相同
    
        1  
    2  
    3  
    4  
    5  
    6  
    

|

    
        a = {1,2}  
    print(table.unpack(a)) --> 1	2  
    a[6] = 6  
    print(table.unpack(a)) --> 1	2  
    a[3] = 3  
    print(table.unpack(a)) --> 1	2	3  
      
  
---|---  

    
    
    1  
    2  
    3  
    

|

    
    
    f = string.find  
    a = {"hello", "ll"}  
    print(f(table.unpack(a)))  
      
  
---|---  
  
  * 可以明确的选取table.unpack()的结果
    
        1  
    

|

    
        print(table.unpack({"Sun", "Mon", "Tue", "Wed", "Thi", "Fri", "Sat"}, 2, 4)) --> Mon	Tue	Wed  
      
  
---|---  
  * 用Lua写unpack
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
        function unpack(t, i, n)   
    	i = i or 1  
    	n = n or #t  
    	if i <= n then  
    		retur t[i], unpack(t, i + 1, n)  
    	end  
    end  
      
  
---|---  
  * 可变参数函数，参数用`...`表示，叫做vararg expression
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
        function add(...)  
    	local s = 0  
    	for i, v in ipairs{...} do  
    		s = s + v  
    	end  
    	return s  
    end  
    print(add(3, 4, 10, 25, 12)) --> 54  
      
  
---|---  

`print(...)`可以打印出所有参数  
`return ...`返回所有参数

  * `{...}`是由所有参数构造的table

  * 多参数可转变为可变参数
    
        1  
    

|

    
        function foo(a, b, c)  
      
  
---|---  

变为  

    
    
    1  
    2  
    

|

    
    
    function foo(...)  
    	local a, b, c = ...  
      
  
---|---  
  
  * 跟踪函数调用：
    
        1  
    2  
    3  
    4  
    

|

    
        function foo1(...)  
    	print("calling foo:", ...)  
    	return foo(...)  
    end  
      
  
---|---  
  * 在`...`之前可以加上任意个固定参数
    
        1  
    2  
    3  
    

|

    
        functin fwrite(fmt, ...)  
    	return io.write(string.format(fmt, ...))  
    end  
      
  
---|---  
  * `{...}`不能检测到末尾的nil，这时可用`table.pack(...)`构造一个带有表示元素个数的域n的table，但是`{...}` is cleaner and faster than `table.pack(...)`
    
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

    
        function nonils(...)  
    	local arg = table.pack(...)  
    	for i = 1, arg.n do  
    		if arg[i] == nil then return false end  
    	end  
    	return true  
    end  
    print(nonils(2, 3, nil)) --> false  
    print(nonils(2, 3))      --> true  
    print(nonils())          --> true  
    print(nonils(nil))       --> false  
      
  
---|---  
  * named arguments并非Lua直接提供的语法，而是通过传递给函数一个table来间接实现的
    
        1  
    2  
    3  
    4  
    

|

    
        function rename(arg)  
    	return os.rename(arg.old, arg.new)  
    end  
    rename{old="old.lua", new="new.lua"}  
      
  
---|---