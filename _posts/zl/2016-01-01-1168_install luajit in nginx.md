---
layout: post
title: install luajit in nginx 
tags: [lua文章]
categories: [topic]
---
<div class="text" id="js-post-content">
        <p>https://github.com/openresty/lua-nginx-module</p>

<h4 id="安装">安装</h4>
<ul>
  <li>安装 <a href="http://luajit.org/download.html">LuaJit</a></li>
  <li>下载 <a href="https://github.com/simpl/ngx_devel_kit/tags">ngx_devel_kit</a></li>
  <li>下载 <a href="https://github.com/openresty/lua-nginx-module/tags">ngx_lua</a></li>
  <li>下载 <a href="http://nginx.org/en/download.html">Nginx</a></li>
</ul>

<figure class="highlight"><pre><code class="language-shell" data-lang="shell"> wget <span class="s1">&#39;http://nginx.org/download/nginx-1.9.7.tar.gz&#39;</span>
 <span class="nb">tar</span> <span class="nt">-xzvf</span> nginx-1.9.7.tar.gz
 <span class="nb">cd </span>nginx-1.9.7/

 <span class="c"># tell nginx&#39;s build system where to find LuaJIT 2.0:</span>
 <span class="nb">export </span><span class="nv">LUAJIT_LIB</span><span class="o">=</span>/path/to/luajit/lib
 <span class="nb">export </span><span class="nv">LUAJIT_INC</span><span class="o">=</span>/path/to/luajit/include/luajit-2.0

 <span class="c"># tell nginx&#39;s build system where to find LuaJIT 2.1:</span>
 <span class="nb">export </span><span class="nv">LUAJIT_LIB</span><span class="o">=</span>/path/to/luajit/lib
 <span class="nb">export </span><span class="nv">LUAJIT_INC</span><span class="o">=</span>/path/to/luajit/include/luajit-2.1

 <span class="c"># or tell where to find Lua if using Lua instead:</span>
 <span class="c">#export LUA_LIB=/path/to/lua/lib</span>
 <span class="c">#export LUA_INC=/path/to/lua/include</span>

 <span class="c"># Here we assume Nginx is to be installed under /opt/nginx/.</span>
 ./configure <span class="nt">--prefix</span><span class="o">=</span>/opt/nginx <span class="se"></span>
         <span class="nt">--with-ld-opt</span><span class="o">=</span><span class="s2">&#34;-Wl,-rpath,/path/to/luajit-or-lua/lib&#34;</span> <span class="se"></span>
         <span class="nt">--add-module</span><span class="o">=</span>/path/to/ngx_devel_kit <span class="se"></span>
         <span class="nt">--add-module</span><span class="o">=</span>/path/to/lua-nginx-module

 make <span class="nt">-j2</span>
 make <span class="nb">install</span></code></pre></figure>

<h4 id="测试">测试</h4>

<p>在nginx.conf中添加下面部分内容</p>

<figure class="highlight"><pre><code class="language-conf" data-lang="conf"><span class="n">location</span> /<span class="n">hello</span> {
    <span class="n">default_type</span> <span class="s1">&#39;text/plain&#39;</span>;
    <span class="n">content_by_lua</span> <span class="s1">&#39;ngx.say(&#34;hello, world&#34;)&#39;</span>;
}</code></pre></figure>

<h5 id="验证结果">验证结果</h5>
<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">[</span>root@1184c0eeaa0a nginx-1.8.1]# curl 127.0.0.1:8080/hello
hello, world
</code></pre></div></div>

<h4 id="示例2-操作redis">示例2 操作Redis</h4>
<p>github地址: <a href="https://github.com/openresty/lua-resty-redis">lua-resty-redis</a></p>

<p>下载lib/resty/redis.lua文件到pato/lib/resty/redis.lua</p>

<p>修改nginx.conf文件</p>

<figure class="highlight"><pre><code class="language-lua" data-lang="lua"><span class="o">#</span> <span class="n">you</span> <span class="k">do</span> <span class="ow">not</span> <span class="n">need</span> <span class="n">the</span> <span class="n">following</span> <span class="n">line</span> <span class="k">if</span> <span class="n">you</span> <span class="n">are</span> <span class="n">using</span>
<span class="o">#</span> <span class="n">the</span> <span class="n">OpenResty</span> <span class="n">bundle</span><span class="p">:</span>
<span class="n">lua_package_path</span> <span class="s2">&#34;/path/to/lua-resty-redis/lib/?.lua;;&#34;</span><span class="p">;</span>

<span class="n">server</span> <span class="p">{</span>
    <span class="n">location</span> <span class="o">/</span><span class="n">test</span> <span class="p">{</span>
        <span class="n">content_by_lua</span> <span class="s1">&#39;
            local redis = require &#34;resty.redis&#34;
            local red = redis:new()

            red:set_timeout(1000) -- 1 sec

            -- or connect to a unix domain socket file listened
            -- by a redis server:
            --     local ok, err = red:connect(&#34;unix:/path/to/redis.sock&#34;)

            local ok, err = red:connect(&#34;127.0.0.1&#34;, 6379)
            if not ok then
                ngx.say(&#34;failed to connect: &#34;, err)
                return
            end

            ok, err = red:set(&#34;dog&#34;, &#34;an animal&#34;)
            if not ok then
                ngx.say(&#34;failed to set dog: &#34;, err)
                return
            end

            ngx.say(&#34;set result: &#34;, ok)

            local res, err = red:get(&#34;dog&#34;)
            if not res then
                ngx.say(&#34;failed to get dog: &#34;, err)
                return
            end

            if res == ngx.null then
                ngx.say(&#34;dog not found.&#34;)
                return
            end

            ngx.say(&#34;dog: &#34;, res)

            red:init_pipeline()
            red:set(&#34;cat&#34;, &#34;Marry&#34;)
            red:set(&#34;horse&#34;, &#34;Bob&#34;)
            red:get(&#34;cat&#34;)
            red:get(&#34;horse&#34;)
            local results, err = red:commit_pipeline()
            if not results then
                ngx.say(&#34;failed to commit the pipelined requests: &#34;, err)
                return
            end

            for i, res in ipairs(results) do
                if type(res) == &#34;table&#34; then
                    if res[1] == false then
                        ngx.say(&#34;failed to run command &#34;, i, &#34;: &#34;, res[2])
                    else
                        -- process the table value
                    end
                else
                    -- process the scalar value
                end
            end

            -- put it into the connection pool of size 100,
            -- with 10 seconds max idle time
            local ok, err = red:set_keepalive(10000, 100)
            if not ok then
                ngx.say(&#34;failed to set keepalive: &#34;, err)
                return
            end

            -- or just close the connection right away:
            -- local ok, err = red:close()
            -- if not ok then
            --     ngx.say(&#34;failed to close: &#34;, err)
            --     return
            -- end
        &#39;</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span></code></pre></figure>


    </div>