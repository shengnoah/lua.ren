---
layout: post
title: leetcode  150 evaluate reverse polish notation 
tags: [lua文章]
categories: [topic]
---
<p>Evaluate the value of an arithmetic expression in <a href="http://en.wikipedia.org/wiki/Reverse_Polish_notation" target="_blank" rel="noopener noreferrer">Reverse Polish Notation</a>.</p>
<p>Valid operators are <code>+</code>, <code>-</code>, <code>*</code>, <code>/</code>. Each operand may be an integer or another expression.</p>
<p><strong>Note:</strong></p>
<ul>
<li>Division between two integers should truncate toward zero.</li>
<li>The given RPN expression is always valid. That means the expression would always evaluate to a result and there won’t be any divide by zero operation.</li>
</ul>
<p><strong>Example 1:</strong></p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line">Input: [&#34;2&#34;, &#34;1&#34;, &#34;+&#34;, &#34;3&#34;, &#34;*&#34;]</span><br/><span class="line">Output: 9</span><br/><span class="line">Explanation: ((2 + 1) * 3) = 9</span><br/></pre></td></tr></tbody></table></figure>
<p><strong>Example 2:</strong></p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line">Input: [&#34;4&#34;, &#34;13&#34;, &#34;5&#34;, &#34;/&#34;, &#34;+&#34;]</span><br/><span class="line">Output: 6</span><br/><span class="line">Explanation: (4 + (13 / 5)) = 6</span><br/></pre></td></tr></tbody></table></figure>
<p><strong>Example 3:</strong></p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/></pre></td><td class="code"><pre><span class="line">Input: [&#34;10&#34;, &#34;6&#34;, &#34;9&#34;, &#34;3&#34;, &#34;+&#34;, &#34;-11&#34;, &#34;*&#34;, &#34;/&#34;, &#34;*&#34;, &#34;17&#34;, &#34;+&#34;, &#34;5&#34;, &#34;+&#34;]</span><br/><span class="line">Output: 22</span><br/><span class="line">Explanation: </span><br/><span class="line">  ((10 * (6 / ((9 + 3) * -11))) + 17) + 5</span><br/><span class="line">= ((10 * (6 / (12 * -11))) + 17) + 5</span><br/><span class="line">= ((10 * (6 / -132)) + 17) + 5</span><br/><span class="line">= ((10 * 0) + 17) + 5</span><br/><span class="line">= (0 + 17) + 5</span><br/><span class="line">= 17 + 5</span><br/><span class="line">= 22</span><br/></pre></td></tr></tbody></table></figure>
<hr/>
<ul>
<li>使用栈能够简单的实现</li>
</ul>
<figure class="highlight java"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/><span class="line">11</span><br/><span class="line">12</span><br/><span class="line">13</span><br/><span class="line">14</span><br/><span class="line">15</span><br/><span class="line">16</span><br/><span class="line">17</span><br/><span class="line">18</span><br/><span class="line">19</span><br/><span class="line">20</span><br/><span class="line">21</span><br/><span class="line">22</span><br/><span class="line">23</span><br/><span class="line">24</span><br/><span class="line">25</span><br/><span class="line">26</span><br/><span class="line">27</span><br/><span class="line">28</span><br/><span class="line">29</span><br/><span class="line">30</span><br/><span class="line">31</span><br/><span class="line">32</span><br/><span class="line">33</span><br/><span class="line">34</span><br/><span class="line">35</span><br/><span class="line">36</span><br/><span class="line">37</span><br/></pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span>  </span>{</span><br/><span class="line">    <span class="function"><span class="keyword">public</span> <span class="keyword">int</span> <span class="title">evalRPN</span><span class="params">(String[] tokens)</span> </span>{</span><br/><span class="line">        Stack&lt;Integer&gt; s = <span class="keyword">new</span> Stack&lt;&gt;();</span><br/><span class="line">        <span class="keyword">for</span> (String t: tokens){</span><br/><span class="line">            <span class="keyword">if</span> (<span class="string">&#34;+&#34;</span>.equals(t)){</span><br/><span class="line">                <span class="keyword">int</span> b = s.pop();</span><br/><span class="line">                <span class="keyword">int</span> a = s.pop();</span><br/><span class="line">                <span class="keyword">int</span> ans = a + b;</span><br/><span class="line">                s.push(ans);</span><br/><span class="line">            }</span><br/><span class="line">            <span class="keyword">else</span> <span class="keyword">if</span> (<span class="string">&#34;-&#34;</span>.equals(t)){</span><br/><span class="line">                <span class="keyword">int</span> b = s.pop();</span><br/><span class="line">                <span class="keyword">int</span> a = s.pop();</span><br/><span class="line">                <span class="keyword">int</span> ans = a - b;</span><br/><span class="line">                s.push(ans);</span><br/><span class="line"></span><br/><span class="line">            }</span><br/><span class="line">            <span class="keyword">else</span> <span class="keyword">if</span> (<span class="string">&#34;*&#34;</span>.equals(t)){</span><br/><span class="line">                <span class="keyword">int</span> b = s.pop();</span><br/><span class="line">                <span class="keyword">int</span> a = s.pop();</span><br/><span class="line">                <span class="keyword">int</span> ans = a * b;</span><br/><span class="line">                s.push(ans);</span><br/><span class="line">            }</span><br/><span class="line">            <span class="keyword">else</span> <span class="keyword">if</span> (<span class="string">&#34;/&#34;</span>.equals(t)){</span><br/><span class="line">                <span class="keyword">int</span> b = s.pop();</span><br/><span class="line">                <span class="keyword">int</span> a = s.pop();</span><br/><span class="line">                <span class="keyword">int</span> ans = a / b;</span><br/><span class="line">                s.push(ans);</span><br/><span class="line">            }</span><br/><span class="line">            <span class="keyword">else</span>{</span><br/><span class="line">                <span class="keyword">int</span> num = Integer.parseInt(t);</span><br/><span class="line">                s.push(num);</span><br/><span class="line">            }</span><br/><span class="line">        }</span><br/><span class="line">        <span class="keyword">return</span> s.peek();</span><br/><span class="line">    }</span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>

                

                <hr/>
                
                

                <br/>

                
                
                

                <br/>
                
                
                
                <br/>