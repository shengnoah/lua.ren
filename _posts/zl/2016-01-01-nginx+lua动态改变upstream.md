
---
layout: post
title: nginx+lua动态改变upstream 
tags: [lua文章]
categories: [lua文章]
---
在做毕设的时候需要动态改变通过nginx代理的服务器数量。背景大概是我有一个generator不断产生负载打在nginx上，还有一个monitor根据generator生成负载的qps来动态决定需要多少台服务器刚好能够承担这个qps的请求，因此需要动态修改nginx中的upstream。以前在配置nginx的时候，upstream都是写死在nginx.conf文件中的，现在要动态改变upstream，这需要在nginx运行过程中用脚本修改，那么就用Lua来实现吧。

### nginx+lua安装

之前安装过了nginx，但是没有安装lua模块，下面按照官网的步骤开始集成Lua。  
需要下载：

  * [LuaJIT](http://luajit.org/download.html)：Lua编译器(a Just-In-Time Compiler for Lua),2.0或2.1版本均可
  * [ngx_devel_kit (NDK) module](https://github.com/simpl/ngx_devel_kit/tags)
  * [lua-nginx-module](https://github.com/openresty/lua-nginx-module/tags)

安装好LuaJIT，解压NDK module和lua-nginx-module，继续。

    
    
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

|

    
    
     cd ~/work/nginx-1.12.1
    
    # 告诉nginx编译程序在哪里能找到LuaJIT:
    
     export LUAJIT_LIB=/path/to/luajit/lib
    
     export LUAJIT_INC=/path/to/luajit/include/luajit-2.0
    
    # 配置nginx模块:
    
     ./configure 
    
             --with-ld-opt="-Wl,-rpath,/path/to/luajit-or-lua/lib" 
    
             --add-module=/path/to/ngx_devel_kit 
    
             --add-module=/path/to/lua-nginx-module
    
    # 编译&安装:
    
     make & sudo make install  
  
---|---  
  
测试一下是否成功：  
在nginx.conf中加入  

    
    
    1
    
    2
    
    3
    
    4

|

    
    
    location /hello { 
    
          default_type 'text/plain'; 
    
          content_by_lua 'ngx.say("hello, lua")'; 
    
    }  
  
---|---  
  
访问127.0.0.1/hello,出现”hello, lua”即为成功。

### 动态改变upstream

这里使用了lua中的”lua_shared_dict”创建一个共享内存空间，通过该命令定义的共享内存对象对于nginx中所有worker进程都是可见的，通过./nginx
-s reload是不会改变共享内存对象的内容的，只有nginx重启才重新清空内容。

下面就是nginx.conf中的配置了。

    
    
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
    
    60
    
    61
    
    62
    
    63
    
    64
    
    65
    
    66
    
    67
    
    68
    
    69
    
    70
    
    71
    
    72
    
    73
    
    74
    
    75
    
    76
    
    77
    
    78
    
    79
    
    80
    
    81
    
    82
    
    83
    
    84
    
    85
    
    86
    
    87
    
    88
    
    89
    
    90
    
    91
    
    92
    
    93
    
    94
    
    95
    
    96
    
    97
    
    98
    
    99
    
    100
    
    101
    
    102
    
    103
    
    104
    
    105
    
    106
    
    107
    
    108
    
    109
    
    110
    
    111
    
    112
    
    113
    
    114
    
    115
    
    116
    
    117

|

    
    
    # nginx结合lua动态修改uptream
    
    #user  nobody;
    
    worker_processes  1; #工作进程个数
    
    #error_log  logs/error.log;
    
    #error_log  logs/error.log  notice;
    
    #error_log  logs/error.log  info;
    
    #pid        logs/nginx.pid;
    
    events {
    
        worker_connections  1024; #单个进程最大连接数
    
    }
    
    http {
    
        lua_shared_dict _upstream_G 1m; #定义upstream共享内存空间
    
        include       mime.types;
    
        default_type  application/octet-stream;
    
        #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    
        #                  '$status $body_bytes_sent "$http_referer" '
    
        #                  '"$http_user_agent" "$http_x_forwarded_for"';
    
        #access_log  logs/access.log  main;
    
        sendfile        on;
    
        #tcp_nopush     on;
    
        #keepalive_timeout  0;
    
        keepalive_timeout  65;
    
        #gzip  on;
    
        upstream energyaware_servers_default {
    
            server 127.0.0.1:8080;
    
            server 127.0.0.1:8090;
    
            server 127.0.0.1:8091;
    
        }
    
        upstream energyaware_servers_3 {
    
            server 127.0.0.1:8080;
    
            server 127.0.0.1:8090;
    
            server 127.0.0.1:8091;
    
        }
    
        upstream energyaware_servers_2 {
    
            server 127.0.0.1:8080;
    
            server 127.0.0.1:8090;
    
        }
    
        upstream energyaware_servers_1 {
    
            server 127.0.0.1:8080;
    
        }
    
        server {
    
            listen       80;
    
            server_name  energyaware.com localhost; #可以有多个，用空格分隔
    
            #charset koi8-r;
    
            #access_log  logs/host.access.log  main;
    
            # 切换 upstream 接口  
    
            location = /upstream_switch {
    
                content_by_lua '
    
                    local ups = ngx.req.get_uri_args()["ups"]
    
                    if ups == nil then
    
                        ngx.say("usage: /upstream_switch?ups=x.x.x.x")
    
                        return
    
                    end
    
                    local host = ngx.var.http_host
    
                    local ups_from = ngx.shared._upstream_G:get(host)
    
                    if ups_from == nil then
    
                        ups_from = "energyaware_servers_default"
    
                    end
    
                    ngx.log(ngx.ERR, host, " switch upstream from ", ups_from, " to ", ups)
    
                    ngx.shared._upstream_G:set(host, ups)
    
                ';
    
            }
    
            location / {
    
                # 动态设置当前 upstream, 未设置则使用默认 upstream  
    
                set_by_lua $cur_ups '
    
                    local host = ngx.var.http_host
    
                    local ups = ngx.shared._upstream_G:get(host)
    
                    if ups ~= nil then
    
                        ngx.log(ngx.ERR, "get [", ups, "] from ngx.shared._upstream_G")
    
                        return ups
    
                    end
    
                    ngx.shared._upstream_G:set(host, "energyaware_servers_default")
    
                    ngx.log(ngx.ERR, "use default upstream: energyaware_servers_default")
    
                    return "energyaware_servers_default"
    
                ';
    
                #proxy_next_upstream off;
    
                proxy_set_header Host $host:$server_port;
    
                #proxy_set_header    Connection  "";
    
                #proxy_redirect  http://$cur_ups http://$cur_ups:80;
    
                #proxy_redirect http://energyaware_servers_default http://energyaware_servers_default:80;
    
                proxy_pass http://$cur_ups;
    
                #proxy_pass http://energyaware_servers_default;
    
            }
    
    	location /hello { 
    
          	    default_type 'text/plain'; 
    
                content_by_lua 'ngx.say("hello, lua")'; 
    
    	}
    
       
    
        }
    
    }  
  
---|---  
  
日志打在了nginx/log/error.log中  
定义了切换upstream的接口，下面命令可以将upstream切换为energyaware_servers_2(一组服务器)，而无需./nginx -s
reload  

    
    
    1

|

    
    
    curl http://energyaware.com/upstream_switch?ups=energyaware_servers_2  
  
---|---  
  
切换完成之后，可以用下面脚本生成请求，然后看看这些请求都打到了哪台服务器上  

    
    
    1

|

    
    
    for i in $(seq 10); do curl http://energyaware.com  ;done  
  
---|---  
  
我这里energyaware_servers_2中有两台服务器，负载均衡策略就是最简单的轮询，所以请求应该是平均地被这两台服务器消费了。

当然也可以把upstream设置成单台服务器，直接写成ip:port的形式即可。比如：  

    
    
    1

|

    
    
    curl http://energyaware.com/upstream_switch?ups=127.0.0.1:8090  
  
---|---  
  
> 参考：  
> <https://github.com/openresty/lua-nginx-module#installation>  
> <http://blog.csdn.net/force_eagle/article/details/51966333>  
> <http://blog.csdn.net/toontong/article/details/49633829>

### 利用redis持久化upstream

有的时候我们希望把upstream持久化，否则nginx因为各种各样的原因需要重启的时候 **lua_shared_dict**
被清空，从而失去了我们原来的upstream；而且，当不止有一台nginx的时候，由于 **lua_shared_dict**
为单个nginx所有不被共享，造成不同nginx对应着不同的upstream，失去了nginx集群的意义。总之，我们需要持久化upstream。

实现与上面方法类似，将upstream以的形式存在redis中，key是host名，value是upstream，对外提供restful接口将value修改成我们想要的upstream即可。当然如果我们每次请求都要从redis里面查找显然是不合理的，可以利用前面提到的
**lua_shared_dict** 做缓存，先从缓存里取，缓存中没有再从redis中取，然后更新缓存。

> 参考：  
>  <https://sosedoff.com/2012/06/11/dynamic-nginx-upstreams-with-lua-and-
> redis.html>  
>  <http://blog.csdn.net/imlsz/article/details/47720235>

### 其他方法

  * 其实我最初的想法是想能不能动态地增加/删除upstream里面的server，这样做需要懂nginx内核了。。。好在发现了一个开源的模块[ngx_http_dyups_module](https://github.com/yzprofile/ngx_http_dyups_module)，淘宝大神写的。不过我还没有尝试，先记录下万一以后用到。

  * nginx官方实现了这个功能，但是在[nginx plus](https://www.nginx.com/products/nginx/load-balancing/#load-balancing-api)版本中(收费)

> “NGINX Plus offers an API that falls closely to the REST architecture and is
> unified under a single API endpoint. Use the NGINX Plus API to update
> upstream configurations and key‑value stores on the fly with zero downtime.
> “  
> –摘自官网

  * 另外还找到一篇[基于 consul + upsync 的动态upstream管理](http://www.jianshu.com/p/35b03c82f9fd)

