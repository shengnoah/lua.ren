---
layout: post
title: 基于Nginx&Lua 和Netflix Eureka的微服务网关 
tags: [lua文章]
categories: [topic]
---
依赖：lua-resty-http 基于Nginx&Lua 和Netflix Eureka的微服务网关。

重新架构了内部组件，采用插件模式。

  * 服务发现 
    * Eureka Discovery
    * 抽象discovery，用来支持多种服务发现？规划中…
  * 动态路由
  * 负载均衡 
    * 加权轮询
    * 基于响应时间的动态权重轮询？开发中…
  * 简单监控
  * 隔离降级
  * 限流
  * metrics
  * 认证安全？规划中。。。
  * 监控页面？开发中…

## 架构图：

![img](https://github.com/tietang/ngx-lua-zuul/raw/_dev_pilot/doc/arch.png)

## 使用方法

基于Nginx和Lua module。需要[安装Nginx
Lua环境](https://www.jianshu.com/p/7c320140c6aa)或者直接下载[openresty](https://link.jianshu.com/?t=http://openresty.org/en/download.html)编译安装。

## 安装和配置ngx-lua-zuul

### 下载代码到/path/to/nginx/lua/lib/

> git clone [http://github.com/tietang/ngx-lua-
> zuul](https://link.jianshu.com/?t=http://github.com/tietang/ngx-lua-zuul)
> –depth=1

## 例子Eureka 服务

如果没有Eureka环境，也可以编译安装本例子中的EurekaDemo服务，参考[编译和运行eureka-
demo服务](https://link.jianshu.com/?t=run-eureka-demo.md)中的相关内容。

### 部署dicovery例子服务：

下载代码后：

> cd /path/to/ngx_lua-zuul/demo/java mvn clean install

将下载的代码中的lua文件夹放到部署目录`/path/to/nginx`，修改`/path/to/nginx/lua/ngx_conf/lua.ngx_conf`文件中的`lua_package_path`为你的真实路径：
`lua_package_path "/path/to/nginx/lua/lib/?.lua;;";`

### 修改`/path/to/nginx/conf/nginx.conf`文件

**http 节点中添加**

    
    
    1
    

|

    
    
    include "/path/to/lua/ngx_conf/ngx_inlude_http.conf";  
  
---|---  
  
**server节点中添加**

    
    
    1
    

|

    
    
    include "/path/to/nginx/lua/ngx_conf/ngx_inlude_server.conf";  
  
---|---  
  
## 参考配置

    
    
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
    25
    26
    27
    28
    29
    30
    31
    32
    33
    34
    35
    36
    37
    38
    39
    40
    41
    42
    43
    44
    45
    46
    47
    48
    49
    50
    51
    52
    53
    54
    55
    56
    57
    58
    59
    60
    61
    

|

    
    
     #user  nobody;
    worker_processes  2;
    
    #error_log  logs/error.log;
    #error_log  logs/error.log  notice;
    #error_log  logs/error.log  info;
    
    #pid        logs/nginx.pid;
    
    
    events {
        worker_connections  1024;
    }
    
    
    http {
        include       mime.types;
        default_type  application/octet-stream;
    
        #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
        #                  '$status $body_bytes_sent "$http_referer" '
        #                  '"$http_user_agent" "$http_x_forwarded_for"';
    
        #access_log  logs/access.log  main;
    
        sendfile        on;
        #tcp_nopush     on;
    
        #keepalive_timeout  0;
        keepalive_timeout  65;
    
        #gzip  on;
        include "/Users/tietang/nginx/nginx/lua/ngx_conf/ngx_inlude_http.conf";
    
    
        server {
    
            include "/Users/tietang/nginx/nginx/lua/ngx_conf/ngx_inlude_server.conf";
            #error_page  404              /404.html;
    
            # redirect server error pages to the static page /50x.html
            #
    
            location = / {
                set $dir $document_root;
                root   $dir/html;
                index  index.html index.htm;
            }
    
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
    
            
        }
    
    
       
    
    }  
  
---|---  
  
### 运行测试

启动所有的demo服务：discovery,api,zuul；

启动nginx；

打开浏览器：[http://127.0.0.1:8000/api/test/0/0](https://link.jianshu.com/?t=http://127.0.0.1:8000/api/test/0/0)

其测试api参考[编译和运行eureka-demo服务](https://link.jianshu.com/?t=run-eureka-
demo.md)中的相关内容。