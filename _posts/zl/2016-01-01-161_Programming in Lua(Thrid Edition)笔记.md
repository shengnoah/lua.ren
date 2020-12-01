---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 3 Expressions

  * `a % b == a - math.floor(a / b) * b`，可以用于浮点数，`x % 1`为x的小数部分，`x - x % 1`为x的整数部分，`x-x%0.01`可以保留x两位小数，也可以用于角度对360取模和弧度对2PI取模`angle%(2*math.pi)`

  * `~=`与`==`作用相反

  * Lua可以根据本地字符编码比较字符串大小

  * 逻辑操作符`and`，`or`，`not`

  * `x = x or v`相当于
    
        1  
    

|

    
        if not x then x = v end  
      
  
---|---  

可以在x未被赋值时对其赋值

  * `(a and b) or c`与C的`a ? b : c`等价
    
        1  
    

|

    
        max = (x > y) and x or y  
      
  
---|---  
  * `..`用于连接字符串
    
        1  
    2  
    3  
    

|

    
        print("Hello " .. "world")   
    print(0 .. 1)              --> 01  
    print(000 .. 01)           --> 01  
      
  
---|---  

`print(000 .. 01)`中会先求值，再转换为字符串，再连接

  * `a[#a + ] = v`把v加到列表a的末尾

  * 关于有洞的列表的长度
    
        1  
    2  
    3  
    4  
    5  
    6  
    

|

    
        c = {1, 2, 3}  
    print(#c) --> 3  
    c[2] = nil  
    print(#c) --> 3  
    c[3] = nil  
    print(#c) --> 1  
      
  
---|---  
  * `a = {1, 2, 3, nil, nil}`与`a = {1, 2, 3}`相同，长度均为3

  * 优先级：
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
        ^  
    not # -(unary)  
    * / %  
    + -  
    ..  
    < > <= >= ~= ==  
    and  
    or  
      
  
---|---  
  * `a = {x = 10, y = 20}`比`a = {}; a.x = 10; a.y = 20`快一些 

  * 混用list风格和record风格构造table
    
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

    
        polyline = {  
    color = "blue",  
    thickness = 2,  
    npoints = 4,  
    {x = 0, y = 0},   -- polyline[1]  
    {x = -10, y = 0}, -- polyline[2]  
    {x = -10, y = 1}, -- polyline[3]  
    {x = 0, y = 1},   -- polyline[4]  
    }  
      
  
---|---  
  * 几种特殊的table构造方式：
    
        1  
    2  
    3  
    

|

    
        a = {["+"] = "add", ["-"] = "sub",  
         ["*"] = "mul", ["/"] = "div"}  
    b = {[1]="red", [2]="green", [3]="blu",}  
      
  
---|---  

最后一个entry后的逗号可选

  * 在table的构造中，可以用分号代替逗号来分隔不同部分，例如list和record