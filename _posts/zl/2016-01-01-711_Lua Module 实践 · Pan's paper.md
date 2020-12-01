---
layout: post
title: Lua Module 实践 · Pan's paper 
tags: [lua文章]
categories: [topic]
---
最近用openresty写接口，每个接口都要要链接数据库，写一个通用的模块来实现，顺便学习下lua module的相关知识。

### lua 模块

Lua中的一个模块就是一个table，table元素包括变量，函数等。“因此创建一个模块很简单，就是创建一个
table，然后把需要导出的常量、函数放入其中，最后返回这个 table”。

    
    
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
    

|

    
    
      
    -- 定义一个名为 module 的模块  
    module = {}  
      
    -- 定义一个常量  
    module.constant = "这是一个常量"  
      
    -- 定义一个函数  
    function ()  
        io.write("这是一个公有函数！n")  
    end  
      
    local function func2()  
        print("这是一个私有函数！")  
    end  
      
    function module.func3()  
        func2()  
    end  
      
    return module  
      
  
---|---  
  
#### require函数

require函数用来加载模块。

    
    
    1  
    2  
    3  
    

|

    
    
    require("<模块名>")  
    -- 或  
    require"<模块名>"  
      
  
---|---  
  
也可以给模块一个别名，方便调用

    
    
    1  
    2  
    3  
    

|

    
    
    local m = require("module")  
      
    m.func1  
      
  
---|---  
  
#### 加载机制

对于我们自定义的模块，并不是放在那个目录都行，require有一套自己的文件加载机制。

> require 用于搜索 Lua 文件的路径是存放在全局变量 package.path 中，当 Lua 启动后，会以环境变量 LUA_PATH
> 的值来初始这个环境变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。

如果我们设置LUA_PATH环境变量（参考1）

    
    
    1  
    2  
    

|

    
    
    #LUA_PATH  
    export LUA_PATH="~/lua/?.lua"  
      
  
---|---  
  
那么调用 require(“module”) 时就会尝试打开以下文件目录去搜索目标。

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    /Users/dengjoe/lua/module.lua;  
    ./module.lua  
    /usr/local/share/lua/5.1/module.lua  
    /usr/local/share/lua/5.1/module/init.lua  
    /usr/local/lib/lua/5.1/module.lua  
    /usr/local/lib/lua/5.1/module/init.lua  
      
  
---|---  
  
### openresty 实践

写一个链接数据库的module

#### 设置module路径

在openresty中设置module路径的方式：

    
    
    1  
    

|

    
    
    lua_package_path 'lua/?.lua;/Users/zhangpan/Project/XinanServer/XinanServer/lua/module/?.lua;;';  
      
  
---|---  
  
其中`lua/?.lua;` 代表的绝对路径是 `~/lua/?.lua;`；后面一个是绝对路径。

#### 设计module

    
    
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
    24  
    

|

    
    
    xinandb = {}  
      
    -- 创建数据库链接实例  
    local mysql = require "resty.mysql"  
      
    xinandb.db = mysql:new()  
      
    -- 连接数据库  
    function xinandb.connect()  
         if not xinandb.db then  
              return false  
         end  
      
         local ok, err, errno, sqlstate = (xinandb.db):connect {  
                        host = "127.0.0.1",  
                        port = 3306,  
                        database = "xinan",  
                        user = "***",  
                        password = "***",  
                        max_packet_size = 1024 * 1024 }  
         return ok                 
    end  
      
    return xinandb  
      
  
---|---  
  
#### yield

这里遇到的一个问题：开始想创建db时就直接链接

    
    
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

    
    
    xinandb.db = mysql:new()  
      
    local ok, err, errno, sqlstate = (xinandb.db):connect {  
                        host = "127.0.0.1",  
                        port = 3306,  
                        database = "xinan",  
                        user = "***",  
                        password = "***",  
                        max_packet_size = 1024 * 1024 }  
      
  
---|---  
  
这样就会报一个错误

> [error] 7091#0: *291 lua entry thread aborted: runtime error: attempt to
> yield across C-call boundary

最终参考文章[关于在config.lua中加入resolver包，出现跨界问题](https://orchina.org/topic/50/view)解决。

解决方式：把连接数据库操作封装在一个函数中供调用。

#### 使用

使用就比较简单了，和其他一样

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
    
    -- 连接DB  
    local xinandb = require("xinandb")  
    local res = xinandb:connect()  
    if not res then  
    	ngx.say("{"errorCode" : 1003, "errorMessage" : "连接数据库错误"}")  
    	return  
    end  
    local db = xinandb.db;  
      
  
---|---  
  
### 参考

#### 参考1：设置环境变量的方式：

> 如果没有 LUA_PATH 这个环境变量，也可以自定义设置，在当前用户根目录下打开 .profile文件（没有则创建，打开 .bashrc
> 文件也可以），例如把 “~/lua/“ 路径加入 LUA_PATH 环境变量里;

然后执行

    
    
    1  
    

|

    
    
    source ~/.profile  
      
  
---|---  
  
#### 坑：再次打开终端环境变量失效

用的是Mac系统，shell是zsh。按照上面的方式设置了环境变量后，关闭终端，再次打开后环境变量就失效了。

解决方式：将LUA_PATH添加到.zshrc 执行

    
    
    1  
    

|

    
    
    source ~/.zshrc  
      
  
---|---