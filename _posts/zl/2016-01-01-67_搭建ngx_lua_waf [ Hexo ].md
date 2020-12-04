---
layout: post
title: 搭建ngx_lua_waf [ Hexo ] 
tags: [lua文章]
categories: [topic]
---
<p class="page-title-sub">
      <span id="post-title-date">Created at 2019-05-26</span>
      
        <span id="post-title-updated">Updated at 2019-08-15</span>
      
      
      <span id="post-title-categories">Category
      
      
        
        
        <a href="/categories/搭建服务/">搭建服务</a>
      
      </span>
      
      
      <span id="post-title-tags">
      Tag
      
      
        
        
        <a href="/tags/搭建ngx-lua-waf/">搭建ngx_lua_waf</a>
      
      </span>
      
    </p>
    
    <hr/>
<h1 id="搭建ngx-lua-waf"><a href="#搭建ngx-lua-waf" class="headerlink" title="搭建ngx_lua_waf"></a>搭建ngx_lua_waf</h1><p>工作环境：centos7<br/> <br/>1.下载nginx源文件<br/>下载地址 ： <a href="http://nginx.org/en/download.html" target="_blank" rel="noopener noreferrer">http://nginx.org/en/download.html</a><br/>cd /usr/local/src<br/>wget <a href="http://nginx.org/download/nginx-1.12.2.tar.gz" target="_blank" rel="noopener noreferrer">http://nginx.org/download/nginx-1.12.2.tar.gz</a><br/>tar -xvf nginx-1.12.2.tar.gz<br/> <br/> <br/>2.下载安装LuaJIT<br/>下载地址：<a href="http://luajit.org/download.html" target="_blank" rel="noopener noreferrer">http://luajit.org/download.html</a><br/>cd /usr/local/src<br/>wget <a href="http://luajit.org/download/LuaJIT-2.1.0-beta3.tar.gz" target="_blank" rel="noopener noreferrer">http://luajit.org/download/LuaJIT-2.1.0-beta3.tar.gz</a><br/>tar -zxf LuaJIT-2.1.0-beta3.tar.gz<br/>cd LuaJIT-2.1.0-beta3<br/>make PREFIX=/usr/local/src/luajit<br/>make install PREFIX=/usr/local/src/luajit<br/> <br/> <br/>3.下载ngx_devel_kit（NDK）模块<br/>下载地址：<a href="https://github.com/simplresty/ngx_devel_kit/tags" target="_blank" rel="noopener noreferrer">https://github.com/simplresty/ngx_devel_kit/tags</a><br/>cd /usr/local/src<br/>wget <a href="https://github.com/simplresty/ngx_devel_kit/archive/v0.3.0.tar.gz" target="_blank" rel="noopener noreferrer">https://github.com/simplresty/ngx_devel_kit/archive/v0.3.0.tar.gz</a><br/>tar -xvf v0.3.0.tar.gz<br/> <br/> <br/>4.下载lua-nginx-module模块<br/>下载地址：<a href="https://github.com/openresty/lua-nginx-module/tags" target="_blank" rel="noopener noreferrer">https://github.com/openresty/lua-nginx-module/tags</a><br/>cd /usr/local/src<br/>wget <a href="https://github.com/openresty/lua-nginx-module/archive/v0.10.11.tar.gz" target="_blank" rel="noopener noreferrer">https://github.com/openresty/lua-nginx-module/archive/v0.10.11.tar.gz</a><br/>tar -xvf v0.10.11.tar.gz<br/> <br/> <br/>5.编译安装Nginx<br/>cd /usr/local/src/nginx-1.12.2<br/>1.切换到Nginx源文件目录，设置环境变量<br/>export LUAJIT_LIB=/usr/local/src/luajit/lib<br/>export LUAJIT_INC=/usr/local/src/luajit/include/luajit-2.1<br/> <br/> <br/>2.编译<br/>./configure <br/>–prefix=/data/server/nginx <br/>–error-log-path=/data/server/nginx/error.log <br/>–http-log-path=/data/server/nginx/access.log <br/>–with-http_ssl_module <br/>–with-http_v2_module <br/>–with-http_realip_module <br/>–with-http_addition_module <br/>–with-http_image_filter_module <br/>–with-http_geoip_module <br/>–with-http_sub_module <br/>–with-http_dav_module <br/>–with-http_flv_module <br/>–with-http_mp4_module <br/>–with-http_gunzip_module <br/>–with-http_gzip_static_module <br/>–with-http_random_index_module <br/>–with-http_secure_link_module <br/>–with-http_degradation_module <br/>–with-http_slice_module <br/>–with-http_stub_status_module <br/>–with-pcre <br/>–with-pcre-jit <br/>–with-stream <br/>–with-stream_ssl_module <br/>–with-debug <br/>–add-module=/usr/local/src/ngx_devel_kit-0.3.0 <br/>–add-module=/usr/local/src/lua-nginx-module-0.10.11 <br/>–with-ld-opt=”-Wl,-rpath,$LUAJIT_LIB” ;<br/>make &amp;&amp; make install<br/> <br/> <br/>6.安装ngx_lua_waf模块<br/>下载地址：<a href="https://github.com/loveshell/ngx_lua_waf/tree/master" target="_blank" rel="noopener noreferrer">https://github.com/loveshell/ngx_lua_waf/tree/master</a><br/><a href="https://imgchr.com/i/mkNvrV" target="_blank" rel="noopener noreferrer"><img src="https://s2.ax1x.com/2019/08/14/mkNvrV.png" alt="mkNvrV.png"/></a><br/>克隆<br/>cd /usr/local/src/<br/>git clone <a href="https://github.com/loveshell/ngx_lua_waf.git" target="_blank" rel="noopener noreferrer">https://github.com/loveshell/ngx_lua_waf.git</a><br/>mv ngx_lua_waf waf<br/>mv waf /data/server/nginx/conf/<br/>修改waf的模块的规则配置路径<br/>vi /data/server/nginx/conf/waf/config.lua<br/>修改为下图所示页面(主要修改路径)<br/><a href="https://imgchr.com/i/mkUkx1" target="_blank" rel="noopener noreferrer"><img src="https://s2.ax1x.com/2019/08/14/mkUkx1.png" alt="mkUkx1.png"/></a><br/>7.修改nginx的配置文件，使其加载waf功能模块<br/>vi /data/server/nginx/conf/nginx.conf<br/>添加以下内容<br/>    lua_package_path “/data/server/nginx/conf/waf/?.lua”;<br/>    lua_shared_dict limit 10m;<br/>    init_by_lua_file  /data/server/nginx/conf/waf/init.lua;<br/>    access_by_lua_file /data/server/nginx/conf/waf/waf.lua;<br/> <br/> <br/><a href="https://imgchr.com/i/mkUosx" target="_blank" rel="noopener noreferrer"><img src="https://s2.ax1x.com/2019/08/14/mkUosx.png" alt="mkUosx.png"/></a><br/> <br/> <br/>8、关闭防火墙、SELINUX，启动<br/>systemctl stop firewalld<br/>setenforce 0<br/>/data/server/nginx/sbin/nginx<br/>访问：192.168.43.198出现nginx页面<br/> <br/> <br/><a href="https://imgchr.com/i/mkUOFe" target="_blank" rel="noopener noreferrer"><img src="https://s2.ax1x.com/2019/08/14/mkUOFe.png" alt="mkUOFe.png"/></a><br/>访问：<a href="http://192.168.43.198/test.php?id=../etc/passwd" target="_blank" rel="noopener noreferrer">http://192.168.43.198/test.php?id=../etc/passwd</a><br/> <br/> <br/><a href="https://imgchr.com/i/mkwtZ6" target="_blank" rel="noopener noreferrer"><img src="https://s2.ax1x.com/2019/08/14/mkwtZ6.png" alt="mkwtZ6.png"/></a></p>