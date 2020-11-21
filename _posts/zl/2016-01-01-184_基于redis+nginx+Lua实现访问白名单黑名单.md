---
layout: post
title: 基于redis+nginx+Lua实现访问白名单黑名单 
tags: [lua文章]
categories: [topic]
---
  1. 添加`echo-nginx-module`模块

    * git clone <https://github.com/openresty/echo-nginx-module.git> /usr/local/echo-nginx-module
    * 切换到nginx源码目录 cd /usr/local/src/nginx
    * 重新执行 ./configure –prefix=/usr/local/nginx –add-module=/usr/local/echo-nginx-module
    * 若提示错误少什么依赖就安装相应的依赖，无提示则执行 make && make install

    * 网上还有另一种方法,同理可以添加别的模块
    
        1  
    2  
    3  
    4  
    

|

    
        git clone https://github.com/nginx/nginx.git  
    git submodule add git@github.com:openresty/echo-nginx-module.git  
    #重新配置nginx,把echo-nginx-module模块编译进nginx可执行文档  
    sudo ./configure --add-module=echo-nginx-module  
      
  
---|---  
  2. 安装 ngx_devel_kit (NDK) 模块

    * cd /usr/local 
    * git clone <https://github.com/simpl/ngx_devel_kit.git>
  3. 添加`lua-nginx-module`模块

    * cd /usr/local 
    * git clone <https://github.com/chaoslawful/lua-nginx-module.git>
  4. 重新编译nginx
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
        ./configure --prefix=/usr/local/nginx   
                --with-ld-opt="-Wl,-rpath,$LUAJIT_LIB"   
                --add-module=/usr/local/ngx_devel_kit    
                --add-module=/usr/local/echo-nginx-module   
                --add-module=/usr/local/lua-nginx-module   
    make -j2   
    make install  
      
  
---|---  
  5. 重启nginx服务器

  6. 测试Lua，在nginx.conf配置文档server代码块中加入以下代码：
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
        location /echo {   
        default_type 'text/plain';   
        echo 'install echo module success!';   
    }   
    location /lua {   
        default_type 'text/plain';   
        content_by_lua 'ngx.say("install lua module success!")';   
    }  
      
  
---|---  
  7. curl [http://localhost/echo](https://liusir.me/http://localhost/echo) // 正常的话输出：install echo module success!

  8. curl [http://localhost/lua](https://liusir.me/http://localhost/lua) // 正常的话输出：install lua module success!

#  [](https://liusir.me/#%E4%BA%8C%E3%80%81%E5%AE%9E%E7%8E%B0 "二、实现")二、实现

  1. 实现原理：通过在nginx上进行访问限制，通过lua来灵活实现业务需求，redis用于存储黑名单列表
  2. 具体过程

    * step1:lua代码(post请求，ip地址黑名单，请求参数中imsi,tel值和黑名单)
    
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
    

|

    
        [root@git-server ~]# cat /usr/local/nginx/conf/lua/ipblacklist.lua  
    ngx.req.read_body()  
      
    local redis = require "resty.redis"  
    local red = redis.new()  
    red.connect(red, '127.0.0.1', '6379')  
      
    local myIP = ngx.req.get_headers()["X-Real-IP"]  
    if myIP == nil then  
       myIP = ngx.req.get_headers()["x_forwarded_for"]  
    end  
    if myIP == nil then  
       myIP = ngx.var.remote_addr  
    end  
      
    if ngx.re.match(ngx.var.uri,"^(/webapi/).*$") then  
        local method = ngx.var.request_method  
        if method == 'POST' then  
            local args = ngx.req.get_post_args()  
      
            local hasIP = red:sismember('black.ip',myIP)  
            local hasIMSI = red:sismember('black.imsi',args.imsi)  
            local hasTEL = red:sismember('black.tel',args.tel)  
            if hasIP==1 or hasIMSI==1 or hasTEL==1 then  
                --ngx.say("This is 'Black List' request")  
                ngx.exit(ngx.HTTP_FORBIDDEN)  
              end  
            else  
                --ngx.say("This is 'POST' request")  
                ngx.exit(ngx.HTTP_FORBIDDEN)  
              end  
             end  
      
  
---|---  
    * step2:修改nginx.conf
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
        location / {  
    root html;  
    index index.html index.htm;  
    access_by_lua_file /usr/local/nginx/conf/lua/ipblacklist.lua;  
    proxy_pass http://127.0.0.1:8080;  
       client_max_body_size 1m;  
    }  
      
  
---|---  
    * step3:添加黑名单规则数据
    
        1  
    2  
    3  
    

|

    
        redis-cli sadd black.ip '192.160.10.10'  
    redis-cli sadd black.imsi '460123456789'  
    redis-cli sadd black.tel '15888888888'  
      
  
---|---  
    * step4:验证结果
    
        1  
    

|

    
        curl -d "imsi=460123456789&tel=15800000000" "http://yourdomain/index.php"  
      
  
---|---