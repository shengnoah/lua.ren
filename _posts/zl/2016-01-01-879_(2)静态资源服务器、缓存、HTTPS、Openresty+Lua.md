---
layout: post
title: (2)静态资源服务器、缓存、HTTPS、Openresty+Lua 
tags: [lua文章]
categories: [topic]
---
<p>1.alias 与 root</p>
<p>访问固定目录的做法</p>
<figure class="highlight plain"><table><tbody><tr><td class="code"><pre><span class="line">Location / {</span><br/><span class="line">	alias dlib/;</span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>
<p>root 的问题：会把url 的路径带到文件目录中来，所以通常使用alias</p>
<p>2.gzip</p>
<p>gzip_min_length 1;表示小于1字节的就不再压缩了。因为有的文件小，一个TCP报文就发出去了。这时候如果还做gzip压缩，消耗CPU。意义不大了。</p>
<p>3.auto_index</p>
<p>应用场景：假定我们把dlib文件下的某个文件分享给用户，让用户决定用哪些文件。</p>
<p>当我们访问反斜线结尾的目录的时候。对应到指定目录，显示目录的结构。</p>
<figure class="highlight plain"><table><tbody><tr><td class="code"><pre><span class="line">location / {</span><br/><span class="line">	alias dlib/;</span><br/><span class="line">	auto_index on:</span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>
<p>共享静态资源的一种手段。</p>
<p>4.限制访问速度<br/>比如访问大文件的时候，我们限制流出速度。</p>
<figure class="highlight plain"><table><tbody><tr><td class="code"><pre><span class="line">location / {</span><br/><span class="line">	alias dlib/;</span><br/><span class="line">	auto_index on:</span><br/><span class="line">	set $limit_rate 1k; #限制向浏览器发送的速度。每秒1k字节。</span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>
<p>$limit_rate 内置变量，限制访问速度。</p>
<p>5.记录access日志</p>
<p>log_format 命名的场景</p>
<p>我们可能会对不同的域名下，做不同的日志记录。<br/>或者做不同用处时（比如：对一些大文件进行反向代理。），我们记录不同的日志。</p>
<p>内置的变量都可以记录到 access_log日志中</p>
<p>6.生产环境不建议使用root</p>
<p>在静态网站访问的时候，出现了403的错误，在nginx.conf文件中添加了 user root; 也就是和当前启动用户一致，然后重新reload，再次访问正常了。</p>
<p> 作者回复<br/>如果是生产环境，不建议使用root用户哦，修改静态资源的权限与user指定的用户权限相匹配更好。</p>
<p>二，反向代理服务</p>
<ol>
<li>server{…listen 127.0.0.1:8000….}</li>
</ol>
<p>加上127.0.0.1 只允许本机的进程访问NGINX。</p>
<ul>
<li>代理服务器配置 upstream 模块</li>
</ul>
<figure class="highlight plain"><table><tbody><tr><td class="code"><pre><span class="line">upstream 名字 {</span><br/><span class="line">    server  服务器地址</span><br/><span class="line"> }</span><br/><span class="line"></span><br/><span class="line">Location / {</span><br/><span class="line">   proxy_pass http://你的upstream名</span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>
<p>3.proxy_set_header 指令</p>
<p>被代理的服务器无法直接拿到用户发送的header。prox_set_header相当于转发。</p>
<p>4.proxy_cache</p>
<p>配置proxy_cache</p>
<figure class="highlight plain"><table><tbody><tr><td class="code"><pre><span class="line">http {</span><br/><span class="line">    ……</span><br/><span class="line">    proxy_cache_path /tmp/nginxcache levels=1:2 keys_zone=my_cache:10m max_sie=10g inactive=60m use_temp_path=off;</span><br/><span class="line">   ……</span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>
<p>文件放在哪个目录下 /tmp/nginxcache<br/>这些文件的命名方式 levels=1:2<br/>文件的关键字要放入共享内存（比如：起名叫my_cache，10m就是10MB）</p>
<p>使用proxy_cache</p>
<p>放在你想要缓存的url下。</p>
<figure class="highlight plain"><table><tbody><tr><td class="code"><pre><span class="line">locaction / {</span><br/><span class="line">    ……</span><br/><span class="line">    proxy_cache my_cace;#my_cace 开辟的10MB的共享内存</span><br/><span class="line">    proxy_cache_key $host$uri$is_args$args;#缓存key</span><br/><span class="line">    proxy_cache_valid 200 304 302 1d; #哪些响应不返回</span><br/><span class="line">   ……</span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>
<p>测试缓存两种方法</p>
<ul>
<li>请求一次，停掉提供服务的机器，再请求看页面是否缓存</li>
<li>看http的header头</li>
</ul>
<p>5.一个解决问题的思路：二分法，逐渐缩小问题范围</p>
<p>6.选nginx还是openresty</p>
<p>如果不需要使用openresty 提供的独有功能，尽量使用更稳定更轻量的nginx。</p>
<p>7.如果想启用gzip压缩，是应该在反向代理上做，还是web服务器上的nginx上配置？</p>
<p> 作者回复<br/>我猜测你的环境是nginx-&gt;nginx，前者是负载均衡的作用，后者是静态资源的作用？如果是这样，建议后者。</p>
<p>8.$host和$http_host区别</p>
<p>用反向代理一些有跳转的页面的时候会出现CSS样式加载不出来 然后我使用’$http_host就可以加载出来</p>
<p> 作者回复<br/>这两个变量的生成方式不同：http_host只会从请求头部中的Host: xxx中取值，而host有三种取值方式：1、先从请求行中取，比如<a href="http://xxx/index.htm；2、如果1取不到，再从Host头部取；3、如果2也取不到，从配置里的server_name里取。你应该根据其含义也选择使用" target="_blank" rel="noopener noreferrer">http://xxx/index.htm；2、如果1取不到，再从Host头部取；3、如果2也取不到，从配置里的server_name里取。你应该根据其含义也选择使用</a></p>
<p>三、日志处理</p>
<ol>
<li>GoAccess 使用</li>
</ol>
<p>goaccess /var/log/nginx/access.log -a -o /etc/nginx/log/report.html –log-format=COMBINED </p>
<p>注意点：对nginx很多变量的输出goaccess是不能解析的</p>
<p>使用文档：<a href="https://goaccess.io/get-started" target="_blank" rel="noopener noreferrer">https://goaccess.io/get-started</a></p>
<p>2.集中日志到一台服务器，用syslog协议。</p>
<p>3.openresty的一致性hash，不均匀，你怎么看，有没有办法修改算法或者虚拟节点的多少</p>
<p> 作者回复<br/>nginx的第三方模块代码都能修改，而openresty的lua代码改起来成本更小，例如你说的应该是chash.lua这个文件。 问题是，你打算用一套新的算法来改么？原来是openresty的consistent hash算法，与nginx官方的consistent hash算法，都是使用了memcached中的ketama实现思想的，是经过验证的。</p>
<p>4.如果做了每天的日志切割，再使用GoAccess 是不是就意味着只能看到当天的统计结果？</p>
<p>不会的，GoAccess会在内存中缓存运行以来的日志分析结果</p>
<ol>
<li>在docker swarm 中，跨主机，多节点的nginx日志收集，聚合，展示，怎么用goaccess进行实现？</li>
</ol>
<p> 作者回复<br/>两个比较简单的方案：</p>
<ul>
<li>1、用syslog把日志汇聚在一台服务上，再goaccess。</li>
<li>2、用NFS把多台主机的日志目录映射在一起，用goaccess再分析。</li>
</ul>
<p>四，安全套接字层</p>
<p>1.SSL</p>
<p>{F8900}</p>
<p>{F8902}</p>
<p>2.ssl第一次握手还是明文传输，也不安全，现在提了一个hsts，老师可以讲一下这个吗？</p>
<p> 作者回复<br/>ssl第一次握手是明文传输，但它只是传输安全套件以及公钥，之后数据是用新生成的对称密钥加密过的，所以SSL是安全的。hsts主要应用在浏览器端，它是强制浏览器使用https方式，对nginx来说，只需要在返回的http头部上添加Strict-Transport-Security，告诉浏览器这个站点只能通过https访问即可。</p>
<p>3.CA</p>
<p>{F8904}</p>
<p>4.证书类型</p>
<p>{F8906}</p>
<p>5.浏览器跟服务器间通讯怎样确认对方是我信赖的人</p>
<p>归根结底验证给这个站点办法的根证书它的发行者是否是有效的。</p>
<p>6.CA是不是只提供签名过的证书，也提供秘钥吗？秘钥不是自己生成的吗？如果提供的话是什么的秘钥呢？</p>
<p> 作者回复<br/>这要根据你所使用的服务商决定哦。比如你在阿里云上购买的证书，都是由阿里云生成好公私钥对的，你需要下载公私钥证书再部署在自己的nginx上。</p>
<p>7.为了可用性我们可以通过nginx 查询OCSP获知证书是否有效。也是为了提升可用性（因为CA机构不对网站的可用性负责）。</p>
<p>{F8908}</p>
<p>8.打开session cache 可以复用密钥</p>
<p>9.性能与优化点</p>
<p>当以小文件为主，主要考验nginx的非对称加密性能。以大文件为主，考验对称加密性能。</p>
<p>当我们处理小文件比较多时，主要优化椭圆曲线算法的一些密码强度。</p>
<p>处理大文件多时，aes算法是否可以替换更有效的算法，或者调整密码强度。</p>
<p>10，非对称加密技术保证：双方生成的新密钥是一致的，这个结论是怎么得出来的呢？ 能验证下吗？</p>
<p> 作者回复<br/>你可以看下椭圆曲线加密的数学证明过程，或者看下RSA的原理验证，网上有很多，不过理解前者需要非常好的数学基础，后者比较简单，可以用来入门。</p>
<p>11.老师的tls流程是最复杂的吧，一般客户端不需要自己的证书密钥，除非银行客户端之类<br/>一般网站类，浏览器只是拿服务端公钥加密随机数发给服务端这样吧~</p>
<p> 作者回复<br/>不是的，我这里的例子是当前最主流的TLS安全套件交互流程，你用firefox、chrome等访问大多数ssl站点，交互流程都是这样的。这些复杂性，其实都被浏览器、nginx悄悄地完成了，所以我们感知不到</p>
<p>12.SSL耗时主要由握手和对称加密构成。</p>
<p>13.CA是怎么知道指定的域名是在当前的nginx下呢？或者有没有推荐的资料或者链接可以参考下呢？</p>
<p>因为域名配置的地址指向nginx所在机器，所以该nginx所在机器拥有域名所有权。中文资料目前《https权威指南》可以参考</p>
<p>14.免费的跟付费的证书有什么差异？</p>
<p>付费的https也分DV证书、OV证书和EV证书。从安全传输这个角度来说，这三种证书效果一样。从浏览器对证书的认可上来，DV证书最差。如果你买的是付费的DV证书，跟这里的例子都一样，因为主流的浏览器都认Lets encrypt。</p>
<p>五，lua 简单服务</p>
<p>1.openresty/bundles目录</p>
<p>ngx_前缀是c模块，lua_打头的是lua模块。主要编译是c模块</p>
<p>2.使用openresty连接 数据库 或 redis ，性能和稳定性怎么样？（身边几乎没有人使用它去连接数据库和缓存，不敢随便使用它！）</p>
<p> 作者回复<br/>性能高,稳定性好,值得一试.</p>
<p>3.请问：你们经常会使用openresty直连redis或mysql？在什么场景下使用呢？</p>
<p>有些接口流量很大，所以不想对上游的应用开发服务器造成影响，就会直连redis,mysql。</p>
<p>4.请问：你们会不会经常使用openresty合并http接口请求？</p>
<p>微服务架构、前后端分离技术的应用，对RESTful接口的依赖越来越多。繁多的接口，流水线式的请求，使耗费的时间越来越长。<br/>解决这种效率的问题常见的方法：</p>
<ul>
<li>1、增加前端的请求数的请求数，这种方法最大的障碍就是浏览器最大并发连接数，如果请求过多会影响整体页面的加载。<ul>
<li>2、后端代码聚合接口，将多个接口的数据聚合到一起返回给前端，这样做最大缺点是代码的耦合很高。<br/>既要做到对请求的合并又要不影响各子系统的划分，更好的方案：<br/>使用OpenResty的ngx.location.capture_multi函数来聚合多个请求，ngx.location.capture_multi对多个内部请求是非阻塞的并行执行的，相对于流水线式的请求大大提交了效率。</li>
</ul>
</li>
</ul>
<p> 作者回复<br/>我很少这么做。你可以试试http2协议(第4部分有介绍)，对大量小请求的性能提升很有效果。</p>
<p>5.openresty做API接口，api网关优势，例如：降低请求/往返次数，请问：为什么会降低请求/往返次数？（网上说是因为：API网关能够确保客户端在单次往返中就从多项服务中检索出数据。）</p>
<p> 作者回复<br/>我对这句话的理解是有这么个场景：<br/>正常的请求是nginx-&gt;应用服务-&gt;mysql，用openresty后，有些简单的业务只需要查询数据库，那么可以变为nginx-&gt;mysql，这样就降低了请求次数及往返次数。</p>
<p>6.ngx.say</p>
<p>ngx.say 会生成响应。是在http中的body中的。<br/>放在body中的文本，可以通过ngx.say添加进去。</p>