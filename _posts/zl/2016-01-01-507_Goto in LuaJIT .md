---
layout: post
title: Goto in LuaJIT  
tags: [lua文章]
categories: [topic]
---
<p>Lua 在 5.2 之后的版本，加入了 <code class="highlighter-rouge">goto</code> 这个关键字，用来控制程序跳转到指定 <code class="highlighter-rouge">label</code>。我们可以利用这个特性，来模拟 continue 的实现。需要注意的是 <code class="highlighter-rouge">goto</code> 只能跳转到 <code class="highlighter-rouge">label</code>，而 <code class="highlighter-rouge">::name::</code> 的格式就可以设置一个 <code class="highlighter-rouge">label</code>。</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">for</span> <span class="n">i</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span><span class="mi">5</span> <span class="k">do</span>
    <span class="k">if</span> <span class="n">i</span> <span class="o">==</span> <span class="mi">3</span> <span class="k">then</span>
        <span class="n">goto</span> <span class="n">continue</span>
    <span class="k">end</span>
    <span class="nb">print</span><span class="p">(</span><span class="n">i</span><span class="p">)</span>
    <span class="p">::</span><span class="n">continue</span><span class="p">::</span>
<span class="k">end</span>
</code></pre></div></div>

<p>这样就简单实现了 continue。但是有些同学可能会有疑问，这是 Lua 5.2 的特性，我们都知道 OR 中使用的 Lua 5.1，那如何使用？</p>

<p>其实我们只需要使用 LuaJIT 就可以解决这个问题了。在 LuaJIT 的主页上有这个介绍：</p>

<blockquote>
  <p>LuaJIT supports some language and library extensions from Lua 5.2. Features that are unlikely to break existing code are unconditionally enabled:
<br/><br/></p>
  <ul>
    <li>goto and ::labels::.</li>
  </ul>
</blockquote>

<p>也就是说 LuaJIT 把 5.2 的这个特性拿过来了。那我们来验证下：</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">[</span><span class="n">root</span><span class="err">@</span><span class="n">master</span><span class="p">:</span><span class="err">~</span><span class="p">]</span><span class="o">#</span> <span class="n">cd</span> <span class="o">/</span><span class="n">usr</span><span class="o">/</span><span class="kd">local</span><span class="o">/</span><span class="n">openresty</span><span class="o">/</span><span class="n">luajit</span><span class="o">/</span><span class="n">bin</span><span class="o">/</span>
<span class="p">[</span><span class="n">root</span><span class="err">@</span><span class="n">master</span><span class="p">:</span><span class="n">bin</span><span class="p">]</span><span class="o">#</span> <span class="p">.</span><span class="o">/</span><span class="n">luajit</span>
<span class="n">LuaJIT</span> <span class="mi">2</span><span class="p">.</span><span class="mi">1</span><span class="p">.</span><span class="mi">0</span><span class="o">-</span><span class="n">beta1</span> <span class="c1">-- Copyright (C) 2005-2015 Mike Pall. http://luajit.org/</span>
<span class="n">JIT</span><span class="p">:</span> <span class="n">ON</span> <span class="n">SSE2</span> <span class="n">SSE3</span> <span class="n">SSE4</span><span class="p">.</span><span class="mi">1</span> <span class="n">fold</span> <span class="n">cse</span> <span class="n">dce</span> <span class="n">fwd</span> <span class="n">dse</span> <span class="n">narrow</span> <span class="n">loop</span> <span class="n">abc</span> <span class="n">sink</span> <span class="n">fuse</span>
<span class="o">&gt;</span>
<span class="o">&gt;</span> <span class="k">for</span> <span class="n">i</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span><span class="mi">5</span> <span class="k">do</span>
    <span class="k">if</span> <span class="n">i</span> <span class="o">==</span> <span class="mi">3</span> <span class="k">then</span>
        <span class="n">goto</span> <span class="n">continue</span>
    <span class="k">end</span>
    <span class="nb">print</span><span class="p">(</span><span class="n">i</span><span class="p">)</span>
    <span class="p">::</span><span class="n">continue</span><span class="p">::</span>
<span class="k">end</span><span class="o">&gt;&gt;</span> <span class="o">&gt;&gt;</span> <span class="o">&gt;&gt;</span> <span class="o">&gt;&gt;</span> <span class="o">&gt;&gt;</span> <span class="o">&gt;&gt;</span>
<span class="mi">1</span>
<span class="mi">2</span>
<span class="mi">4</span>
<span class="mi">5</span>
<span class="o">&gt;</span>
</code></pre></div></div>

<p>继续验证 OR ：</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">[</span><span class="n">root</span><span class="err">@</span><span class="n">master</span><span class="p">:</span><span class="n">demo</span><span class="p">]</span><span class="o">#</span> <span class="n">nl</span> <span class="n">app</span><span class="o">/</span><span class="n">app</span><span class="p">.</span><span class="n">lua</span>
     <span class="mi">1</span>	<span class="kd">local</span> <span class="n">lor</span> <span class="o">=</span> <span class="nb">require</span><span class="p">(</span><span class="s2">&#34;lor.index&#34;</span><span class="p">)</span>
     <span class="mi">2</span>	<span class="kd">local</span> <span class="n">app</span> <span class="o">=</span> <span class="n">lor</span><span class="p">()</span>

     <span class="mi">3</span>	<span class="n">app</span><span class="p">:</span><span class="n">get</span><span class="p">(</span><span class="s2">&#34;/index&#34;</span><span class="p">,</span> <span class="k">function</span><span class="p">(</span><span class="n">req</span><span class="p">,</span><span class="n">res</span><span class="p">,</span><span class="nb">next</span><span class="p">)</span>
     <span class="mi">4</span>	    <span class="n">res</span><span class="p">:</span><span class="n">send</span><span class="p">(</span><span class="s2">&#34;hello&#34;</span><span class="p">)</span>
     <span class="mi">5</span>	<span class="k">end</span><span class="p">)</span>

     <span class="mi">6</span>	<span class="n">app</span><span class="p">:</span><span class="n">get</span><span class="p">(</span><span class="s2">&#34;goto&#34;</span><span class="p">,</span> <span class="k">function</span><span class="p">(</span><span class="n">req</span><span class="p">,</span><span class="n">res</span><span class="p">,</span><span class="nb">next</span><span class="p">)</span>
     <span class="mi">7</span>	    <span class="k">for</span> <span class="n">i</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span> <span class="mi">5</span> <span class="k">do</span>
     <span class="mi">8</span>	        <span class="k">if</span> <span class="n">i</span> <span class="o">==</span> <span class="mi">3</span> <span class="k">then</span>
     <span class="mi">9</span>	            <span class="n">goto</span> <span class="n">continue</span>
    <span class="mi">10</span>	        <span class="k">end</span>
    <span class="mi">11</span>	        <span class="n">ngx</span><span class="p">.</span><span class="n">say</span><span class="p">(</span><span class="n">i</span><span class="p">)</span>
    <span class="mi">12</span>	        <span class="p">::</span><span class="n">continue</span><span class="p">::</span>
    <span class="mi">13</span>	    <span class="k">end</span>
    <span class="mi">14</span>	<span class="k">end</span><span class="p">)</span>

    <span class="mi">15</span>	<span class="k">return</span> <span class="n">app</span>

<span class="p">[</span><span class="n">root</span><span class="err">@</span><span class="n">master</span><span class="p">:</span><span class="n">demo</span><span class="p">]</span><span class="o">#</span>
<span class="p">[</span><span class="n">root</span><span class="err">@</span><span class="n">master</span><span class="p">:</span><span class="n">demo</span><span class="p">]</span><span class="o">#</span> <span class="n">http</span> <span class="p">:</span><span class="mi">8888</span><span class="o">/</span><span class="n">goto</span>
<span class="n">HTTP</span><span class="o">/</span><span class="mi">1</span><span class="p">.</span><span class="mi">1</span> <span class="mi">200</span> <span class="n">OK</span>
<span class="n">Connection</span><span class="p">:</span> <span class="n">keep</span><span class="o">-</span><span class="n">alive</span>
<span class="n">Content</span><span class="o">-</span><span class="n">Type</span><span class="p">:</span> <span class="n">text</span><span class="o">/</span><span class="n">plain</span>
<span class="n">Date</span><span class="p">:</span> <span class="n">Fri</span><span class="p">,</span> <span class="mi">02</span> <span class="n">Dec</span> <span class="mi">2016</span> <span class="mi">02</span><span class="p">:</span><span class="mi">55</span><span class="p">:</span><span class="mi">43</span> <span class="n">GMT</span>
<span class="n">Server</span><span class="p">:</span> <span class="n">openresty</span><span class="o">/</span><span class="mi">1</span><span class="p">.</span><span class="mi">9</span><span class="p">.</span><span class="mi">7</span><span class="p">.</span><span class="mi">4</span>
<span class="n">Transfer</span><span class="o">-</span><span class="n">Encoding</span><span class="p">:</span> <span class="n">chunked</span>
<span class="n">X</span><span class="o">-</span><span class="n">Powered</span><span class="o">-</span><span class="n">By</span><span class="p">:</span> <span class="n">Lor</span> <span class="n">Framework</span>

<span class="mi">1</span>
<span class="mi">2</span>
<span class="mi">4</span>
<span class="mi">5</span>

<span class="p">[</span><span class="n">root</span><span class="err">@</span><span class="n">master</span><span class="p">:</span><span class="n">demo</span><span class="p">]</span><span class="o">#</span>
</code></pre></div></div>

<p>那么 <code class="highlighter-rouge">goto</code> 还有哪些玩法？尾递归优化使用 <code class="highlighter-rouge">goto</code> 来实现：</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">do</span>
    <span class="kd">local</span> <span class="k">function</span> <span class="nf">fact</span><span class="p">(</span><span class="n">n</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">fact_iter</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
    <span class="k">end</span>

    <span class="k">function</span> <span class="nf">fact_iter</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">now</span><span class="p">)</span>
        <span class="p">::</span><span class="n">call</span><span class="p">::</span>
        <span class="k">if</span> <span class="n">n</span> <span class="o">&lt;=</span> <span class="mi">1</span> <span class="k">then</span>
            <span class="k">return</span> <span class="n">now</span>
        <span class="k">else</span>
            <span class="c1">-- return fact_iter(n-1, now*n)</span>
            <span class="n">n</span><span class="p">,</span> <span class="n">now</span> <span class="o">=</span> <span class="n">n</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span> <span class="n">now</span><span class="o">*</span><span class="n">n</span>
            <span class="n">goto</span> <span class="n">call</span>
        <span class="k">end</span>
    <span class="k">end</span>

    <span class="nb">print</span><span class="p">(</span><span class="n">fact</span><span class="p">(</span><span class="mi">5</span><span class="p">))</span>
    <span class="nb">print</span><span class="p">(</span><span class="n">fact</span><span class="p">(</span><span class="mi">10000</span><span class="p">))</span>
    <span class="nb">print</span><span class="p">(</span><span class="n">fact</span><span class="p">(</span><span class="mi">10000000</span><span class="p">))</span>
<span class="k">end</span>
</code></pre></div></div>

<p>然而感觉并没什么卵用。其他玩法需要各位同学自己去挖掘了。。不过需要注意的是 Lua 也有对 <code class="highlighter-rouge">goto</code> 的滥用限制：</p>

<ul>
  <li>A label is visible in the entire block where it is defined (including nested blocks, but not nested functions).</li>
  <li>A goto may jump to any visible label as long as it does not enter into the scope of a local variable.</li>
</ul>


                <hr style="visibility: hidden;"/>