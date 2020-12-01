---
layout: post
title: install luajit in nginx 
tags: [lua文章]
categories: [topic]
---
https://github.com/openresty/lua-nginx-module

#### 安装

  * 安装 [LuaJit](http://luajit.org/download.html)
  * 下载 [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit/tags)
  * 下载 [ngx_lua](https://github.com/openresty/lua-nginx-module/tags)
  * 下载 [Nginx](http://nginx.org/en/download.html)

    
    
     wget 'http://nginx.org/download/nginx-1.9.7.tar.gz'
     tar -xzvf nginx-1.9.7.tar.gz
     cd nginx-1.9.7/
    
     # tell nginx's build system where to find LuaJIT 2.0:
     export LUAJIT_LIB=/path/to/luajit/lib
     export LUAJIT_INC=/path/to/luajit/include/luajit-2.0
    
     # tell nginx's build system where to find LuaJIT 2.1:
     export LUAJIT_LIB=/path/to/luajit/lib
     export LUAJIT_INC=/path/to/luajit/include/luajit-2.1
    
     # or tell where to find Lua if using Lua instead:
     #export LUA_LIB=/path/to/lua/lib
     #export LUA_INC=/path/to/lua/include
    
     # Here we assume Nginx is to be installed under /opt/nginx/.
     ./configure --prefix=/opt/nginx 
             --with-ld-opt="-Wl,-rpath,/path/to/luajit-or-lua/lib" 
             --add-module=/path/to/ngx_devel_kit 
             --add-module=/path/to/lua-nginx-module
    
     make -j2
     make install

#### 测试

在nginx.conf中添加下面部分内容

    
    
    location /hello {
        default_type 'text/plain';
        content_by_lua 'ngx.say("hello, world")';
    }

##### 验证结果

    
    
    [root@1184c0eeaa0a nginx-1.8.1]# curl 127.0.0.1:8080/hello
    hello, world
    

#### 示例2 操作Redis

github地址: [lua-resty-redis](https://github.com/openresty/lua-resty-redis)

下载lib/resty/redis.lua文件到pato/lib/resty/redis.lua

修改nginx.conf文件

    
    
    # you do not need the following line if you are using
    # the OpenResty bundle:
    lua_package_path "/path/to/lua-resty-redis/lib/?.lua;;";
    
    server {
        location /test {
            content_by_lua '
                local redis = require "resty.redis"
                local red = redis:new()
    
                red:set_timeout(1000) -- 1 sec
    
                -- or connect to a unix domain socket file listened
                -- by a redis server:
                --     local ok, err = red:connect("unix:/path/to/redis.sock")
    
                local ok, err = red:connect("127.0.0.1", 6379)
                if not ok then
                    ngx.say("failed to connect: ", err)
                    return
                end
    
                ok, err = red:set("dog", "an animal")
                if not ok then
                    ngx.say("failed to set dog: ", err)
                    return
                end
    
                ngx.say("set result: ", ok)
    
                local res, err = red:get("dog")
                if not res then
                    ngx.say("failed to get dog: ", err)
                    return
                end
    
                if res == ngx.null then
                    ngx.say("dog not found.")
                    return
                end
    
                ngx.say("dog: ", res)
    
                red:init_pipeline()
                red:set("cat", "Marry")
                red:set("horse", "Bob")
                red:get("cat")
                red:get("horse")
                local results, err = red:commit_pipeline()
                if not results then
                    ngx.say("failed to commit the pipelined requests: ", err)
                    return
                end
    
                for i, res in ipairs(results) do
                    if type(res) == "table" then
                        if res[1] == false then
                            ngx.say("failed to run command ", i, ": ", res[2])
                        else
                            -- process the table value
                        end
                    else
                        -- process the scalar value
                    end
                end
    
                -- put it into the connection pool of size 100,
                -- with 10 seconds max idle time
                local ok, err = red:set_keepalive(10000, 100)
                if not ok then
                    ngx.say("failed to set keepalive: ", err)
                    return
                end
    
                -- or just close the connection right away:
                -- local ok, err = red:close()
                -- if not ok then
                --     ngx.say("failed to close: ", err)
                --     return
                -- end
            ';
        }
    }