---
layout: post
title: Lua 学习 chapter8  
tags: [lua文章]
categories: [lua文章]
---
### 目录

  1. 局部变量和代码块
  2. 控制结构
  3. break,return和goto

## 局部变量和代码块

Lua语言中的变量默认情况下是全局变量，所有的局部变量在使用前必须声明。一个代码块是一个控制结构的主体，或是函数的主体，或是一个代码段(即变量被声明时所在的文档或字符串）。
在乱终可以显示的声明代码块，使用do end

    
    
    1
    2
    3
    

|

    
    
    do
    	--代码块
    end
      
  
---|---  
`

在lua中，尽可能的使用局部变量时是一种良好的变成风格。局部变量便于回收，不会造成命名冲突和混乱。
在lua中，又一个检查全局变量的模块strict.lua,如果视图在一个函数中对不存在的全局变量赋值或者使用，将会抛出异常。

## 控制结构

  1. if end终结
  2. while end终结
  3. repeat 使用untile终结
  4. for end终结

### if

    
    
    1
    2
    3
    4
    5
    6
    7
    8
    

|

    
    
    local a = 100
    if a > 0 then
        print(a)
    elseif a > 5 then
        print(a)
    else
        print(a)
    end
      
  
---|---  
`

### while

    
    
    1
    2
    3
    4
    5
    

|

    
    
    --如果不对a进行+1的操作很容易陷入死循环
    while a > 10 do
        print(a)
        a = a + 1
    end 
      
  
---|---  
`

### repeat

和其它语言do while的用法很像

    
    
    1
    2
    3
    4
    

|

    
    
    repeat
        print(a)
        a = a - 1
    until a > 20
      
  
---|---  
`

### for

    
    
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
    

|

    
    
    for var = exp1,exp2,exp3 do
    	someting
    end
    --- 从exp1变成exp2 exp3是步长 默认是1，所以这个参数是可选参数
    for i = 1, 10 do
        print(i)
    end
    
    for i, v in ipairs(b) do
        print(v)
    end
    
    for i, v in pairs(b) do
        print(v)
    end
      
  
---|---  
`

## break,return和goto

break和return语句用于从当前的循环结构中跳出，goto语句则能够跳转到函数中的任何位置。 break只能在循环中使用，跳出当前层次的循环。

return用来返回函数的执行结果或者结束函数的执行，在每个函数的结尾都存在一个隐形的return。

goto 的使用，跳转到标签的位置。 标签的定义：

    
    
    1
    2
    3
    4
    5
    6
    

|

    
    
    goto isMe
    while a < 200 do
        print(a)
        a = a + 1
    end
    ::isMe::
      
  
---|---  
`

goto的使用在lua中存在限制条件，标签遵循可见性原则，因此不能直接跳转到一个代码块中的标签（因为代码快中的标签对外不可见）。
其次goto不能跳转到函数外，最后goto不能跳转到局部变量的作用域。

* * *