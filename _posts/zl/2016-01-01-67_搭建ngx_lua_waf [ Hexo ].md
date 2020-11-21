---
layout: post
title: 搭建ngx_lua_waf [ Hexo ] 
tags: [lua文章]
categories: [topic]
---
Created at 2019-05-26 Updated at 2019-08-15 Category [搭建服务](/categories/搭建服务/)
Tag [搭建ngx_lua_waf](/tags/搭建ngx-lua-waf/)

* * *

# 搭建ngx_lua_waf

工作环境：centos7  
  
1.下载nginx源文件  
下载地址 ： <http://nginx.org/en/download.html>  
cd /usr/local/src  
wget <http://nginx.org/download/nginx-1.12.2.tar.gz>  
tar -xvf nginx-1.12.2.tar.gz  
  
  
2.下载安装LuaJIT  
下载地址：<http://luajit.org/download.html>  
cd /usr/local/src  
wget <http://luajit.org/download/LuaJIT-2.1.0-beta3.tar.gz>  
tar -zxf LuaJIT-2.1.0-beta3.tar.gz  
cd LuaJIT-2.1.0-beta3  
make PREFIX=/usr/local/src/luajit  
make install PREFIX=/usr/local/src/luajit  
  
  
3.下载ngx_devel_kit（NDK）模块  
下载地址：<https://github.com/simplresty/ngx_devel_kit/tags>  
cd /usr/local/src  
wget <https://github.com/simplresty/ngx_devel_kit/archive/v0.3.0.tar.gz>  
tar -xvf v0.3.0.tar.gz  
  
  
4.下载lua-nginx-module模块  
下载地址：<https://github.com/openresty/lua-nginx-module/tags>  
cd /usr/local/src  
wget <https://github.com/openresty/lua-nginx-module/archive/v0.10.11.tar.gz>  
tar -xvf v0.10.11.tar.gz  
  
  
5.编译安装Nginx  
cd /usr/local/src/nginx-1.12.2  
1.切换到Nginx源文件目录，设置环境变量  
export LUAJIT_LIB=/usr/local/src/luajit/lib  
export LUAJIT_INC=/usr/local/src/luajit/include/luajit-2.1  
  
  
2.编译  
./configure  
–prefix=/data/server/nginx  
–error-log-path=/data/server/nginx/error.log  
–http-log-path=/data/server/nginx/access.log  
–with-http_ssl_module  
–with-http_v2_module  
–with-http_realip_module  
–with-http_addition_module  
–with-http_image_filter_module  
–with-http_geoip_module  
–with-http_sub_module  
–with-http_dav_module  
–with-http_flv_module  
–with-http_mp4_module  
–with-http_gunzip_module  
–with-http_gzip_static_module  
–with-http_random_index_module  
–with-http_secure_link_module  
–with-http_degradation_module  
–with-http_slice_module  
–with-http_stub_status_module  
–with-pcre  
–with-pcre-jit  
–with-stream  
–with-stream_ssl_module  
–with-debug  
–add-module=/usr/local/src/ngx_devel_kit-0.3.0  
–add-module=/usr/local/src/lua-nginx-module-0.10.11  
–with-ld-opt=”-Wl,-rpath,$LUAJIT_LIB” ;  
make && make install  
  
  
6.安装ngx_lua_waf模块  
下载地址：<https://github.com/loveshell/ngx_lua_waf/tree/master>  
[![mkNvrV.png](https://s2.ax1x.com/2019/08/14/mkNvrV.png)](https://imgchr.com/i/mkNvrV)  
克隆  
cd /usr/local/src/  
git clone <https://github.com/loveshell/ngx_lua_waf.git>  
mv ngx_lua_waf waf  
mv waf /data/server/nginx/conf/  
修改waf的模块的规则配置路径  
vi /data/server/nginx/conf/waf/config.lua  
修改为下图所示页面(主要修改路径)  
[![mkUkx1.png](https://s2.ax1x.com/2019/08/14/mkUkx1.png)](https://imgchr.com/i/mkUkx1)  
7.修改nginx的配置文件，使其加载waf功能模块  
vi /data/server/nginx/conf/nginx.conf  
添加以下内容  
lua_package_path “/data/server/nginx/conf/waf/?.lua”;  
lua_shared_dict limit 10m;  
init_by_lua_file /data/server/nginx/conf/waf/init.lua;  
access_by_lua_file /data/server/nginx/conf/waf/waf.lua;  
  
  
[![mkUosx.png](https://s2.ax1x.com/2019/08/14/mkUosx.png)](https://imgchr.com/i/mkUosx)  
  
  
8、关闭防火墙、SELINUX，启动  
systemctl stop firewalld  
setenforce 0  
/data/server/nginx/sbin/nginx  
访问：192.168.43.198出现nginx页面  
  
  
[![mkUOFe.png](https://s2.ax1x.com/2019/08/14/mkUOFe.png)](https://imgchr.com/i/mkUOFe)  
访问：<http://192.168.43.198/test.php?id=../etc/passwd>  
  
  
[![mkwtZ6.png](https://s2.ax1x.com/2019/08/14/mkwtZ6.png)](https://imgchr.com/i/mkwtZ6)