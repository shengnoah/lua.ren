---
layout: post
title: Lua 学习 chapter7  
tags: [lua文章]
categories: [lua文章]
---
### 目录

  1. 简单I/O模型
  2. 完整I/O模型
  3. 其它文档操作
  4. 其它系统调用

## 简单I/O模型

对于文档操作，I/O提供了两种不同的模型。简单模型虚拟了一个当前输入流和一个当前的输出流，其I/O是通过这些流实现的。
I/O库把当前的输入流初始化为进程的标准输入(C中的stdin),将当前的输出流初始化进程的标准输出(C中的stdout)。因此当执行io.read()这样的语句的时候就可以从标准的输入中读取一行。

函数的io.input和函数io.output用来改变当前的输入输出流。
io.input(fileName)会以只读的方式打开指定的文档，并将文档设置为当前的输入流。之后所有的输入都将来自于这个文档，除非重新调用io.input函数来改变。对于输出与之类似。

**函数write()** : 由于调用该函数可以传入多个参数，所以应该避免使用io.write(a..b..b),而是使用write(a,b,c)。
作为原则应该只在用后 **即弃** 的代码或者调试代码的时候调用print输出，当需要完全控制输出是，应该使用io.write()输出，和print不同，
函数io.write最终的输出不会添加诸如制表符或者换行符这样的额外内容。
此外函数io.write允许对输出进行重定向，而函数print只能使用标准输出。函数print会为其参数调用tostring函数。

    
    
    1
    2
    3
    4
    

|

    
    
    io.write("sin(3)=",math.sin(3),"n") -- >sin(3) = 0.1411200080
    io.write(string.format("sin(3)=%.4fn",math.sin(3))) -- >sin(3) = 0.1411
    
    io.read("a") -->从当前位置开始的读取当前输入文档的全部内容
      
  
---|---  
`

函数io.read可以从当前的输入流中读取字符串，其参数决定了其要读取的数据：

  1. “a” 读取整个文档
  2. “l” 读取下一行 （丢弃换行符） 默认参数
  3. “L” 读取下一行（保留换行符）
  4. “n” 读取一个数值(number 类型)
  5. num 以字符串读取num个字符

获取一个文档的所有行数可以使用 io.lines() 函数

    
    
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

    
    
    local lines = {}
    for line in io.lines() do
        lines[#lines + 1] = line
    end
    
    table.sort(lines)
    
    for i, v in ipairs(lines) do
        io.write(v, "n")
    end
      
  
---|---  
`

io.write(n)
从当前的输入流中最多读取n个字符，如果没有字符返回nil,io.write(0)是一个特烈，它常用于测试是否达到了文档末尾。如果仍然有数据可供读取，他就会返回一个空字符串，否则返回nil。

## 完整I/O模型

可以使用io.open来打开一个文档。该函数拥有两个参数，一个是文档名，一个是打开的模式。

  1. “r” 只读
  2. “w” 只写(可以用来删除文档中原来的内容)
  3. “a” 追加
  4. “b” 打开二进制文档

io.open函数打开发生错的时候会返回两个值，一个是nil另一个是系统相关错误的错误码

检测错误的一种典型方法是使用函数assert: local f = assert(io.open(fileName,mode))
发生错的时候错误信息会作为函数assert的第二个餐宿被传入。 然后assert会将错误信息展示出来。

文档打开之后可以使用从流中读取和写入。

    
    
    1
    2
    3
    4
    5
    

|

    
    
    local f = assert(io.open(file,"r"))
    local t = f:read("a")
    f:close()
    -- io.stdin, io.stdout, io.stderr
    io.stderr:write(message)
      
  
---|---  
`

Io库提供了三个预定义的C语言流的句柄，可以直接调用。

可以通过io.input 和 io.output 来改变的当前的输入输出流

    
    
    1
    2
    3
    4
    5
    

|

    
    
    local temp = io.input()
    io.input("newInput")
    --对新的输入流进行操作
    io.input():close
    io.input(temp)  --恢复之前的输入流
      
  
---|---  
`

**注意：** io.read(args)
实际上是io.input():read(args)，同样的io.write(args)和io.output():write(args)的简写。
io.lines() 函数可以带参数，filename,读取文档中的内容，不带参数就是从当前输入流中读，现在次函数还可以接收像read函数一样的参数。

## 其他文档操作

io.tmpfile返回一个操作临时文档的句柄，该句柄是以读写模式打开的，当进程运行完成后，该句柄就会被回收。
函数flush将所有的缓冲数据写入到文档中。与函数write一样，我们也可以把它当作io.flush()使用，以刷新当前的输出流，或者f:flush()，用来刷新当前流。

函数setvbuf用于设置流的缓冲模式。

  1. “no” 不缓冲
  2. “full” 缓冲区满或者显示刷新文档时才写入数据
  3. “line” 遇到换行符或从特定文档中读到了数据

对于后面两个模式，可以跟第二个参数来设置缓冲区大小。 一般情况下:io.stderr 是不缓冲的，io.stdout是按行缓冲的。

函数seek用来获取和设置文档当前位置。常使用f:seek(whence,offset)的形式来调用。whence有一下三种模式：

  1. “set” 文档开头
  2. “cur” 文档当前位置
  3. “end” 文档结尾

偏移是以字节为单位的。 whence默认值为”cur”,offset的默认值是0。file:seek()
返回当前位置且不改变当前位置。file:seek(“set”)会将位置重置到文档开头并返回0，file:seek(“end”)会将文档当前位置重置到文档结尾，并返回文档的大小。

os.rename改变文档名字 os.remove删除文档。

## 其它系统调用

os.exit用于终止进程的执行。 os.getev用于获取环境变量。

os.execute用来运行系统命令，等价于c中的system。还用io.popen也是运行一条系统命令,但是该函数可以重定向命令的输入输出流。

    
    
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

    
    
    function create(dirname)
    	os.execute("mkdir "...dirname)
    end
    
    local f = io.popen("dir /B","r") -- 只读
    local dir = {}
    for entry in f:lines() do
    	dir[#dir + 1] = entry
    end
      
  
---|---  
`

* * *