---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
<h3 id="4-Statements"><a href="#4-Statements" class="headerlink" title="4 Statements"></a>4 Statements</h3>
<ul>
<li><p>多重赋值：先求出所有值，再赋值</p>
</li>
<li><p>交换两值</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">x, y = y, x</span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p>多余的变量赋nil，多余的值丢弃</p>
</li>
<li><p>用局部变量保护全局变量</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">local</span> foo = foo</span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p><code>if-then-elseif-else-end</code>充当<code>switch</code></p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/><span class="line">11</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">if</span> op == <span class="string">&#34;+&#34;</span> <span class="keyword">then</span></span><br/><span class="line">	r = a + b</span><br/><span class="line"><span class="keyword">elseif</span> op == <span class="string">&#34;-&#34;</span> <span class="keyword">then</span></span><br/><span class="line">	r = a - b</span><br/><span class="line"><span class="keyword">elseif</span> op == <span class="string">&#34;*&#34;</span> <span class="keyword">then</span></span><br/><span class="line">	r = a * b</span><br/><span class="line"><span class="keyword">elseif</span> op == <span class="string">&#34;/&#34;</span> <span class="keyword">then</span></span><br/><span class="line">	r = a / b</span><br/><span class="line"><span class="keyword">else</span></span><br/><span class="line">	<span class="built_in">error</span>(<span class="string">&#34;invalid operation&#34;</span>)</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p><code>numeric for</code></p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">for</span> var = exp1, exp2, exp3 <span class="keyword">do</span></span><br/><span class="line">	&lt;somethin&gt;</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<p>exp3可选，为变量的增量<br/></p><figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">for</span> i= <span class="number">10</span>， <span class="number">1</span>， <span class="number">-1</span> <span class="keyword">do</span> <span class="built_in">print</span>(i) <span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure><p></p>
<ul>
<li><p><code>math.huge</code>用作无穷大</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">for</span> i = <span class="number">1</span>, <span class="built_in">math</span>.<span class="built_in">huge</span> <span class="keyword">do</span></span><br/><span class="line">	<span class="keyword">if</span>(<span class="number">0.3</span>*i^<span class="number">3</span> - <span class="number">20</span>*i^<span class="number">2</span> - <span class="number">500</span> &gt;= <span class="number">0</span>) <span class="keyword">then</span></span><br/><span class="line">		<span class="built_in">print</span>(i)</span><br/><span class="line">		<span class="keyword">break</span></span><br/><span class="line">	<span class="keyword">end</span></span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p>for循环的表达式只求值一次，变量为局部变量</p>
</li>
<li><p><code>generic for</code></p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">for</span> k, v <span class="keyword">in</span> <span class="built_in">pairs</span>(t) <span class="keyword">do</span> <span class="built_in">print</span>(k, v) <span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p>用<code>generic for</code>和迭代器建立反向表</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line">days = {<span class="string">&#34;Sunday&#34;</span>, <span class="string">&#34;Monday&#34;</span>, <span class="string">&#34;Tuesday&#34;</span>, <span class="string">&#34;Wednesday&#34;</span>, <span class="string">&#34;Thursday&#34;</span>， <span class="string">&#34;Friday&#34;</span>, <span class="string">&#34;Saturday&#34;</span>}</span><br/><span class="line">revDays = {}</span><br/><span class="line"><span class="keyword">for</span> k, v <span class="keyword">in</span> <span class="built_in">pairs</span>(days) <span class="keyword">do</span></span><br/><span class="line">	revDays[v] = k</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p><code>goto</code>实现<code>redo</code>和<code>continue</code></p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">while</span> some_condition <span class="keyword">do</span></span><br/><span class="line">	::redo::</span><br/><span class="line">	<span class="keyword">if</span> some_other_condition <span class="keyword">then</span> <span class="keyword">goto</span> continue</span><br/><span class="line">	<span class="keyword">else</span> <span class="keyword">if</span> yet_another_condition <span class="keyword">then</span> <span class="keyword">goto</span> redo</span><br/><span class="line">	<span class="keyword">end</span></span><br/><span class="line">	&lt;some code&gt;</span><br/><span class="line">	::continue::</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p><code>goto</code>的域</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">while</span> some_condition <span class="keyword">do</span></span><br/><span class="line">	<span class="keyword">if</span> some_other_condition <span class="keyword">then</span> <span class="keyword">goto</span> continue <span class="keyword">end</span></span><br/><span class="line">	<span class="keyword">local</span> var = something</span><br/><span class="line">	&lt;some code&gt;</span><br/><span class="line">	::continue::</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<p>局部变量的域结束在最后一个非空语句，<code>continue</code>标签出现在最后一个非空语句之后，所以<code>goto</code>并没有跳入<code>var</code>的域。</p>
<ul>
<li><code>goto</code>不能从一个函数内跳到函数外</li>
</ul>