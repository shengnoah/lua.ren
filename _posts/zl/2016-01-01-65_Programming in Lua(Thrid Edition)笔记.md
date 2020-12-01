---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 4 Statements

  * 多重赋值：先求出所有值，再赋值

  * 交换两值
    
        1  
    

|

    
        x, y = y, x  
      
  
---|---  
  * 多余的变量赋nil，多余的值丢弃

  * 用局部变量保护全局变量
    
        1  
    

|

    
        local foo = foo  
      
  
---|---  
  * `if-then-elseif-else-end`充当`switch`
    
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

    
        if op == "+" then  
    	r = a + b  
    elseif op == "-" then  
    	r = a - b  
    elseif op == "*" then  
    	r = a * b  
    elseif op == "/" then  
    	r = a / b  
    else  
    	error("invalid operation")  
    end  
      
  
---|---  
  * `numeric for`
    
        1  
    2  
    3  
    

|

    
        for var = exp1, exp2, exp3 do  
    	<somethin>  
    end  
      
  
---|---  

exp3可选，为变量的增量  

    
    
    1  
    

|

    
    
    for i= 10， 1， -1 do print(i) end  
      
  
---|---  
  
  * `math.huge`用作无穷大
    
        1  
    2  
    3  
    4  
    5  
    6  
    

|

    
        for i = 1, math.huge do  
    	if(0.3*i^3 - 20*i^2 - 500 >= 0) then  
    		print(i)  
    		break  
    	end  
    end  
      
  
---|---  
  * for循环的表达式只求值一次，变量为局部变量

  * `generic for`
    
        1  
    

|

    
        for k, v in pairs(t) do print(k, v) end  
      
  
---|---  
  * 用`generic for`和迭代器建立反向表
    
        1  
    2  
    3  
    4  
    5  
    

|

    
        days = {"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday"， "Friday", "Saturday"}  
    revDays = {}  
    for k, v in pairs(days) do  
    	revDays[v] = k  
    end  
      
  
---|---  
  * `goto`实现`redo`和`continue`
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
        while some_condition do  
    	::redo::  
    	if some_other_condition then goto continue  
    	else if yet_another_condition then goto redo  
    	end  
    	<some code>  
    	::continue::  
    end  
      
  
---|---  
  * `goto`的域
    
        1  
    2  
    3  
    4  
    5  
    6  
    

|

    
        while some_condition do  
    	if some_other_condition then goto continue end  
    	local var = something  
    	<some code>  
    	::continue::  
    end  
      
  
---|---  

局部变量的域结束在最后一个非空语句，`continue`标签出现在最后一个非空语句之后，所以`goto`并没有跳入`var`的域。

  * `goto`不能从一个函数内跳到函数外