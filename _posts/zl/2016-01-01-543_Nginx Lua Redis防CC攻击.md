---
layout: post
title: Nginx Lua Redis防CC攻击 
tags: [lua文章]
categories: [topic]
---
__web安全 __

### 原理

Nginx Lua Redis防止CC攻击实现原理：

同一个外网IP、同一个网址(ngx.var.request_uri)、同一个客户端(http_user_agent)在某一段时间(CCseconds)内访问某个网址(ngx.var.request_uri)超过指定次数(CCcount)，则禁止这个外网IP+同一个客户端(md5(IP+ngx.var.http_user_agent)访问这个网址(ngx.var.request_uri)一段时间(blackseconds)。

该脚本使用lua编写(依赖nginx+lua)，将信息写到redis(依赖redis.lua)。

### Nginx lua模块安装

重新编译nginx，安装lua模块，或者直接使用《[OneinStack](https://oneinstack.com/)》安装OpenResty自带改模块

    
    
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
    

|

    
    
    pushd /root/oneinstack/src  
    wget -c http://nginx.org/download/nginx-1.12.1.tar.gz  
    wget -c http://mirrors.linuxeye.com/oneinstack/src/openssl-1.0.2l.tar.gz  
    wget -c http://mirrors.linuxeye.com/oneinstack/src/pcre-8.41.tar.gz  
    wget -c http://luajit.org/download/LuaJIT-2.0.5.tar.gz  
    git clone https://github.com/simpl/ngx_devel_kit.git  
    git clone https://github.com/openresty/lua-nginx-module.git  
    tar xzf nginx-1.12.1.tar.gz  
    tar xzf openssl-1.0.2l.tar.gz  
    tar xzf pcre-8.41.tar.gz  
    tar xzf LuaJIT-2.0.5.tar.gz  
    pushd LuaJIT-2.0.5  
    make && make install  
    popd  
    pushd nginx-1.12.1  
    ./configure --prefix=/usr/local/nginx --user=www --group=www --with-http_stub_status_module --with-http_v2_module --with-http_ssl_module --with-http_gzip_static_module --with-http_realip_module --with-http_flv_module --with-http_mp4_module --with-openssl=../openssl-1.0.2l --with-pcre=../pcre-8.41 --with-pcre-jit --with-ld-opt=-ljemalloc --add-module=../lua-nginx-module --add-module=../ngx_devel_kit  
    make  
    mv /usr/local/nginx/sbin/nginx{,_bk}  
    cp objs/nginx /usr/local/nginx/sbin  
    nginx -t   
      
  
---|---  
  
### 加载redis.lua

    
    
    1  
    2  
    3  
    

|

    
    
    mkdir /usr/local/nginx/conf/lua  
    cd /usr/local/nginx/conf/lua  
    wget https://github.com/openresty/lua-resty-redis/raw/master/lib/resty/redis.lua  
      
  
---|---  
  
在/usr/local/nginx/conf/nginx.conf http { }中添加：

    
    
    1  
    2  
    

|

    
    
    #the Nginx bundle:  
    lua_package_path "/usr/local/nginx/conf/lua/redis.lua";  
      
  
---|---  
  
### 防止CC规则waf.lua

将下面内容保存在/usr/local/nginx/conf/lua/waf.lua

    
    
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
    

|

    
    
    local get_headers = ngx.req.get_headers  
    local ua = ngx.var.http_user_agent  
    local uri = ngx.var.request_uri  
    local url = ngx.var.host .. uri  
    local redis = require 'redis'  
    local red = redis.new()  
    local CCcount = 20  
    local CCseconds = 60  
    local RedisIP = '127.0.0.1'  
    local RedisPORT = 6379  
    local blackseconds = 7200  
    if ua == nil then  
        ua = "unknown"  
    end  
    if (uri == "/wp-admin.php") then  
        CCcount=20  
        CCseconds=60  
    end  
    red:set_timeout(100)  
    local ok, err = red.connect(red, RedisIP, RedisPORT)  
    if ok then  
        red.connect(red, RedisIP, RedisPORT)  
        function ()  
            IP = ngx.req.get_headers()["X-Real-IP"]  
            if IP == nil then  
                IP = ngx.req.get_headers()["x_forwarded_for"]  
            end  
            if IP == nil then  
                IP  = ngx.var.remote_addr  
            end  
            if IP == nil then  
                IP  = "unknown"  
            end  
            return IP  
        end  
        local token = getClientIp() .. "." .. ngx.md5(url .. ua)  
        local req = red:exists(token)  
        if req == 0 then  
            red:incr(token)  
            red:expire(token,CCseconds)  
        else  
            local times = tonumber(red:get(token))  
            if times >= CCcount then  
                local blackReq = red:exists("black." .. token)  
                if (blackReq == 0) then  
                    red:set("black." .. token,1)  
                    red:expire("black." .. token,blackseconds)  
                    red:expire(token,blackseconds)  
                    ngx.exit(580)  
                else  
                    ngx.exit(580)  
                end  
                return  
            else  
                red:incr(token)  
            end  
        end  
        return  
    end  
      
  
---|---  
  
### Nginx虚拟主机加载waf.lua

在虚拟主机配置文件/usr/local/nginx/conf/vhost/oneinstack.com.conf

    
    
    1  
    

|

    
    
    access_by_lua_file "/usr/local/nginx/conf/lua/waf.lua";  
      
  
---|---  
  
### 测试

一分钟之内，一个页面快速点击20次以上，登录redis，看到black开通的key即被禁止访问（nginx 503）

![](https://jiesunn.github.io//2019/02/02/2019-02-02-Nginx%20Lua%20Redis防CC攻击/pic1.png)