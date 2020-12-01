---
layout: post
title: centos下安装openresty+ngx_lua_waf防火墙部署 
tags: [lua文章]
categories: [topic]
---
全文默认安装路径：/usr/local/src

### [](https://stepwen.github.io/#1%E3%80%81%E5%AE%89%E8%A3%85Luagit
"1、安装Luagit:")1、安装Luagit:

    
    
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
  
### [](https://stepwen.github.io/#2%E3%80%81%E5%AE%89%E8%A3%85openresty
"2、安装openresty:")2、安装openresty:

    
    
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
  
### [](https://stepwen.github.io/#3%E3%80%81%E4%B8%8B%E8%BD%BDngx-lua-waf
"3、下载ngx_lua_waf")3、下载ngx_lua_waf

    
    
    1  
    2  
    

|

    
    
    # cd /usr/local/openresty/nginx/  
    # git clone https://github.com/loveshell/ngx_lua_waf.git  
      
  
---|---  
  
###
[](https://stepwen.github.io/#4%E3%80%81%E4%BF%AE%E6%94%B9nginx%E6%B7%BB%E5%8A%A0%E9%85%8D%E7%BD%AE%EF%BC%8C%E6%94%AF%E6%8C%81lua%E8%84%9A%E6%9C%AC%E5%9C%B0%E5%9D%80%EF%BC%8C%E5%9C%A8http%E6%AE%B5%E4%BD%8D%E7%BD%AE%EF%BC%9A
"4、修改nginx添加配置，支持lua脚本地址，在http段位置：")4、修改nginx添加配置，支持lua脚本地址，在http段位置：

    
    
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
  
### [](https://stepwen.github.io/#5%E3%80%81%E4%BF%AE%E6%94%B9ngx-lua-
waf%E7%9B%B8%E5%85%B3%E9%85%8D%E7%BD%AE%EF%BC%9A
"5、修改ngx_lua_waf相关配置：")5、修改ngx_lua_waf相关配置：

    
    
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
  
###
[](https://stepwen.github.io/#6%E3%80%81%E5%88%9B%E5%BB%BA%E6%97%A5%E5%BF%97%E5%AD%98%E6%94%BE%E7%9B%AE%E5%BD%95%EF%BC%9A
"6、创建日志存放目录：")6、创建日志存放目录：

    
    
    1  
    2  
    

|

    
    
    # mkdir /usr/local/openresty/nginx/logs/hack/  
    # chown -R nobody:nobody /usr/local/openresty/nginx/logs/hack/  
      
  
---|---  
  
###
[](https://stepwen.github.io/#7%E3%80%81%E5%90%AF%E5%8A%A8nginx-%E6%B5%8B%E8%AF%95
"7、启动nginx 测试")7、启动nginx 测试

    
    
    1  
    2  
    

|

    
    
    # cd /usr/local/openresty/nginx/sbin  
    # ./nginx  
      
  
---|---