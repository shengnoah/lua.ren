---
layout: post
title: react native 报undefined is not an object(evaluating this.state.currentPage) 
tags: [lua文章]
categories: [topic]
---
<p> <strong>之前由于项目不忙，学习了一个多月的React Native，现将遇到的问题及解决方案记录一下</strong></p>
<hr/>
<p>  在方法中使用state中的属性的时候出现这样的错误，原因是此时的this指向的对象是当前的方法而不是这个对象类</p>
<p>  解决方法有两个：</p>
<p>  1.绑定这个对象类</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">onPress={this.onClick.bind(this)}</span><br/></pre></td></tr></tbody></table></figure>
<p>  2.使用箭头函数</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">onPress={()=&gt;this.onClick();}</span><br/></pre></td></tr></tbody></table></figure>
<hr/>
<pre><code>个人比较推荐第二种方式
</code></pre>