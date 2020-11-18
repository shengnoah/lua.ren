---
layout: post
title: ngx_lua灵活定制fastcgi_cache缓存key 
tags: [lua文章]
categories: [lua文章]
---
## Web开发常见的几种缓存

>   * 常用缓存（memcached和redis）
>   * Nginx的缓存（标准模块缓存: proxy_cache和fastcgi_cache / 第三方模块做缓存: ngx_lua）
>   * CDN缓存
>   * 浏览器缓存（Cache-Control和LocalStorage）
>

## proxy_cache和fastcgi_cache

proxy_cache和fastcgi_cache都为Nginx的内置缓存，proxy_cache主要用于反向代理时，对后端内容源服务器进行缓存，fastcgi_cache主要用于对FastCGI的动态程序进行缓存。  
两者相关配置类似，以下为fastcgi_cache举例

### 原理

针对fastcgi（如：php-
fpm）返回的内容缓存为静态文件(文件名是用Md5算法对Key进行哈希后所得，而Key可使用fastcgi_cache的相关指令来进行控制)  
，在用户浏览时，无需重复请求后端fastcgi，而直接返回缓存的内容，减少了后端的语言解析以及数据库连接的消耗。

### 数据流程图

![](https://sohow.cc//images/http_process.png)

### 指令注释

#### nginx的http作用域:

fastcgi_cache_path /home/wwwroot/yii.me/runtime/logs levels=1:2
keys_zone=keys_zone=zone:512m:1m inactive=1d  
max_size=1g;
#指定一个路径，目录结构等级，关键字区域存储时间和非活动删除时间。以及最大占用空间（keys_zone主要缓存key和文件元信息，不会缓存页面）

#### nginx的location作用域:

fastcgi_cache zone; #表示开启FastCGI缓存并为其指定一个名称:zone  
fastcgi_cache_valid 1m; #设置缓存时间1分钟  
fastcgi_cache_min_uses 1; #设置链接请求1次就被缓存  
fastcgi_cache_use_stale error timeout invalid_header http_500;
#定义哪些情况下用过期缓存（如果对实效要求不高建议加updating，关闭fastcgi_cache_lock，可提高性能）  
fastcgi_cache_methods GET POST; #缓存GET和POST请求  
fastcgi_cache_key “$cache_path$containerid$containerpage”;
#缓存key=页面+containerid+分页页码  
fastcgi_ignore_headers Cache-Control Expires Set-Cookie; #包含这些header的响应不缓存  
fastcgi_cache_lock on; #同时有请求处理的时候只有一个请求允许访问后端服务器，其余请求等待缓存结果或等待超时再进行响应  
fastcgi_cache_lock_timeout 5s; #等待超时时间5秒，超时则穿透，且不缓存穿透结果  
fastcgi_cache_bypass $skip_cache; #非0不从cache中取  
fastcgi_no_cache $skip_cache; #非0不保存到cache

### 性能提升

以下测试使用的vps机器  
ab -c10 -n50000 <http://127.0.0.1:8080/echo.php>

  * fastcgi_cache off Requests per second: 2441.56 [#/sec] (mean)
  * fastcgi_cache zone Requests per second: 3662.79 [#/sec] (mean)

## ngx_lua模块

### 参考资料

  * [OpenResty 最佳实践-子查询](http://wiki.jikexueyuan.com/project/openresty/openresty/sub_request.html)
  * [OpenResty 最佳实践-缓存](http://wiki.jikexueyuan.com/project/openresty/ngx_lua/cache.html)

### 在ngx_lua模块中使用共享内存

  * 定义一个共享内存对象  
语法：lua_shared_dict  
该命令主要是定义一块名为name的共享内存空间，内存大小为size。通过该命令定义的共享内存对象对于Nginx中所有worker进程都是可见的，当Nginx通过reload命令重启时，共享内存字典项会从新获取它的内容，而当Nginx  
退出时，字典项的值将会丢失。  
例子：

    
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

|

    
        location /testngxlua {
    
            default_type application/json;
    
            charset utf8;
    
            access_by_lua '
    
                function get_from_cache(key)
    
                    local cache_ngx = ngx.shared.my_cache
    
                    local value = cache_ngx:get(key)
    
                    return value
    
                end
    
                function set_to_cache(key, value, exptime)
    
                    if not exptime then
    
                        exptime = 0
    
                    end
    
                    local cache_ngx = ngx.shared.my_cache
    
                    local succ, err, forcible = cache_ngx:set(key, value, exptime)
    
                    return succ
    
                end
    
                ngx.req.read_body()
    
                local c = ngx.req.get_uri_args()["c"] or ""
    
                local str = get_from_cache(c)
    
                if (str ~= nil) then
    
                    ngx.print("ngx_cache: "..str)
    
                else
    
                    local options = {}
    
                    options["method"] = ngx.var.request_method == "GET" and ngx.HTTP_GET or ngx.HTTP_POST
    
                    options["body"] = ngx.var.request_body
    
                    options["args"] = ngx.var.args
    
                    local res = ngx.location.capture("/index.php"..ngx.var.uri, options)
    
                    if res.status == 200 then
    
                        set_to_cache(c, res.body, 300)
    
                        ngx.print(res.body)
    
                    else
    
                        ngx.say(res.status)
    
                    end
    
                end
    
            ';
    
        }  
  
---|---