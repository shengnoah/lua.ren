---
layout: post
title: 如何用 openresty + lua 快速搭建一个简单网站 
tags: [lua文章]
categories: [topic]
---
最近在折腾 OpenResty，OpenResty 就不用多介绍了，这方面国内质量好的资料也不多，  
如果非要选一个入门级别的资料，自然是《OpenResty 最佳实践》。断断续续看了一些 Lua 的语法和 OpenResty 基础，  
就想做点什么练练手。碰巧某天。同事介绍了几个 OpenResty 的 lua 库，一看刚刚好，可以做来搭个简单的网站。  
那么我们就开始吧…

### 安装 OpenResty

在上一篇文章，记录了几个 Mac 安装 openresty 的几个坑。有兴趣的可以看看，不多介绍了。

### 搭建项目

项目目录很简单，刚刚介绍的那个 lua 库 lua-resty-template，看库的名字就知道这是个模板相关的库，主要是提供模板渲染的功能。  
有一些自己的模板语法，这和其他语法的模板语法一样的。

项目目录结构如下：  

    
    
    1  
    

|

    
    
    .
    ├── conf
    │   └── nginx.conf
    ├── html
    │   ├── _base.html
    │   └── view.html
    ├── lua_file
    │   └── content.lua
    ├── readme.md
    └── requirements.txt  
      
  
---|---  
  
基本机构有了，开始安装相关依赖，我自己的openresty是安装在/opt目录下，大致看下结构  

    
    
    1  
    

|

    
    
    .
    ├── bin
    ├── lualib
    │   └── cjson.so
    │   └── ngx
    │   └── rds
    │   └── redis
    │   └── resty
    ├── luajit
    ├── nginx
    └── pod
    └── resty.index  
      
  
---|---  
  
lualib/resty 主要就是openresty内置的resty 模块，为了将第三方模块区分出来，最好新建一个目录  

    
    
    1  
    

|

    
    
    mkdir /opt/openresty/lualib/3rd_resty/resty  
      
  
---|---  
  
可能会奇怪为什么需要`3rd_resty/resty`这样的嵌套目录，因为 template.lua
文件的模块是`resty.template`，这样才能正确的被lua导入，  
将需要的lua-resty-template/template.lua
文件放置于`/opt/openresty/lualib/3rd_resty/resty`，这一步就结束了。

### Coding

开始编码，其实首先需要修改原始的Nginx配置文件，在nginx.conf 添加以下2行。  

    
    
    1  
    

|

    
    
    lua_package_path '/opt/openresty/lualib/3rd_resty/?.lua;;'
    include /path/to/your/nginx.conf;  
      
  
---|---  
  
lua_package_path 是 lua_nginx_module 库的内置参数，设置了lua的包解析路径。

项目的nginx.conf 是这样。  

    
    
    1  
    

|

    
    
    server {
        listen 6699;
        set $tempalte_root /path/to/project/html;
    
        location / {
            default_type text/html;
            content_by_lua_file /path/to/lua_file/content.lua;
        }
    }  
      
  
---|---  
  
`set $tempalte_root /path/to/project/html;` 是设置html文件的解析路径，  
`content_by_lua_file` 也是 lua_nginx_module
库的内置参数，我们完成功能最关键的语句，会调用lua文件执行代码，获得返回结果，直接响应请求。

其他lua代码和html代码就不贴上来了，有兴趣的去[github](https://github.com/JackyXiong/openresty-
lua-site)看吧。

### 其他

不得不说，openresty通过lua扩展nginx的功能，使其更加的强大。使用它来搭建网站不能真正发挥其在服务端的作用，本文只是管中窥豹，后续还有更多值得深入的地方。