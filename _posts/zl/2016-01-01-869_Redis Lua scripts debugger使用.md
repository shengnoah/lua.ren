---
layout: post
title: Redis Lua scripts debugger使用 
tags: [lua文章]
categories: [topic]
---
## 背景说明

使用`Redis`开发分布式应用时，难免会遇到需要使用分布式锁来确保某一小段逻辑的原子性操作，如：当存在某个`key`对应的值`A`大于值`B`时，则返回`false`；否则`A
+ 1`。试想一下，如果用到分布式锁，是不是有点感觉像是杀鸡用宰牛刀？

由于`Redis`的操作都是原子性的，所以我们可以将如上所述的类似逻辑采用`Lua`脚本表述作为一个原子任务向`redisClient`提交，可以避免采用分布式锁。如脚本:

    
    
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

    
    
    local key = KEYS[1]  
    local expire_time = tonumber(ARGV[1])  
    local max_count = tonumber(ARGV[2])  
      
      
    local is_key_exists = redis.call("EXISTS", key)  
      
    -- 存在  
    if is_key_exists == 1 then  
        local key_count = redis.call("get", key)  
        if tonumber(key_count) >= max_count then  
            return false;  
        else  
            redis.call("incr", key)  
            return true;  
        end  
    else  
        redis.call("incr", key);  
        redis.call("expire", key, expire_time)  
        return true;  
    end  
      
  
---|---  
  
但此时发现，在`Redis`中调试是个困难的事情。但幸好，从`Redis 3,2`版本开始，提供了`Lua script debugger`来解决此问题。

## Redis Lua调试器特点

`Redis Lua`调试器代号为`LDB`，特点如下：

  * 支持单步调试
  * 支持静态和动态断点
  * 支持将被调试的脚本载入至调试终端
  * 支持对`Lua`变量进行观察
  * 支持追踪脚本执行的`Redis`命令
  * 支持以美观方式打印`Redis`值及`Lua`值
  * 能够在无线循环及长时间执行步骤中模拟出断点
  * 采用`CS`模型，`S`即`Redis`服务器，`C`即`redis-cli`客户端
  * 默认情况下，每个调试会话都是`fork`子进程进行的。所以调试过程中不会影响`Redis`其他操作；调试结束后，本次会话中的内容都会被回滚
  * 你也可以将调试会话设置成同步模式，但是此时必须注意调试时会影响`Redis`所有其他的操作，且调试会话产生的结果都会被保存下来

## Redis Lua调试器快速入门

想看视频的直接点[这里](https://www.bilibili.com/video/av9437433/)，这是`Redis`之父录制的视频。

想看文字的继续。以我们在刚开始提供的`Lua`脚本，假设其目录为`~/Desktop/tmp/test.lua`。此时执行命令（需要先`cd`到`redis-
cli`命令目录下）：

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    -- async mode  
    redis-cli --ldb --eval redis_lua_file redis_key , param1 param2  
      
    -- sync mode  
    redis-cli --ldb-sync-mode --eval redis_lua_file redis_key , param1 param2  
      
  
---|---  
      
    
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

    
    
     ~  redis-cli --ldb --eval ~/Desktop/tmp/test.lua key , 1 3  
    Lua debugging session started, please use:  
    quit    -- End the session.  
    restart -- Restart the script in debug mode again.  
    help    -- Show Lua script debugging commands.  
      
    * Stopped at 6, stop reason = step over  
     6   local key = KEYS[1]  
    lua debugger>  
      
  
---|---  
  
> 注意：`redis-cli`里的逗号前后都是有空格的。我这里示例是本地有开启`redis-server`，如果没有开启也可以连接到远程`redis
> server`进行调试。另外，`key`表示的是要传入脚本中处理的`redis key`，其后的1和3则是参数，中间用空格隔开。

之后所要做的就是在`debugger`里输入`s`命令回车，表示执行下一步，直至程序结束或有异常退出为止。

## Redis Lua debug命令

括号中字母代表命令缩写。

![redis_lua_debug命令.png](https://s2.ax1x.com/2019/09/11/nwnpOU.png)

## 参考文章

  1. [Redis Lua scripts debugger](https://redis.io/topics/ldb)