---
layout: post
title: centos下安装openresty+ngx_lua_waf防火墙部署 
tags: [lua文章]
categories: [topic]
---
全文默认安装路径：/usr/local/src

### 1、安装Luagit:

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    # cd /usr/local/src  
    # wget http://luajit.org/download/LuaJIT-2.1.0-beta3.tar.gz  
    # tar -zxvf LuaJIT-2.1.0-beta3.tar.gz  
    # cd LuaJIT-2.1.0-beta3  
    # make  
    # make install  
    #ln -sf luajit-2.1.0-beta3 /usr/local/bin/luajit  
      
  
---|---  
  
### 2、安装openresty:

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
    
    # cd /usr/local/src  
    # wget http://openresty.org/download/ngx_openresty-1.13.6.2.tar.gz  
    # wget https://sourceforge.net/projects/pcre/files/pcre/8.42/pcre-8.42.tar.gz  
    # tar -zvxf pcre-8.42.tar.gz  
    # tar -zvxf ngx_openresty-1.13.6.2.tar.gz  
    # cd ngx_openresty-1.13.6.2.tar.gz  
    # ./configure --prefix=/usr/local/openresty --with-luajit --with-pcre=/usr/local/src/pcre-8.42/ --with-http_stub_status_module --with-http_gzip_static_module --with-http_ssl_module  
    # gmake && gmake install  
      
  
---|---  
  
### 3、下载ngx_lua_waf

    
    
    1  
    2  
    

|

    
    
    # cd /usr/local/openresty/nginx/  
    # git clone https://github.com/loveshell/ngx_lua_waf.git  
      
  
---|---  
  
### 4、修改nginx添加配置，支持lua脚本地址，在http段位置：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    ###相关项目存放地址  
    lua_package_path "/usr/local/openresty/nginx/ngx_lua_waf/?.lua";  
    ###存放limit表的大小  
    lua_shared_dict limit 10m;  
    ###相应地址  
    init_by_lua_file  /usr/local/openresty/nginx/ngx_lua_waf/init.lua;  
    access_by_lua_file /usr/local/openresty/nginx/ngx_lua_waf/waf.lua;  
      
  
---|---  
  
### 5、修改ngx_lua_waf相关配置：

    
    
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
    

|

    
    
    RulePath = "/usr/local/openresty/nginx/ngx_lua_waf/wafconf/"   ##指定相应位置  
    attacklog = "on"                            ##开启日志  
    logdir = "/usr/local/openresty/nginx/logs/hack/"           ##日志存放位置  
    UrlDeny="on"                              ##是否开启URL防护  
    Redirect="on"                             ##地址重定向  
    CookieMatch="on"                           ##cookie拦截  
    postMatch="on"                            ##post拦截  
    whiteModule="on"                           ##白名单  
    black_fileExt={"php","jsp"}                          
    ipWhitelist={"127.0.0.1"}                    ##白名单IP  
    ipBlocklist={"1.0.0.1"}                     ##黑名单IP  
    CCDeny="on"                             ##开启CC防护          
    CCrate="100/60"                          ##60秒内允许同一个IP访问100次  
      
  
---|---  
  
### 6、创建日志存放目录：

    
    
    1  
    2  
    

|

    
    
    # mkdir /usr/local/openresty/nginx/logs/hack/  
    # chown -R nobody:nobody /usr/local/openresty/nginx/logs/hack/  
      
  
---|---  
  
### 7、启动nginx 测试

    
    
    1  
    2  
    

|

    
    
    # cd /usr/local/openresty/nginx/sbin  
    # ./nginx  
      
  
---|---