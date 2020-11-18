---
layout: post
title: OpenResty运行Lua示例 
tags: [lua文章]
categories: [lua文章]
---
OpenResty 是一个基于 Nginx 与 Lua 的高性能 Web 平台。

更多信息，参考 [OpenResty 官网](http://openresty.org/)。

安装过程跟 Nginx 基本相同，区别在于安装完成之后，默认安装了很多 Module。

安装完成后，执行 ./nginx -V 的结果

    
    
    nginx version: openresty/1.13.6.2  
    built by gcc 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC)   
    built with OpenSSL 1.0.2k-fips  26 Jan 2017  
    TLS SNI support enabled  
    configure arguments: --prefix=/work/admin/openresty/nginx --with-cc-opt=-O2 --add-module=../ngx_devel_kit-0.3.0 --add-module=../echo-nginx-module-0.61 --add-module=../xss-nginx-module-0.06 --add-module=../ngx_coolkit-0.2rc3 --add-module=../set-misc-nginx-module-0.32 --add-module=../form-input-nginx-module-0.12 --add-module=../encrypted-session-nginx-module-0.08 --add-module=../srcache-nginx-module-0.31 --add-module=../ngx_lua-0.10.13 --add-module=../ngx_lua_upstream-0.07 --add-module=../headers-more-nginx-module-0.33 --add-module=../array-var-nginx-module-0.05 --add-module=../memc-nginx-module-0.19 --add-module=../redis2-nginx-module-0.15 --add-module=../redis-nginx-module-0.3.7 --add-module=../rds-json-nginx-module-0.15 --add-module=../rds-csv-nginx-module-0.09 --add-module=../ngx_stream_lua-0.0.5 --with-ld-opt=-Wl,-rpath,/work/admin/openresty/luajit/lib --with-stream --with-stream_ssl_module --with-http_ssl_module  
      
  
---  
  
OpenResty 配置文件格式同 Nginx 完全兼容，区别在于可在各个处理阶段中，嵌入 Lua 代码。

下面是一个示例，在 rewrite 阶段，读取 Post 包请求体中的 JSON 字段，将请求重定向到不同的后端服务上去。

    
    
    server {  
        listen       8080;  
        server_name  localhost;  
      
          
      
        #access_log  logs/myweb.log  access;  
        access_log  off;  
      
        location / {  
      
            set $backend '';  
      
            rewrite_by_lua_block {  
      
                local json = require("cjson.safe")  
      
                ngx.req.read_body()  
      
                local body_data = ngx.req.get_body_data()  
                local body_data_json = json.decode(body_data)  
      
                if string.find(body_data_json["key"],"value") then  
                    back = "127.0.0.1:10000"  
                else  
                    back = "myweb"  
                end  
      
                ngx.var.backend = back  
      
            }  
      
            proxy_pass http://$backend;  
      
            proxy_set_header Host $host;  
            proxy_set_header X_Real_IP $remote_addr;  
            proxy_set_header X_Forwarded_For $proxy_add_x_forwarded_for;  
      
            proxy_next_upstream error timeout invalid_header http_404 http_500 http_502 http_503;  
      
        }  
      
    }  
      
  
---