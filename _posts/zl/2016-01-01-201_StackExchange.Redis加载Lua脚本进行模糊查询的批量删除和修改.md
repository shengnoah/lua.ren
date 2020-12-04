---
layout: post
title: StackExchange.Redis加载Lua脚本进行模糊查询的批量删除和修改 
tags: [lua文章]
categories: [topic]
---

        <ul id="markdown-toc">
  <li><a href="https://lxljh398.github.io/#%E9%80%9A%E8%BF%87keys%E8%BF%9B%E8%A1%8C%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2%E5%90%8E%E7%9A%84%E6%89%B9%E9%87%8F%E6%93%8D%E4%BD%9C" id="markdown-toc-通过keys进行模糊查询后的批量操作">通过keys进行模糊查询后的批量操作</a></li>
  <li><a href="https://lxljh398.github.io/#%E5%AF%B9hash%E9%9B%86%E5%90%88%E4%B8%8B%E7%9A%84key%E8%BF%9B%E8%A1%8C%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2%E5%90%8E%E7%9A%84%E6%89%B9%E9%87%8F%E6%93%8D%E4%BD%9C" id="markdown-toc-对hash集合下的key进行模糊查询后的批量操作">对Hash集合下的key进行模糊查询后的批量操作</a></li>
  <li><a href="https://lxljh398.github.io/#%E5%AF%B9set%E9%9B%86%E5%90%88%E4%B8%8B%E7%9A%84%E5%80%BC%E8%BF%9B%E8%A1%8C%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2%E5%90%8E%E7%9A%84%E6%89%B9%E9%87%8F%E6%93%8D%E4%BD%9C" id="markdown-toc-对set集合下的值进行模糊查询后的批量操作">对Set集合下的值进行模糊查询后的批量操作</a></li>
  <li><a href="https://lxljh398.github.io/#%E6%B3%A8%E6%84%8F" id="markdown-toc-注意">注意</a></li>
</ul>

<h2 id="通过keys进行模糊查询后的批量操作">通过keys进行模糊查询后的批量操作</h2>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code>     <span class="n">var</span> <span class="n">redis</span> <span class="o">=</span> <span class="n">ConnectionMultiplexer</span><span class="o">.</span><span class="na">Connect</span><span class="o">(</span><span class="s">"127.0.0.1:6379,allowAdmin = true"</span><span class="o">);</span>
     <span class="n">redis</span><span class="o">.</span><span class="na">GetDatabase</span><span class="o">().</span><span class="na">ScriptEvaluate</span><span class="o">(</span><span class="n">LuaScript</span><span class="o">.</span><span class="na">Prepare</span><span class="o">(</span>
         <span class="c1">//Redis的keys模糊查询：</span>
         <span class="s">" local ks = redis.call('KEYS', @keypattern) "</span> <span class="o">+</span> <span class="c1">//local ks为定义一个局部变量，其中用于存储获取到的keys</span>
         <span class="s">" for i=1,#ks,5000 do "</span> <span class="o">+</span>    <span class="c1">//#ks为ks集合的个数, 语句的意思： for(int i = 1; i &lt;= ks.Count; i+=5000)</span>
         <span class="s">"     redis.call('del', unpack(ks, i, math.min(i+4999, #ks))) "</span> <span class="o">+</span> <span class="c1">//Lua集合索引值从1为起始，unpack为解包，获取ks集合中的数据，每次5000，然后执行删除</span>
         <span class="s">" end "</span> <span class="o">+</span>
         <span class="s">" return true "</span>
         <span class="o">),</span>
         <span class="k">new</span> <span class="o">{</span> <span class="n">keypattern</span> <span class="o">=</span> <span class="s">"mykey*"</span> <span class="o">});</span>
</code></pre></div></div>

<h2 id="对hash集合下的key进行模糊查询后的批量操作">对Hash集合下的key进行模糊查询后的批量操作</h2>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code>     <span class="n">redis</span><span class="o">.</span><span class="na">GetDatabase</span><span class="o">().</span><span class="na">ScriptEvaluate</span><span class="o">(</span><span class="n">LuaScript</span><span class="o">.</span><span class="na">Prepare</span><span class="o">(</span>
         <span class="s">" local ks = redis.call('hkeys', @hashid) "</span> <span class="o">+</span>
         <span class="s">" local fkeys = {} "</span> <span class="o">+</span>
         <span class="s">" for i=1,#ks do "</span> <span class="o">+</span>
         <span class="c1">//使用string.find进行匹配操作</span>
         <span class="s">"   if string.find(ks[i], @keypattern) then "</span> <span class="o">+</span>
         <span class="s">"      fkeys[#fkeys + 1] = ks[i] "</span> <span class="o">+</span>
         <span class="s">"   end "</span> <span class="o">+</span>
         <span class="s">" end "</span> <span class="o">+</span>
         <span class="s">" for i=1,#fkeys,5000 do "</span> <span class="o">+</span>
         <span class="s">"   redis.call('hdel', @hashid, unpack(fkeys, i, math.min(i+4999, #fkeys))) "</span> <span class="o">+</span>
         <span class="s">" end "</span> <span class="o">+</span>
         <span class="s">" return true "</span>
         <span class="o">),</span>
         <span class="k">new</span> <span class="o">{</span> <span class="n">hashid</span> <span class="o">=</span> <span class="s">"hkey"</span><span class="o">,</span> <span class="n">keypattern</span> <span class="o">=</span> <span class="s">"^mykey"</span> <span class="o">});</span>   <span class="c1">//keypattern为可使用正则表达式</span>
</code></pre></div></div>
<p>或</p>
<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code>     <span class="n">redis</span><span class="o">.</span><span class="na">GetDatabase</span><span class="o">().</span><span class="na">ScriptEvaluate</span><span class="o">(</span><span class="n">LuaScript</span><span class="o">.</span><span class="na">Prepare</span><span class="o">(</span>
         <span class="s">" local ks = redis.call('hkeys', @hashid) "</span> <span class="o">+</span>
         <span class="s">" local fkeys = {} "</span> <span class="o">+</span>
         <span class="s">" for i=1,#ks do "</span> <span class="o">+</span>
         <span class="s">"   if string.find(ks[i], @keypattern) then "</span> <span class="o">+</span>
         <span class="s">"      fkeys[#fkeys + 1] = ks[i] "</span> <span class="o">+</span>
         <span class="s">"   end "</span> <span class="o">+</span>
         <span class="s">" end "</span> <span class="o">+</span>
         <span class="s">" for i=1,#fkeys do "</span> <span class="o">+</span>
         <span class="s">"   redis.call('hset', @hashid, fkeys[i], @value) "</span> <span class="o">+</span>
         <span class="s">" end "</span> <span class="o">+</span>
         <span class="s">" return true "</span>
         <span class="o">),</span>
         <span class="k">new</span> <span class="o">{</span> <span class="n">hashid</span> <span class="o">=</span> <span class="s">"hkey"</span><span class="o">,</span> <span class="n">keypattern</span> <span class="o">=</span> <span class="s">"^key"</span><span class="o">,</span> <span class="n">value</span> <span class="o">=</span> <span class="s">"hashValue"</span> <span class="o">});</span>   <span class="c1">//keypattern为可使用正则表达式</span>
</code></pre></div></div>
<h2 id="对set集合下的值进行模糊查询后的批量操作">对Set集合下的值进行模糊查询后的批量操作</h2>

<div class="language-java highlighter-rouge"><div class="highlight"><pre class="highlight"><code>     <span class="n">redis</span><span class="o">.</span><span class="na">GetDatabase</span><span class="o">().</span><span class="na">ScriptEvaluate</span><span class="o">(</span><span class="n">LuaScript</span><span class="o">.</span><span class="na">Prepare</span><span class="o">(</span>
         <span class="s">" local ks = redis.call('smembers', @keyid) "</span> <span class="o">+</span>
         <span class="s">" local fkeys = {} "</span> <span class="o">+</span>
         <span class="s">" for i=1,#ks do "</span> <span class="o">+</span>
         <span class="s">"   if string.find(ks[i], @keypattern) then "</span> <span class="o">+</span>
         <span class="s">"      fkeys[#fkeys + 1] = ks[i] "</span> <span class="o">+</span>
         <span class="s">"   end "</span> <span class="o">+</span>
         <span class="s">" end "</span> <span class="o">+</span>
         <span class="s">" for i=1,#fkeys,5000 do "</span> <span class="o">+</span>
         <span class="s">"   redis.call('srem', @keyid, unpack(fkeys, i, math.min(i+4999, #fkeys))) "</span> <span class="o">+</span>
         <span class="s">" end "</span> <span class="o">+</span>
         <span class="s">" return true "</span>
         <span class="o">),</span>
         <span class="k">new</span> <span class="o">{</span> <span class="n">keyid</span> <span class="o">=</span> <span class="s">"setkey"</span><span class="o">,</span> <span class="n">keypattern</span> <span class="o">=</span> <span class="s">"^myval"</span> <span class="o">});</span>   <span class="c1">//keypattern为可使用正则表达式</span>
</code></pre></div></div>

<h2 id="注意">注意</h2>
<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>从 Redis 2.6.0 版本开始，才可通过内置的 Lua 解释器，使用 EVAL 命令对 Lua 脚本进行求值。
</code></pre></div></div>