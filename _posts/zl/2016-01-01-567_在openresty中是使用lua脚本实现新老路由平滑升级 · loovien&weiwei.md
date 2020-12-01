---
layout: post
title: 在openresty中是使用lua脚本实现新老路由平滑升级 · loovien&weiwei 
tags: [lua文章]
categories: [topic]
---
想升级PHP框架phalcon到3.x, 但是发现升级后与老版本基本不兼容,
也就意味着代码基本要重写了。考虑到不可能一下把所有的接口切换到新的框架上去（不能短时间内全部迁移所有的接口，新的框架提供的接口需要测试时间）。想到的方案是，
一方面提供新的接口使用新的框架编写，然后网关判断，
如果是新的接口就路由到新的框架部署的服务器上。空闲的时间，慢慢的把老的接口往新的框架上迁移。一下简单的实现了一个demo案列。

**环境以及依赖软件**

  * window 10。
  * openresty/1.13.6.1 提供网关服务。
  * redis-server 提供新路由存储。
  * golang 模拟提供web服务。

这里不对[openresty](http://openresty.org), [golang](https://golang.org/)做介绍了。

**openresty配置**

编辑`nginx.conf`文件, 默认使用`phalcon1`upstream,当`rewrite_by_lua_file`执行完后, 如果是新的路由,
会重写`phalcon1`到`phalcon3`。

    
    
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
    

|

    
    
    upstream phalcon1 {  
        server 127.0.0.1:8088 fail_timeout=53 weight=4 max_fails=100;  
    }  
    upstream phalcon3 {  
        server 127.0.0.1:8089 fail_timeout=53 weight=10 max_fails=100;  
    }  
    server {  
        listen       80;  
        server_name  localhost;  
        #charset koi8-r;  
        access_log  logs/host.access.log  main;  
        error_page   500 502 503 504  /50x.html;  
        location / {  
            root   html;  
            index  index.html index.htm;  
        }  
        location /api/user {  
            default_type text/html;  
            set $upstream "phalcon1";  
            # set_by_lua_file $upstream "lualib/zq/chrbyuri.lua"; # set_by_lua_file 关闭了 resty_redis API  
            rewrite_by_lua_file "lualib/zq/chrbyuri.lua";  
            proxy_pass http://$upstream;  
        }  
        location = /50x.html {  
            root   html;  
        }  
    }  
      
  
---|---  
  
**chrbyuri.lua**

从redis中查找是否是新的路由, 新的话重写`phalcon1`到`phalcon3`, 这里有个小规则, 框架中有的路由是
`/api/user/{page}/{num}.json` 由于这些路由是动态, 不好固化存在redis中, 这列只做了简单的处理, 就是替换成
`{num}`, 比如请求路由: `/api/user/100/10/post.json` =>
`/api/user/{num}/{num}/post.json`, 对应的在redis存替换的key就好了。

    
    
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
    

|

    
    
    local newupstream = "phalcon3"  
    local redis_host = "127.0.0.1"  
    local redis_port = 6379  
    local redis = require "resty.redis"  
    local uri = ngx.var.uri  
    local r = uri:gsub("/%d+", "/{num}")  
    ngx.log(ngx.INFO, "origin document_uri:", uri)  
    ngx.log(ngx.INFO, "replace document_uri:", r)  
      
      
    local route_redis = redis:new()  
    local ok, err = route_redis:connect(redis_host, redis_port)  
    if not ok then  
        ngx.log(ngx.ERR, "connect to redis error", err)  
        return  
    end  
    -- query document_uri is exists or not  
    local newsrv, err = route_redis:get(r)  
    if newsrv == ngx.null then  
      ngx.log(ngx.INFO, string.format("route %s is not new route", r))  
      return  
    end  
      
    -- override upstream  
    ngx.var.upstream = newupstream  
      
  
---|---  
  
**golang实现的web服务**

端口8088为老的接口服务器，提供 `/api/user/user.info1` `/api/user/user.info2` 2个接口。

    
    
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
    

|

    
    
    # filename: phalcon1.go  
    package main  
      
    import (  
    	"net/http"  
    	"log"  
    )  
      
    func ()  {  
    	http.HandleFunc("/api/user/user.info1", func(rw http.ResponseWriter, r *http.Request) {  
    		rw.Write([]byte("phalcon1:8088 handle route: {api/user/user.info2}"))  
    	})  
    	http.HandleFunc("/api/user/user.info2", func(rw http.ResponseWriter, r *http.Request) {  
    		rw.Write([]byte("phalcon1:8088 handle route: {api/user/user.info2}"))  
    	})  
    	log.Fatalf("server exception: %+v n", http.ListenAndServe(":8088", nil))  
    }  
      
  
---|---  
  
端口8089为新的接口服务器，提供 `/api/user/user.info3` 1个接口。

    
    
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

    
    
    package main  
    # filename: phalcon3.go  
    import (  
    	"net/http"  
    	"log"  
    )  
      
    func ()  {  
    	http.HandleFunc("/api/user/user.info3", func(rw http.ResponseWriter, r *http.Request) {  
    		rw.Write([]byte("phalcon1:8089 handle route: {api/user/user.info2}"))  
    	})  
    	log.Fatalf("server exception: %+v n", http.ListenAndServe(":8089", nil))  
    }  
      
  
---|---  
  
**测试**

  1. 启动openresty `cd /path/to/openresty/nginx.exe`。
  2. 启动redis-server `cd /path/to/redis-server/redis-server -c /path/to/redis-server.conf`。
  3. 启动phalcon2 `go run phalcon3.go`。
  4. 启动phalcon1 `go run phalcon1.go`。

  5. 将新的路由写入到redis中 `redis-cli set /api/user/user.info3 1`。

  6. 访问 `http://localhost/api/user/user.info1` `http://localhost/api/user/user.info2` 还是代理到了phalcon1上。

  7. 访问 `http://localhost/api/user/user.info3` 就代理到了phalcon3上了。

**后续**

当有新的接口发布时候, 记得往redis写入新的路由标识即可了。