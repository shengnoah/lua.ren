---
layout: post
title: Linux安装ngx_lua_waf实现waf功能 
tags: [lua文章]
categories: [topic]
---
# 一、ngx_lua_waf用途

1、防止SQL注入，本地包含，本地溢出，fuzzing测试，XSS，SSRF等web攻击;  
2、防止SVN/备份之类文件泄漏;  
3、防止apachebench之类的压力测试工具的攻击;  
4、屏蔽常见的扫描黑客工具，扫描器;  
5、屏蔽常见的网络请求;  
6、屏蔽照片附件类目录php执行权限;  
7、防止webshell上传。

# 二、安装

## 1、首先安装所需要的依赖环境

`yum -y install gcc gcc-c++ wget git geoip-devel gd-devel pcre-deve libcurl-
devel libxml2 libxml2-devel libgd-devel openssl-devel lua-devel`

## 2、LuaJIT

下载并安装LuaJIT2.0.5，首先来到/usr/local/src（压缩包存放目录）目录下。  
`cd /usr/local/src`  
`wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz`  
`tar -zxvf LuaJIT-2.0.5.tar.gz`  
`cd LuaJIT-2.0.5`  
`make install PREFIX=/usr/local/src/luajit`  
然后创建一条软连接：  
`ln -s /usr/local/src/luajit/lib/libluajit-5.1.so.2 /lib64/libluajit-5.1.so.2`

## 3、ngx_devel_kit

下载并安装ngx_devel_kit  
`cd /usr/local/src`  
`wget https://github.com/simpl/ngx_devel_kit/archive/v0.3.0.tar.gz`  
`tar -zxvf v0.3.0.tar.gz`

## 4、lua-nginx-model

下载并安装lua-nginx-model（nginx的lua模块）  
`wget https://github.com/openresty/lua-nginx-
module/archive/v0.10.14rc3.tar.gz`  
`tar -zxvf v0.10.14rc3.tar.gz`

## 5、安装nginx

下载并安装nginx，这里我选择的是1.15.2版本的。  
`wget http://nginx.org/download/nginx-1.15.2.tar.gz`  
然后开始编译安装。  
`./configure --user=www --group=www --prefix=/data/server/nginx --error-log-
path=/data/server/nginx/error.log --http-log-
path=/data/server/nginx/access.log --with-http_ssl_module --with-
http_v2_module --with-http_realip_module --with-http_addition_module --with-
http_image_filter_module --with-http_geoip_module --with-http_sub_module
--with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-
http_gunzip_module --with-http_gzip_static_module --with-
http_random_index_module --with-http_secure_link_module --with-
http_degradation_module --with-http_slice_module --with-
http_stub_status_module --with-pcre --with-pcre-jit --with-stream --with-
stream_ssl_module --with-debug --add-module=/usr/local/src/ngx_devel_kit-0.3.0
--add-module=/usr/local/src/lua-nginx-module-0.10.14rc3 --with-ld-
opt="-Wl,-rpath,$LUAJIT_LIB" ;`  
检查没问题的话开始安装：  
`make && make install`

## 6、下载并安装waf模块

`wget https://github.com/hack-umbrella/ngx_lua_waf/archive/master.zip`  
解压并改名为waf，移动到nginx的配置目录下  
`unzip master.zip`  
`mv /usr/local/src/ngx_lua_waf-0.7.2/ waf`  
`cp -rf /usr/local/src/waf/ /data/server/nginx/conf/`  
修改waf模块的规则配置路径  
`vim /data/server/nginx/conf/waf/config.lua`  
修改配置文件为如下：  
RulePath = “/data/server/nginx/conf/waf/wafconf/“  
–规则存放目录  
attacklog = “off”  
–是否开启攻击信息记录，需要配置logdir  
logdir = “/data/server/nginx/logs/hack/“  
–log存储目录，该目录需要用户自己新建，切需要nginx用户的可写权限  
UrlDeny=”on”  
–是否拦截url访问  
Redirect=”on”  
–是否拦截后重定向  
CookieMatch = “on”  
–是否拦截cookie攻击  
postMatch = “on”  
–是否拦截post攻击  
whiteModule = “on”  
–是否开启URL白名单  
black_fileExt={“php”,”jsp”}  
–填写不允许上传文件后缀类型  
ipWhitelist={“127.0.0.1”}  
–ip白名单，多个ip用逗号分隔  
ipBlocklist={“1.0.0.1”}  
–ip黑名单，多个ip用逗号分隔  
CCDeny=”on”  
–是否开启拦截cc攻击(需要nginx.conf的http段增加lua_shared_dict limit 10m;)  
CCrate = “30/60”  
–设置cc攻击频率，单位为秒.  
–默认1分钟同一个IP只能请求同一个地址30次  
html=[[Please go away~~]]  
–警告内容,可在中括号内自定义  
备注:不要乱动双引号，区分大小写._

修改nginx的配置文件使其加载waf功能模块。  
`vim /data/server/nginx/conf/nginx.conf`  
http里面添加如下（注意文件内的格式）  
`lua_package_path "/data/server/nginx/conf/waf/?.lua";`  
`lua_shared_dict limit 10m;`  
`init_by_lua_file /data/server/nginx/conf/waf/init.lua;`  
`access_by_lua_file /data/server/nginx/conf/waf/waf.lua;`  
创建nginx的启动脚本  
`vim /lib/systemd/system/nginx.service`  
内容如下：  
== [Unit]  
Description=The NGINX HTTP and reverse proxy server  
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]  
Type=forking  
PIDFile=/data/server/nginx/logs/nginx.pid  
ExecStartPre=/data/server/nginx/sbin/nginx -t  
ExecStart=/data/server/nginx/sbin/nginx  
ExecReload=/data/server/nginx/sbin/nginx -s reload  
ExecStop=/usr/bin/kill -s QUIT $MAINPID  
PrivateTmp=true

[Install]  
WantedBy=multi-user.target==

启动nginx并设置为开机自启  
`systemctl start nginx.service`  
`systemctl enable nginx.service`  
创建nginx的软连接:  
`ln -s /data/server/nginx/sbin/* /usr/local/sbin/`

# 三、测试

浏览器访问：  
http://安装waf的IP/test.txt?id=../../etc/passwd  
![](https://app.yinxiang.com/FileSharing.action?hash=1/13ad558d231e9bde3653bb3f97920f15-23899)  
如上图所示，WAF成功起了作用。还可以根据自己的需求给WAF添加过滤规则，使其更安全可靠，到这一个简单的WAF就搭建完成了。