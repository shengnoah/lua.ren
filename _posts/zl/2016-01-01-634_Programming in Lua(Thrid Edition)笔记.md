---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 1 Getting Started

  * statements之间的分号可选

  * `-i`选项会让Lua在执行完指定chunk后进入交互模式
    
        1  
    

|

    
         lua -i prog  
      
  
---|---  
  * 在交互模式中可用dofile()函数运行一个外部文件chunk

  * 退出交互模式：ctrl-D in UNIX，ctrl-Z in Windows，os.exit()

  * 标识符尽量不要以一个或多个下划线开头，其多在Lua内有特殊用处

  * 单行注释`--`，多行注释`--[[`和`]]--`，`---[[`只起到单行注释的作用

  * use Lua as a script interpreter in UNIX systems: 在脚本的第一行加上：
    
        1  
    

|

    
        #!/usr/local/bin/lua  
      
  
---|---  

或者  

    
    
    1  
    

|

    
    
    #!/usr/local/env lua  
      
  
---|---  
  
  * `-e`选项允许直接执行命令行里的代码
    
        1  
    

|

    
        % lua -e "print(math.sin(12))"  
      
  
---|---  
  * `-l`选项会加载一个库
    
        1  
    

|

    
        % lua -llib  
      
  
---|---  

会加载文件名为`lib.lua`的库

  * 命令行中的脚本参数，以脚本名为0，右边的为正，左边的为负
    
        1  
    

|

    
        % lua -e "sin=math.sin" script a b  
      
  
---|---  

参数会被存在arg表中，内容如下：  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    arg[-3] = "lua"  
    arg[-2] = "-e"  
    arg[-1] = "sin=math.sin"  
    arg[0] = "script"  
    arg[1] = "a"  
    arg[2] = "b"  
      
  
---|---