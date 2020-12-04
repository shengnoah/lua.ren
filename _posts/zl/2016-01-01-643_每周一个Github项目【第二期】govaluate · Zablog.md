---
layout: post
title: 每周一个Github项目【第二期】govaluate · Zablog 
tags: [lua文章]
categories: [topic]
---
<p>golang环境下任意表达式的求值 // Arbitrary expression evaluation for golang</p>

<table>
<thead>
<tr>
<th>名称</th>
<th>govaluate</th>
</tr>
</thead>
<tbody>
<tr>
<td>地址</td>
<td><a href="https://github.com/Knetic/govaluate" target="_blank" rel="noopener noreferrer">Github</a></td>
</tr>
<tr>
<td>作者</td>
<td>Knetic等</td>
</tr>
<tr>
<td>brief intro</td>
<td>Arbitrary expression evaluation for golang</td>
</tr>
<tr>
<td>简要介绍</td>
<td>golang环境下任意表达式的求值</td>
</tr>
<tr>
<td>LICENSE</td>
<td>MIT</td>
</tr>
<tr>
<td>Stars</td>
<td>245</td>
</tr>
</tbody>
</table>
<p>govaluate提供了任意类似C语言的算术/字符串表达式的求值。</p>
<h2 id="为什么你不应该直接在代码中书写表达式"><a href="#为什么你不应该直接在代码中书写表达式" class="headerlink" title="为什么你不应该直接在代码中书写表达式"></a>为什么你不应该直接在代码中书写表达式</h2><p>有些时候，你并没有办法提前得知表达式的样子，或者你希望表达式可设置。如果你有一堆运行在你的应用上的数据，或者你想要允许你的用户自定义一些内容，或者你写的是一个监控框架，可以获得很多metrics信息，然后进行一些公式计算，那么这个库就会非常有用。</p>
<h2 id="如何使用"><a href="#如何使用" class="headerlink" title="如何使用"></a>如何使用</h2><p>可以创建一个新的EvaluableExpression，然后调用它的”Evaluate”方法。</p>
<figure class="highlight go"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div></pre></td><td class="code"><pre><div class="line">   expression, err := govaluate.NewEvaluableExpression(<span class="string">&#34;10 &gt; 0&#34;</span>);</div><div class="line">result, err := expression.Evaluate(<span class="literal">nil</span>);</div><div class="line"></div></pre></td></tr></tbody></table></figure>
<p>那么，如何使用参数？</p>
<figure class="highlight go"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div></pre></td><td class="code"><pre><div class="line">expression, err := govaluate.NewEvaluableExpression(<span class="string">&#34;foo &gt; 0&#34;</span>);</div><div class="line"></div><div class="line">parameters := <span class="built_in">make</span>(<span class="keyword">map</span>[<span class="keyword">string</span>]<span class="keyword">interface</span>{}, <span class="number">8</span>)</div><div class="line">parameters[<span class="string">&#34;foo&#34;</span>] = <span class="number">-1</span>;</div><div class="line"></div><div class="line">result, err := expression.Evaluate(parameters);</div><div class="line"><span class="comment">// result is now set to &#34;false&#34;, the bool value.</span></div></pre></td></tr></tbody></table></figure>
<p>这很棒，但是这些基本上可以使用代码直接实现。那么如果计算中牵扯到一些数学计算呢？</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div><div class="line">8</div></pre></td><td class="code"><pre><div class="line">expression, err := govaluate.NewEvaluableExpression(&#34;(requests_made * requests_succeeded / 100) &gt;= 90&#34;);</div><div class="line"></div><div class="line">parameters := make(map[string]interface{}, 8)</div><div class="line">parameters[&#34;requests_made&#34;] = 100;</div><div class="line">parameters[&#34;requests_succeeded&#34;] = 80;</div><div class="line"></div><div class="line">result, err := expression.Evaluate(parameters);</div><div class="line">// result is now set to &#34;false&#34;, the bool value.</div></pre></td></tr></tbody></table></figure>
<p>上述例子返回的都是布尔值，事实上，它是可以返回数字的。</p>
<figure class="highlight golang"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div><div class="line">8</div></pre></td><td class="code"><pre><div class="line">expression, err := govaluate.NewEvaluableExpression(<span class="string">&#34;(mem_used / total_mem) * 100&#34;</span>);</div><div class="line"></div><div class="line">parameters := <span class="built_in">make</span>(<span class="keyword">map</span>[<span class="keyword">string</span>]<span class="keyword">interface</span>{}, <span class="number">8</span>)</div><div class="line">parameters[<span class="string">&#34;total_mem&#34;</span>] = <span class="number">1024</span>;</div><div class="line">parameters[<span class="string">&#34;mem_used&#34;</span>] = <span class="number">512</span>;</div><div class="line"></div><div class="line">result, err := expression.Evaluate(parameters);</div><div class="line"><span class="comment">// result is now set to &#34;50.0&#34;, the float64 value.</span></div></pre></td></tr></tbody></table></figure>
<p>你也可以做一些日期的转化，只要符合RF3339,ISO8061,Unix Date，或者ruby日期格式标准即可。如果你还是不太确定，那么可以看一下支持的<a href="https://github.com/Knetic/govaluate/blob/0580e9b47a69125afa0e4ebd1cf93c49eb5a43ec/parsing.go#L258" target="_blank" rel="noopener noreferrer">日期标准</a>。</p>
<figure class="highlight go"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div></pre></td><td class="code"><pre><div class="line">   expression, err := govaluate.NewEvaluableExpression(<span class="string">&#34;&#39;2014-01-02&#39; &gt; &#39;2014-01-01 23:59:59&#39;&#34;</span>);</div><div class="line">result, err := expression.Evaluate(<span class="literal">nil</span>);</div><div class="line"></div><div class="line"><span class="comment">// result is now set to true</span></div></pre></td></tr></tbody></table></figure>
<p>表达式只需要进行一次句法分析，就可以多次复用。</p>
<figure class="highlight go"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div></pre></td><td class="code"><pre><div class="line">   expression, err := govaluate.NewEvaluableExpression(<span class="string">&#34;response_time &lt;= 100&#34;</span>);</div><div class="line">parameters := <span class="built_in">make</span>(<span class="keyword">map</span>[<span class="keyword">string</span>]<span class="keyword">interface</span>{}, <span class="number">8</span>)</div><div class="line"></div><div class="line"><span class="keyword">for</span> {</div><div class="line">	parameters[<span class="string">&#34;response_time&#34;</span>] = pingSomething();</div><div class="line">	result, err := expression.Evaluate(parameters)</div><div class="line">}</div></pre></td></tr></tbody></table></figure>
<p>关于执行顺序，本库支持正常C标准的执行顺序。编写表达式时，请确保您正确地书写操作符，或使用括号来明确表达式的哪些部分应先运行。</p>
<p>govaluate采用或者[]来完成转义。</p>
<p>支持自定义函数</p>
<p>支持简单的结构体（访问器）</p>
<h2 id="运算符支持"><a href="#运算符支持" class="headerlink" title="运算符支持"></a>运算符支持</h2><p>ruleplatform的表达式引擎支持以下运算：<br/>二元计算符 : + - / <em> &amp; | ^ *</em> % &gt;&gt; &lt;&lt;<br/>二元比较符 : &gt; &gt;= &lt; &lt;= == != =~ !~<br/>逻辑操作符 : || &amp;&amp;<br/>括号 : ( )<br/>数组相关 : , IN (例子1 IN (1, 2, ‘foo’)，返回值true)<br/>一元计算符 : ! - ~<br/>三元运算符 : ? :<br/>空值聚合符: ??</p>
<p>更多内容请查看<a href="https://github.com/Knetic/govaluate" target="_blank" rel="noopener noreferrer">https://github.com/Knetic/govaluate</a></p>