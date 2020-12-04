---
layout: post
title: StackExchange.Redis加载Lua脚本进行模糊查询的批量删除和修改 
tags: [lua文章]
categories: [topic]
---
<ul id="markdown-toc">
  <li><a href="#通过keys进行模糊查询后的批量操作" id="markdown-toc-通过keys进行模糊查询后的批量操作">通过keys进行模糊查询后的批量操作</a></li>
  <li><a href="#对hash集合下的key进行模糊查询后的批量操作" id="markdown-toc-对hash集合下的key进行模糊查询后的批量操作">对Hash集合下的key进行模糊查询后的批量操作</a></li>
  <li><a href="#对set集合下的值进行模糊查询后的批量操作" id="markdown-toc-对set集合下的值进行模糊查询后的批量操作">对Set集合下的值进行模糊查询后的批量操作</a></li>
  <li><a href="#注意" id="markdown-toc-注意">注意</a></li>
</ul>

<h2 id="通过keys进行模糊查询后的批量操作">通过keys进行模糊查询后的批量操作</h2>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code>     <span class="n">var</span> <span class="n">redis</span> <span class="o">=</span> <span class="n">ConnectionMultiplexer</span><span class="o">.</span><span class="na">Connect</span><span class="o">(</span><span class="s">&#34;127.0.0.1:6379,allowAdmin = true&#34;</span><span class="o">);</span>
     <span class="n">redis</span><span class="o">.</span><span class="na">GetDatabase</span><span class="o">().</span><span class="na">ScriptEvaluate</span><span class="o">(</span><span class="n">LuaScript</span><span class="o">.</span><span class="na">Prepare</span><span class="o">(</span>
         <span class="c1">//Redis的keys模糊查询：</span>
         <span class="s">&#34; local ks = redis.call(&#39;KEYS&#39;, @keypattern) &#34;</span> <span class="o">+</span> <span class="c1">//local ks为定义一个局部变量，其中用于存储获取到的keys</span>
         <span class="s">&#34; for i=1,#ks,5000 do &#34;</span> <span class="o">+</span>    <span class="c1">//#ks为ks集合的个数, 语句的意思： for(int i = 1; i &lt;= ks.Count; i+=5000)</span>
         <span class="s">&#34;     redis.call(&#39;del&#39;, unpack(ks, i, math.min(i+4999, #ks))) &#34;</span> <span class="o">+</span> <span class="c1">//Lua集合索引值从1为起始，unpack为解包，获取ks集合中的数据，每次5000，然后执行删除</span>
         <span class="s">&#34; end &#34;</span> <span class="o">+</span>
         <span class="s">&#34; return true &#34;</span>
         <span class="o">),</span>
         <span class="k">new</span> <span class="o">{</span> <span class="n">keypattern</span> <span class="o">=</span> <span class="s">&#34;mykey*&#34;</span> <span class="o">});</span>
</code></pre></div></div>

<h2 id="对hash集合下的key进行模糊查询后的批量操作">对Hash集合下的key进行模糊查询后的批量操作</h2>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code>     <span class="n">redis</span><span class="o">.</span><span class="na">GetDatabase</span><span class="o">().</span><span class="na">ScriptEvaluate</span><span class="o">(</span><span class="n">LuaScript</span><span class="o">.</span><span class="na">Prepare</span><span class="o">(</span>
         <span class="s">&#34; local ks = redis.call(&#39;hkeys&#39;, @hashid) &#34;</span> <span class="o">+</span>
         <span class="s">&#34; local fkeys = {} &#34;</span> <span class="o">+</span>
         <span class="s">&#34; for i=1,#ks do &#34;</span> <span class="o">+</span>
         <span class="c1">//使用string.find进行匹配操作</span>
         <span class="s">&#34;   if string.find(ks[i], @keypattern) then &#34;</span> <span class="o">+</span>
         <span class="s">&#34;      fkeys[#fkeys + 1] = ks[i] &#34;</span> <span class="o">+</span>
         <span class="s">&#34;   end &#34;</span> <span class="o">+</span>
         <span class="s">&#34; end &#34;</span> <span class="o">+</span>
         <span class="s">&#34; for i=1,#fkeys,5000 do &#34;</span> <span class="o">+</span>
         <span class="s">&#34;   redis.call(&#39;hdel&#39;, @hashid, unpack(fkeys, i, math.min(i+4999, #fkeys))) &#34;</span> <span class="o">+</span>
         <span class="s">&#34; end &#34;</span> <span class="o">+</span>
         <span class="s">&#34; return true &#34;</span>
         <span class="o">),</span>
         <span class="k">new</span> <span class="o">{</span> <span class="n">hashid</span> <span class="o">=</span> <span class="s">&#34;hkey&#34;</span><span class="o">,</span> <span class="n">keypattern</span> <span class="o">=</span> <span class="s">&#34;^mykey&#34;</span> <span class="o">});</span>   <span class="c1">//keypattern为可使用正则表达式</span>
</code></pre></div></div>
<p>或</p>
<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code>     <span class="n">redis</span><span class="o">.</span><span class="na">GetDatabase</span><span class="o">().</span><span class="na">ScriptEvaluate</span><span class="o">(</span><span class="n">LuaScript</span><span class="o">.</span><span class="na">Prepare</span><span class="o">(</span>
         <span class="s">&#34; local ks = redis.call(&#39;hkeys&#39;, @hashid) &#34;</span> <span class="o">+</span>
         <span class="s">&#34; local fkeys = {} &#34;</span> <span class="o">+</span>
         <span class="s">&#34; for i=1,#ks do &#34;</span> <span class="o">+</span>
         <span class="s">&#34;   if string.find(ks[i], @keypattern) then &#34;</span> <span class="o">+</span>
         <span class="s">&#34;      fkeys[#fkeys + 1] = ks[i] &#34;</span> <span class="o">+</span>
         <span class="s">&#34;   end &#34;</span> <span class="o">+</span>
         <span class="s">&#34; end &#34;</span> <span class="o">+</span>
         <span class="s">&#34; for i=1,#fkeys do &#34;</span> <span class="o">+</span>
         <span class="s">&#34;   redis.call(&#39;hset&#39;, @hashid, fkeys[i], @value) &#34;</span> <span class="o">+</span>
         <span class="s">&#34; end &#34;</span> <span class="o">+</span>
         <span class="s">&#34; return true &#34;</span>
         <span class="o">),</span>
         <span class="k">new</span> <span class="o">{</span> <span class="n">hashid</span> <span class="o">=</span> <span class="s">&#34;hkey&#34;</span><span class="o">,</span> <span class="n">keypattern</span> <span class="o">=</span> <span class="s">&#34;^key&#34;</span><span class="o">,</span> <span class="n">value</span> <span class="o">=</span> <span class="s">&#34;hashValue&#34;</span> <span class="o">});</span>   <span class="c1">//keypattern为可使用正则表达式</span>
</code></pre></div></div>
<h2 id="对set集合下的值进行模糊查询后的批量操作">对Set集合下的值进行模糊查询后的批量操作</h2>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code>     <span class="n">redis</span><span class="o">.</span><span class="na">GetDatabase</span><span class="o">().</span><span class="na">ScriptEvaluate</span><span class="o">(</span><span class="n">LuaScript</span><span class="o">.</span><span class="na">Prepare</span><span class="o">(</span>
         <span class="s">&#34; local ks = redis.call(&#39;smembers&#39;, @keyid) &#34;</span> <span class="o">+</span>
         <span class="s">&#34; local fkeys = {} &#34;</span> <span class="o">+</span>
         <span class="s">&#34; for i=1,#ks do &#34;</span> <span class="o">+</span>
         <span class="s">&#34;   if string.find(ks[i], @keypattern) then &#34;</span> <span class="o">+</span>
         <span class="s">&#34;      fkeys[#fkeys + 1] = ks[i] &#34;</span> <span class="o">+</span>
         <span class="s">&#34;   end &#34;</span> <span class="o">+</span>
         <span class="s">&#34; end &#34;</span> <span class="o">+</span>
         <span class="s">&#34; for i=1,#fkeys,5000 do &#34;</span> <span class="o">+</span>
         <span class="s">&#34;   redis.call(&#39;srem&#39;, @keyid, unpack(fkeys, i, math.min(i+4999, #fkeys))) &#34;</span> <span class="o">+</span>
         <span class="s">&#34; end &#34;</span> <span class="o">+</span>
         <span class="s">&#34; return true &#34;</span>
         <span class="o">),</span>
         <span class="k">new</span> <span class="o">{</span> <span class="n">keyid</span> <span class="o">=</span> <span class="s">&#34;setkey&#34;</span><span class="o">,</span> <span class="n">keypattern</span> <span class="o">=</span> <span class="s">&#34;^myval&#34;</span> <span class="o">});</span>   <span class="c1">//keypattern为可使用正则表达式</span>
</code></pre></div></div>

<h2 id="注意">注意</h2>
<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>从 Redis 2.6.0 版本开始，才可通过内置的 Lua 解释器，使用 EVAL 命令对 Lua 脚本进行求值。
</code></pre></div></div>