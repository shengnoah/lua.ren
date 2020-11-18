---
layout: post
title: 基于OpenResty安装配置Nginx+LUA并实现请求分发的双层Nginx架构策略 
tags: [lua文章]
categories: [lua文章]
---
台CentOS6.x  
192.168.1.210  
192.168.1.211  
192.168.1.212  
网络拓扑  
210和211作为应用层web服务器  
212作为网络请求分发代理服务器

# Step1:安装Linux依赖

    
    
    1  
    

|

    
    
    yum install -y readline-devel pcre-devel openssl-devel gcc  
      
  
---|---  
  
# Step2:安装Nginx Openresty

    
    
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
    

|

    
    
    wget http://openresty.org/download/ngx_openresty-1.7.7.2.tar.gz    
    tar -xzvf ngx_openresty-1.7.7.2.tar.gz    
    cd /usr/servers/ngx_openresty-1.7.7.2/  
      
    cd bundle/LuaJIT-2.1-20150120/    
    make clean && make && make install    
    ln -sf luajit-2.1.0-alpha /usr/local/bin/luajit  
      
    cd bundle    
    wget https://github.com/FRiCKLE/ngx_cache_purge/archive/2.3.tar.gz    
    tar -xvf 2.3.tar.gz    
      
    cd bundle    
    wget https://github.com/yaoweibin/nginx_upstream_check_module/archive/v0.3.0.tar.gz    
    tar -xvf v0.3.0.tar.gz    
      
    cd /usr/servers/ngx_openresty-1.7.7.2    
    ./configure --prefix=/usr/servers --with-http_realip_module  --with-pcre  --with-luajit --add-module=./bundle/ngx_cache_purge-2.3/ --add-module=./bundle/nginx_upstream_check_module-0.3.0/ -j2    
    make && make install  
      
  
---|---  
  
# Step3:检查安装并启动

安装完成后有以下目录：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    cd /usr/servers/    
    ll  
      
    /usr/servers/luajit  
    /usr/servers/lualib  
    /usr/servers/nginx  
      
  
---|---  
  
检查nginx版本

    
    
    1  
    

|

    
    
    /usr/servers/nginx/sbin/nginx -V  
      
  
---|---  
  
启动nginx:

    
    
    1  
    

|

    
    
    /usr/servers/nginx/sbin/nginx  
      
  
---|---  
  
# Step4:Nginx添加LUA配置

使LUA配置按照工程化目录结构进行配置。

    
    
    1  
    2  
    3  
    

|

    
    
    mkdir /usr/hello  
      
    cp -r /usr/servers/lualib /usr/hello/  
      
  
---|---  
  
打开nginx.conf配置文件

    
    
    1  
    

|

    
    
    vi /usr/servers/nginx/conf/nginx.conf  
      
  
---|---  
  
在http部分添加：

    
    
    1  
    2  
    3  
    

|

    
    
    lua_package_path "/usr/hello/lualib/?.lua;;";    
    lua_package_cpath "/usr/hello/lualib/?.so;;";    
    include /usr/hello/hello.conf;  
      
  
---|---  
  
创建hello的lua配置和脚本

    
    
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

    
    
    vi /usr/hello/hello.conf  
      
    server {    
        listen       80;    
        server_name  _;    
        
        location /hello/test {    
            default_type 'text/html';     
            content_by_lua_file /usr/hello/lua/test.lua;    
        }    
    }  
      
  
---|---  
  
编辑lua脚本:

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    mkdir /usr/hello/lua  
      
    vi /usr/hello/lua/test.lua  
      
    ngx.say("hello world")  
      
  
---|---  
  
Nginx检查配置是否正确

    
    
    1  
    2  
    3  
    

|

    
    
    [root@eshop-cache01 hello]# /usr/servers/nginx/sbin/nginx -t    
    nginx: the configuration file /usr/servers/nginx/conf/nginx.conf syntax is ok  
    nginx: configuration file /usr/servers/nginx/conf/nginx.conf test is successful  
      
  
---|---  
  
Nginx重新加载配置生效

    
    
    1  
    

|

    
    
    /usr/servers/nginx/sbin/nginx -s reload  
      
  
---|---  
  
# Step5:访问LUA配置的location路径

    
    
    1  
    

|

    
    
    curl http://192.168.1.210/hello/test  
      
  
---|---  
  
返回test.lua中脚本输出的“hello world”。

# Step6:分发层Nginx安装LUA脚本实现请求分发

我们作为一个流量分发的nginx，会发送http请求到后端的应用nginx上面去，所以要先引入lua http lib包。

    
    
    1  
    2  
    3  
    

|

    
    
    cd /usr/hello/lualib/resty/    
    wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http_headers.lua    
    wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http.lua  
      
  
---|---  
  
在分发层Nginx写LUA脚本：

    
    
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
    

|

    
    
    nano /usr/hello/lua/test.lua  
      
    local uri_args = ngx.req.get_uri_args()  
    local ecourseId = uri_args["ecourseId"]  
      
    local host = {"192.168.1.210", "192.168.1.211"}  
    local hash = ngx.crc32_long(ecourseId)  
    hash = (hash % 2) + 1  
    backend = "http://"..host[hash]  
      
    local paras = "";  
    local request_args_tab = ngx.req.get_uri_args()  
    for k, v in pairs(request_args_tab) do  
        paras=paras..k.."="..v.."&"  
    end  
      
    local requestPath = ngx.var.uri  
    requestPath = requestPath.."?"..paras  
      
    local http = require("resty.http")  
    local httpc = http.new()  
      
    local resp, err = httpc:request_uri(backend, {  
        method = "GET",  
        path = requestPath  
    })  
      
    if not resp then  
        ngx.say("request error :", err)  
        return  
    end  
      
    ngx.say(resp.body)  
      
    httpc:close()  
      
  
---|---  
  
Nginx重新加载配置生效

    
    
    1  
    

|

    
    
    /usr/servers/nginx/sbin/nginx -s reload  
      
  
---|---  
  
测试请求分发,在浏览器地址栏输入

<http://192.168.1.212/hello/test?ecourseId=1>  
<http://192.168.1.212/hello/test?ecourseId=2>  
<http://192.168.1.212/hello/test?ecourseId=3>  
<http://192.168.1.212/hello/test?ecourseId=4>  
查看返回的结果，请求被分发到应用层web服务器节点了。根据ecourseId与应用层web服务节点数取模，找到对应的应用层服务器节点。

# Step7:按照相同的方法部署另外两台机器的Nginx

安装过程，略…  
应用层Nginx添加http请求功能包即可。

    
    
    1  
    2  
    3  
    

|

    
    
    cd /usr/hello/lualib/resty/    
    wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http_headers.lua    
    wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http.lua  
      
  
---|---  
  
# 补充：LUA脚本获取Nginx Http请求的相关参数说明

1.获取当前请求的url相关信息

    
    
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
    

|

    
    
    function test()  
    -- 这个变量等于包含一些客户端请求参数的原始URI，它无法修改，请查看$uri更改或重写URI。  
    local request_uri = ngx.var.request_uri  
     log(tools.gbk_to_u8("获取当前请求的url==") .. tools.u8_to_gbk(cjson.encode(request_uri)) )  
      
     -- HTTP方法（如http，https）。按需使用，例：  
     local scheme = ngx.var.scheme server_addr  
     log(tools.gbk_to_u8("获取当前请求的url scheme==") .. tools.u8_to_gbk(cjson.encode(scheme)) )  
      
     -- 服务器地址，在完成一次系统调用后可以确定这个值，如果要绕开系统调用，则必须在listen中指定地址并且使用bind参数。  
     local server_addr = ngx.var.server_addruri   
     log(tools.gbk_to_u8("获取当前请求的url server_addr==") .. tools.u8_to_gbk(cjson.encode(server_addr)) )  
      
    -- 请求中的当前URI(不带请求参数，参数位于$args)，可以不同于浏览器传递的$request_uri的值，它可以通过内部重定向，或者使用index指令进行修改。  
     local uri = ngx.var.uri   
     log(tools.gbk_to_u8("获取当前请求的url uri==") .. tools.u8_to_gbk(cjson.encode(uri)) )  
      
     -- 服务器名称  
     local server_name  = ngx.var.server_name    
     log(tools.gbk_to_u8("获取当前请求的url server_name ==") .. tools.u8_to_gbk(cjson.encode(server_name ))   
      
     -- 请求到达服务器的端口号。  
    local server_port  = ngx.var.server_name    
     log(tools.gbk_to_u8("获取当前请求的url server_port ==") .. tools.u8_to_gbk(cjson.encode(server_port ))   
    end  
      
      
    test()  
      
  
---|---  
  
2.获取发送请求端过来的url相关信息

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    -- 获取远程的IP地址。  
    local remote_addr  = ngx.var.remote_addr   
     log(m_uuid,tools.gbk_to_u8("获取发送请求过来的远程请求remote_addr ==") .. tools.u8_to_gbk(cjson.encode(remote_addr )) )  
      
     -- 获取远程的端口号  
     local remote_port  = ngx.var.remote_port    
     log(m_uuid,tools.gbk_to_u8("获取发送请求过来的远程请求remote_port ==") .. tools.u8_to_gbk(cjson.encode(remote_port )) )  
      
  
---|---