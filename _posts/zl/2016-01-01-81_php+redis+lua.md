---
layout: post
title: php+redis+lua 
tags: [lua文章]
categories: [topic]
---
<p>发两个php+redis+lua的例子。<br/></p>
<h2 id="一、直接在redis上运行命令demo"><a href="#一、直接在redis上运行命令demo" class="headerlink" title="一、直接在redis上运行命令demo"></a>一、直接在redis上运行命令demo</h2><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">eval &#34;return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}&#34; 2 key1 key2 first second</span><br/></pre></td></tr></tbody></table></figure>
<ul>
<li>eval 命令代表后面接的是lua脚本，需要redis使用lua解析器；</li>
<li>2 代表接下来两个参数为为KEY的参数，即为 key1 key2；</li>
<li>first、second代表ARGV的附加参数；</li>
</ul>
<h2 id="二、PHP的demo"><a href="#二、PHP的demo" class="headerlink" title="二、PHP的demo"></a>二、PHP的demo</h2><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/></pre></td><td class="code"><pre><span class="line">$lua = &#34;return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}&#34;;</span><br/><span class="line">$s = $redis-&gt;eval($lua,array(&#39;key1&#39;,&#39;key2&#39;,&#39;first&#39;,&#39;second&#39;),2);</span><br/></pre></td></tr></tbody></table></figure>
<ul>
<li>eval(lua脚本字符串,参数数组,前几个为key参数);</li>
</ul>
<h4 id="1、一次性获取所有的hash结构的所有值"><a href="#1、一次性获取所有的hash结构的所有值" class="headerlink" title="1、一次性获取所有的hash结构的所有值"></a>1、一次性获取所有的hash结构的所有值</h4><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/></pre></td><td class="code"><pre><span class="line">$lua = &#34;local ret={}; for i,v in pairs(KEYS) do ret[i]=redis.call(&#39;hgetall&#39;, v) end; return ret&#34;;</span><br/><span class="line">$obj = new Dao_RedisBase();</span><br/><span class="line">$arr_hash_key = [&#39;hash1&#39;,&#39;hash2&#39;,&#39;hash3&#39;,&#39;hash4&#39;];</span><br/><span class="line">$hashresult=$obj-&gt;getCon()-&gt;eval($lua,$arr_hash_key,count($arr_hash_key));</span><br/></pre></td></tr></tbody></table></figure>
<h4 id="2、如果值一样则删除"><a href="#2、如果值一样则删除" class="headerlink" title="2、如果值一样则删除"></a>2、如果值一样则删除</h4><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line">if redis.call(&#34;get&#34;,KEYS[1]) == ARGV[1] then</span><br/><span class="line">    return redis.call(&#34;del&#34;,KEYS[1])</span><br/><span class="line">else</span><br/><span class="line">    return 0</span><br/><span class="line">end</span><br/></pre></td></tr></tbody></table></figure>
<h2 id="注意："><a href="#注意：" class="headerlink" title="注意："></a>注意：</h2><h4 id="redis的proxy可能不支持lua脚本。至少百度修改的Twemproxy就不支持proxy。所以如果你的redis集群是分布式，含有多个分片。使用了proxy，需要先判断能不能使用lua脚本。"><a href="#redis的proxy可能不支持lua脚本。至少百度修改的Twemproxy就不支持proxy。所以如果你的redis集群是分布式，含有多个分片。使用了proxy，需要先判断能不能使用lua脚本。" class="headerlink" title="redis的proxy可能不支持lua脚本。至少百度修改的Twemproxy就不支持proxy。所以如果你的redis集群是分布式，含有多个分片。使用了proxy，需要先判断能不能使用lua脚本。"></a>redis的proxy可能不支持lua脚本。至少百度修改的Twemproxy就不支持proxy。所以如果你的redis集群是分布式，含有多个分片。使用了proxy，需要先判断能不能使用lua脚本。</h4>