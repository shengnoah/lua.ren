---
layout: post
title: nginx+nginx_lua实现WAF防护功能 
tags: [lua文章]
categories: [topic]
---
> Do one thing at a time, and do well.

## nginx_lua

nginx_lua模块是nginx的第三方模块，它可以将lua语言嵌入到nginx配置中，从而极大的扩展了nginx的能力，nginx以高并发而知名，而lua作为嵌入式语言轻便，两者的结合可以做到在nginx层就实现编程,而这里我们加入waf的lua过滤编程来实现waf。  

## 安装

需要的程序包：

  * nginx
  * nginx_devel_kit（拓展nginx服务器核心功能的模块）
  * lua-nginx-module（nginx_lua模块）
  * nginx_lua_waf（waf策略 web应用防火墙）
  * LuaJIT（c实现的lua解释器）

### LuaJIT

下载网站：

    
    
    1
    
    2
    
    3
    
    4

|

    
    
    ~]
    
    ~]# tar xf LuaJIT-2.0.5.tar.gz
    
    ~]# cd LuaJIT-2.0.5
    
    ~]# make -j 2 && make install  
  
---|---  
  
`lib和include是直接放在/usr/local/lib和/usr/local/include`

设置环境变量(nginx编译时需要)  

    
    
    1
    
    2
    
    3
    
    4
    
    5

|

    
    
    ~]# ~]# vim /etc/profile.d/LuaJIT.conf
    
    export LUAJIT_LIB=/usr/local/lib
    
    export LUAJIT_INC=/usr/local/include/luajit-2.0
    
    export LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH
    
    ~]# . /etc/profile.d/LuaJIT.conf  
  
---|---  
  
### nginx_devel_kit

第三方模块，我们可以到nginx wiki是查找：www.nginx.com/resources/wiki/modules/index.html

    
    
    1

|

    
    
    ~]# git clone https://github.com/simpl/ngx_devel_kit  
  
---|---  
  
### lua-nginx-module

    
    
    1

|

    
    
    ~]# git clone https://github.com/openresty/lua-nginx-module  
  
---|---  
  
### nginx

编译nginx_devel_kit和lua-nginx-module进nginx  

    
    
    1
    
    2
    
    3
    
    4
    
    5

|

    
    
    ~]# wget http://nginx.org/download/nginx-1.12.1.tar.gz
    
    ~]# tar xf nginx-1.12.1.tar.gz
    
    ~]# cd nginx-1.12.1
    
    ~]# yum install -y openssl-devel pcre-devel
    
    ~]# ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --add-module=../ngx_devel_kit --add-module=../lua-nginx-module --user=nginx --group=nginx  
  
---|---  
  
### nginx_lua_waf

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7

|

    
    
    ~]# wget -c https://github.com/loveshell/ngx_lua_waf/archive/master.zip
    
    ~]# mkdir -p /usr/local/nginx/conf/waf
    
    ~]# unzip master.zip 
    
    ~]# cd ngx_lua_waf-master
    
    ~]# cp -rf * /usr/local/nginx/conf/waf/
    
    ~]# mkdir -p /usr/local/nginx/logs/hack
    
    ~]# chown -R nginx /usr/local/nginx/logs/hack  
  
---|---  
  
#### nginx_lua_waf的用途

  * 防止sql注入，本地包含，部分溢出，fuzzing测试，xss,SSRF等web攻击
  * 防止svn/备份之类文件泄漏
  * 防止ApacheBench之类压力测试工具的攻击
  * 屏蔽常见的扫描黑客工具，扫描器
  * 屏蔽异常的网络请求
  * 屏蔽图片附件类目录php执行权限
  * 防止webshell上传

#### nginx_lua_waf的使用

nginx安装路径假设为:/usr/local/nginx/conf/  
在nginx.conf的http段添加  

    
    
    1
    
    2
    
    3
    
    4
    
    5

|

    
    
    lua_need_request_body on;
    
          lua_package_path "/usr/local/nginx/conf/waf/?.lua";
    
          lua_shared_dict limit 10m;
    
          init_by_lua_file  /usr/local/nginx/conf/waf/init.lua; 
    
          access_by_lua_file /usr/local/nginx/conf/waf/waf.lua;  
  
---|---  
  
配置config.lua里的waf规则目录(一般在waf/conf/目录下)  

    
    
    1

|

    
    
    RulePath = "/usr/local/nginx/conf/waf/wafconf/"  
  
---|---  
  
#### config.lua 配置说明

    
    
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

|

    
    
    RulePath = "/usr/local/nginx/conf/waf/wafconf/"
    
          --规则存放目录
    
          attacklog = "off"
    
          --是否开启攻击信息记录，需要配置logdir
    
          logdir = "/usr/local/nginx/logs/hack/"
    
          --log存储目录，该目录需要用户自己新建，切需要nginx用户的可写权限
    
          UrlDeny="on"
    
          --是否拦截url访问
    
          Redirect="on"
    
          --是否拦截后重定向
    
          CookieMatch = "on"
    
          --是否拦截cookie攻击
    
          postMatch = "on" 
    
          --是否拦截post攻击
    
          whiteModule = "on" 
    
          --是否开启URL白名单
    
          black_fileExt={"php","jsp"}
    
          --填写不允许上传文件后缀类型
    
          ipWhitelist={"127.0.0.1"}
    
          --ip白名单，多个ip用逗号分隔
    
          ipBlocklist={"1.0.0.1"}
    
          --ip黑名单，多个ip用逗号分隔
    
          CCDeny="on"
    
          --是否开启拦截cc攻击(需要nginx.conf的http段增加lua_shared_dict limit 10m;)
    
          CCrate = "100/60"
    
          --设置cc攻击频率，单位为秒.
    
          --默认1分钟同一个IP只能请求同一个地址100次
    
          html=[[Please go away~~]]
    
          --警告内容,可在中括号内自定义
    
          备注:不要乱动双引号，区分大小写  
  
---|---  
  
### waf.conf 自定义过滤规则

  * args里面的规则get参数进行过滤的
  * url是只在get请求url过滤的规则
  * post是只在post请求过滤的规则
  * whitelist是白名单，里面的url匹配到不做过滤
  * user-agent是对user-agent的过滤规则

`注意：默认开启了get和post过滤，需要开启cookie过滤的，编辑waf.lua取消部分--注释即可`

## WAF测试

![](https://jusene.github.io//image/98.png)

看下日志：  

    
    
    1
    
    2
    
    3

|

    
    
    hack]# tail -f localhost_2017-09-16_sec.log 
    
    10.211.55.2 [2017-09-16 14:01:40] "GET localhost/?id=select%20*%20from%20mysql;" "-"  "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:56.0) Gecko/20100101 Firefox/56.0" "select.+(from|limit)"
    
    10.211.55.2 [2017-09-16 14:05:51] "GET localhost/?id=union%20select%20*%20from%20mysql;" "-"  "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:56.0) Gecko/20100101 Firefox/56.0" "select.+(from|limit)"  
  
---|---  
  
从日志中我们可以看见我么测试的url的args请求触发了那条规则。

从config.lua中我们还可以看见cc防护，测试下：  

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6

|

    
    
    ~]# cat /usr/local/nginx/conf/waf/config.lua
    
    ...
    
    CCDeny="on"     #开启cc防护
    
    CCrate="5/60"   #降低触发阀值便于测试
    
    ...
    
    ~]# nginx -s reload  
  
---|---  
  
测试结果，当我在1分钟频繁请求超过5次，返回404错误，最后结果返回的是503错误，当我们停止访问，过一会就可以恢复访问，经过测试这个防护是针对请求ip的，证明cc防护还是可以达到一定的效果的。

![](https://jusene.github.io//image/99.png)

这是nginx+lua的一种扩展，而nginx的另一个分支openresty将nginx与lua做了很多扩展，有空一定需要好好研究下。