---
layout: post
title: 基于redis+nginx+Lua实现访问白名单黑名单 
tags: [lua文章]
categories: [topic]
---

    

    <span hidden itemprop="author" itemscope itemtype="http://schema.org/Person">
      <meta itemprop="name" content="刘三儿">
      <meta itemprop="description" content="天道酬勤 自强不息">
      <meta itemprop="image" content="/images/kid.jpg">
    </span>

    <span hidden itemprop="publisher" itemscope itemtype="http://schema.org/Organization">
      <meta itemprop="name" content="liusir.me">
    </span>

    
      
    

    
    
    
    <div class="post-body" itemprop="articleBody">

      
      

      
        
<ol>
<li>
<p>添加<code>echo-nginx-module</code>模块</p>
<ul>
<li>git clone <a href="https://github.com/openresty/echo-nginx-module.git" target="_blank" rel="noopener noreferrer">https://github.com/openresty/echo-nginx-module.git</a> /usr/local/echo-nginx-module</li>
<li>切换到nginx源码目录 cd /usr/local/src/nginx</li>
<li>重新执行 ./configure –prefix=/usr/local/nginx –add-module=/usr/local/echo-nginx-module</li>
<li>
<p>若提示错误少什么依赖就安装相应的依赖，无提示则执行 make &amp;&amp; make install</p>
</li>
<li>
<p>网上还有另一种方法,同理可以添加别的模块</p>
</li>
</ul>
<figure class="highlight plain"><table><tr>
<td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br></pre></td>
<td class="code"><pre><span class="line">git clone https://github.com/nginx/nginx.git</span><br><span class="line">git submodule add git@github.com:openresty/echo-nginx-module.git</span><br><span class="line">#重新配置nginx,把echo-nginx-module模块编译进nginx可执行文档</span><br><span class="line">sudo ./configure --add-module=echo-nginx-module</span><br></pre></td>
</tr></table></figure>
</li>
<li>
<p>安装 ngx_devel_kit (NDK) 模块</p>
<ul>
<li>cd /usr/local </li>
<li>git clone <a href="https://github.com/simpl/ngx_devel_kit.git" target="_blank" rel="noopener noreferrer">https://github.com/simpl/ngx_devel_kit.git</a>  </li>
</ul>
</li>
<li>
<p>添加<code>lua-nginx-module</code>模块</p>
<ul>
<li>cd /usr/local </li>
<li>git clone <a href="https://github.com/chaoslawful/lua-nginx-module.git" target="_blank" rel="noopener noreferrer">https://github.com/chaoslawful/lua-nginx-module.git</a>
</li>
</ul>
</li>
<li>
<p>重新编译nginx</p>
<figure class="highlight plain"><table><tr>
<td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br></pre></td>
<td class="code"><pre><span class="line">./configure --prefix=/usr/local/nginx </span><br><span class="line">            --with-ld-opt="-Wl,-rpath,$LUAJIT_LIB" </span><br><span class="line">            --add-module=/usr/local/ngx_devel_kit  </span><br><span class="line">            --add-module=/usr/local/echo-nginx-module </span><br><span class="line">            --add-module=/usr/local/lua-nginx-module </span><br><span class="line">make -j2 </span><br><span class="line">make install</span><br></pre></td>
</tr></table></figure>
</li>
<li>
<p>重启nginx服务器</p>
</li>
<li>
<p>测试Lua，在nginx.conf配置文档server代码块中加入以下代码：</p>
 <figure class="highlight plain"><table><tr>
<td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br></pre></td>
<td class="code"><pre><span class="line">location /echo { </span><br><span class="line">    default_type 'text/plain'; </span><br><span class="line">    echo 'install echo module success!'; </span><br><span class="line">} </span><br><span class="line">location /lua { </span><br><span class="line">    default_type 'text/plain'; </span><br><span class="line">    content_by_lua 'ngx.say("install lua module success!")'; </span><br><span class="line">}</span><br></pre></td>
</tr></table></figure>
</li>
<li>
<p>curl <a href="https://liusir.me/http://localhost/echo" target="_blank" rel="noopener noreferrer">http://localhost/echo</a>  // 正常的话输出：install echo module success!</p>
</li>
<li>curl <a href="https://liusir.me/http://localhost/lua" target="_blank" rel="noopener noreferrer">http://localhost/lua</a>   // 正常的话输出：install lua module success!</li>
</ol>
<h1 id="二、实现">
<a href="https://liusir.me/#%E4%BA%8C%E3%80%81%E5%AE%9E%E7%8E%B0" class="headerlink" title="二、实现"></a>二、实现</h1>
<ol>
<li>实现原理：通过在nginx上进行访问限制，通过lua来灵活实现业务需求，redis用于存储黑名单列表</li>
<li>
<p>具体过程</p>
<ul>
<li>step1:lua代码(post请求，ip地址黑名单，请求参数中imsi,tel值和黑名单)</li>
</ul>
<figure class="highlight plain"><table><tr>
<td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br><span class="line">17</span><br><span class="line">18</span><br><span class="line">19</span><br><span class="line">20</span><br><span class="line">21</span><br><span class="line">22</span><br><span class="line">23</span><br><span class="line">24</span><br><span class="line">25</span><br><span class="line">26</span><br><span class="line">27</span><br><span class="line">28</span><br><span class="line">29</span><br><span class="line">30</span><br><span class="line">31</span><br><span class="line">32</span><br></pre></td>
<td class="code"><pre><span class="line">[root@git-server ~]# cat /usr/local/nginx/conf/lua/ipblacklist.lua</span><br><span class="line">ngx.req.read_body()</span><br><span class="line"></span><br><span class="line">local redis = require "resty.redis"</span><br><span class="line">local red = redis.new()</span><br><span class="line">red.connect(red, '127.0.0.1', '6379')</span><br><span class="line"></span><br><span class="line">local myIP = ngx.req.get_headers()["X-Real-IP"]</span><br><span class="line">if myIP == nil then</span><br><span class="line">   myIP = ngx.req.get_headers()["x_forwarded_for"]</span><br><span class="line">end</span><br><span class="line">if myIP == nil then</span><br><span class="line">   myIP = ngx.var.remote_addr</span><br><span class="line">end</span><br><span class="line"></span><br><span class="line">if ngx.re.match(ngx.var.uri,"^(/webapi/).*$") then</span><br><span class="line">    local method = ngx.var.request_method</span><br><span class="line">    if method == 'POST' then</span><br><span class="line">        local args = ngx.req.get_post_args()</span><br><span class="line"></span><br><span class="line">        local hasIP = red:sismember('black.ip',myIP)</span><br><span class="line">        local hasIMSI = red:sismember('black.imsi',args.imsi)</span><br><span class="line">        local hasTEL = red:sismember('black.tel',args.tel)</span><br><span class="line">        if hasIP==1 or hasIMSI==1 or hasTEL==1 then</span><br><span class="line">            --ngx.say("This is 'Black List' request")</span><br><span class="line">            ngx.exit(ngx.HTTP_FORBIDDEN)</span><br><span class="line">          end</span><br><span class="line">        else</span><br><span class="line">            --ngx.say("This is 'POST' request")</span><br><span class="line">            ngx.exit(ngx.HTTP_FORBIDDEN)</span><br><span class="line">          end</span><br><span class="line">         end</span><br></pre></td>
</tr></table></figure>
<ul>
<li>step2:修改nginx.conf</li>
</ul>
<figure class="highlight plain"><table><tr>
<td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br></pre></td>
<td class="code"><pre><span class="line">location / {</span><br><span class="line">root html;</span><br><span class="line">index index.html index.htm;</span><br><span class="line">access_by_lua_file /usr/local/nginx/conf/lua/ipblacklist.lua;</span><br><span class="line">proxy_pass http://127.0.0.1:8080;</span><br><span class="line">   client_max_body_size 1m;</span><br><span class="line">}</span><br></pre></td>
</tr></table></figure>
<ul>
<li>step3:添加黑名单规则数据</li>
</ul>
<figure class="highlight plain"><table><tr>
<td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br></pre></td>
<td class="code"><pre><span class="line">redis-cli sadd black.ip '192.160.10.10'</span><br><span class="line">redis-cli sadd black.imsi '460123456789'</span><br><span class="line">redis-cli sadd black.tel '15888888888'</span><br></pre></td>
</tr></table></figure>
<ul>
<li>step4:验证结果</li>
</ul>
<figure class="highlight plain"><table><tr>
<td class="gutter"><pre><span class="line">1</span><br></pre></td>
<td class="code"><pre><span class="line">curl -d "imsi=460123456789&amp;tel=15800000000" "http://yourdomain/index.php"</span><br></pre></td>
</tr></table></figure>
</li>
</ol>

      
    </div>