---
layout: post
title: 基于Nginx&Lua 和Netflix Eureka的微服务网关 
tags: [lua文章]
categories: [topic]
---
<p>依赖：lua-resty-http
基于Nginx&amp;Lua 和Netflix Eureka的微服务网关。</p>

<p>重新架构了内部组件，采用插件模式。</p>

<ul>
<li>服务发现

<ul>
<li>Eureka Discovery</li>
<li>抽象discovery，用来支持多种服务发现？规划中…</li>
</ul></li>
<li>动态路由</li>
<li>负载均衡

<ul>
<li>加权轮询</li>
<li>基于响应时间的动态权重轮询？开发中…</li>
</ul></li>
<li>简单监控</li>
<li>隔离降级</li>
<li>限流</li>
<li>metrics</li>
<li>认证安全？规划中。。。</li>
<li>监控页面？开发中…</li>
</ul>

<h2 id="架构图">架构图：</h2>

<p><img src="https://github.com/tietang/ngx-lua-zuul/raw/_dev_pilot/doc/arch.png" alt="img"/></p>

<h2 id="使用方法">使用方法</h2>

<p>基于Nginx和Lua module。需要<a href="https://www.jianshu.com/p/7c320140c6aa">安装Nginx Lua环境</a>或者直接下载<a href="https://link.jianshu.com/?t=http://openresty.org/en/download.html">openresty</a>编译安装。</p>

<h2 id="安装和配置ngx-lua-zuul">安装和配置ngx-lua-zuul</h2>

<h3 id="下载代码到-path-to-nginx-lua-lib">下载代码到/path/to/nginx/lua/lib/</h3>

<blockquote>
<p>git clone <a href="https://link.jianshu.com/?t=http://github.com/tietang/ngx-lua-zuul">http://github.com/tietang/ngx-lua-zuul</a> –depth=1</p>
</blockquote>

<h2 id="例子eureka-服务">例子Eureka 服务</h2>

<p>如果没有Eureka环境，也可以编译安装本例子中的EurekaDemo服务，参考<a href="https://link.jianshu.com/?t=run-eureka-demo.md">编译和运行eureka-demo服务</a>中的相关内容。</p>

<h3 id="部署dicovery例子服务">部署dicovery例子服务：</h3>

<p>下载代码后：</p>

<blockquote>
<p>cd /path/to/ngx_lua-zuul/demo/java
mvn clean install</p>
</blockquote>

<p>将下载的代码中的lua文件夹放到部署目录<code>/path/to/nginx</code>，修改<code>/path/to/nginx/lua/ngx_conf/lua.ngx_conf</code>文件中的<code>lua_package_path</code>为你的真实路径：
<code>lua_package_path &#34;/path/to/nginx/lua/lib/?.lua;;&#34;;</code></p>

<h3 id="修改-path-to-nginx-conf-nginx-conf-文件">修改<code>/path/to/nginx/conf/nginx.conf</code>文件</h3>

<p><strong>http 节点中添加</strong></p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><span class="lnt">1
</span></pre></td>
<td class="lntd">
<pre class="chroma">include &#34;/path/to/lua/ngx_conf/ngx_inlude_http.conf&#34;;</pre></td></tr></tbody></table>
</div>
</div>
<p><strong>server节点中添加</strong></p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><span class="lnt">1
</span></pre></td>
<td class="lntd">
<pre class="chroma">include &#34;/path/to/nginx/lua/ngx_conf/ngx_inlude_server.conf&#34;;</pre></td></tr></tbody></table>
</div>
</div>
<h2 id="参考配置">参考配置</h2>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><span class="lnt"> 1
</span><span class="lnt"> 2
</span><span class="lnt"> 3
</span><span class="lnt"> 4
</span><span class="lnt"> 5
</span><span class="lnt"> 6
</span><span class="lnt"> 7
</span><span class="lnt"> 8
</span><span class="lnt"> 9
</span><span class="lnt">10
</span><span class="lnt">11
</span><span class="lnt">12
</span><span class="lnt">13
</span><span class="lnt">14
</span><span class="lnt">15
</span><span class="lnt">16
</span><span class="lnt">17
</span><span class="lnt">18
</span><span class="lnt">19
</span><span class="lnt">20
</span><span class="lnt">21
</span><span class="lnt">22
</span><span class="lnt">23
</span><span class="lnt">24
</span><span class="lnt">25
</span><span class="lnt">26
</span><span class="lnt">27
</span><span class="lnt">28
</span><span class="lnt">29
</span><span class="lnt">30
</span><span class="lnt">31
</span><span class="lnt">32
</span><span class="lnt">33
</span><span class="lnt">34
</span><span class="lnt">35
</span><span class="lnt">36
</span><span class="lnt">37
</span><span class="lnt">38
</span><span class="lnt">39
</span><span class="lnt">40
</span><span class="lnt">41
</span><span class="lnt">42
</span><span class="lnt">43
</span><span class="lnt">44
</span><span class="lnt">45
</span><span class="lnt">46
</span><span class="lnt">47
</span><span class="lnt">48
</span><span class="lnt">49
</span><span class="lnt">50
</span><span class="lnt">51
</span><span class="lnt">52
</span><span class="lnt">53
</span><span class="lnt">54
</span><span class="lnt">55
</span><span class="lnt">56
</span><span class="lnt">57
</span><span class="lnt">58
</span><span class="lnt">59
</span><span class="lnt">60
</span><span class="lnt">61
</span></pre></td>
<td class="lntd">
<pre class="chroma"> #user  nobody;
worker_processes  2;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  &#39;$remote_addr - $remote_user [$time_local] &#34;$request&#34; &#39;
    #                  &#39;$status $body_bytes_sent &#34;$http_referer&#34; &#39;
    #                  &#39;&#34;$http_user_agent&#34; &#34;$http_x_forwarded_for&#34;&#39;;

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    include &#34;/Users/tietang/nginx/nginx/lua/ngx_conf/ngx_inlude_http.conf&#34;;


    server {

        include &#34;/Users/tietang/nginx/nginx/lua/ngx_conf/ngx_inlude_server.conf&#34;;
        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #

        location = / {
            set $dir $document_root;
            root   $dir/html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        
    }


   

}</pre></td></tr></tbody></table>
</div>
</div>
<h3 id="运行测试">运行测试</h3>

<p>启动所有的demo服务：discovery,api,zuul；</p>

<p>启动nginx；</p>

<p>打开浏览器：<a href="https://link.jianshu.com/?t=http://127.0.0.1:8000/api/test/0/0">http://127.0.0.1:8000/api/test/0/0</a></p>

<p>其测试api参考<a href="https://link.jianshu.com/?t=run-eureka-demo.md">编译和运行eureka-demo服务</a>中的相关内容。</p>