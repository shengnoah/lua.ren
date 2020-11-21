---
layout: post
title: nginx + lua 开发中过程中 post body 过大返回 4xx 
tags: [lua文章]
categories: [topic]
---
## 背景

基于 OpenResty 提供 post 接口，调用方调用该接口 post 数据，该接口接收 post 过来的数据，复用 Nginx access
日志落盘。

## 问题

当用户的 body 体过大时，`ngx.req.get_body_data()` 读请求体，会出现读取不到直接返回 `nil` 的情况。

## 问题原因

究其原因，主要是 Nginx 诞生之初主要是为了解决负载均衡情况，而这种情况，是不需要读取 body 就可以决定负载策略的，所以这个点对于 API
Server 和 Web Application 开发的同学有点怪。

## 解决办法

  1. 如果你只是某个接口需要读取 body（并非全局行为），那么这时候也可以显示调用 `ngx.req.read_body()` 接口
  2. 如果想全局生效的话需要使用命令`lua_need_request_body on;`

当选择上述其中一种，甚至两种方案都使用了，依旧还解决不了问题，这时候，需要坚持 body 体是不是太大了。这是需要设置如下两个命令：  

    
    
    1  
    2  
    

|

    
    
    client_body_buffer_size 256k; #默认8k|16k  
    client_max_body_size 256k; #默认1m  
      
  
---|---  
  
上述两条命令将强制将 body 写入内存，这样就可以读取较大的 body 体的数据了。

如果请求体已经被存入临时文件，请使用`ngx.req.get_body_file`函数代替。