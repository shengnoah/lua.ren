---
layout: post
title: How  computer program are interpreted and evaluated 
tags: [lua文章]
categories: [topic]
---
<p><img src="https://user-images.githubusercontent.com/12626454/53999102-28a30300-417d-11e9-9cb5-c5cfa2642bc8.png" alt=""/></p>

<p>花了一些时间重新捋了一遍自己关于解释器的学习笔记：<a href="https://github.com/xxleyi/learning_list/issues/28">CS 61A Summer 2018 Lecture 20: Interpreters</a></p>

<p>稍稍得了一些门道，把自己的想法记录一下。</p>

<p>自己目前的编程以动态语言 Python 为主，想要对动态语言有一个完整的理解和掌握，解释器这部分是必须要掌握的点。</p>

<p>简单来说，解释器就是程序的求值器。当我们花费心思构建起针对一个个功能实现的程序之后，下一步就是把那些代码喂给解释器，然后解释器把我们的代码当作一段文本，当作一些字符串一样的东西，进行解释求值。</p>

<p>本质上就是一句话：我们写的代码之于解释器来说就是一些待求值的数据。正所谓：<strong>code is data, data is code.</strong></p>

<p>具体来讲，解释器求值有一个通用流程：Read-Eval-Print Loop，读入-求值-打印输出 循环进行。</p>

<p>光是这么讲还是太抽象。还需要继续拆分：</p>

<ul>
  <li>读入
    <ul>
      <li>词法分析生成「token 序列」</li>
      <li>语义分析生成「recursive expression representation」</li>
    </ul>
  </li>
  <li>求值
    <ul>
      <li>互递归
        <ul>
          <li>eval ：对表达式求值，得到过程和参数，调用 apply</li>
          <li>apply：将过程应用于操作数，操作数本身可能是又一个表达式，调用 eval</li>
        </ul>
      </li>
      <li>递归到最后，得到「程序的值」</li>
    </ul>
  </li>
  <li>大部分时候，价值发生在副作用上</li>
  <li>而我们的程序，就是让副作用可控且符合期待</li>
</ul>



		 	    <div class="push"></div>
					<br/>
					<em style="margin: 0; font-size: 1rem; color: lightgray; padding-left: 1rem;">不知是该恭喜，还是该怎样，总之阅读到该文的，你是第 <span id="busuanzi_value_page_pv"></span> 人。每一次刷新，都是不同的自己。</em>
					<hr/>