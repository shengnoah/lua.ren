---
layout: post
title: nginx+lua+kafka实现日志统一收集汇总 
tags: [lua文章]
categories: [topic]
---
### 一:场景描述

对于线上大流量服务或者需要上报日志的nginx服务，每天会产生大量的日志，这些日志非常有价值。可用于计数上报、用户行为分析、接口质量、性能监控等需求。但传统nginx记录日志的方式数据会散落在各自nginx上，而且大流量日志本身对磁盘也是一种冲击。

我们需要把这部分nginx日志统一收集汇总起来,收集过程和结果需要满足如下需求:

  * 支持不同业务获取数据,如监控业务，数据分析统计业务，推荐业务等。
  * 数据实时性
  * 高性能保证

### 二:技术方案

得益于openresty和kafka的高性能，我们可以非常轻量高效的实现当前需求，架构如下:

![](https://img.dazhuanlan.com/2019/11/26/5ddd23983b88b.png)

方案描述:

  * 1:线上请求打向nginx后，使用lua完成日志整理:如统一日志格式，过滤无效请求，分组等。
  * 2:根据不同业务的nginx日志,划分不同的topic。
  * 3:lua实现producter异步发送到kafka集群。
  * 4:对不同日志感兴趣的业务组实时消费获取日志数据。

### 三:相关技术

  * openresty: <http://openresty.org>
  * kafka: <http://kafka.apache.org>
  * lua-resty-kafka: <https://github.com/doujiang24/lua-resty-kafka>

### 四:安装配置

为了简单直接，我们采用单机形式配置部署，集群情况类似。

#### 1.准备openresty依赖:

    
    
    1  
    

|

    
    
    apt-get install libreadline-dev libncurses5-dev libpcre3-dev libssl-dev perl make build-essential  
      
  
---|---  
  
或者:

    
    
    1  
    

|

    
    
    yum install readline-devel pcre-devel openssl-devel gcc  
      
  
---|---  
  
#### 2\. 安装编译openresty:

##### (1):安装openresty:

    
    
    1  
    2  
    3  
    

|

    
    
    cd /opt/nginx/ # 安装文件所在目录    
    wget https://openresty.org/download/openresty-1.9.7.4.tar.gz    
    tar -xzf openresty-1.9.7.4.tar.gz /opt/nginx/  
      
  
---|---  
  
配置:  
指定目录为/opt/openresty,默认在/usr/local。  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    ./configure --prefix=/opt/openresty     
                --with-luajit     
                --without-http_redis2_module     
                --with-http_iconv_module    
    make    
    make install  
      
  
---|---  
  
#### 3.安装lua-resty-kafka

下载lua-resty-kafka:  

    
    
    1  
    2  
    

|

    
    
    wget https://github.com/doujiang24/lua-resty-kafka/archive/master.zip    
    unzip lua-resty-kafka-master.zip -d /opt/nginx/  
      
  
---|---  
  
拷贝lua-resty-kafka到openresty  

    
    
    1  
    2  
    

|

    
    
    mkdir /opt/openresty/lualib/kafka    
    cp -rf /opt/nginx/lua-resty-kafka-master/lib/resty /opt/openresty/lualib/kafka/  
      
  
---|---  
  
#### 4:安装单机kafka

    
    
    1  
    2  
    3  
    

|

    
    
    cd /opt/nginx/    
    wget http://apache.fayea.com/kafka/0.9.0.1/kafka_2.10-0.9.0.1.tgz    
    tar xvf kafka_2.10-0.9.0.1.tgz  
      
  
---|---  
  
开启单机zookeeper  

    
    
    1  
    

|

    
    
    nohup sh bin/zookeeper-server-start.sh config/zookeeper.properties > ./zk.log 2>&1 &  
      
  
---|---  
  
绑定broker ip,必须绑定  
在config/servier.properties下修改host.name  

    
    
    1  
    

|

    
    
    #host.name={your_server_ip}  
      
  
---|---  
  
启动kafka服务  

    
    
    1  
    

|

    
    
    nohup sh bin/kafka-server-start.sh config/server.properties > ./server.log 2>&1 &  
      
  
---|---  
  
创建测试topic

    
    
    1  
    

|

    
    
    sh bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic test1 --partitions 1 --replication-factor 1  
      
  
---|---  
  
### 五:配置运行

开发编辑/opt/openresty/nginx/conf/nginx.conf 实现kafka记录nginx日志功能，源码如下:

    
    
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
    

|

    
    
    worker_processes  12;    
         
    events {    
        use epoll;    
        worker_connections  65535;    
    }    
         
    http {    
        include       mime.types;    
        default_type  application/octet-stream;    
        sendfile        on;    
        keepalive_timeout  0;    
        gzip on;    
        gzip_min_length  1k;    
        gzip_buffers     4 8k;    
        gzip_http_version 1.1;    
        gzip_types       text/plain application/x-javascript text/css application/xml application/X-JSON;    
        charset UTF-8;    
        # 配置后端代理服务    
        upstream rc{    
            server 10.10.*.15:8080 weight=5 max_fails=3;    
            server 10.10.*.16:8080 weight=5 max_fails=3;    
            server 10.16.*.54:8080 weight=5 max_fails=3;    
            server 10.16.*.55:8080 weight=5 max_fails=3;    
            server 10.10.*.113:8080 weight=5 max_fails=3;    
            server 10.10.*.137:8080 weight=6 max_fails=3;    
            server 10.10.*.138:8080 weight=6 max_fails=3;    
            server 10.10.*.33:8080 weight=4 max_fails=3;    
            # 最大长连数    
            keepalive 32;    
        }    
        # 配置lua依赖库地址    
        lua_package_path "/opt/openresty/lualib/kafka/?.lua;;";    
         
        server {    
            listen       80;    
            server_name  localhost;    
            location /favicon.ico {    
                root   html;    
                    index  index.html index.htm;    
            }    
            location / {    
                proxy_connect_timeout 8;    
                proxy_send_timeout 8;    
                proxy_read_timeout 8;    
                proxy_buffer_size 4k;    
                proxy_buffers 512 8k;    
                proxy_busy_buffers_size 8k;    
                proxy_temp_file_write_size 64k;    
                proxy_next_upstream http_500 http_502  http_503 http_504  error timeout invalid_header;    
                root   html;    
                index  index.html index.htm;    
                proxy_pass http://rc;    
                proxy_http_version 1.1;    
                proxy_set_header Connection "";    
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;    
                # 使用log_by_lua 包含lua代码,因为log_by_lua指令运行在请求最后且不影响proxy_pass机制    
                log_by_lua '    
                    -- 引入lua所有api    
                    local cjson = require "cjson"    
                    local producer = require "resty.kafka.producer"    
                    -- 定义kafka broker地址，ip需要和kafka的host.name配置一致    
                    local broker_list = {    
                        { host = "10.10.78.52", port = 9092 },    
                    }    
                    -- 定义json便于日志数据整理收集    
                    local log_json = {}    
                    log_json["uri"]=ngx.var.uri    
                    log_json["args"]=ngx.var.args    
                    log_json["host"]=ngx.var.host    
                    log_json["request_body"]=ngx.var.request_body    
                    log_json["remote_addr"] = ngx.var.remote_addr    
                    log_json["remote_user"] = ngx.var.remote_user    
                    log_json["time_local"] = ngx.var.time_local    
                    log_json["status"] = ngx.var.status    
                    log_json["body_bytes_sent"] = ngx.var.body_bytes_sent    
                    log_json["http_referer"] = ngx.var.http_referer    
                    log_json["http_user_agent"] = ngx.var.http_user_agent    
                    log_json["http_x_forwarded_for"] = ngx.var.http_x_forwarded_for    
                    log_json["upstream_response_time"] = ngx.var.upstream_response_time    
                    log_json["request_time"] = ngx.var.request_time    
                    -- 转换json为字符串    
                    local message = cjson.encode(log_json);    
                    -- 定义kafka异步生产者    
                    local bp = producer:new(broker_list, { producer_type = "async" })    
                    -- 发送日志消息,send第二个参数key,用于kafka路由控制:    
                    -- key为nill(空)时，一段时间向同一partition写入数据    
                    -- 指定key，按照key的hash写入到对应的partition    
                    local ok, err = bp:send("test1", nil, message)    
         
                    if not ok then    
                        ngx.log(ngx.ERR, "kafka send err:", err)    
                        return    
                    end    
                ';    
            }    
            error_page   500 502 503 504  /50x.html;    
            location = /50x.html {    
                root   html;    
            }    
        }    
    }  
      
  
---|---  
  
### 六:检测&运行

1.检测配置,只检测nginx配置是否正确，lua错误日志在nginx的error.log文件中  

    
    
    1  
    

|

    
    
    ./nginx -t /opt/openresty/nginx/conf/nginx.conf  
      
  
---|---  
  
2.启动

    
    
    1  
    

|

    
    
    ./nginx -c /opt/openresty/nginx/conf/nginx.conf  
      
  
---|---  
  
3.重启

    
    
    1  
    

|

    
    
    ./nginx -s reload  
      
  
---|---  
  
### 七:测试

#### 1:使用任意http请求发送给当前nginx，如:

    
    
    1  
    

|

    
    
    http://10.10.78.52/m/personal/AC8E3BC7-6130-447B-A9D6-DF11CB74C3EF/rc/v1?passport=83FBC7337D681E679FFBA1B913E22A0D@qq.sohu.com&page=2&size=10  
      
  
---|---  
  
#### 2:查看upstream代理是否工作正常

#### 3:查看kafka 日志对应的topic是否产生消息日志，如下:

从头消费topic数据命令  

    
    
    1  
    

|

    
    
    sh kafka-console-consumer.sh --zookeeper 10.10.78.52:2181 --topic test1 --from-beginning  
      
  
---|---  
  
效果监测:  
![](https://img.dazhuanlan.com/2019/11/26/5ddd23b06172d.png)

#### 4:ab压力测试

单nginx+upstream测试:  

    
    
    1  
    

|

    
    
    ab -n 10000 -c 100 -k http://10.10.34.15/m/personal/AC8E3BC7-6130-447B-A9D6-DF11CB74C3EF/rc/v1?passport=83FBC7337D681E679FFBA1B913E22A0D@qq.sohu.com&page=2&size=10  
      
  
---|---  
  
结果  

    
    
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
    

|

    
    
    Server Software:        nginx  
    Server Hostname:        10.10.34.15  
    Server Port:            80  
    Document Path:          /m/personal/AC8E3BC7-6130-447B-A9D6-DF11CB74C3EF/rc/v1?passport=83FBC7337D681E679FFBA1B913E22A0D@qq.sohu.com  
    Document Length:        13810 bytes  
    Concurrency Level:      100  
    Time taken for tests:   2.148996 seconds  
    Complete requests:      10000  
    Failed requests:        9982  
       (Connect: 0, Length: 9982, Exceptions: 0)  
    Write errors:           0  
    Keep-Alive requests:    0  
    Total transferred:      227090611 bytes  
    HTML transferred:       225500642 bytes  
    Requests per second:    4653.34 [#/sec] (mean)  
    Time per request:       21.490 [ms] (mean)  
    Time per request:       0.215 [ms] (mean, across all concurrent requests)  
    Transfer rate:          103196.10 [Kbytes/sec] received  
    Connection Times (ms)  
                  min  mean[+/-sd] median   max  
    Connect:        0    0   0.1      0       2  
    Processing:     5   20  23.6     16     701  
    Waiting:        4   17  20.8     13     686  
    Total:          5   20  23.6     16     701  
    Percentage of the requests served within a certain time (ms)  
      50%     16  
      66%     20  
      75%     22  
      80%     25  
      90%     33  
      95%     41  
      98%     48  
      99%     69  
    100%    701 (longest request)  
      
  
---|---  
  
单nginx+upstream+log_lua_kafka接入测试:  

    
    
    1  
    

|

    
    
    ab -n 10000 -c 100 -k http://10.10.78.52/m/personal/AC8E3BC7-6130-447B-A9D6-DF11CB74C3EF/rc/v1?passport=83FBC7337D681E679FFBA1B913E22A0D@qq.sohu.com&page=2&size=10  
      
  
---|---  
  
结果

    
    
    Server Software:        openresty/1.9.7.4
    Server Hostname:        10.10.78.52
    Server Port:            80
    Document Path:          /m/personal/AC8E3BC7-6130-447B-A9D6-DF11CB74C3EF/rc/v1?passport=83FBC7337D681E679FFBA1B913E22A0D@qq.sohu.com
    Document Length:        34396 bytes
    Concurrency Level:      100
    Time taken for tests:   2.234785 seconds
    Complete requests:      10000
    Failed requests:        9981
       (Connect: 0, Length: 9981, Exceptions: 0)
    Write errors:           0
    Keep-Alive requests:    0
    Total transferred:      229781343 bytes
    HTML transferred:       228071374 bytes
    Requests per second:    4474.70 [#/sec] (mean)
    Time per request:       22.348 [ms] (mean)
    Time per request:       0.223 [ms] (mean, across all concurrent requests)
    Transfer rate:          100410.10 [Kbytes/sec] received
    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    0   0.2      0       3
    Processing:     6   20  27.6     17    1504
    Waiting:        5   15  12.0     14     237
    Total:          6   20  27.6     17    1504
    Percentage of the requests served within a certain time (ms)
      50%     17
      66%     19
      75%     21
      80%     23
      90%     28
      95%     34
      98%     46
      99%     67
    100%   1004 (longest request)