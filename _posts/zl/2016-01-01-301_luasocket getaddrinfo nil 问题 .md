---
layout: post
title: luasocket getaddrinfo nil 问题  
tags: [lua文章]
categories: [topic]
---
<p>使用 luarocks 安装 luasocket，在调用 bind 时，报：</p>

<blockquote>
  <p>socket.lua:29: attempt to call field ‘getaddrinfo’ (a nil value)</p>
</blockquote>

<p>继续执行以下 lua 代码片段：</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">Lua</span> <span class="mi">5</span><span class="p">.</span><span class="mi">1</span><span class="p">.</span><span class="mi">4</span>  <span class="n">Copyright</span> <span class="p">(</span><span class="n">C</span><span class="p">)</span> <span class="mi">1994</span><span class="o">-</span><span class="mi">2008</span> <span class="n">Lua</span><span class="p">.</span><span class="n">org</span><span class="p">,</span> <span class="n">PUC</span><span class="o">-</span><span class="n">Rio</span>
<span class="o">&gt;</span>
<span class="o">&gt;</span> <span class="k">do</span>
<span class="o">&gt;&gt;</span>   <span class="kd">local</span> <span class="n">socket</span> <span class="o">=</span> <span class="nb">require</span><span class="p">(</span><span class="s2">&#34;socket&#34;</span><span class="p">)</span>
<span class="o">&gt;&gt;</span>   <span class="k">for</span> <span class="n">k</span><span class="p">,</span><span class="n">v</span> <span class="k">in</span> <span class="nb">pairs</span><span class="p">(</span><span class="n">socket</span><span class="p">.</span><span class="n">dns</span><span class="p">)</span> <span class="k">do</span>
<span class="o">&gt;&gt;</span>     <span class="nb">print</span><span class="p">(</span><span class="n">k</span><span class="p">,</span><span class="n">v</span><span class="p">)</span>
<span class="o">&gt;&gt;</span>   <span class="k">end</span>
<span class="o">&gt;&gt;</span> <span class="k">end</span>
<span class="n">gethostname</span>	<span class="k">function</span><span class="err">:</span> <span class="err">0</span><span class="nf">xc774f0</span>
<span class="n">toip</span>	<span class="k">function</span><span class="err">:</span> <span class="err">0</span><span class="nf">xc77ea0</span>
<span class="n">tohostname</span>	<span class="k">function</span><span class="err">:</span> <span class="err">0</span><span class="nf">xc77f00</span>
<span class="o">&gt;</span>
<span class="o">&gt;</span>
</code></pre></div></div>

<p>发现确实没有加载进来，而 <code class="highlighter-rouge">getaddrinfo</code> 是一个标准的 posix 系统调用，所以基本可以确定是安装的问题了。。</p>

<p>于是从 github 拉下最新的 luasocket v3.0-rc1 手动编译完成后，可以 bind，可以看到加载进来了：</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">&gt;</span>
<span class="o">&gt;&gt;</span> <span class="k">do</span>
<span class="o">&gt;&gt;</span>   <span class="kd">local</span> <span class="n">socket</span> <span class="o">=</span> <span class="nb">require</span><span class="p">(</span><span class="s2">&#34;socket&#34;</span><span class="p">)</span>
<span class="o">&gt;&gt;</span>   <span class="k">for</span> <span class="n">k</span><span class="p">,</span><span class="n">v</span> <span class="k">in</span> <span class="nb">pairs</span><span class="p">(</span><span class="n">socket</span><span class="p">.</span><span class="n">dns</span><span class="p">)</span> <span class="k">do</span>
<span class="o">&gt;&gt;</span>     <span class="nb">print</span><span class="p">(</span><span class="n">k</span><span class="p">,</span><span class="n">v</span><span class="p">)</span>
<span class="o">&gt;&gt;</span>   <span class="k">end</span>
<span class="o">&gt;&gt;</span> <span class="k">end</span>
<span class="n">getnameinfo</span>	<span class="k">function</span><span class="err">:</span> <span class="err">0</span><span class="nf">xc77490</span>
<span class="n">getaddrinfo</span>	<span class="k">function</span><span class="err">:</span> <span class="err">0</span><span class="nf">xc77380</span>
<span class="n">gethostname</span>	<span class="k">function</span><span class="err">:</span> <span class="err">0</span><span class="nf">xc774f0</span>
<span class="n">toip</span>	<span class="k">function</span><span class="err">:</span> <span class="err">0</span><span class="nf">xc77ea0</span>
<span class="n">tohostname</span>	<span class="k">function</span><span class="err">:</span> <span class="err">0</span><span class="nf">xc77f00</span>
<span class="o">&gt;</span>
<span class="o">&gt;</span>
</code></pre></div></div>

<p>所以结论已经出来了，是 luarocks 安装 luasocket 的问题了。那我们继续追踪问题的根源，使用 <code class="highlighter-rouge">luarocks install luasocket</code> 可以看到 luasocket 的编译参数为：</p>

<div class="language-sh highlighter-rouge"><div class="highlight"><pre class="highlight"><code>gcc <span class="nt">-O2</span> <span class="nt">-fPIC</span> <span class="nt">-I</span>/usr/include <span class="nt">-c</span> src/mime.c <span class="nt">-o</span> src/mime.o <span class="nt">-DLUA_COMPAT_APIINTCASTS</span> <span class="nt">-DLUASOCKET_DEBUG</span> <span class="nt">-DLUASOCKET_API</span><span class="o">=</span>__attribute__<span class="o">((</span>visibility<span class="o">(</span><span class="s2">&#34;default&#34;</span><span class="o">)))</span> <span class="nt">-DUNIX_API</span><span class="o">=</span>__attribute__<span class="o">((</span>visibility<span class="o">(</span><span class="s2">&#34;default&#34;</span><span class="o">)))</span> <span class="nt">-DMIME_API</span><span class="o">=</span>__attribute__<span class="o">((</span>visibility<span class="o">(</span><span class="s2">&#34;default&#34;</span><span class="o">)))</span>
</code></pre></div></div>

<p>而我自己手动编译时，默认参数为：</p>

<div class="language-sh highlighter-rouge"><div class="highlight"><pre class="highlight"><code>gcc <span class="nt">-I</span>/usr/local/lua/include/lua/5.1 <span class="nt">-DLUASOCKET_NODEBUG</span> <span class="nt">-DLUA_NOCOMPAT_MODULE</span> <span class="nt">-DLUASOCKET_API</span><span class="o">=</span><span class="s1">&#39;__attribute__((visibility(&#34;default&#34;)))&#39;</span> <span class="nt">-DUNIX_API</span><span class="o">=</span><span class="s1">&#39;__attribute__((visibility(&#34;default&#34;)))&#39;</span> <span class="nt">-DMIME_API</span><span class="o">=</span><span class="s1">&#39;__attribute__((visibility(&#34;default&#34;)))&#39;</span> <span class="nt">-pedantic</span> <span class="nt">-Wall</span> <span class="nt">-Wshadow</span> <span class="nt">-Wextra</span> <span class="nt">-Wimplicit</span> <span class="nt">-O2</span> <span class="nt">-ggdb3</span> <span class="nt">-fpic</span> <span class="nt">-fvisibility</span><span class="o">=</span>hidden   <span class="nt">-c</span> <span class="nt">-o</span> mime.o mime.c
</code></pre></div></div>

<p>嗯。。。问题似乎已经很明确了，没错，就是几个宏的引用问题：</p>

<div class="language-sh highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nt">-DLUA_COMPAT_APIINTCASTS</span> <span class="nt">-DLUASOCKET_DEBUG</span>
</code></pre></div></div>

<p>而我们手动编译是启用的这几个宏：</p>

<div class="language-sh highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nt">-DLUASOCKET_NODEBUG</span> <span class="nt">-DLUA_NOCOMPAT_MODULE</span>
</code></pre></div></div>

<p>当然如果你嫌手动编译麻烦，也可以用 luarocks 安装 2.0.2 的版本，这个是没有问题的：</p>

<div class="language-sh highlighter-rouge"><div class="highlight"><pre class="highlight"><code>luarocks install luasocket 2.0.2-6
</code></pre></div></div>

<p>另，在调查这个问题的原因时，新发现个 luarocks pack 的使用技巧，pack 就是可以将我们安装过后的 rock 重新生成 rock 文件，之后我们将其上传到需要安装的其他机器，执行 <code class="highlighter-rouge">luarocks install ***.rock</code> 即可：</p>

<div class="language-sh highlighter-rouge"><div class="highlight"><pre class="highlight"><code>luarocks pack luasocket
<span class="c"># 接下来拷贝 luasocket-3.0rc1-2.linux-x86_64.rock 至目标机器并执行以下命令</span>
<span class="c"># 就可以离线安装 luasocket 了</span>
luarocks install luasocket-3.0rc1-2.linux-x86_64.rock
</code></pre></div></div>


                <hr style="visibility: hidden;"/>