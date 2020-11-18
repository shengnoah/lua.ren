---
layout: post
title: lua+kafka+nginx采集nginx日志 
tags: [lua文章]
categories: [lua文章]
---
## [](https://jsonlisky.github.io/#%E8%83%8C%E6%99%AF "背景")背景

  * 解决nginx的访问日志过大文档占用磁盘，文档分割归档、日志实时性的弊端。

## [](https://jsonlisky.github.io/#%E6%8A%80%E6%9C%AF%E6%96%B9%E6%A1%88
"技术方案")技术方案

  * openresty(lua) + kafka组合实现一个日志采集的轻量级架构
  * 大致架构如下：
    * ![架构图](https://jsonlisky.github.io/2019/01/20/lua+kafka+nginx%E9%87%87%E9%9B%86nginx%E6%97%A5%E5%BF%97/architecture.png)

说明:

  * 线上请求打向nginx后，使用openresty(lua)完成日志整理:如统一日志格式,获取关键的请求参数组装成写入kafka的message。

##
[](https://jsonlisky.github.io/#%E4%BD%BF%E7%94%A8%E7%9A%84%E7%BB%84%E4%BB%B6%E5%92%8C%E6%8A%80%E6%9C%AF
"使用的组件和技术")使用的组件和技术

  * docker:容器化技术
  * openresty:将Nginx转变为完整的可编写脚本的Web平台
  * lua_resty_kafka:基于cosocket API的Openresty的Lua kafka客户端驱动进程
  * kafka：分布式消息队列

## [](https://jsonlisky.github.io/#%E5%AE%89%E8%A3%85%E6%AD%A5%E9%AA%A4
"安装步骤")安装步骤

  * 项目结构:

    * ![项目结构图](https://jsonlisky.github.io/2019/01/20/lua+kafka+nginx%E9%87%87%E9%9B%86nginx%E6%97%A5%E5%BF%97/project.png)
  * 准备nginx.conf
    
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
    

|

    
        worker_processes auto;  
    error_log logs/error.log;  
    pid /run/nginx.pid;  
    events {  
        worker_connections 1024;  
    }  
      
    http {  
        log_format  main  '$time_iso8601t$requestt$remote_addrt$http_user_agentt$http_referert'  
                        '$request_body';  
      
        access_log off;  
        sendfile            on;  
        tcp_nopush          on;  
        tcp_nodelay         on;  
        keepalive_timeout   65;  
        types_hash_max_size 2048;  
        default_type        application/octet-stream;  
        lua_package_path "/opt/openresty/lualib/kafka/?.lua;;";      
        server {  
            listen       80 default_server;  
            server_name  _;  
      
            root         /opt/openresty/nginx/html;  
            lua_need_request_body on;  
      
            location /test {  
                expires -1;  
                content_by_lua_file /opt/openresty/nginx/writeToKafka.lua;  
            }  
      
            error_page 404 /404.html;  
                location = /40x.html {  
            }  
      
            error_page 500 502 503 504 /50x.html;  
                location = /50x.html {  
            }  
        }  
    }  
      
  
---|---  
  * 编写lua脚本处理日志

    * 注意:将`borker_list`和`KAFKA_TOPIC_KEY`替换为你自己的kafka地址和主题。
        
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
        

|

        
                  
        local cjson = require "cjson"    
        local producer = require "resty.kafka.producer"    
          
        local broker_list =  {{ host = "xx.xx.xx.xx", port = 9092 },}    
          
        local message = ""  
        local separator = "	"  
        local path = (nil == ngx.var.uri) and "-" or ngx.var.uri  
        local query_string = (nil == ngx.var.query_string) and "-" or ngx.var.query_string  
        local client_ip = ngx.var.remote_addr    
        local referer = (nil == ngx.var.http_referer) and "-" or ngx.var.http_referer  
        local user_agent = ngx.var.http_user_agent    
        local receiveTime = ngx.var.time_iso8601  
        local request_method = ngx.var.request_method  
        local cookie = (nil == ngx.var.http_cookie) and "-" or ngx.var.http_cookie    
          
        message = receiveTime..separator..request_method..separator..path..separator..query_string..separator..client_ip..separator..user_agent..separator..referer..separator..cookie  
          
        -- 新增写入kafka错误日志记录  
        local error_handle = function (topic, partition_id, queue, index, err, retryalbe)  
            ngx.log(ngx.ERR, "origin send message:", message)  
        end	  
        -- 定义kafka异步生产者    
        local bp = producer:new(broker_list, { producer_type = "async", error_handle = error_handle })    
        -- 发送日志消息,send第二个参数key,用于kafka路由控制:    
        -- key为nill(空)时，一段时间向同一partition写入数据    
        -- 指定key，按照key的hash写入到对应的partition    
        bp:send("${KAFKA_TOPIC_KEY}", nil, message)  
          
  
---|---  
  * 编写镜像文档Dockerfile
    
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
    

|

    
        MAINTAINER 阿贤 <jsonlisky@gmail.com>  
    RUN mkdir /opt/nginx;  
    ADD ./master.zip /opt/nginx/  
    RUN yum install -y   
      readline-devel   
      pcre-devel   
      openssl-devel   
      gcc   
      zip   
      unzip   
      wget   
      perl   
      make;  
    RUN  cd /opt/nginx && wget https://openresty.org/download/openresty-1.9.7.4.tar.gz;  
    RUN  cd /opt/nginx && tar -xzf /opt/nginx/openresty-1.9.7.4.tar.gz;  
    RUN  cd /opt/nginx/openresty-1.9.7.4 && ./configure --prefix=/opt/openresty --with-luajit --without-http_redis2_module --with-http_iconv_module && gmake && gmake install;  
    RUN  unzip /opt/nginx/master.zip -d /opt/nginx/;  
    RUN  mkdir /opt/openresty/lualib/kafka;  
    RUN  cp -rf /opt/nginx/lua-resty-kafka-master/lib/resty /opt/openresty/lualib/kafka/;  
    RUN echo "install  complete!"  
    ADD ./nginx.conf /opt/openresty/nginx/conf/  
    ADD ./writeToKafka.lua  /opt/openresty/nginx/  
    RUN ln -sf /dev/stdout /opt/openresty/nginx/logs/access.log   
    	&& ln -sf /dev/stderr /opt/openresty/nginx/logs/error.log  
    EXPOSE 80  
    CMD ["/opt/openresty/nginx/sbin/nginx", "-g", "daemon off;"]  
      
  
---|---  
  * 进入到Dockerfile目录执行构建镜像命令
    
        1  
    

|

    
        sudo docker build -t openresty-nginx .  
      
  
---|---  
  * 运行容器(对外暴露8080端口)
    
        1  
    

|

    
        docker run -d -p 8080:80 openresty-nginx  
      
  
---|---  

## [](https://jsonlisky.github.io/#%E6%B5%8B%E8%AF%95%E6%A0%A1%E9%AA%8C
"测试校验")测试校验

  * 简单的curl查看
    
        1  
    

|

    
        curl 127.0.0.1:8080  
      
  
---|---  
  * 使用kafka消费者命令消费查看
    
        1  
    

|

    
        bin/kafka-console-consumer.sh --bootstrap-server --.--.--.--:9092 --topic topicName  --from-beginning  
      
  
---|---