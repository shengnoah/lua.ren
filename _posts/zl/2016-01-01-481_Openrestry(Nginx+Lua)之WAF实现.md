---
layout: post
title: Openrestry(Nginx+Lua)之WAF实现 
tags: [lua文章]
categories: [topic]
---
<div class="page-image">
      <div class="cover-image" style="background: url(/assets/img/post-9.jpg) center no-repeat; background-size: cover;"></div>
    </div>
    <div class="wrapper">
      <div class="page-content">
        <div class="header-page">
          
          <div class="page-date">
		          <span id="busuanzi_container_page_pv"> | 本文阅读量：<span id="busuanzi_value_page_pv"></span>次</span>
		  </div>
        </div>
        <h1 id="openrestry简介">openrestry简介:</h1>

<ul>
  <li>
    <p>OpenResty®是一个基于Nginx与Lua的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。</p>
  </li>
  <li>
    <p>OpenResty®通过汇聚各种设计精良的 Nginx 模块（主要由 OpenResty 团队自主开发），从而将 Nginx 有效地变成一个强大的通用 Web 应用平台。这样，Web 开发人员和系统工程师可以使用 Lua 脚本语言调动 Nginx 支持的各种 C 以及 Lua 模块，快速构造出足以胜任 10K 乃至 1000K 以上单机并发连接的高性能 Web 应用系统。</p>
  </li>
  <li>
    <p>OpenResty®的目标是让你的Web服务直接跑在 Nginx 服务内部，充分利用 Nginx 的非阻塞 I/O 模型，不仅仅对 HTTP 客户端请求,甚至于对远程后端诸如 MySQL、PostgreSQL、Memcached 以及 Redis 等都进行一致的高性能响应。</p>
  </li>
</ul>

<h2 id="openrestry安装部署">openrestry安装部署:</h2>

<ul>
  <li>部署文档参考资料</li>
</ul>

<blockquote>
  <p>https://openresty.org/cn/installation.html
https://openresty.org/cn/linux-packages.html</p>
</blockquote>

<blockquote>
  <p>我是在CentOS发行版下通过yum方式安装的，图个简单，毕竟openrestry的部署不是我们的重点。</p>
</blockquote>

<h2 id="openrestry的waf功能列表">openrestry的WAF功能列表:</h2>

<blockquote>
  <ol>
    <li>支持IP白名单和黑名单功能，直接将黑名单的IP访问拒绝。</li>
    <li>支持URL白名单，将不需要过滤的URL进行定义。</li>
    <li>支持User-Agent的过滤，匹配自定义规则中的条目，然后进行处理（返回403）。</li>
    <li>支持CC攻击防护，单个URL指定时间的访问次数，超过设定值，直接返回403。</li>
    <li>支持Cookie过滤，匹配自定义规则中的条目，然后进行处理（返回403）。</li>
    <li>支持URL过滤，匹配自定义规则中的条目，如果用户请求的URL包含这些，返回403。</li>
    <li>支持URL参数过滤，原理同上。</li>
    <li>支持日志记录，将所有拒绝的操作，记录到日志中去。</li>
    <li>日志记录为JSON格式，便于日志分析，例如使用ELKStack进行攻击日志收集、存储、搜索和展示。</li>
  </ol>
</blockquote>

<h4 id="waf实现基本原理">WAF实现基本原理:</h4>

<blockquote>
  <p>WAF实现 WAF一句话描述，就是解析HTTP请求（协议解析模块），规则检测（规则模块），做不同的防御动作（动作模块），并将防御过程（日志模块）记录下来。所以本文中的WAF的实现由五个模块(配置模块、协议解析模块、规则模块、动作模块、错误处理模块）组成。</p>
</blockquote>

<h2 id="openrestry启用waf配置">openrestry启用WAF配置:</h2>

<p><em style="color: red">参考资料，感谢这两个项目人员的贡献</em></p>

<blockquote>
  <p>https://github.com/loveshell/ngx_lua_waf
https://github.com/unixhot/waf</p>
</blockquote>

<ul>
  <li>获取WAF实现的lua代码，并放到openrestry配置文件存放路径:</li>
</ul>

<blockquote>
  <p>PS: yum方式安装openrestry后，程序会在部署在:<em style="color: red">/usr/local/openresty</em>路径。</p>
</blockquote>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c">#git clone https://github.com/unixhot/waf.git</span>
<span class="c">#cp -a ./waf/waf /usr/local/openresty/nginx/conf/</span>
</code></pre></div></div>

<ul>
  <li>修改Nginx的配置文件，加入以下配置。在nginx.conf的<em style="color: red">http</em>段添加:</li>
</ul>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c">#WAF</span>
lua_shared_dict limit 50m<span class="p">;</span>
lua_package_path <span class="s2">&#34;/usr/local/openresty/nginx/conf/waf/?.lua&#34;</span><span class="p">;</span>
init_by_lua_file <span class="s2">&#34;/usr/local/openresty/nginx/conf/waf/init.lua&#34;</span><span class="p">;</span>
access_by_lua_file <span class="s2">&#34;/usr/local/openresty/nginx/conf/waf/access.lua&#34;</span><span class="p">;</span>
</code></pre></div></div>

<ul>
  <li>根据需要修改WAF配置参数:</li>
</ul>

<blockquote>
  <p>一般需要修改WAF日志默认存放路径/tmp/,其他配置项参数的修改参考注解即可，被拦截后的提醒页面内容根据需要自行定义：</p>
</blockquote>

<p><img src="https://lichi6174.github.io//assets/img/openrestry-waf.jpg" alt="openrestry-waf"/></p>

<ul>
  <li>验证配置和启动openrestry服务：</li>
</ul>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c">#/usr/local/openresty/nginx/sbin/nginx –t</span>
<span class="c">#/usr/local/openresty/nginx/sbin/nginx</span>
</code></pre></div></div>

<h2 id="nginx配置参考">nginx配置参考:</h2>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>worker_processes  4<span class="p">;</span>
events <span class="o">{</span>
    worker_connections  1024<span class="p">;</span>
<span class="o">}</span>
http <span class="o">{</span>
    include       mime.types<span class="p">;</span>
    default_type  application/octet-stream<span class="p">;</span>
    log_format  main  <span class="s1">&#39;$remote_addr - $remote_user [$time_local] &#34;$request&#34; &#39;</span>
                      <span class="s1">&#39;$status $body_bytes_sent &#34;$http_referer&#34; &#39;</span>
                      <span class="s1">&#39;&#34;$http_user_agent&#34; &#34;$http_x_forwarded_for&#34;&#39;</span><span class="p">;</span>
    access_log  logs/access.log  main<span class="p">;</span>
    sendfile        on<span class="p">;</span>
    keepalive_timeout  65<span class="p">;</span>
    lua_shared_dict limit 50m<span class="p">;</span>
    lua_package_path <span class="s2">&#34;/usr/local/openresty/nginx/conf/waf/?.lua&#34;</span><span class="p">;</span>
    init_by_lua_file <span class="s2">&#34;/usr/local/openresty/nginx/conf/waf/init.lua&#34;</span><span class="p">;</span>
    access_by_lua_file <span class="s2">&#34;/usr/local/openresty/nginx/conf/waf/access.lua&#34;</span><span class="p">;</span>
    server <span class="o">{</span>
        listen       80<span class="p">;</span>
        server_name  localhost<span class="p">;</span>
        location / <span class="o">{</span>
            root   html<span class="p">;</span>
            index  index.html index.htm<span class="p">;</span>
        <span class="o">}</span>
        error_page   500 502 503 504  /50x.html<span class="p">;</span>
        location <span class="o">=</span> /50x.html <span class="o">{</span>
            root   html<span class="p">;</span>
        <span class="o">}</span>
    <span class="o">}</span>
include vhosts/<span class="k">*</span>.conf<span class="p">;</span>
<span class="o">}</span>
</code></pre></div></div>

<h2 id="关于lua脚本获取客户端真实ip方法调整">关于lua脚本获取客户端真实IP方法调整</h2>
<blockquote>
  <p>原代码获取客户端真实IP，如果经过多个代理节点传过来的X_Forwarded_For的IP值不止一个的时候会有问题。</p>
</blockquote>

<h3 id="1-此功能函数定义的脚本位置">1. 此功能函数定义的脚本位置：</h3>
<blockquote>
  <p>/usr/local/openresty/nginx/conf/waf/lib.lua</p>
</blockquote>

<h3 id="2-此功能函数参考">2. 此功能函数参考：</h3>
<h4 id="方法一">方法一：</h4>
<ul>
  <li>原代码：</li>
</ul>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nt">--Get</span> the client IP
<span class="k">function </span>get_client_ip<span class="o">()</span>
    CLIENT_IP <span class="o">=</span> ngx.req.get_headers<span class="o">()[</span><span class="s2">&#34;X_real_ip&#34;</span><span class="o">]</span>
    <span class="k">if </span>CLIENT_IP <span class="o">==</span> nil <span class="k">then
        </span>CLIENT_IP <span class="o">=</span> ngx.req.get_headers<span class="o">()[</span><span class="s2">&#34;X_Forwarded_For&#34;</span><span class="o">]</span>
    end
    <span class="k">if </span>CLIENT_IP <span class="o">==</span> nil <span class="k">then
        </span>CLIENT_IP  <span class="o">=</span> ngx.var.remote_addr
    end
    <span class="k">if </span>CLIENT_IP <span class="o">==</span> nil <span class="k">then
        </span>CLIENT_IP  <span class="o">=</span> <span class="s2">&#34;unknown&#34;</span>
    end
    <span class="k">return </span>CLIENT_IP
end
</code></pre></div></div>

<h4 id="方法二">方法二：</h4>
<blockquote>
  <p>部分客户端访问经过多个代理节点之后，X_Forwarded_For获得的IP地址可能不止一个，我们只取第一个ip地址即为客户端真实IP地址，比如如下日志记录：</p>
</blockquote>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>100.116.224.220 - - <span class="o">[</span>29/Nov/2018:09:14:37 +0800-1543454077.462] <span class="s2">&#34;POST /api/operation/abc HTTP/1.1&#34;</span> 200 873 <span class="s2">&#34;https://abc.com/2018031602388641/0.2.1811161506.32/index.html&#34;</span> <span class="s2">&#34;Mozilla/5.0 (Linux; U; Android 9; zh-CN; ALP-AL00 Build/HUAWEIALP-AL00)&#34;</span> <span class="s2">&#34;223.104.64.51, 11.34.31.180, 110.75.242.180&#34;</span>
</code></pre></div></div>

<ul>
  <li>调整后代码(经验证有效)：</li>
</ul>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nt">--Get</span> the client IP
<span class="k">function </span>get_client_ip<span class="o">()</span>
    CLIENT_IP <span class="o">=</span> ngx.req.get_headers<span class="o">()[</span><span class="s2">&#34;X_real_ip&#34;</span><span class="o">]</span>
    <span class="k">if </span>CLIENT_IP <span class="o">==</span> nil <span class="k">then
        if </span>ngx.var.http_x_forwarded_for ~<span class="o">=</span> nil <span class="k">then
        </span>CLIENT_IP <span class="o">=</span> string.match<span class="o">(</span>ngx.var.http_x_forwarded_for, <span class="s2">&#34;%d+.%d+.%d+.%d+&#34;</span>, 1<span class="o">)</span><span class="p">;</span>
        end
    end
    <span class="k">if </span>CLIENT_IP <span class="o">==</span> nil <span class="k">then
        </span>CLIENT_IP  <span class="o">=</span> ngx.var.remote_addr or <span class="s1">&#39;127.0.0.1&#39;</span>
    end
    <span class="k">if </span>CLIENT_IP <span class="o">==</span> nil <span class="k">then
        </span>CLIENT_IP  <span class="o">=</span> <span class="s2">&#34;unknown&#34;</span>
    end
    <span class="k">return </span>CLIENT_IP
end
</code></pre></div></div>

<h4 id="方法三">方法三：</h4>
<ul>
  <li>调整后代码（验证无效）</li>
</ul>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nt">--Get</span> the client IP
<span class="k">function </span>get_client_ip<span class="o">()</span>
    HEADERS <span class="o">=</span> ngx.req.get_headers<span class="o">()</span>
    CLIENT_IP <span class="o">=</span> HEADERS[<span class="s2">&#34;X_real_ip&#34;</span><span class="o">]</span>
    <span class="k">if </span>CLIENT_IP <span class="o">==</span> nil <span class="k">then
       if </span><span class="nb">type</span><span class="o">(</span>HEADERS[<span class="s2">&#34;x-forwarded-for&#34;</span><span class="o">])</span> <span class="o">==</span> <span class="s2">&#34;table&#34;</span> <span class="k">then
           </span>CLIENT_IP <span class="o">=</span> HEADERS[<span class="s2">&#34;x-forwarded-for&#34;</span><span class="o">][</span>1]
       <span class="k">else
           </span>CLIENT_IP <span class="o">=</span> HEADERS[<span class="s2">&#34;x-forwarded-for&#34;</span><span class="o">]</span>
       end
    end
    <span class="k">if </span>CLIENT_IP <span class="o">==</span> nil <span class="k">then
        </span>CLIENT_IP  <span class="o">=</span> ngx.var.remote_addr or <span class="s1">&#39;127.0.0.1&#39;</span>
    end
    <span class="k">if </span>CLIENT_IP <span class="o">==</span> nil <span class="k">then
        </span>CLIENT_IP  <span class="o">=</span> <span class="s2">&#34;unknown&#34;</span>
    end
    <span class="k">return </span>CLIENT_IP
end
</code></pre></div></div>

        <div class="page-footer">
          <div class="page-tag">
            <span>Tags:</span>
            
            <a href="/tags#Blog" class="tag">| Blog</a>
            
            <a href="/tags#security" class="tag">| security</a>
            
            <a href="/tags#web" class="tag">| web</a>
            
            <a href="/tags#openrestry" class="tag">| openrestry</a>
            
          </div>

          
          <div id="gitmentContainer"></div>
            <link rel="stylesheet" href="https://jjeejj.github.io/css/gitment.css"/>
            <script src="https://jjeejj.github.io/js/gitment.js"></script>
            <script>
              var gitment = new Gitment({
              owner: 'lichi6174',
              repo: 'lichi6174.github.io',
              oauth: {
                client_id: '6a7d368a50aba47c2074',
                client_secret: '21d9d8a3ffba794b0dbda3e2f3276aa9ddd98ec5',
              },
              });
              gitment.render('gitmentContainer');
            </script>

          
		      <section>
			

            <div class="content-play">
              <p><a href="javascript:void(0)" onclick="dashangToggle()" class="dashang" title="打赏，支持一下">打赏一下呗</a></p>
              <div class="hide_box-play"></div>
              <div class="shang_box-play">
                <a class="shang_close-play" href="javascript:void(0)" onclick="dashangToggle()" title="关闭"><img src="https://lichi6174.github.io//assets/img/close.jpg" alt="取消"/></a>
                <div class="shang_tit-play">
                  <p>对你有帮助，那就打赏一下吧</p>
                </div>
				
				<div class="shang_payimg">    
                    <img src="https://lichi6174.github.io//assets/img/wechatpayimg.png" alt="扫码支持" title="扫一扫"/>
                </div>
				
				 <div class="pay_item" data-id="weipay">
                    <span class="pay_logo"><img src="https://lichi6174.github.io//assets/img/wechat.jpg" alt="微信"/></span>
                  </div>
				
                <div class="shang_payimg">
                    <img src="https://lichi6174.github.io//assets/img/alipayimg.jpg" alt="扫码支持" title="扫一扫"/>
                </div>
				
				 <div class="pay_item checked" data-id="alipay">
                    <span class="pay_logo"><img src="https://lichi6174.github.io//assets/img/alipay.jpg" alt="支付宝"/></span>
                  </div>
				
                <div class="pay_explain">扫码打赏,金额随意</div>
               
              </div>
            </div>
            <script type="text/javascript">
            function dashangToggle(){
              $(".hide_box-play").fadeToggle();
              $(".shang_box-play").fadeToggle();
            }
            </script>

            <div style="text-align:center;margin:50px 0; font:normal 14px/24px &#39;MicroSoft YaHei&#39;;"></div>

            <style type="text/css">
              .content-play{width:100%;margin-top: 20px;margin-bottom: 10px;height:40px;text-align:center;}
              .hide_box-play{z-index:999;filter:alpha(opacity=50);background:#666;opacity: 0.5;-moz-opacity: 0.5;left:0;top:0;height:99%;width:100%;position:fixed;display:none;}
              .shang_box-play{width:100%;height:540px;padding:10px;background-color:#fff;border-radius:10px;position:fixed;z-index:1000;left:0%;top:50%;margin-left:0px;margin-top:-280px;border:1px dotted #dedede;display:none;}
              .shang_box-play img{border:none;border-width:0;}
              .dashang{width:100px;margin:5px auto;height:25px;line-height:25px;padding:10px;background-color:#E74851;color:#fff;text-align:center;text-decoration:none;border-radius:10px;font-weight:bold;font-size:16px;transition: all 0.3s;}
              .dashang:hover{opacity:0.8;padding:15px;font-size:18px;}
              .shang_close-play{float:right;display:inline-block;
                margin-right: 10px;margin-top: 20px;
              }
              .shang_logo{display:block;text-align:center;margin:20px auto;}
              .shang_tit-play{width: 100%;height: 75px;text-align: center;line-height: 66px;color: #a3a3a3;font-size: 16px;background: url('/images/payimg/cy-reward-title-bg.jpg');font-family: 'Microsoft YaHei';margin-top: 7px;margin-right:2px;}
              .shang_tit-play p{color:#a3a3a3;text-align:center;font-size:16px;}
              .shang_payimg{padding:10px;/*border:6px solid #EA5F00;**/margin:0 auto;border-radius:3px;height:140px;display:inline-block;}
              .shang_payimg img{display:inline-block;margin-right:10px;float:left;text-align:center;width:140px;height:140px; }
              .pay_explain{text-align:center;margin:10px auto;font-size:12px;color:#545454;}
              .shang_payselect{text-align:center;margin:0 auto;margin-top:40px;cursor:pointer;height:60px;width:500px;margin-left:110px;}
              .shang_payselect .pay_item{display:inline-block;margin-right:140px;float:left;}
              .shang_info-play{clear:both;}
              .shang_info-play p,.shang_info-play a{color:#C3C3C3;text-align:center;font-size:12px;text-decoration:none;line-height:2em;}
            </style>

       
</section>

        </div>
		
        
         

      </div>
    </div>