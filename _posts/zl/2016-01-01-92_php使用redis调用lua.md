---
layout: post
title: php使用redis调用lua 
tags: [lua文章]
categories: [topic]
---
<div class="entry">
      
        <p>  今天看到laravel在队列处理的时候用了lua脚本<br/><a href="https://github.com/laravel/framework/blob/5.8/src/Illuminate/Queue/LuaScripts.php" target="_blank" rel="noopener noreferrer">LuaScripts.php</a><br/><del>后来查了下，原来php还有个lua的扩展，真的是孤陋寡闻了</del></p>
<p>redis可以通过EVAL调用lua脚本 <a href="https://redis.io/commands/eval" target="_blank" rel="noopener noreferrer">EVAL</a><br/>eval的参数顺序： lua脚本，key数量，所有的键名，所有的值<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line">&gt; eval &#34;return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}&#34; 2 key1 key2 first second</span><br/><span class="line">1) &#34;key1&#34;</span><br/><span class="line">2) &#34;key2&#34;</span><br/><span class="line">3) &#34;first&#34;</span><br/><span class="line">4) &#34;second&#34;</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>php中redis的eval参数顺序：lua脚本，参数数组，key数量<a href="https://github.com/phpredis/phpredis/#eval" target="_blank" rel="noopener noreferrer">phpredis-&gt;eval()</a><br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/></pre></td><td class="code"><pre><span class="line">$redis-&gt;eval(&#34;return 1&#34;); // Returns an integer: 1</span><br/><span class="line">$redis-&gt;eval(&#34;return {1,2,3}&#34;); // Returns [1,2,3]</span><br/><span class="line">$redis-&gt;del(&#39;mylist&#39;);</span><br/><span class="line">$redis-&gt;rpush(&#39;mylist&#39;,&#39;a&#39;);</span><br/><span class="line">$redis-&gt;rpush(&#39;mylist&#39;,&#39;b&#39;);</span><br/><span class="line">$redis-&gt;rpush(&#39;mylist&#39;,&#39;c&#39;);</span><br/><span class="line">// Nested response:  [1,2,3,[&#39;a&#39;,&#39;b&#39;,&#39;c&#39;]];</span><br/><span class="line">$redis-&gt;eval(&#34;return {1,2,3,redis.call(&#39;lrange&#39;,&#39;mylist&#39;,0,-1)}&#34;);</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>通常php会用redis的setnx来加锁，但是setnx有一个缺陷，锁过期时间3秒，a程序执行时间超过3秒，锁已经解除，下个程序b进来锁上，a执行完解锁，解的是b的锁<br/>还有一些复杂操作必须通过lua脚本来保证原子性，比如把一个list的数据复制到另一个list  </p>

      
    </div>