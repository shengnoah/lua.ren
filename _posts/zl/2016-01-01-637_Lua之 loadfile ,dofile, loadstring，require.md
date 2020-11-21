---
layout: post
title: Lua之 loadfile ,dofile, loadstring，require 
tags: [lua文章]
categories: [topic]
---
* * *

### loadfile——只编译，不运行

    
    
    1.功能：载入文件但不执行代码块，对于相同的文件每次都会执行。只是编译代码，然后将编译结果作为一个函数返回
    2.调用：loadfile("filename")
    3.错误处理：不引发错误，只返回错误值但不处理错误,即返回nil和错误消息
    4.优点：调用一次之后可以多次调用返回的结果（即函数），
      即“多次调用”只需编译一次（注：这里的多次调用   是指多次调用返回的函数，而不是多次调用loadfile）
    

**dofile可如下定义：**  

    
    
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
      
    
    1
    
    2
    
    3
    
    4
    
    5

|

    
    
    注：加载了程序块并没有定义其中的函数。在Lua中，函数定义是一种赋值操作，是在运行时才完成的操作。
    
    例如：一个文件test.lua中有一个函数 function foo(x) print(x) end ,执行如下代码：
    
    　　　f = loadfile(test.lua) --加载程序块，此时还没有定义函数foo
    
    　　　f() --运行加载的程序块，此时就定义了函数foo
    
         foo("hello lua") -->hello lua --经过上面的步骤才能调用foo  
  
---|---  
  
### dofile——执行

    
    
    1.功能：载入文件并执行代码块，对于相同的文件每次都会执行
    2.调用：dofile("filename")
    3.错误处理：如果代码块中有错误则会引发错误
    4.优点：对简单任务而言，非常便捷
    5.缺点：每次载入文件时都会执行程序块
    6.定位：内置操作，辅助函数
    

### require——我只执行一次

    
    
    require和dofile有点像，不过又很不一样，require在第一次加载文件的时候，会执行里面的代码。
    但是，第二次之后，再次加载文件，则不会重复执行了。换句话说，它会保存已经加载过的文件，不会重复加载。
    

### loadstring

    
    
    1.特点：功能强大，但开销大；
    2.典型用处：执行外部代码，如：用户的输入
    3.错误错里：代码中如果有语法错误就会返回nil
    4.理解：f = loadstring("i = i+1")  可理解为（但不完全是）f = function()  i = i+1  end 
    (注：这里的变量"i"是全局变量，不是指局部变量，如果没有定义全局变量"i",调用f()则会报错！，即loadstring   不涉及词法域)