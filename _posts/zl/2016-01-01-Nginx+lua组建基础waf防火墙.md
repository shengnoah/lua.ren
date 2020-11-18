---
layout: post
title: Nginx+lua组建基础waf防火墙 
tags: [lua文章]
categories: [lua文章]
---
## 一、nginx+Lua环境部署

### 1、系统基础信息

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
      
    192.168.83.129  
    # cat /etc/redhat-release  
    CentOS release 6.5 (Final)  
    # uname -r  
    2.6.32-431.el6.x86_64  
      
  
---|---  
  
### 2、安装基础库

    
    
    1  
    2  
    

|

    
    
    yum -y install gcc gcc-c++  
    yum -y install openssl openssl-devel  
      
  
---|---  
  
### 3、创建Nginx运行的普通用户

    
    
    1  
    

|

    
    
    useradd -s /sbin/nologin -M www  
      
  
---|---  
  
### 4、下载需要的程序并安装

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    

|

    
    
    cd /usr/local/src/  
    wget http://nginx.org/download/nginx-1.9.4.tar.gz  
    wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz  
    wget -c http://luajit.org/download/LuaJIT-2.0.4.tar.gz  
    wget https://github.com/simpl/ngx_devel_kit/archive/v0.2.19.tar.gz  
    wget https://github.com/openresty/lua-nginx-module/archive/v0.9.16.tar.gz  
    tar zxvf ngx_devel_kit-0.2.19.tar.gz  
    tar zxvf lua-nginx-module-0.9.16.tar.gz  
    tar zxvf pcre-8.38.tar.gz  
      
  
---|---  
  
### 5、安装LuaJIT Luajit是Lua即时编译器

    
    
    1  
    2  
    3  
    

|

    
    
    tar zxvf LuaJIT-2.0.4.tar.gz  
    cd /usr/local/src/LuaJIT-2.0.4  
    make && make install  
      
  
---|---  
  
### 6、安装nginx并加载模块

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    cd nginx-1.9.4/  
    export LUAJIT_LIB=/usr/local/lib  
    export LUAJIT_INC=/usr/local/include/luajit-2.0/  
    ./configure --prefix=/usr/local/nginx --user=www --group=www     --with-http_ssl_module --with-http_stub_status_module --with-file-aio --with-http_dav_module --add-module=../ngx_devel_kit-0.2.19/ --add-module=../lua-nginx-module-0.9.16/ --with-pcre=/usr/local/src/pcre-8.38/  
    make -j2 && make install  
    ln -s /usr/local/lib/libluajit-5.1.so.2 /lib64/libluajit-5.1.so.2  
      
  
---|---  
  
安装完毕后，下面可以测试安装了，修改nginx.conf 增加第一个配置  

    
    
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
    

|

    
    
       server {  
           listen       80;  
           server_name  localhost;  
          
           #charset koi8-r;  
          
           #access_log  logs/host.access.log  main;  
          
            location /hello {  
                   default_type 'text/plain';  
                   content_by_lua 'ngx.say("hello,lua")';  
           }  
             
           error_page   500 502 503 504  /50x.html;  
           location = /50x.html {  
           root   html;  
       	}  
    }  
      
  
---|---  
  
启动nginx  

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    /usr/local/nginx/sbin/nginx -t  
        nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok  
        nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful  
    /usr/local/nginx/sbin/nginx  
      
  
---|---  
  
### 7、测试看lua环境是否正常

![Alt text](https://francisblogstatic.oss-cn-
shanghai.aliyuncs.com/images/20181210013.png)

## 二、openresty实现WAF功能

### 1、系统基础信息

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
      
    192.168.83.129  
    # cat /etc/redhat-release  
    CentOS release 6.5 (Final)  
    # uname -r  
    2.6.32-431.el6.x86_64  
      
  
---|---  
  
### 2、安装基础依赖包

    
    
    1  
    

|

    
    
    yum install -y readline-devel pcre-devel openssl-devel  
      
  
---|---  
  
### 3、下载并编译安装openresty

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    cd /usr/local/src/  
    wget https://openresty.org/download/ngx_openresty-1.9.3.2.tar.gz  
    tar zxvf ngx_openresty-1.9.3.2.tar.gz  
    cd ngx_openresty-1.9.3.2  
    ./configure --prefix=/usr/local/openresty-1.9.3.2 --with-luajit --with-http_stub_status_module --with-pcre --with-pcre-jit  
    gmake && gmake install  
    ln -s /usr/local/openresty-1.9.3.2/ /usr/local/openresty  
      
  
---|---  
  
### 4、测试openresty安装

    
    
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

    
    
    # vim /usr/local/openresty/nginx/conf/nginx.conf  
        server {  
            location /hello {  
                    default_type text/html;  
                    content_by_lua_block {  
                        ngx.say("HelloWorld")  
                    }  
                }  
        }  
    # /usr/local/openresty/nginx/sbin/nginx -t  
        nginx: the configuration file /usr/local/openresty-1.9.3.2/nginx/conf/nginx.conf syntax is ok  
        nginx: configuration file /usr/local/openresty-1.9.3.2/nginx/conf/nginx.conf test is successful  
    # /usr/local/openresty/nginx/sbin/nginx  
      
  
---|---  
  
### 5、WAF部署：在github上克隆下代码

    
    
    1  
    2  
    3  
    

|

    
    
    yum -y install git  
    cd /usr/local/openresty/nginx/conf/  
    git clone https://github.com/unixhot/waf.git  
      
  
---|---  
  
### 6、修改Nginx的配置文件，加入（http字段）以下配置。注意路径，同时WAF日志默认存放在/tmp/日期_waf.log

    
    
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

    
    
    # vim /usr/local/openresty/nginx/conf/nginx.conf  
        http {  
            include       mime.types;  
            default_type  application/octet-stream;  
        #WAF  
            lua_shared_dict limit 50m;  
            lua_package_path "/usr/local/openresty/nginx/conf/waf/?.lua";  
            init_by_lua_file "/usr/local/openresty/nginx/conf/waf/init.lua";  
            access_by_lua_file "/usr/local/openresty/nginx/conf/waf/access.lua";  
    # /usr/local/openresty/nginx/sbin/nginx -t  
        nginx: the configuration file /usr/local/openresty-1.9.3.2/nginx/conf/nginx.conf syntax is ok  
        nginx: configuration file /usr/local/openresty-1.9.3.2/nginx/conf/nginx.conf test is successful  
    # /usr/local/openresty/nginx/sbin/nginx -s reload  
      
  
---|---  
  
### 7、根据日志记录位置，创建日志目录

    
    
    1  
    2  
    

|

    
    
    # mkdir /tmp/waf_logs  
    # chown www.www /tmp/waf_logs  
      
  
---|---  
  
### 8、配置信息与注释

    
    
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

    
    
    # cat /usr/local/openresty/nginx/conf/waf/config.lua  
    --WAF config file,enable = "on",disable = "off"   
     --waf status      
     config_waf_enable = "on"   #是否开启配置  
     --log dir   
     config_log_dir = "/tmp/waf_logs"    #日志记录地址  
     --rule setting   
     config_rule_dir = "/usr/local/nginx/conf/waf/rule-config"        #匹配规则缩放地址  
     --enable/disable white url   
     config_white_url_check = "on"  #是否开启url检测  
     --enable/disable white ip   
     config_white_ip_check = "on"   #是否开启IP白名单检测  
     --enable/disable block ip   
     config_black_ip_check = "on"   #是否开启ip黑名单检测  
     --enable/disable url filtering   
     config_url_check = "on"      #是否开启url过滤  
     --enalbe/disable url args filtering   
     config_url_args_check = "on"   #是否开启参数检测  
     --enable/disable user agent filtering   
     config_user_agent_check = "on"  #是否开启ua检测  
     --enable/disable cookie deny filtering   
     config_cookie_check = "on"    #是否开启cookie检测  
     --enable/disable cc filtering   
     config_cc_check = "on"   #是否开启防cc攻击  
     --cc rate the xxx of xxx seconds   
     config_cc_rate = "10/60"   #允许一个ip60秒内只能访问10此  
     --enable/disable post filtering   
     config_post_check = "on"   #是否开启post检测  
     --config waf output redirect/html   
     config_waf_output = "html"  #action一个html页面，也可以选择跳转  
     --if config_waf_output ,setting url   
     config_waf_redirect_url = "http://www.baidu.com"   
     config_output_html=[[  #下面是html的内容  
     	<html>   
     <head>   
     <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />   
     <meta http-equiv="Content-Language" content="zh-cn" />   
     <title>网站防火墙</title>   
     </head>   
     <body>   
     <h1 align="center"> # 您的行为已违反本网站相关规定，注意操作规范。   
     </body>   
     </html>   
     ]]  
      
  
---|---  
  
## 三、启用waf并做测试

### 1、模拟sql注入即url攻击

![Alt text](https://francisblogstatic.oss-cn-
shanghai.aliyuncs.com/images/20181210014.png)  
日志显示如下,记录了UA，匹配规则，URL，客户端类型，攻击的类型，请求的数据  

    
    
    1  
    2  
    

|

    
    
    # tail -f /tmp/2018-07-30_waf.log  
    {"user_agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36","rule_tag":"\.(bak|inc|old|mdb|sql|backup|java|class|tgz|gz|tar|zip)$","req_url":"/eastmonet.sql","client_ip":"192.168.83.1","local_time":"2018-07-30 10:46:52","attack_method":"Deny_URL","req_data":"-","server_name":"localhost"}  
      
  
---|---  
  
### 2、使用ab压力测试工具模拟防CC攻击

    
    
    1  
    2  
    

|

    
    
      
    192.168.83.131  
      
  
---|---  
  
![Alt text](https://francisblogstatic.oss-cn-
shanghai.aliyuncs.com/images/20181210015.png)  
将对方IP放入黑名单  

    
    
    1  
    2  
    

|

    
    
    # echo 192.168.83.131 >> /usr/local/openresty/nginx/conf/waf/rule-config/blackip.rule  
    # /usr/local/openresty/nginx/sbin/nginx -s reload  
      
  
---|---  
  
再拿192.168.83.131访问的时候就提示403了  
![Alt text](https://francisblogstatic.oss-cn-
shanghai.aliyuncs.com/images/20181210016.png)  
将对方IP放入白名单  

    
    
    1  
    2  
    

|

    
    
    [root@tiejiang-src1 ~]# echo 192.168.83.131 >> /usr/local/openresty/nginx/conf/waf/rule-config/whiteip.rule  
    [root@tiejiang-src1 ~]# /usr/local/openresty/nginx/sbin/nginx -s reload  
      
  
---|---  
  
此时将不对此ip进行任何防护措施，所以sql注入时应该返回404  
![Alt text](https://francisblogstatic.oss-cn-
shanghai.aliyuncs.com/images/20181210017.png)  
目录：

  * waf目录：/usr/local/openresty/nginx/conf/waf
  * 配置文件：/usr/local/openresty/nginx/conf/waf/config.lua
  * Waf的ip黑名单：/usr/local/openresty/nginx/conf/waf/rule-config/blackip.rule
  * Waf的ip白名单：/usr/local/openresty/nginx/conf/waf/rule-config/whiteip.rule