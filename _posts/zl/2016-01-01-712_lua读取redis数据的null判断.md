---
layout: post
title: lua读取redis数据的null判断 
tags: [lua文章]
categories: [topic]
---
最近在配合移动端调试的时候，被抓去debug一个在清除redis缓存之后才会出现的网关错误。于是打开服务器上的log定位到类似错误:

    
    
    1  
    

|

    
    
    [error] 7#7: *12030 lua entry thread aborted: runtime error: /data/share/apps/lua/access_check.lua:133: bad argument #1 to 'decode' (string expected, got userdata)  
      
  
---|---  
  
该段代码的主要作用是在`openresty`中`lua`读取`redis`中数据并解码为`json`：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
    
    local access_token = redis_client:read_by_key(token_key)  
       if access_token == nil then  
             
           return false  
       end  
      
       local obj_token = cjson.decode(access_token)  
       -- do something  
      
  
---|---  
  
通过查询资料得知原因： **`lua`读取`redis`数据返回结果为空时，返回的结果不是`nil`而是`userdata`类型的`ngx.null`。**

* * *

#### [为什么要这么设计？](https://github.com/openresty/lua-resty-redis/issues/90)

>
> 因为`nil`在`lua`中有特殊的意义，如果一个变量被设置为`nil`相当于告知该变量`未定义`(不存在)一样，如果把`redis`查询的结果为空设置为`nil`，而该查询的`key`对应在`redis`中又是存在的，就无法把`查询为空`和`未定义`区分开来了，这样显然是不合理的。所以必须使用一个`userdata`类型的值来表示这个查询记录为空，但是又不等同于`未定义变量`（ngx.null)。

* * *

因此，代码做如下修改即可:

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
    
    local access_token = redis_client:read_by_key(token_key)  
       if access_token == ngx.null or access_token == nil then  
             
           return false  
       end  
      
       local obj_token = cjson.decode(access_token)  
       -- do something  
      
  
---|---