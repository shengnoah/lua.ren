---
layout: post
title: How TLA+ Evaluation Next Action 
tags: [lua文章]
categories: [topic]
---
<h2 id="tlc是如何计算状态的">TLC是如何计算状态的</h2>

<p>当TLC计算一个invariant，直接计算值，返回TRUE/FALSE
当TLC计算Init和Next，返回一个状态集合（这个集合被加入到sq中）:</p>

<ol>
  <li>Init: 所有可能的初始状态；</li>
  <li>Next：所有可能的后继状态；</li>
</ol>

<h2 id="tlc如何计算next">TLC如何计算Next</h2>

<ul>
  <li>状态：是对变量的赋值；</li>
  <li>TLC计算一个状态s的后继状态：
    <ol>
      <li>对所有unprimed变量进行赋值；</li>
      <li>对所有的primed变量赋值为null；</li>
      <li>开始计算next Action；</li>
    </ol>
  </li>
  <li>TLC在计算Next Action和普通的表达式是不同的</li>
</ul>

<h3 id="第一个不同点">第一个不同点</h3>

<ul>
  <li>TLC对‘或’表达式并不是从左到右计算：
    <ul>
      <li>当计算 A1 / … / An，首先拆分成n个子表达式；</li>
      <li>当计算E x in S : p，对于S中的每个元素，拆分成若干个子表达式；</li>
      <li>P =&gt; Q，等价于(非P) / Q</li>
    </ul>
  </li>
  <li>
    <p>举个例子</p>

    <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  (A =&gt; B) / ( C / (E i in S : D(i) / E)) 
</code></pre></div>    </div>

    <p>拆分成3个子表达式：</p>

    <ol>
      <li>非A；</li>
      <li>B；</li>
      <li>C / (E i in S : D(i) / E)；</li>
    </ol>

    <p>计算第3个表达式的过程是：</p>

    <ol>
      <li>计算C</li>
      <li>如果C为TRUE，把后边的E i in S : D(i) / E 根据S中的元素i拆分成多个表达式D(i) / E;</li>
      <li>计算D(i) / E时，应用同样的规则，先计算D(i)</li>
      <li>
        <p>如果D(i)为TRUE，计算E</p>

        <p># 第二个不同点</p>
      </li>
    </ol>
  </li>
  <li>如何计算primed变量
    <ul>
      <li>计算x’ = e时，首先把x’赋值为null，然后计算e的值给x’，返回TRUE；</li>
      <li>计算x’ in S，等价于 E v in S : x’ = v;</li>
      <li>UNCHANGED«e1, … , en»，等价于 (UNCHANGED e1) / … / (UNCHANGED en)</li>
    </ul>
  </li>
  <li>当primed变量没有被赋值时会报错；</li>
  <li>当一个‘与’返回FALSE，这个子表达式计算就停止了，没有找到任何状态；</li>
  <li>当一个表达式计算完毕，就找到一些状态，这些状态就是primed变量的赋值；</li>
</ul>

<h2 id="一个具体的例子">一个具体的例子</h2>
<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/ / x&#39; in 1..Len(y)
	/ y&#39; = Append(Tail(y), x&#39;)
/ / x&#39; = x + 1
	/ y&#39; = Append(y, x&#39;)
</code></pre></div></div>
<ul>
  <li>假设Init初始化之后的状态是： x = 1, y = «2, 3»
    <ul>
      <li>由于最外层是两个‘或’，TLC把表达式拆分成两个子表达式；</li>
      <li>计算第一个子表达式：x’ in 1..Len(y) / y’ = Append(Tail(y), x’) 这个表达式一个‘与’因此从左往右依次计算，由于Len(y)为2，因此，这个表达式被拆成两个:
        <ul>
          <li>第一个：
            <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  / x&#39; = 1
  / y&#39; = Append(Tail(y), x&#39;)
</code></pre></div>            </div>
            <p>得到x = 1, y = «3, 1»</p>
          </li>
          <li>第二个：
            <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  / x&#39; = 2
  / y&#39; = Append(Tail(y), x&#39;)
</code></pre></div>            </div>
            <p>得到x = 2, y = «3, 2»</p>
          </li>
        </ul>
      </li>
      <li>计算第二个子表达式：这是一个‘与’表达式，从左往右依次计算两个
        <ul>
          <li>计算第一个x = 2</li>
          <li>计算第二个y = «2, 3, 2»</li>
        </ul>
      </li>
      <li>整个Next的后继状态一共有3个。</li>
    </ul>
  </li>
  <li>假设Init初始化之后的状态是： x = 1, y = « »
    <ul>
      <li>由于最外层是两个‘或’，TLC把表达式拆分成两个子表达式；</li>
      <li>
        <p>计算第一个子表达式：<code class="highlighter-rouge">x&#39; in 1..Len(y) / y&#39; = Append(Tail(y), x&#39;)</code>
  由于Len(y)为0，第一个‘与’的子表达式：</p>

        <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  / x&#39; in 1..0
  / y&#39; = Append(Tail(y), x&#39;)
</code></pre></div>        </div>
      </li>
    </ul>

    <p>拆分后为：</p>

    <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  / i in {} : x&#39; = i
  / y&#39; = Append(Tail(y), x&#39;)
</code></pre></div>    </div>

    <p>这个表达式是一个‘与’，从左往右依次计算，第一个表达式返回FALSE，计算终止</p>
    <ul>
      <li>计算第二个子表达式：
        <div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  / x&#39; = 2
  / y&#39; = Append(Tail(y), x&#39;)
</code></pre></div>        </div>
        <p>得到 x = 2, y = «2, 2»</p>
      </li>
      <li>整个Next的后继状态一共有1个。</li>
    </ul>
  </li>
</ul>

<p>​</p>

<p>​</p>