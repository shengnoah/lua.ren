---
layout: post
title: Redis Lua scripts debugger使用 
tags: [lua文章]
categories: [topic]
---
<h2 id="背景说明"><a href="#背景说明" class="headerlink" title="背景说明"></a>背景说明</h2><p>使用<code>Redis</code>开发分布式应用时，难免会遇到需要使用分布式锁来确保某一小段逻辑的原子性操作，如：当存在某个<code>key</code>对应的值<code>A</code>大于值<code>B</code>时，则返回<code>false</code>；否则<code>A + 1</code>。试想一下，如果用到分布式锁，是不是有点感觉像是杀鸡用宰牛刀？</p>
<p>由于<code>Redis</code>的操作都是原子性的，所以我们可以将如上所述的类似逻辑采用<code>Lua</code>脚本表述作为一个原子任务向<code>redisClient</code>提交，可以避免采用分布式锁。如脚本:</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/><span class="line">11</span><br/><span class="line">12</span><br/><span class="line">13</span><br/><span class="line">14</span><br/><span class="line">15</span><br/><span class="line">16</span><br/><span class="line">17</span><br/><span class="line">18</span><br/><span class="line">19</span><br/><span class="line">20</span><br/><span class="line">21</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">local</span> key = KEYS[<span class="number">1</span>]</span><br/><span class="line"><span class="keyword">local</span> expire_time = <span class="built_in">tonumber</span>(ARGV[<span class="number">1</span>])</span><br/><span class="line"><span class="keyword">local</span> max_count = <span class="built_in">tonumber</span>(ARGV[<span class="number">2</span>])</span><br/><span class="line"></span><br/><span class="line"></span><br/><span class="line"><span class="keyword">local</span> is_key_exists = redis.call(<span class="string">&#34;EXISTS&#34;</span>, key)</span><br/><span class="line"></span><br/><span class="line"><span class="comment">-- 存在</span></span><br/><span class="line"><span class="keyword">if</span> is_key_exists == <span class="number">1</span> <span class="keyword">then</span></span><br/><span class="line">    <span class="keyword">local</span> key_count = redis.call(<span class="string">&#34;get&#34;</span>, key)</span><br/><span class="line">    <span class="keyword">if</span> <span class="built_in">tonumber</span>(key_count) &gt;= max_count <span class="keyword">then</span></span><br/><span class="line">        <span class="keyword">return</span> <span class="literal">false</span>;</span><br/><span class="line">    <span class="keyword">else</span></span><br/><span class="line">        redis.call(<span class="string">&#34;incr&#34;</span>, key)</span><br/><span class="line">        <span class="keyword">return</span> <span class="literal">true</span>;</span><br/><span class="line">    <span class="keyword">end</span></span><br/><span class="line"><span class="keyword">else</span></span><br/><span class="line">    redis.call(<span class="string">&#34;incr&#34;</span>, key);</span><br/><span class="line">    redis.call(<span class="string">&#34;expire&#34;</span>, key, expire_time)</span><br/><span class="line">    <span class="keyword">return</span> <span class="literal">true</span>;</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>

<p>但此时发现，在<code>Redis</code>中调试是个困难的事情。但幸好，从<code>Redis 3,2</code>版本开始，提供了<code>Lua script debugger</code>来解决此问题。</p>
<h2 id="Redis-Lua调试器特点"><a href="#Redis-Lua调试器特点" class="headerlink" title="Redis Lua调试器特点"></a>Redis Lua调试器特点</h2><p><code>Redis Lua</code>调试器代号为<code>LDB</code>，特点如下：</p>
<ul>
<li>支持单步调试</li>
<li>支持静态和动态断点</li>
<li>支持将被调试的脚本载入至调试终端</li>
<li>支持对<code>Lua</code>变量进行观察</li>
<li>支持追踪脚本执行的<code>Redis</code>命令</li>
<li>支持以美观方式打印<code>Redis</code>值及<code>Lua</code>值</li>
<li>能够在无线循环及长时间执行步骤中模拟出断点</li>
<li>采用<code>CS</code>模型，<code>S</code>即<code>Redis</code>服务器，<code>C</code>即<code>redis-cli</code>客户端</li>
<li>默认情况下，每个调试会话都是<code>fork</code>子进程进行的。所以调试过程中不会影响<code>Redis</code>其他操作；调试结束后，本次会话中的内容都会被回滚</li>
<li>你也可以将调试会话设置成同步模式，但是此时必须注意调试时会影响<code>Redis</code>所有其他的操作，且调试会话产生的结果都会被保存下来</li>
</ul>
<h2 id="Redis-Lua调试器快速入门"><a href="#Redis-Lua调试器快速入门" class="headerlink" title="Redis Lua调试器快速入门"></a>Redis Lua调试器快速入门</h2><p>想看视频的直接点<a href="https://www.bilibili.com/video/av9437433/" target="_blank" rel="noopener noreferrer">这里</a>，这是<code>Redis</code>之父录制的视频。</p>
<p>想看文字的继续。以我们在刚开始提供的<code>Lua</code>脚本，假设其目录为<code>~/Desktop/tmp/test.lua</code>。此时执行命令（需要先<code>cd</code>到<code>redis-cli</code>命令目录下）：</p>
<figure class="highlight shell"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line">-- async mode</span><br/><span class="line">redis-cli --ldb --eval redis_lua_file redis_key , param1 param2</span><br/><span class="line"></span><br/><span class="line">-- sync mode</span><br/><span class="line">redis-cli --ldb-sync-mode --eval redis_lua_file redis_key , param1 param2</span><br/></pre></td></tr></tbody></table></figure>

<figure class="highlight shell"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/></pre></td><td class="code"><pre><span class="line"> ~  redis-cli --ldb --eval ~/Desktop/tmp/test.lua key , 1 3</span><br/><span class="line">Lua debugging session started, please use:</span><br/><span class="line">quit    -- End the session.</span><br/><span class="line">restart -- Restart the script in debug mode again.</span><br/><span class="line">help    -- Show Lua script debugging commands.</span><br/><span class="line"></span><br/><span class="line">* Stopped at 6, stop reason = step over</span><br/><span class="line"><span class="bash"> 6   <span class="built_in">local</span> key = KEYS[1]</span></span><br/><span class="line">lua debugger&gt;</span><br/></pre></td></tr></tbody></table></figure>

<blockquote>
<p>注意：<code>redis-cli</code>里的逗号前后都是有空格的。我这里示例是本地有开启<code>redis-server</code>，如果没有开启也可以连接到远程<code>redis server</code>进行调试。另外，<code>key</code>表示的是要传入脚本中处理的<code>redis key</code>，其后的1和3则是参数，中间用空格隔开。</p>
</blockquote>
<p>之后所要做的就是在<code>debugger</code>里输入<code>s</code>命令回车，表示执行下一步，直至程序结束或有异常退出为止。</p>
<h2 id="Redis-Lua-debug命令"><a href="#Redis-Lua-debug命令" class="headerlink" title="Redis Lua debug命令"></a>Redis Lua debug命令</h2><p>括号中字母代表命令缩写。</p>
<p><img src="https://s2.ax1x.com/2019/09/11/nwnpOU.png" alt="redis_lua_debug命令.png"/></p>
<h2 id="参考文章"><a href="#参考文章" class="headerlink" title="参考文章"></a>参考文章</h2><ol>
<li><a href="https://redis.io/topics/ldb" target="_blank" rel="noopener noreferrer">Redis Lua scripts debugger</a></li>
</ol>