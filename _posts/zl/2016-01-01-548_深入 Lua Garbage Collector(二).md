---
layout: post
title: 深入 Lua Garbage Collector(二) 
tags: [lua文章]
categories: [topic]
---
<p>

</p><p>这一篇我们主要介绍一下 <strong>Lua</strong> 的 <strong>GC</strong> 机制</p>
<h2 id="Lua垃圾回收器函数"><a href="#Lua垃圾回收器函数" class="headerlink" title="Lua垃圾回收器函数"></a>Lua垃圾回收器函数</h2><p><strong>Lua</strong> 提供了以下函数 <strong>collectgarbage ([opt [, arg]])</strong> 用来控制自动内存管理:</p>
<ul>
<li><p>collectgarbage(“collect”): 做一次完整的垃圾收集循环。通过参数 opt 它提供了一组不同的功能：</p>
</li>
<li><p>collectgarbage(“count”): 以 K 字节数为单位返回 Lua 使用的总内存数。 这个值有小数部分，所以只需要乘上 1024 就能得到 Lua 使用的准确字节数（除非溢出）。</p>
</li>
<li><p>collectgarbage(“restart”): 重启垃圾收集器的自动运行。</p>
</li>
<li><p>collectgarbage(“setpause”): 将 arg 设为收集器的 间歇率 。 返回 间歇率 的前一个值。</p>
</li>
<li><p>collectgarbage(“setstepmul”): 返回 步进倍率 的前一个值。</p>
</li>
<li><p>collectgarbage(“step”): 单步运行垃圾收集器。 步长”大小”由 arg 控制。 传入 0 时，收集器步进（不可分割的）一步。 传入非 0 值， 收集器收集相当于 Lua 分配这些多（K 字节）内存的工作。 如果收集器结束一个循环将返回 true 。</p>
</li>
<li><p>collectgarbage(“stop”): 停止垃圾收集器的运行。 在调用重启前，收集器只会因显式的调用运行。</p>
</li>
</ul>

<p>以下演示了一个简单的垃圾回收实例:</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div><div class="line">8</div><div class="line">9</div><div class="line">10</div><div class="line">11</div></pre></td><td class="code"><pre><div class="line">mytable = {<span class="string">&#34;apple&#34;</span>, <span class="string">&#34;orange&#34;</span>, <span class="string">&#34;banana&#34;</span>}</div><div class="line"></div><div class="line"><span class="built_in">print</span>(<span class="built_in">collectgarbage</span>(<span class="string">&#34;count&#34;</span>))</div><div class="line"></div><div class="line">mytable = <span class="keyword">nil</span></div><div class="line"></div><div class="line"><span class="built_in">print</span>(<span class="built_in">collectgarbage</span>(<span class="string">&#34;count&#34;</span>))</div><div class="line"></div><div class="line"><span class="built_in">print</span>(<span class="built_in">collectgarbage</span>(<span class="string">&#34;collect&#34;</span>))</div><div class="line"></div><div class="line"><span class="built_in">print</span>(<span class="built_in">collectgarbage</span>(<span class="string">&#34;count&#34;</span>))</div></pre></td></tr></tbody></table></figure>
<hr/>
<h2 id="基本算法"><a href="#基本算法" class="headerlink" title="基本算法"></a>基本算法</h2><p>基本算法就是我们之前的 <strong>Mark &amp; Sweep</strong></p>
<blockquote>
<p>首先，系统管理着所有已经创建了的对象。每个对象都有对其他对象的引用。 <strong>root</strong> 集合代表着已知的系统级别的对象引用。我们从 <strong>root</strong> 集合出发，就可以访问到系统引用到的所有对象。而没有被访问到的对象就是垃圾对象，需要被销毁。</p>
</blockquote>
<p>我们可以将所有对象分成三个状态：</p>
<p>①. White状态，也就是待访问状态。表示对象还没有被垃圾回收的标记过程访问到。</p>
<p>②. Gray状态，也就是待扫描状态。表示对象已经被垃圾回收访问到了，但是对象本身对于其他对象的引用还没有进行遍历访问。</p>
<p>③. Black状态，也就是已扫描状态。表示对象已经被访问到了，并且也已经遍历了对象本身对其他对象的引用</p>
<p>伪代码如下：</p>
<figure class="highlight gcode"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div><div class="line">8</div><div class="line">9</div><div class="line">10</div><div class="line">11</div><div class="line">12</div><div class="line">13</div><div class="line">14</div><div class="line">15</div><div class="line">16</div><div class="line">17</div></pre></td><td class="code"><pre><div class="line">当前所有对象都是White状态;  </div><div class="line">将root集合引用到的对象从White设置成Gray，并放到Gray集合中;  </div><div class="line"><span class="keyword">while</span>  </div><div class="line">{  </div><div class="line">    从Gray集合中移除一个对象O，并将O设置成Black状态;  </div><div class="line">    for<span class="comment">(O中每一个引用到的对象O1)</span> {  </div><div class="line">        <span class="keyword">if</span><span class="comment">(O1在White状态)</span> {  </div><div class="line">            将从White设置成Gray，并放到到Gray集合中；  </div><div class="line">        }  </div><div class="line">    }  </div><div class="line">}  </div><div class="line">for<span class="comment">(任意一个对象O)</span>{  </div><div class="line">    <span class="keyword">if</span><span class="comment">(O在White状态)</span>  </div><div class="line">        销毁对象O;  </div><div class="line">    else  </div><div class="line">        将O设置成White状态;  </div><div class="line">}</div></pre></td></tr></tbody></table></figure>
<hr/>
<h2 id="Incremental-Garbage-Collection"><a href="#Incremental-Garbage-Collection" class="headerlink" title="Incremental Garbage Collection"></a>Incremental Garbage Collection</h2><blockquote>
<p>上面的算法如果一次性执行，在对象很多的情况下，会执行很长时间，严重影响程序本身的响应速度。其中一个解决办法就是，可以将上面的算法分步执行，这样每个步骤所耗费的时间就比较小了。我们可以将上述算法改为以下下几个步骤:</p>
</blockquote>
<ol>
<li><p>首先标识所有的 <strong>root</strong> 对象</p>
</li>
<li><p>遍历访问所有的 <strong>gray</strong> 对象。如果超出了本次计算量上限，退出等待下一次遍历</p>
</li>
<li><p>销毁垃圾对象</p>
</li>
</ol>
<blockquote>
<p>在每个步骤之间，由于程序可以正常执行，所以会破坏当前对象之间的引用关系。black对象表示已经被扫描的对象，所以他应该不可能引用到一个white对象。当程序的改变使得一个black对象引用到一个white对象时，就会造成错误。解决这个问题的办法就是设置barrier。barrier在程序正常运行过程中，监控所有的引用改变。如果一个black对象需要引用一个white对象，存在两种处理办法：</p>
</blockquote>
<p>①. 将white对象设置成gray，并添加到gray列表中等待扫描。这样等于帮助整个GC的标识过程向前推进了一步。<br/>②. 将black对象该回成gray，并添加到gray列表中等待扫描。这样等于使整个GC的标识过程后退了一步。</p>
<blockquote>
<p>这种垃圾回收方式被称为 <strong>Incremental Garbage Collection</strong> (简称为 <strong>IGC</strong> ， <strong>Lua</strong>所采用的就是这种方法。使用 <strong>IGC</strong> 并不是没有代价的。 <strong>IGC</strong> 所检测出来的垃圾对象集合比实际的集合要小，也就是说，有些在 <strong>GC</strong> 过程中变成垃圾的对象，有可能在本轮 <strong>GC</strong> 中检测不到。不过，这些残余的垃圾对象一定会在下一轮 <strong>GC</strong> 被检测出来，不会造成泄露。</p>
</blockquote>
<p>在下一篇中我们将会具体的来看一看 <strong>Lua</strong> 的 <strong>GC源码</strong></p>