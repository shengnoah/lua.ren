---
layout: post
title: 在原有的nginx上面安装Openresty和lua相关的模块 
tags: [lua文章]
categories: [topic]
---
突然有一天出了个需求，做文件防盗链的，而且需要通过nginx来做，这个时候必然想到了`Openresty`，Openresty本身其实已经安装有nginx了，但是要求在公司原有的nginx上面装一些Openresty里面的模块，这个时候就有点复杂了，但是最终还是研究出来了，庆幸啊，这里做一个笔记，以便下次安装使用。  

# 安装openresty

  1. 下载openresty 

下载地址：<https://github.com/openresty/openresty/releases>  

    
    
    1  
    

|

    
    
    wget https://github.com/openresty/openresty/releases/download/v1.13.6.1/openresty-1.13.6.1.tar.gz  
      
  
---|---  
  
  2. 编译安装 

解压  

    
    
    1  
    2  
    3  
    

|

    
    
    tar -xvf openresty-1.13.6.1.tar.gz  
      
    cd openresty-1.13.6.1  
      
  
---|---  
  
编译安装  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    ./configure -j2  
      
    gmake  
      
    gmake install  
      
  
---|---  
  
# 安装lua

在下载`openresty`安装包的时候，里面其实已经依赖了`lua`了，只需要安装就好了

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    cd openresty-1.13.6.1/bundle/LuaJIT-2.1-20171103/  
      
    make  
      
    make install  
      
  
---|---  
  
# nginx添加相关模块

  1. 配置lua位置 

找到以前`nginx`的源码包，配置lua位置

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    cd nginx-1.15.0  
      
    export LUAJIT_LIB=/usr/local/openresty/lualib/  
    export LUAJIT_INC=/usr/local/openresty/luajit/include/luajit-2.1/  
      
  
---|---  
  
  2. 重新编译nginx 

    
    
    1  
    

|

    
    
    ./configure --prefix=/usr/local/nginx --with-cc-opt=-O2 --add-module=/root/openresty-1.13.6.1/bundle/ngx_devel_kit-0.3.0 --add-module=/root/openresty-1.13.6.1/bundle/echo-nginx-module-0.61 --add-module=/root/openresty-1.13.6.1/bundle/xss-nginx-module-0.05 --add-module=/root/openresty-1.13.6.1/bundle/ngx_coolkit-0.2rc3 --add-module=/root/openresty-1.13.6.1/bundle/set-misc-nginx-module-0.31 --add-module=/root/openresty-1.13.6.1/bundle/form-input-nginx-module-0.12 --add-module=/root/openresty-1.13.6.1/bundle/encrypted-session-nginx-module-0.07 --add-module=/root/openresty-1.13.6.1/bundle/srcache-nginx-module-0.31 --add-module=/root/openresty-1.13.6.1/bundle/ngx_lua-0.10.11 --add-module=/root/openresty-1.13.6.1/bundle/ngx_lua_upstream-0.07 --add-module=/root/openresty-1.13.6.1/bundle/headers-more-nginx-module-0.33 --add-module=/root/openresty-1.13.6.1/bundle/array-var-nginx-module-0.05 --add-module=/root/openresty-1.13.6.1/bundle/memc-nginx-module-0.18 --add-module=/root/openresty-1.13.6.1/bundle/redis2-nginx-module-0.14 --add-module=/root/openresty-1.13.6.1/bundle/redis-nginx-module-0.3.7 --add-module=/root/openresty-1.13.6.1/bundle/rds-json-nginx-module-0.15 --add-module=/root/openresty-1.13.6.1/bundle/rds-csv-nginx-module-0.08 --add-module=/root/openresty-1.13.6.1/bundle/ngx_stream_lua-0.0.3 --with-ld-opt=-Wl,-rpath,/usr/local/lib/ --with-stream --with-stream_ssl_module --with-http_ssl_module  
      
  
---|---  
  
编译完成了，执行`make`，记住，这里不要执行`make install`，不然会把以前安装的会覆盖的

    
    
    1  
    

|

    
    
    make  
      
  
---|---  
  
这里有几个参数说明一下：

  * –prefix=/usr/local/nginx：nginx安装目录
  * –add-module=/root/openresty-1.13.6.1/bundle：这个是刚刚下载的openresty安装包
  * –with-ld-opt=-Wl,-rpath,/usr/local/lib/：lua安装的路径，上面lua安装的时候，默认是这个位置的

编译完成后，会新生成一个nginx执行文件，在nginx-1.15.0/objs目录下，测试一下对应的依赖有没有装上

    
    
    1  
    2  
    3  
    

|

    
    
    cd nginx-1.15.0/objs  
      
    ./nginx -V  
      
  
---|---  
  
显示以下，说明完美  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    nginx version: nginx/1.15.0  
    built by gcc 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC)  
    built with OpenSSL 1.0.2k-fips  26 Jan 2017  
    TLS SNI support enabled  
    configure arguments: --prefix=/usr/local/nginx --with-cc-opt=-O2 --add-module=/root/openresty-1.13.6.1/bundle/ngx_devel_kit-0.3.0 --add-module=/root/openresty-1.13.6.1/bundle/echo-nginx-module-0.61 --add-module=/root/openresty-1.13.6.1/bundle/xss-nginx-module-0.05 --add-module=/root/openresty-1.13.6.1/bundle/ngx_coolkit-0.2rc3 --add-module=/root/openresty-1.13.6.1/bundle/set-misc-nginx-module-0.31 --add-module=/root/openresty-1.13.6.1/bundle/form-input-nginx-module-0.12 --add-module=/root/openresty-1.13.6.1/bundle/encrypted-session-nginx-module-0.07 --add-module=/root/openresty-1.13.6.1/bundle/srcache-nginx-module-0.31 --add-module=/root/openresty-1.13.6.1/bundle/ngx_lua-0.10.11 --add-module=/root/openresty-1.13.6.1/bundle/ngx_lua_upstream-0.07 --add-module=/root/openresty-1.13.6.1/bundle/headers-more-nginx-module-0.33 --add-module=/root/openresty-1.13.6.1/bundle/array-var-nginx-module-0.05 --add-module=/root/openresty-1.13.6.1/bundle/memc-nginx-module-0.18 --add-module=/root/openresty-1.13.6.1/bundle/redis2-nginx-module-0.14 --add-module=/root/openresty-1.13.6.1/bundle/redis-nginx-module-0.3.7 --add-module=/root/openresty-1.13.6.1/bundle/rds-json-nginx-module-0.15 --add-module=/root/openresty-1.13.6.1/bundle/rds-csv-nginx-module-0.08 --add-module=/root/openresty-1.13.6.1/bundle/ngx_stream_lua-0.0.3 --with-ld-opt=-Wl,-rpath,/usr/local/lib/ --with-stream --with-stream_ssl_module --with-http_ssl_module  
      
  
---|---  
  
  3. 复制nginx命令覆盖以前的nginx 

复制前，最好把之前的nginx备份一下，以防不测  

    
    
    1  
    2  
    3  
    

|

    
    
    cd /usr/local/nginx/sbin/  
      
    cp nginx nginx.old  
      
  
---|---  
  
赢新的覆盖,覆盖之前，最好停掉nginx  

    
    
    1  
    2  
    3  
    

|

    
    
    cd nginx-1.15.0/  
      
    cp objs/nginx /usr/local/nginx/sbin/  
      
  
---|---  
  
这里会提示是否覆盖，输入y，然后回车就好了

# 测试

  1. 先测试nginx有没有被玩坏，先检查一下

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    cd /usr/local/nginx  
      
    ./sbin/nginx -t  
      
    ./sbin/nginx  
      
  
---|---  
  
启动完成，访问下以前的站点还能不能正常打开，目测是没问题的

  2. 测试lua模块

创建一个专门存放lua文件的文件夹,我习惯创建在nginx目录下  

    
    
    1  
    2  
    3  
    

|

    
    
    cd /usr/local/nginx  
      
    mkdir lua  
      
  
---|---  
  
创建一个lua文件  

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    vim hello.lua  
      
      
    ngx.log(ngx.ERR,"hello");  
      
  
---|---  
  
把这个lua文件依赖到nginx里面试试  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    location / {  
            root /workspace/hexo/public/;  
            index index.html index.htm;  
            access_by_lua_file lua/hello.lua;  
    }  
      
  
---|---  
  
老规矩，先检查下有没问题没，然后重启  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    cd /usr/local/nginx  
      
    ./sbin/nginx -t  
      
    ./sbin/nginx -s reload  
      
  
---|---  
  
然后打开日志，准备看有没有打印对应的日志信息  

    
    
    1  
    

|

    
    
    tail -f logs/error.log  
      
  
---|---  
  
正常会看到以下日志  

    
    
    1  
    

|

    
    
    2018/07/04 11:58:38 [error] 15646#0: *52 [lua] hello.lua:2: hello,  
      
  
---|---  
  
完美！

* * *

  

[ 上一篇：程序员必备开发工具，提高开发效率的神兵利器，大多都是免费的哦 ](/tech/cheng-xu-yuan-bi-bei-kai-fa-gong-
ju-ti-gao-gong-zuo-xiao-lv/ "程序员必备开发工具，提高开发效率的神兵利器，大多都是免费的哦")

[ 下一篇：Apollo分布式配置中心部署以及使用 ](/tech/apollo-pei-zhi-zhong-xin-an-zhuang-bu-shu/
"Apollo分布式配置中心部署以及使用")