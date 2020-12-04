---
layout: post
title: OpenResty Lua 的使用 (一) 环境搭建 
tags: [lua文章]
categories: [topic]
---
<p>OpenResty Lua 的使用 (一) 环境搭建</p>

<h2 id="前言">前言</h2>

<p>开始学习Openresty Lua的使用,顺便也熟悉了下Nginx。</p>

<h2 id="一准备">一、准备</h2>

<ul>
  <li>Linux环境，我这里是在虚拟机安装的CentOS 7。</li>
  <li>Openresty lua 安装包。</li>
</ul>

<h2 id="二安装">二、安装</h2>

<ul>
  <li>
    <p>安装 <code class="highlighter-rouge">wget</code> 命令用以下载使用，有的话可以不安装，<code class="highlighter-rouge">wget -V</code> 测试是否安装。  <br/>
 <code class="highlighter-rouge">yum -y install wget</code></p>
  </li>
  <li>
    <p>安装 <code class="highlighter-rouge">unzip</code> 命令用以解压zip文件，有的话可以不安装。   <br/>
 <code class="highlighter-rouge">yum install -y unzip zip</code></p>
  </li>
  <li>
    <p>安装 <code class="highlighter-rouge">vim</code> 命令用以编辑文件，有的话可以不安装，在命令行敲入vi，按“tab”键，可以看到，是否已经有vim命令的存在。  <br/>
 <code class="highlighter-rouge">yum -y install vim*</code></p>
  </li>
  <li>
    <p>安装openresty其他需要的环境。  <br/>
 <code class="highlighter-rouge">yum install readline-devel pcre-devel openssl-devel gcc gcc-c++ curl perl</code></p>
  </li>
  <li>
    <p>为了方便我在opt下新建文件夹。  <br/>
<code class="highlighter-rouge">cd /opt/</code>   <br/>
<code class="highlighter-rouge">mkdir -p download backup app work</code></p>
  </li>
  <li>
    <p>进入/opt/download文件夹下<code class="highlighter-rouge">cd download/</code>。</p>
  </li>
  <li>
    <p>下载OpenrestyLua,可以在Win系统下载后上传，也可以直接在服务器下载。[<a href="http://openresty.org/cn/download.html">中文下载地址</a>],[<a href="https://openresty.org/en/download.html">英文下载地址</a>]，选择对应版本进行下载。此处介绍在服务器直接下载：  <br/>
<code class="highlighter-rouge">wget https://openresty.org/download/openresty-1.13.6.2.tar.gz</code></p>
  </li>
  <li>
    <p>同时再下载一些一会需要的模块和依赖的软件。</p>

    <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  * 下载ngx_cache_purge模块，该模块用于清理nginx缓存，非必需，建议安装。    
    `wget https://github.com/FRiCKLE/ngx_cache_purge/archive/2.3.tar.gz`
    
  * 下载nginx_upstream_check_module模块，该模块用于ustream健康检查，非必需，建议安装。    
   `wget https://github.com/yaoweibin/nginx_upstream_check_module/archive/v0.3.0.tar.gz`    
    也可以到GitHub主页自己打包下载zip，然后上传解压，https://github.com/yaoweibin/nginx_upstream_check_module 
    
  * 下载openssl模块,可以到 https://www.openssl.org/source/ 下载最新版，https://ftp.openssl.org/source/old/1.0.2/ 下载旧版。    
   `wget https://www.openssl.org/source/openssl-1.1.1.tar.gz`    
   `wget https://ftp.openssl.org/source/old/1.0.2/openssl-1.0.2l.tar.gz`
    
  * 下载pcre，地址：https://ftp.pcre.org/pub/pcre/ 自行下载。    
   `wget https://ftp.pcre.org/pub/pcre/pcre-8.41.tar.gz`
    
  * 安装drizzle模块(访问mysql数据库模块，非必需，建议安装)。    
   `wget http://agentzh.org/misc/nginx/drizzle7-2011.07.21.tar.gz`
    
  * 下载nginx-http-concat(合并静态文件请求模块，非必需，建议安装)。    
   `wget https://github.com/alibaba/nginx-http-concat/archive/master.zip`
    
  * 安装Zlib。    
   `wget https://zlib.net/fossils/zlib-1.2.8.tar.gz`
    
  * 下载luajit，此处使用OpenResty，忽略此步骤。    
   `wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz`
</code></pre></div>    </div>
  </li>
  <li>
    <p>解压上面下载的几个文件。  <br/>
<code class="highlighter-rouge">tar -zxvf XXXX</code></p>

    <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  此处扩展一点其他知识点，此处不操作，了解即可：
  安装PCRE
      yum install -y gcc gcc-c++
      cd pcre-8.41
      ./configure
      make
      make install
  安装openSSL
      cd openssl-1.0.2l 
      设定Openssl 安装，( --prefix )参数为欲安装之目录，也就是安装后的档案会出现在该目录下：
      ./config --prefix=/usr/local/openssl
      ./config -t
      make
      make install 
  安装drizzle模块   
      cd drizzle7-2011.07.21  
      ./configure --without-server  
      make libdrizzle-1.0  
      make install-libdrizzle-1.0 
  安装Zlib
      cd zlib-1.2.8
      ./configure    --prefix=/data/progam/zlib
      make
      make install
      再进行配置一下系统的文件，加载刚才编译安装的zlib生成的库文件
      vi /etc/ld.so.conf.d/zlib.conf
      加入如下内容后保存退出: /data/program/zlib/lib
      也就是添加安装目录的文件路径，库文件。ldconfig  运行之后就会加载安装的库文件了。
  安装luajit,注意使用openresty不用安装luajit
      cd LuaJIT-2.0.3 
      make &amp;&amp; make install PREFIX=/usr/local/luajit
      export LUAJIT_LIB=/usr/local/luajit/lib 
      export LUAJIT_INC=/usr/local/luajit/include/luajit-2.0
</code></pre></div>    </div>
  </li>
  <li>
    <p>进入解压后的Opentesty文件下，<code class="highlighter-rouge">cd openresty-1.13.6.2</code> ,可以看到存在一个名为configure的可执行脚本程序。它是用于检查系统是否有编译时所需的库，以及库的版本是否满足编译的需要等安装所需要的系统信息。为随后的编译工作做准备。在当前文件夹下执行如下设置，注意不能有换行,注意版本，注意刚刚下载的几个模块路径需要修改，自带的一下模块自动会包含在内不用再配置。  <br/>
  <code class="highlighter-rouge">./configure --prefix=/usr/local/openresty    --with-debug    --add-module=/opt/download/ngx_cache_purge-2.3 --add-module=/opt/download/nginx_upstream_check_module-0.3.0 --with-http_sub_module --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module --with-pcre-jit --with-http_v2_module --with-openssl=/opt/download/openssl-1.0.2l --with-pcre=/opt/download/pcre-8.41 --with-http_gzip_static_module --with-http_flv_module --with-stream --with-stream_ssl_module --with-stream --with-stream_ssl_module </code>  <br/>
 如觉得太长 可以再尾部使用  来代表不换行，如 ：</p>
    <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>      ./configure 
      --prefix=/usr/local/openresty  
      --with-debug 
      xxx等
</code></pre></div>    </div>
    <p>可以在测试环境打开debug，使用<code class="highlighter-rouge">--with-debug</code> ，nginx默认是info以上的。  <br/>
有的地方看到-j2参数，应该是说明电脑支持多核工作，假设是双核，可以使用-j2。</p>
    <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>    如下这些参数会默认加载 不需要另外操作，如果加上的话重复加载模块，容易出错。
    --with-cc-opt=&#39;-DNGX_LUA_USE_ASSERT -DNGX_LUA_ABORT_AT_PANIC -O2&#39; 
    --add-module=../ngx_devel_kit-0.3.0 
    --add-module=../echo-nginx-module-0.61 
    --add-module=../xss-nginx-module-0.06 
    --add-module=../ngx_coolkit-0.2rc3 
    --add-module=../set-misc-nginx-module-0.32 
    --add-module=../form-input-nginx-module-0.12 
    --add-module=../encrypted-session-nginx-module-0.08 
    --add-module=../srcache-nginx-module-0.31 
    --add-module=../ngx_lua-0.10.13 
    --add-module=../ngx_lua_upstream-0.07 
    --add-module=../headers-more-nginx-module-0.33 
    --add-module=../array-var-nginx-module-0.05 
    --add-module=../memc-nginx-module-0.19 
    --add-module=../redis2-nginx-module-0.15 
    --add-module=../redis-nginx-module-0.3.7 
    --add-module=../rds-json-nginx-module-0.15 
    --add-module=../rds-csv-nginx-module-0.09 
    --add-module=../ngx_stream_lua-0.0.5 
    --with-ld-opt=&#39;-Wl,-rpath,/usr/local/openresty/nginx/luajit/lib&#39;
</code></pre></div>    </div>
  </li>
</ul>

<p>其他具体参数介绍可以看之前的一篇文章[[^Nginx的使用之编译配置参数]]</p>

<ul>
  <li>如果执行出错说明配置的参数有问题，一般是可能版本与安装的不对应， 具体原因可以看源码包目录下的 build/nginx-VERSION/objs/autoconf.err文件查看。如果上部没有出错，会出现提示<code class="highlighter-rouge">gmake / gmake install</code> 的提示，根据提示一步步执行操作，先执行：<code class="highlighter-rouge">gmake</code> 执行完毕后执行：<code class="highlighter-rouge">gmake install</code>。</li>
</ul>

<h2 id="三启动">三、启动</h2>
<p>注意：</p>
<ol>
  <li>
    <p>如果需要指定其他的目录的配置文件，可以复制/usr/local/openresty/nginx/conf/下的mime.types和nginx.conf。mime.types是必须，nginx.conf按照自己的需求修改。</p>
  </li>
  <li>
    <p>如果需要指定其他的目录的配置文件用以存放代码执行目录的话，需要把/usr/local/openresty/nginx/目录下的html目录文件也复制过来，一般是一个index.html文件和50x.html文件。</p>
  </li>
</ol>

<ul>
  <li>
    <p>查看版本信息：  <br/>
<code class="highlighter-rouge">/usr/local/openresty/nginx/sbin/nginx -v</code></p>
  </li>
  <li>
    <p>查看配置信息：  <br/>
<code class="highlighter-rouge">/usr/local/openresty/nginx/sbin/nginx -V</code></p>
  </li>
  <li>配置文件,可以先简单设置 如打印hello world 或者使用默认配置启动：
    <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  worker_processes  1;
    
  error_log  /opt/logs/error.log debug;
    
  pid        /opt/logs/lua-nginx.pid;
    
  events {
  	use epoll;
  	multi_accept on;
      worker_connections  1024;
  }
      
  http {
        
      include       mime.types;
      default_type  application/octet-stream;
    
      log_format  main  &#39;$remote_addr - $remote_user [$time_local] &#34;$request&#34; &#39;
                        &#39;$status $body_bytes_sent &#34;$http_referer&#34; &#39;
                        &#39;&#34;$http_user_agent&#34; &#34;$http_x_forwarded_for&#34;&#39;;
    
      sendfile        on;

      keepalive_timeout  65;

  	lua_package_path  &#39;${prefix}?.lua;${prefix}?/init.lua;;&#39;;
      lua_package_cpath &#39;${prefix}?.so;;&#39;;
    
      server {
          listen       80;
          server_name  www.luatest.com;

          access_log  /opt/logs/lua.access.log  main;
    
          location / {
              root   html;
              index  index.html index.htm;
          }
    		
  		location =/lua {
              default_type text/plain;
              content_by_lua &#39;ngx.say(&#34;hello,lua&#34;)&#39;;
          }
    
          error_page  404              /404.html;
    
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   html;
          }
      }
  }
</code></pre></div>    </div>
  </li>
  <li>
    <p>启动：  <br/>
<code class="highlighter-rouge">/usr/local/openresty/nginx/sbin/nginx -p /opt/app/lua/ -c /opt/work/nginx.conf </code>，-p指定代码的root目录，相当于prefix， -c 指定配置文件。</p>
  </li>
  <li>
    <p>查看进程：  <br/>
<code class="highlighter-rouge">ps -ef|grep nginx</code></p>
  </li>
  <li>
    <p>终止进程：  <br/>
<code class="highlighter-rouge">kill  进程id</code></p>
  </li>
  <li>如果觉得每次前面太长可以设置环境变量 。
    <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  vi  /etc/profile
    
  PATH=/usr/local/openresty/nginx/sbin:$PATH
  export PATH
  或者
  export PATH=/usr/local/openresty/nginx/sbin:$PATH
    
  保存后执行 source /etc/profile  重新加载设置
</code></pre></div>    </div>
  </li>
  <li>测试，内部请求：curl 127.0.0.1,外部可以使用配置的域名，如果访问不了需要开放80端口。
    <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  firewall-cmd --zone=public --add-port=80/tcp --permanent
  firewall-cmd --reload
</code></pre></div>    </div>
  </li>
  <li>重新加载文件,如果修改了文件 可以重新加载文件。  <br/>
  <code class="highlighter-rouge">/usr/local/openresty/nginx/sbin/nginx -c /usr/local/openresty/.../conf/nginx.conf -p /usr/local/.../ -s reload</code></li>
</ul>

<h2 id="结束">结束</h2>