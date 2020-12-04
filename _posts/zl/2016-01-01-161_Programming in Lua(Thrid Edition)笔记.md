---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
<h3 id="3-Expressions"><a href="#3-Expressions" class="headerlink" title="3 Expressions"></a>3 Expressions</h3>
<ul>
<li><p><code>a % b == a - math.floor(a / b) * b</code>，可以用于浮点数，<code>x % 1</code>为x的小数部分，<code>x - x % 1</code>为x的整数部分，<code>x-x%0.01</code>可以保留x两位小数，也可以用于角度对360取模和弧度对2PI取模<code>angle%(2*math.pi)</code></p>
</li>
<li><p><code>~=</code>与<code>==</code>作用相反</p>
</li>
<li><p>Lua可以根据本地字符编码比较字符串大小</p>
</li>
<li><p>逻辑操作符<code>and</code>，<code>or</code>，<code>not</code></p>
</li>
<li><p><code>x = x or v</code>相当于</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">if</span> <span class="keyword">not</span> x <span class="keyword">then</span> x = v <span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<p>可以在x未被赋值时对其赋值</p>
<ul>
<li><p><code>(a and b) or c</code>与C的<code>a ? b : c</code>等价</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line"><span class="built_in">max</span> = (x &gt; y) <span class="keyword">and</span> x <span class="keyword">or</span> y</span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p><code>..</code>用于连接字符串</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line"><span class="built_in">print</span>(<span class="string">&#34;Hello &#34;</span> .. <span class="string">&#34;world&#34;</span>) </span><br/><span class="line"><span class="built_in">print</span>(<span class="number">0</span> .. <span class="number">1</span>)              <span class="comment">--&gt; 01</span></span><br/><span class="line"><span class="built_in">print</span>(<span class="number">000</span> .. <span class="number">01</span>)           <span class="comment">--&gt; 01</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<p><code>print(000 .. 01)</code>中会先求值，再转换为字符串，再连接</p>
<ul>
<li><p><code>a[#a + ] = v</code>把v加到列表a的末尾</p>
</li>
<li><p>关于有洞的列表的长度</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/></pre></td><td class="code"><pre><span class="line">c = {<span class="number">1</span>, <span class="number">2</span>, <span class="number">3</span>}</span><br/><span class="line"><span class="built_in">print</span>(#c) <span class="comment">--&gt; 3</span></span><br/><span class="line">c[<span class="number">2</span>] = <span class="literal">nil</span></span><br/><span class="line"><span class="built_in">print</span>(#c) <span class="comment">--&gt; 3</span></span><br/><span class="line">c[<span class="number">3</span>] = <span class="literal">nil</span></span><br/><span class="line"><span class="built_in">print</span>(#c) <span class="comment">--&gt; 1</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p><code>a = {1, 2, 3, nil, nil}</code>与<code>a = {1, 2, 3}</code>相同，长度均为3</p>
</li>
<li><p>优先级：</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/></pre></td><td class="code"><pre><span class="line">^</span><br/><span class="line"><span class="keyword">not</span> # -(unary)</span><br/><span class="line">* / %</span><br/><span class="line">+ -</span><br/><span class="line">..</span><br/><span class="line">&lt; &gt; &lt;= &gt;= ~= ==</span><br/><span class="line"><span class="keyword">and</span></span><br/><span class="line"><span class="keyword">or</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p><code>a = {x = 10, y = 20}</code>比<code>a = {}; a.x = 10; a.y = 20</code>快一些 </p>
</li>
<li><p>混用list风格和record风格构造table</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/></pre></td><td class="code"><pre><span class="line">polyline = {</span><br/><span class="line">color = <span class="string">&#34;blue&#34;</span>,</span><br/><span class="line">thickness = <span class="number">2</span>,</span><br/><span class="line">npoints = <span class="number">4</span>,</span><br/><span class="line">{x = <span class="number">0</span>, y = <span class="number">0</span>},   <span class="comment">-- polyline[1]</span></span><br/><span class="line">{x = <span class="number">-10</span>, y = <span class="number">0</span>}, <span class="comment">-- polyline[2]</span></span><br/><span class="line">{x = <span class="number">-10</span>, y = <span class="number">1</span>}, <span class="comment">-- polyline[3]</span></span><br/><span class="line">{x = <span class="number">0</span>, y = <span class="number">1</span>},   <span class="comment">-- polyline[4]</span></span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p>几种特殊的table构造方式：</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line">a = {[<span class="string">&#34;+&#34;</span>] = <span class="string">&#34;add&#34;</span>, [<span class="string">&#34;-&#34;</span>] = <span class="string">&#34;sub&#34;</span>,</span><br/><span class="line">     [<span class="string">&#34;*&#34;</span>] = <span class="string">&#34;mul&#34;</span>, [<span class="string">&#34;/&#34;</span>] = <span class="string">&#34;div&#34;</span>}</span><br/><span class="line">b = {[<span class="number">1</span>]=<span class="string">&#34;red&#34;</span>, [<span class="number">2</span>]=<span class="string">&#34;green&#34;</span>, [<span class="number">3</span>]=<span class="string">&#34;blu&#34;</span>,}</span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<p>最后一个entry后的逗号可选</p>
<ul>
<li>在table的构造中，可以用分号代替逗号来分隔不同部分，例如list和record</li>
</ul>