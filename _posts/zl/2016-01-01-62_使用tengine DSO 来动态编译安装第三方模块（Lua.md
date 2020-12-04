---
layout: post
title: 使用tengine DSO 来动态编译安装第三方模块（Lua 
tags: [lua文章]
categories: [topic]
---
<p>在使用nginx时，需要增加或者改动一个模块，都需要重新编译nginx文件。最近使用了tengine，它提供的DSO工具来动态加载模块。本例测试需要动态增加lua-nginx模块，记录一下安装过程。</p>




<h2>1. 编译安装 luajit</h2>

<p>Lua 环境推荐使用成熟快速的 luajit。</p>

<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
</pre></td><td class="code"><pre><code class=""><span class="line">cd /root/Download/
</span><span class="line">wget http://luajit.org/download/LuaJIT-2.0.4.tar.gz
</span><span class="line">tar zxvf LuaJIT-2.0.4.tar.gz
</span><span class="line">cd /root/Downloads/LuaJIT-2.0.4
</span><span class="line">make &amp;&amp; make install PREFIX=/usr/local/luajit-2.0.4
</span></code></pre></td></tr></tbody></table></div></figure>


<h2>2. 设置环境变量：</h2>

<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
</pre></td><td class="code"><pre><code class=""><span class="line"># tell nginx&#39;s build system where to find LuaJIT 2.0:
</span><span class="line">export LUAJIT_LIB=/usr/local/luajit-2.0.4/lib
</span><span class="line">export LUAJIT_INC=/usr/local/luajit-2.0.4/include/luajit-2.0</span></code></pre></td></tr></tbody></table></div></figure>


<h2>3. 你能够使用dso_tool(在Nginx安装目录的sbin下)这个工具来编译第三方模块 lua-nginx-module.</h2>

<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
</pre></td><td class="code"><pre><code class=""><span class="line">cd /root/Download/
</span><span class="line">wget --no-check-certificate https://github.com/openresty/lua-nginx-module/archive/v0.9.19.tar.gz
</span><span class="line">tar zxvf v0.9.19.tar.gz 
</span><span class="line">cd /usr/local/tengine/sbin/
</span><span class="line">./dso_tool --add-module=/root/Downloads/LuaJIT-2.0.4</span></code></pre></td></tr></tbody></table></div></figure>


<h2>4. 编译成功时，如下图</h2>

<p><img src="https://swmwlm.github.io//images/tengine/tengine-2.png"/></p>

<h2>5. 设置动态库</h2>

<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td><td class="code"><pre><code class=""><span class="line">echo &#39;/usr/local/luajit-2.0.4/lib&#39;&gt;&gt;/etc/ld.so.conf.d/usr_local_luajit_lib.conf 
</span><span class="line">ldconfig</span></code></pre></td></tr></tbody></table></div></figure>


<h2>6. 配置tengine的配置文件，动态加载模块，如下图所示：</h2>

<p><img src="https://swmwlm.github.io//images/tengine/tengine-1.png"/></p>

<p>使用./nginx -t 测试配置文件是否通过。提示成功，则打开浏览器访问/lua路径，测试第三方模块lua-nginx-module是否正常。</p>

<h2>7. 全新编译安装Tengine+Lua</h2>

<h3>7.1 编译安装nginx需要的依赖包</h3>

<pre><code>yum install gcc gcc-c++ make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl openssl-devel
</code></pre>

<h3>7.2 编译安装Lua需要的依赖包（centos默认可能装过了，可省略，rpm -qa|grep readline查询）</h3>

<pre><code>yum install readline readline-devel

或者手动编译安装

wget -c ftp://ftp.gnu.org/gnu/readline/readline-6.2.tar.gz 
tar -zxvf readline-6.2.tar.gz 
cd readline-6.2
./configure
make &amp;&amp; make installl

手动编译后如果指定了 prefix ，要修改一下 lua 的编译脚本
修改lua src/Makefile
修改MYCFLAGS=-I/usr/local/readline/include
修改MYLDFLAGS=-L/usr/local/readline/lib
直接执行lua命令，出现加载不了readline.so的问题，软链接 /usr/local/readline/lib/libreadline.so.6 到 /usr/lib/
</code></pre>

<h3>7.3 安装lua</h3>

<pre><code>yum安装:
yum install lua lua-devel

安装Lua或者LuaJIT都是可以的，但是出于效率的考虑，推荐安装LuaJIT。源码安装LuaGIT如下：
LuaGIT官方地址: http://luajit.org/
</code></pre>

<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
</pre></td><td class="code"><pre><code class=""><span class="line">wget http://luajit.org/download/LuaJIT-2.0.4.tar.gz
</span><span class="line">cd /usr/local/src 
</span><span class="line">tar -zxvf LuaJIT-2.0.4.tar.gz 
</span><span class="line">cd LuaJIT-2.0.4
</span><span class="line">make
</span><span class="line">make install                                                       (也可以使用make install PREFIX=/usr/local/luagit-2.0.4)
</span><span class="line">ln -s /usr/local/lib/libluajit-5.1.so.2 /lib64/libluajit-5.1.so.2    (ln -s /usr/local/luagit-2.0.4/lib/libluajit-5.1.so.2 /lib64/libluajit-5.1.so.2)
</span><span class="line">export LUAJIT_LIB=/usr/local/lib                                       (export LUAJIT_LIB=/usr/local/luagit-2.0.4/lib)
</span><span class="line">export LUAJIT_INC=/usr/local/include/luajit-2.0/                   (export LUAJIT_INC=/usr/local/luagit-2.0.4/include/luajit-2.0/)</span></code></pre></td></tr></tbody></table></div></figure>


<pre><code>因为安装在缺省路径，所以LuaJIT对应的lib，include均在/usr/local目录里。
</code></pre>

<blockquote><p><a href="http://huoding.com/2012/08/31/156">请参考博文</a></p></blockquote>

<p>创建www用户和组:</p>

<pre><code>sudo groupadd www
sudo useradd www -g www -s /sbin/nologin -M 
</code></pre>

<p>另外，可以安装jemalloc，优化内存管理</p>

<pre><code>wget http://www.canonware.com/download/jemalloc/jemalloc-3.6.0.tar.bz2
tar xjf jemalloc-3.6.0.tar.bz2
cd jemalloc-3.6.0
./configure
make &amp;&amp; make install
echo &#39;/usr/local/lib&#39; &gt; /etc/ld.so.conf.d/local.conf
ldconfig
</code></pre>

<h4>7.3.1 使用jemalloc优化MySQL</h4>

<pre><code>方法1.

MySQL/MaridDB 5.5编译方法，cmake预编译时加上下面参数

-DCMAKE_EXE_LINKER_FLAGS=&#34;-ljemalloc&#34; -DWITH_SAFEMALLOC=OFF

方法2.

修改mysqld_safe直接加载:

查找文件 /usr/local/mysql/bin/mysqld_safe
在#executing mysqld_safe 下面加上

LD_PRELOAD=/usr/local/lib/libjemalloc.so

重新启动MYSQL

使用下面代码自动修改mysqld_safe文件

sed -i &#39;s@executing mysqld_safe@executing mysqld_safenexport LD_PRELOAD=/usr/local/lib/libjemalloc.so@&#39; /usr/local/mysql/bin/mysqld_safe
service mysqld restart
</code></pre>

<h4>7.3.2 使用jemalloc优化Nginx</h4>

<pre><code>编译NGINX时添加以下参数:
--with-ld-opt=&#34;-ljemalloc&#34;

具体实现：

cd lnmp/src/nginx-1.4.2
make clean
./configure --prefix=/usr/local/nginx --user=www --group=www   
--with-http_stub_status_module --with-http_ssl_module --with-http_flv_module  
--with-http_gzip_static_module --with-ld-opt=&#34;-ljemalloc&#34; (或者 --with-jemalloc)
make &amp;&amp; make install
</code></pre>

<p>验证 jemalloc 是否运行:</p>

<pre><code>lsof -n | grep jemalloc
</code></pre>

<h3>7.4 编译Tengine</h3>

<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
</pre></td><td class="code"><pre><code class=""><span class="line">shell&gt; wget http://tengine.taobao.org/download/tengine-2.1.2.tar.gz
</span><span class="line">shell&gt; tar zxvf tengine-2.1.2.tar.gz
</span><span class="line">shell&gt; cd tengine-2.1.2
</span><span class="line">shell&gt; ./configure --prefix=/usr/local/tengine-2.1.2 --with-http_lua_module --with-luajit-lib=/usr/local/luagit-2.0.4/lib/ --with-luajit-inc=/usr/local/luagit-2.0.4/include/luajit-2.0/ --pid-path=/var/run/nginx.pid --group=www --user=www --dso-path=/usr/local/tengine-2.1.2/modules --with-http_gzip_static_module --with-http_stub_status_module --with-poll_module --with-http_sub_module --with-http_realip_module --with-http_concat_module --http-proxy-temp-path=/var/tmp/tengine-2.1.2/proxy_temp --http-fastcgi-temp-path=/var/tmp/tengine-2.1.2/fastcgi_temp --http-uwsgi-temp-path=/var/tmp/tengine-2.1.2/uwsgi_temp --http-scgi-temp-path=/var/tmp/tengine-2.1.2/cgi_temp --http-client-body-temp-path=/var/tmp/tengine-2.1.2/client_body_temp --http-log-path=/var/log/tengine-2.1.2/access.log --error-log-path=/var/log/tengine-2.1.2/error.log --with-ld-opt=&#34;-Wl,-rpath,$LUAJIT_LIB,-ljemalloc&#34;
</span><span class="line">
</span><span class="line">shell&gt; make&amp;&amp; make install
</span><span class="line">
</span><span class="line">shell&gt; ln -s /usr/local/tengine-2.1.2 /usr/local/nginx （创建软连接，可省略）
</span><span class="line">shell&gt; mkdir /var/tmp/tengine-2.1.2 -p           （创建所需的目录以及赋权）
</span><span class="line">shell&gt; chown -R www:www /var/tmp/tengine-2.1.2
</span><span class="line">
</span><span class="line">
</span></code></pre></td></tr></tbody></table></div></figure>


<pre><code>参数说明：由于Pcre是通过yum安装的，不用使用--with-pcre=/path/to/ 指定。
lua是手动安装的需要配置参数
--with-http_lua_module 
--with-luajit-lib=/usr/local/luagit-2.0.4/lib/  --with-luajit-inc=/usr/local/luagit-2.0.4/include/luajit-2.0/ 

--prefix安装的目录

作用：不指定prefix，则可执行文件默认放在/usr/local/bin,库文件默认在/usr/local/lib,配置文件在/usr/local/etc,其他资源文件放在/usr/local/share。若要卸载这个程序，要么在原来的make目录下执行make uninstall(前提是make文件指定过uninstall),要么去上述默认目录里把相关的文件夹一个个的删掉。但是，对于指定了prefix的目录的好处是卸载软件或者移植软件。当某个软件不再需要时，只须简单删除该文件安装目录，就可以把软件卸载干净；移植软件只需要拷贝整个目录到另外一个机器即可（相同的操作系统及环境依赖）。

--with-ld-opt

其中的 --with-ld-opt=-Wl,-rpath,/usr/local/lib:/opt/openresty/luajit/lib 意思是：这是链接器选项，目的是把/usr/loca/lib和/opt/openresty/luajit/lib这两个路径添加进nginx可执行文件的RPATH头重，这样在启动的时候，系统就可以找到正确的动态链接库文件。

若不使用该参数指定动态的链接库，试着使用 ./nginx 启动tengine,如果运气不好的话，可能遇到如下错误：

cannot open shared object file: No such file or directory

这是什么情况呢？可以使用ldd命令来查看：
ldd /usr/local/tengine-2.1.0/sbin/nginx
会出现xxxx-xxxx.so =&gt; not found

此类问题通常使用ldconfig命令就能解决，例如本例有可能出现的
libluajit-&lt;VERSION&gt;.so =&gt; not found

我们使用如下命令：（由于我们默认安装libluagit动态链接库默认安装至/usr/local/lib）
</code></pre>

<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td><td class="code"><pre><code class=""><span class="line">shell&gt; echo &#34;/usr/local/lib&#34; &gt; /etc/ld.so.conf.d/usr_local_lib.conf
</span><span class="line">shell&gt; ldconfig</span></code></pre></td></tr></tbody></table></div></figure>


<pre><code>再次启动nginx，应该就OK了。
</code></pre>

<h3>7.5 测试lua是否可以使用了</h3>

<p>配置下面代码至nginx.conf的server 中，并重新加载配置文件（./nginx -s reload），</p>

<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
</pre></td><td class="code"><pre><code class=""><span class="line">location /lua {
</span><span class="line">    set $test &#34;hello, world.&#34;;
</span><span class="line">    content_by_lua &#39;
</span><span class="line">        ngx.header.content_type = &#34;text/plain&#34;;
</span><span class="line">        ngx.say(ngx.var.test);
</span><span class="line">    &#39;;
</span><span class="line">}</span></code></pre></td></tr></tbody></table></div></figure>