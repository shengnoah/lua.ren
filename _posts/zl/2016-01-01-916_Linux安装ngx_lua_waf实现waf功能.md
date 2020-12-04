---
layout: post
title: Linux安装ngx_lua_waf实现waf功能 
tags: [lua文章]
categories: [topic]
---
<h1 id="一、ngx-lua-waf用途"><a href="#一、ngx-lua-waf用途" class="headerlink" title="一、ngx_lua_waf用途"></a>一、ngx_lua_waf用途</h1><p> 1、防止SQL注入，本地包含，本地溢出，fuzzing测试，XSS，SSRF等web攻击;<br/>2、防止SVN/备份之类文件泄漏;<br/>3、防止apachebench之类的压力测试工具的攻击;<br/>4、屏蔽常见的扫描黑客工具，扫描器;<br/>5、屏蔽常见的网络请求;<br/>6、屏蔽照片附件类目录php执行权限;<br/>7、防止webshell上传。</p>
<h1 id="二、安装"><a href="#二、安装" class="headerlink" title="二、安装"></a>二、安装</h1><h2 id="1、首先安装所需要的依赖环境"><a href="#1、首先安装所需要的依赖环境" class="headerlink" title="1、首先安装所需要的依赖环境"></a>1、首先安装所需要的依赖环境</h2><p><code>yum -y install gcc gcc-c++ wget git geoip-devel gd-devel pcre-deve libcurl-devel libxml2 libxml2-devel libgd-devel openssl-devel lua-devel</code></p>
<h2 id="2、LuaJIT"><a href="#2、LuaJIT" class="headerlink" title="2、LuaJIT"></a>2、LuaJIT</h2><p> 下载并安装LuaJIT2.0.5，首先来到/usr/local/src（压缩包存放目录）目录下。<br/> <code>cd /usr/local/src</code><br/> <code>wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz</code><br/><code>tar -zxvf LuaJIT-2.0.5.tar.gz</code><br/><code>cd LuaJIT-2.0.5</code><br/><code>make install PREFIX=/usr/local/src/luajit</code><br/>然后创建一条软连接：<br/><code>ln -s /usr/local/src/luajit/lib/libluajit-5.1.so.2 /lib64/libluajit-5.1.so.2</code></p>
<h2 id="3、ngx-devel-kit"><a href="#3、ngx-devel-kit" class="headerlink" title="3、ngx_devel_kit"></a>3、ngx_devel_kit</h2><p>下载并安装ngx_devel_kit<br/><code>cd /usr/local/src</code><br/><code>wget https://github.com/simpl/ngx_devel_kit/archive/v0.3.0.tar.gz</code><br/><code>tar -zxvf v0.3.0.tar.gz</code></p>
<h2 id="4、lua-nginx-model"><a href="#4、lua-nginx-model" class="headerlink" title="4、lua-nginx-model"></a>4、lua-nginx-model</h2><p>下载并安装lua-nginx-model（nginx的lua模块）<br/><code>wget https://github.com/openresty/lua-nginx-module/archive/v0.10.14rc3.tar.gz</code><br/><code>tar -zxvf v0.10.14rc3.tar.gz</code></p>
<h2 id="5、安装nginx"><a href="#5、安装nginx" class="headerlink" title="5、安装nginx"></a>5、安装nginx</h2><p>下载并安装nginx，这里我选择的是1.15.2版本的。<br/><code>wget http://nginx.org/download/nginx-1.15.2.tar.gz</code><br/>然后开始编译安装。<br/><code>./configure 
--user=www 
--group=www 
--prefix=/data/server/nginx 
--error-log-path=/data/server/nginx/error.log 
--http-log-path=/data/server/nginx/access.log 
--with-http_ssl_module 
--with-http_v2_module 
--with-http_realip_module 
--with-http_addition_module 
--with-http_image_filter_module 
--with-http_geoip_module 
--with-http_sub_module 
--with-http_dav_module 
--with-http_flv_module 
--with-http_mp4_module 
--with-http_gunzip_module 
--with-http_gzip_static_module 
--with-http_random_index_module 
--with-http_secure_link_module 
--with-http_degradation_module 
--with-http_slice_module 
--with-http_stub_status_module 
--with-pcre 
--with-pcre-jit 
--with-stream 
--with-stream_ssl_module 
--with-debug 
--add-module=/usr/local/src/ngx_devel_kit-0.3.0 
--add-module=/usr/local/src/lua-nginx-module-0.10.14rc3 
--with-ld-opt=&#34;-Wl,-rpath,$LUAJIT_LIB&#34; ;</code><br/>检查没问题的话开始安装：<br/><code>make &amp;&amp; make install</code></p>
<h2 id="6、下载并安装waf模块"><a href="#6、下载并安装waf模块" class="headerlink" title="6、下载并安装waf模块"></a>6、下载并安装waf模块</h2><p><code>wget https://github.com/hack-umbrella/ngx_lua_waf/archive/master.zip</code><br/>解压并改名为waf，移动到nginx的配置目录下<br/><code>unzip master.zip</code><br/><code>mv /usr/local/src/ngx_lua_waf-0.7.2/ waf</code><br/><code>cp -rf /usr/local/src/waf/ /data/server/nginx/conf/</code><br/>修改waf模块的规则配置路径<br/><code>vim /data/server/nginx/conf/waf/config.lua</code><br/>修改配置文件为如下：<br/>   RulePath = “/data/server/nginx/conf/waf/wafconf/“<br/>    –规则存放目录<br/>    attacklog = “off”<br/>    –是否开启攻击信息记录，需要配置logdir<br/>    logdir = “/data/server/nginx/logs/hack/“<br/>    –log存储目录，该目录需要用户自己新建，切需要nginx用户的可写权限<br/>    UrlDeny=”on”<br/>    –是否拦截url访问<br/>    Redirect=”on”<br/>    –是否拦截后重定向<br/>    CookieMatch = “on”<br/>    –是否拦截cookie攻击<br/>    postMatch = “on”<br/>    –是否拦截post攻击<br/>    whiteModule = “on”<br/>    –是否开启URL白名单<br/>    black_fileExt={“php”,”jsp”}<br/>    –填写不允许上传文件后缀类型<br/>    ipWhitelist={“127.0.0.1”}<br/>    –ip白名单，多个ip用逗号分隔<br/>    ipBlocklist={“1.0.0.1”}<br/>    –ip黑名单，多个ip用逗号分隔<br/>    CCDeny=”on”<br/>    –是否开启拦截cc攻击(需要nginx.conf的http段增加lua_shared_dict limit 10m;)<br/>    CCrate = “30/60”<br/>    –设置cc攻击频率，单位为秒.<br/>    –默认1分钟同一个IP只能请求同一个地址30次<br/>    html=[[Please go away~~]]<br/>    –警告内容,可在中括号内自定义<br/>    备注:不要乱动双引号，区分大小写._</p>
<p>修改nginx的配置文件使其加载waf功能模块。<br/><code>vim /data/server/nginx/conf/nginx.conf</code><br/>http里面添加如下（注意文件内的格式）<br/>    <code>lua_package_path &#34;/data/server/nginx/conf/waf/?.lua&#34;;</code><br/>    <code>lua_shared_dict limit 10m;</code><br/>    <code>init_by_lua_file  /data/server/nginx/conf/waf/init.lua;</code><br/>    <code>access_by_lua_file /data/server/nginx/conf/waf/waf.lua;</code><br/>创建nginx的启动脚本<br/><code>vim /lib/systemd/system/nginx.service</code><br/>内容如下：<br/>== [Unit]<br/>Description=The NGINX HTTP and reverse proxy server<br/>After=syslog.target network.target remote-fs.target nss-lookup.target</p>
<p>[Service]<br/>Type=forking<br/>PIDFile=/data/server/nginx/logs/nginx.pid<br/>ExecStartPre=/data/server/nginx/sbin/nginx -t<br/>ExecStart=/data/server/nginx/sbin/nginx<br/>ExecReload=/data/server/nginx/sbin/nginx -s reload<br/>ExecStop=/usr/bin/kill -s QUIT $MAINPID<br/>PrivateTmp=true</p>
<p>[Install]<br/>WantedBy=multi-user.target==</p>
<p>启动nginx并设置为开机自启<br/><code>systemctl start nginx.service</code><br/><code>systemctl enable nginx.service</code><br/>创建nginx的软连接:<br/><code>ln -s /data/server/nginx/sbin/* /usr/local/sbin/</code></p>
<h1 id="三、测试"><a href="#三、测试" class="headerlink" title="三、测试"></a>三、测试</h1><p>浏览器访问：<br/>http://安装waf的IP/test.txt?id=../../etc/passwd<br/><img src="https://app.yinxiang.com/FileSharing.action?hash=1/13ad558d231e9bde3653bb3f97920f15-23899" alt=""/><br/>如上图所示，WAF成功起了作用。还可以根据自己的需求给WAF添加过滤规则，使其更安全可靠，到这一个简单的WAF就搭建完成了。</p>