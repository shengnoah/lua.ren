---
layout: post
title: nginx-lua-redis 
tags: [lua文章]
categories: [topic]
---
## lua

    
    
    1  
    2  
    3  
    

|

    
    
    apt-get install lua5.1  
    apt-get install liblua5.1-dev  
    apt-get install liblua5.1-socket2  
      
  
---|---  
  
如果liblua5.1-socket2安装不上，需要添加源  

    
    
    1  
    

|

    
    
    deb http:  
      
  
---|---  
  
然后执行  

    
    
    1  
    

|

    
    
    sudo apt-get update  
      
  
---|---  
  
## 扩展

### lua-redis-parser

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    git clone https://github.com/agentzh/lua-redis-parser.git  
    export LUA_INCLUDE_DIR=/usr/include/lua5.1  
    cd lua-redis-parser  
    make CC=gcc  
    make install CC=gcc  
      
  
---|---  
  
### json

    
    
    1  
    2  
    3  
    

|

    
    
    wget http://files.luaforge.net/releases/json/json/0.9.50/json4lua-0.9.50.zip  
    unzip json4lua-0.9.50.zip  
    cp json4lua-0.9.50/json/json.lua /usr/share/lua/5.1/  
      
  
---|---  
  
### redis-lua

    
    
    1  
    2  
    

|

    
    
    git clone https://github.com/nrk/redis-lua.git  
    cp redis-lua/src/redis.lua /usr/share/lua/5.1/  
      
  
---|---  
  
## nginx

### module

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    git clone https://github.com/simpl/ngx_devel_kit.git  
    git clone https://github.com/chaoslawful/lua-nginx-module.git  
    git clone https://github.com/agentzh/redis2-nginx-module.git  
    git clone https://github.com/agentzh/set-misc-nginx-module.git  
    git clone https://github.com/agentzh/echo-nginx-module.git  
    git clone https://github.com/catap/ngx_http_upstream_keepalive.git  
      
  
---|---  
  
### 编译依赖

    
    
    1  
    2  
    

|

    
    
    apt-get install libpcre3 libpcre3-dev libltdl-dev libssl-dev libjpeg62 libjpeg62-dev libpng12-0 libpng12-dev libxml2-dev libcurl4-openssl-dev libmcrypt-dev autoconf libxslt1-dev  libgeoip-dev libperl-dev -y  
    apt-get install libgd2-noxpm-dev -y  
      
  
---|---  
  
### 下载

    
    
    1  
    2  
    3  
    

|

    
    
    wget http://nginx.org/download/nginx-1.8.0.tar.gz  
    tar zxvf nginx-1.8.0.tar.gz  
    cd nginx-1.8.0  
      
  
---|---  
  
### 配置

    
    
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
    

|

    
    
    ./configure --prefix=/usr/local/nginx --with-debug --with-http_addition_module   
    --with-http_dav_module --with-http_flv_module --with-http_geoip_module   
    --with-http_gzip_static_module --with-http_image_filter_module --with-http_perl_module   
    --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module   
    --with-http_stub_status_module --with-http_ssl_module --with-http_sub_module   
    --with-http_xslt_module --with-ipv6 --with-sha1=/usr/include/openssl   
    --with-md5=/usr/include/openssl --with-mail --with-mail_ssl_module   
    --add-module=../ngx_devel_kit   
    --add-module=../echo-nginx-module   
    --add-module=../lua-nginx-module   
    --add-module=../redis2-nginx-module   
    --add-module=../set-misc-nginx-module  
      
  
---|---  
  
### 编译安装

    
    
    1  
    2  
    

|

    
    
    make  
    make install  
      
  
---|---  
  
nginx被安装在`/usr/local/nginx` 下面,默认环境变量里面没有，需要增加连接  

    
    
    1  
    

|

    
    
    ln -s /usr/local/nginx/sbin/nginx /usr/sbin/nginx  
      
  
---|---  
  
# 配置

## nginx.conf

注释掉 server块，增加引用  

    
    
    1  
    

|

    
    
    include /usr/local/etc/nginx/conf.d/*.conf;  
      
  
---|---  
  
## demo.conf

    
    
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
    

|

    
    
     "/usr/share/lua/5.1/?.lua;;";  
    server {  
        listen 80;  
        server_name demo.local;      
        root  html;  
      
        location /demo {  
            default_type 'text/html';  
            access_by_lua_file "/opt/demo.lua";  
        }  
    }  
      
  
---|---  
  
## demo.lua

    
    
    1  
    

|

    
    
    ngx.say("hello,world")  
      
  
---|---