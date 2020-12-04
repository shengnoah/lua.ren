---
layout: post
title: Lua学习笔记(3) 关于pairs和ipairs 
tags: [lua文章]
categories: [topic]
---
<p>[TOC]</p>



<h2 id="pairs">pairs</h2>

<p>遍历table</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>local tbTestPairs ={
	[1] = 1,
	nTest_1 = 2,
	szTest = &#34;test&#34;,
	tbTest = {},
	nTest_2,
}

for k, v in pairs(tbTestPairs) do
	print (k, v)
end
</code></pre></div></div>

<p>结果</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>szTest	test
tbTest	table: 000000000033a630
nTest_1	2
</code></pre></div></div>

<h2 id="ipairs">ipairs</h2>
<p>按顺序便利table</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>local tbTestIpairs = {1, 3, 5, nil, 7}
for k, v in ipairs(tbTestIpairs) do
	print(k ,v)
end
</code></pre></div></div>

<p>结果</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>1	1
2	3
3	5
</code></pre></div></div>

<h2 id="总结">总结</h2>

<ul>
  <li>pairs是无序的，ipairs是有序的。这点很重要，如果要按存储顺序处理表中内容的时候，pairs就可能得不到预期的结果；</li>
  <li>ipairs遇到nil就停止打印，所以只打印出来前面三个，7没有打印出来</li>
  <li>pairs中为值nil的元素，即使你定义了key是有效的，也是遍历不出来的。所以要小心对key有要求的应用！</li>
  <li>从上面的结果可以看到tbTestPairs中nTest_2没有打印，<em>这不是因为nTest_2这个key对应的值为nil</em>，而是没有定义key的元素在table里面从前往后依次从[1][2]…开始作为key的，所以nTest_2的key为[1],也覆盖了[1]=1，因此[1]=1也没有打印出来</li>
  <li>存储结构上pairs对应的是hash表，ipairs对应的是数组。这个很多帖子都有说明，看代码也更明白</li>
</ul>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code></code></pre></div></div>

<h1 id="理解层次">理解层次</h1>

<h2 id="学习和理解">学习和理解</h2>

<p>说到底，pairs和ipairs都是lua中的迭代器。迭代器是一种可以遍历一种结合中所有元素的机制[1]。可以看泛型for的语义：</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>for &lt;var-list&gt; in &lt;exp-list&gt; do
	&lt;body&gt;
end
</code></pre></div></div>

<p>for在循环过程中保存了迭代器函数。for做的第一件事就是对in后面的表达式求值，这些表达式应该返回3个值供for保存（这里也就知道如果自己实现迭代器需要根据语法要求来写）：</p>

<ul>
  <li>迭代器函数</li>
  <li>恒定状态</li>
  <li>控制变量</li>
</ul>

<p>在初始化完成之后，for会以恒定状态和控制变量来调用迭代器函数。可以看下面的代码，更加清晰：</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>for var_1, ..., var_n in &lt;explist&gt; do 
	&lt;block&gt;
end

-- 等价于以下代码

do
	local _f, _s, _var = &lt;explist&gt;	-- 返回迭代器函数、恒定状态和控制变量的初值
	while true do
		local var_1, ..., var_n = _f(_s, _var)
		_var = var_1
		if _var == nil then
			break
		end
		&lt;block&gt;
	end
end
</code></pre></div></div>

<p>参考[1]中专门对这个泛型for问题说的非常详细！感谢作者^_^</p>

<h2 id="学以致用">学以致用</h2>

<ul>
  <li>pairs</li>
</ul>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>local function iter(a, k)
	k, v = next(a, k);
	if v then
		return k, v;
	end
end

function pairs(t)
	return iter, t, nil;
end
</code></pre></div></div>

<ul>
  <li>iparis</li>
</ul>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>local function iter(a, i)
	i = i + 1;
	if a[i] then
		return i, a[i];
	end
end

function ipairs(t)
	return iter, t, 0;
end
</code></pre></div></div>

<h1 id="参考">参考</h1>

<ul>
  <li>[1]<a href="http://www.jellythink.com/archives/506">lua中的迭代器与泛型for</a></li>
</ul>