---
layout: post
title: [proxy_cache&fastcgi_cache lua] 
tags: [lua文章]
categories: [lua文章]
---
[区别] 网上照搬  
proxy_cache 缓存后端服务器的内容，可以是动态或者静态的任何内容  
fastcg_cache 缓存fastcgi生成的内容，很多情况是php生成的内容  
proxy_cache 主要用于反向代理时，对后端内容原服务器进行缓存，减少了nginx与后端通信的次数，节省了传输时间和后端带宽  
fastcgi_cache 主要用于对fastcgi的动态进程进行缓存，减少了nginx和php通信的次数，减轻了php和数据库的压力  
先来proxy_cache 针对lua  
levels:缓存的级别，哈希 keys_zone:共享内存区 inactive:一个请求60m没有被请求，那么缓存管理会自动删除
(有了这个感觉不用purge了)  
proxy_cache_path /dev/shm/proxy_cache levels=1:2 keys_zone=one_cache:100m
inactive=60m;  
fastcgi_cache_path /dev/shm/fastcgi_cache_${USER}-${PRJ_KEY}-${APP_SYS}
levels=1:2 keys_zone=Action_pk_gift-fcgi-
cache_${USER}-${PRJ_KEY}-${APP_SYS}:100m inactive=60m;

#这个清理模块要写在上面，也是看别人说的，我太懒了 不想实验了  
location ~ /purge/ajax_gift_gifts_get{  
allow all;

    
    
    #deny all;
    proxy_cache_purge one_cache "$host$arg_roomid$arg_rid$http_origin";
    

}

location = /ajax_gift_gifts_get{  
add_header X-Cache-Status “$upstream_cache_status - $upstream_response_time”;

    
    
            #根据key生成相应的缓存  可以通过$arg_参数名，来访问到参数，然后可以设置根据参数不同来指定你想要的缓存条件
            proxy_cache_key "$host$arg_roomid$arg_rid$http_origin";
            #允许哪些方法可以缓存
            proxy_cache_methods GET HEAD;
            #缓存存放的缓存快
            proxy_cache  one_cache;  #这个one_cache 是上面定义proxy_cache_path 时里面定义的共享内存区
           #对于任何响应都缓存 30s
            proxy_cache_valid  any 30s;
            # 请求几次开始缓存
            proxy_cache_min_uses  1;
            #客户端主动断掉链接，Nginx会等待后端处理完（或超时），然后记录后端的返回信息
            proxy_ignore_client_abort on;
            add_header        Host           ${DOMAIN_GATE};
           # 当多个客户端请求一个不存在的缓存时，只有第一个请求被允许发送至服务器
            proxy_cache_lock on;
            # 哪些状态要缓存
            proxy_cache_use_stale error updating timeout http_500 http_503;
           #proxy_cache 和 proxy_pass  必须一起用才可以了，我是代理到另一个接口了，网上看 有人写127.0.0.1:port 我没成功，这个必须得写可以访问的通的server_name 自己到时候慢慢试吧
            proxy_pass http://${DOMAIN_GATE}/ajax_gift_gifts_getinside;
    }
    
     location  = /ajax_gift_gifts_getinside{
        limit_conn mall_one_${USER} 1;
        limit_conn_status 403;
        limit_req zone=mall_addr_${USER} burst=1 nodelay;
        limit_req_status 403;
        default_type application/json;
        content_by_lua_file ${PRJ_ROOT}/src/lua/app/gift.lua;
    }
    

来看fastchi_cache 对php 不能照搬这个 ，环境不一样，参数不一样，变量不一样  
location /pk_gift {

    
    
       #这里重新整理参数
        set $paramstr $uri?groupid=$arg__groupid;
        try_files $uri /localcache$paramstr;
    }
    

location ~ ^/localcache/(w+) {  
internal;

    
    
       #解析action，/xxx => action=xxx
       set $action 'index';
       if ( $request_uri ~ ^/(w+)[^?]* ) {
           set $action $1;
       }
    
       include        fastcgi_params;
       root           ${PRJ_ROOT}/src/apps/${APP_SYS} ;
       fastcgi_pass   $php_sock;
       fastcgi_param  SCRIPT_FILENAME  ${PRJ_ROOT}/src/apps/${APP_SYS}/index.php;
       fastcgi_param  QUERY_STRING     do=$action&$query_string;
       client_max_body_size       100m;
       fastcgi_connect_timeout 1000s;
       fastcgi_send_timeout 1000s;
       fastcgi_read_timeout 1000s;
       #看这里  
       #这个是给response 头加一个变量看看是否缓存的状态 有MISS：没命中 HIT：缓存命中 EXPIRED:缓存已经过期请求被传送到后端 UPDATING：正在更新缓存，将使用旧的应答 STATE:后端将得到过期的应答
       add_header X-Cache-CFC "$upstream_cache_status - $upstream_response_time";
      #这个也是根据参数生成相应的缓存key,到时候匹配也是按照这个key来匹配 下面$1是上面正则匹配参数的第一个
       fastcgi_cache_key "$host$1$query_string$http_origin";
      #允许缓存的方法
       fastcgi_cache_methods GET HEAD;
       #最上面fastcgi_cache_path 定义的路径里面有
       fastcgi_cache  Action_pk_gift-fcgi-cache_fangyuanmei-mall-front;
       #任何状态缓存30s
       fastcgi_cache_valid   any 30;
       #请求几次开始缓存
       fastcgi_cache_min_uses  1;
       #同proxy_cache
       fastcgi_cache_lock on;
       #同proxy_cache
       fastcgi_cache_use_stale error updating timeout   http_500 http_503;
       #同proxy_cache
       fastcgi_ignore_client_abort on;
    }
    
    哎，图片还不出来
    

![](https://YM-FANG.github.io/2018/12/25/proxy-cache-fastcgi-
cache/%60X1@%5D2%~9G_70H\(24Z%DV29.png)  
![](https://YM-FANG.github.io/2018/12/25/proxy-cache-fastcgi-
cache/2018/12/25/proxy-cache-fastcgi-cache/%60X1@%5D2%~9G_70H\(24Z%DV29.png)  
![fuck](https://YM-FANG.github.io/2018/12/25/proxy-cache-fastcgi-
cache/%60X1@%5D2%~9G_70H\(24Z%DV29.png)