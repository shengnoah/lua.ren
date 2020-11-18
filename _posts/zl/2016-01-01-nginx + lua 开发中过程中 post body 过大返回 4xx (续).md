---
layout: post
title: nginx + lua 开发中过程中 post body 过大返回 4xx (续) 
tags: [lua文章]
categories: [lua文章]
---
## 背景

随着业务发展会出现较大的 post body 数据，按照[nginx + lua 开发中过程中 post body 过大返回
4xx](/2019/05/28/nginx-lua-开发中过程中-post-body-过大返回-4xx/ "nginx + lua 开发中过程中 post
body 过大返回 4xx")提到的方式修改后，大部分情况下 post body 正常接收并处理落日志。但会偶现空日志的情况。

## 问题分析

经过多轮本地和沙盒压测，复现了问题。由于在出现空日志情况是 error 日志并没留下相关信息，随后做了如下处理：

  1. 把 error 日志级别调到 debug，当问题复现时，error.log 中会有客户端过早断开连接类似的日子打出。
  2. 在 access 日志中添加 request_time, status,等信息，发现出现空日志时，status=408，request_time 都比较长。

因此，可以明确出现该问题是客户端链接超时造成的。

## 解决方案

为解决该问题，做如下优化：

### 调整超时时间，和 buffer

    
    
    1  
    2  
    3  
    

|

    
    
    client_body_timeout 10s;  
    client_header_timeout 10s;  
    client_body_in_single_buffer on; #这个 directive 让 Nginx 将所有的 request body 存储在一个缓冲当中，它的默认值是 off。启用它可以优化读取 $request_body 变量时的 I/O 性能  
      
  
---|---  
  
### 开启 access buffer 和 if

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    log_format  main escape=json '[$log_time] [$logid] [INFO] $click_info';  
    设置变量loggable，默认$loggable=0;当$status==200时，$loggable=1  
    map $status $loggable {  
    	200  1;  
        default 0;  
    }  
      
  
---|---  
  
只有真确处理了请求才会写日志。客户端在收到 408 时，会将 body 拆分重传。