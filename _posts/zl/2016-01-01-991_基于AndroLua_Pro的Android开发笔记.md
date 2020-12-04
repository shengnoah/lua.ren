---
layout: post
title: 基于AndroLua_Pro的Android开发笔记 
tags: [lua文章]
categories: [topic]
---
<blockquote>
<p>温馨提示:<strong>请使用电脑浏览器打开</strong>,以确保最佳的阅读体验,谢谢.(￣▽￣)”</p>
</blockquote>
<blockquote>
<p>骚年, 学海无涯, 编程你怕不怕? 啥,你说怕? 那么你还看个P啊.</p>
</blockquote>
<p><sup>[0]<strong>英文大小写忽略</strong></sup><br/><sup>[1]文章中的 <strong>alua</strong> 和 <strong>AndroLua</strong>的含义一样</sup><br/><sup>[2]文章中的 <strong>AndroLua_Pro</strong> 和 <strong>AndroLua+</strong> 的含义一样</sup></p>
<ul>
<li><p>我接触androlua时长是两年半了(<strong>cxk警告</strong>), 因为AndroLua_Pro是真的很冷门, 所以基本上没有什么学习资料, 但是它的优势很好, 很适合用于学习, 我谈不上大神, 但是我可以把我的经验与你们分享.(<strong>话说都开这么多坑了</strong>)</p>
</li>
<li><p>我理解的AndroLua+逻辑应该是, <strong>Java–(jni)–C–(lua栈)–Lua</strong>, 要是想玩得通, 这些小细节必不可少啊.</p>
</li>
<li><p><a href="https://www.jianshu.com/p/87ce6f565d37" target="_blank" rel="noopener noreferrer">JNI文章推荐</a></p>
</li>
</ul>
<h2 id="gt-1-AndroLua-Pro开发的优势"><a href="#gt-1-AndroLua-Pro开发的优势" class="headerlink" title="&gt;1. AndroLua_Pro开发的优势"></a>&gt;1. AndroLua_Pro开发的优势</h2><ul>
<li><strong>简单</strong>:Lua语法简单, 上手<strong>容易</strong>.</li>
<li><strong>方便</strong>:可以在手机上敲代码, 可以用业余时间来敲敲(<strong>我的大部分代码是打发时间敲的</strong>).</li>
<li><strong>其他</strong>:效率高, 还可以热更新.</li>
</ul>
<hr/>
<h2 id="gt-2-AndroLua-Pro的历史"><a href="#gt-2-AndroLua-Pro的历史" class="headerlink" title="&gt;2. AndroLua_Pro的历史"></a>&gt;2. AndroLua_Pro的历史</h2><ul>
<li><a href="https://baike.baidu.com/item/AndroLua/17344562?fr=aladdin" target="_blank" rel="noopener noreferrer">AndroLua+的百度百科</a></li>
<li>GitHub上的<a href="https://github.com/nirenr/AndroLua_Pro" target="_blank" rel="noopener noreferrer">AndroLua_Pro</a></li>
</ul>
<p>是由国内大佬<strong>nirenr</strong>基于<strong>androlua</strong>优化而来的, 这个是大佬的原话:</p>
<blockquote>
<p><strong>AndroLua+</strong>是我基于GitHub开源项目优化增强而来的一个工程，主要是<strong>效率提高100倍</strong>以上，原来Lua调用Java方法速度大约一秒只能200次左右，经过不断优化，现在大约在10000-30000不等，使其可以在实际项目使用而不明显拖慢程序速度。另外一个就是<strong>修复其中关于JNI的局部引用溢出</strong>问题，就如原作者说的，他做这个只是为了练手ndk开发，所以luajava1.1中的各种bug一概没有修复，说到JNI的局部引用溢出，真是一个非常无语的问题，有时间专门写篇博文唠叨下。最后还有是<strong>把Lua从5.1.5升级到现在最新的5.3.3</strong>，貌似目前还没有其他安卓工程使用Lua5.3.3<br/>感谢泥人, 让我开始了掉头发的新生活.</p>
</blockquote>
<hr/>
<h1 id="2-开始"><a href="#2-开始" class="headerlink" title="2. 开始"></a>2. 开始</h1><h2 id="gt-1-工具准备"><a href="#gt-1-工具准备" class="headerlink" title="&gt;1. 工具准备"></a>&gt;1. 工具准备</h2><p>AndroLua_Pro的衍生物<strong>很多</strong>比如</p>
<blockquote>
<p>ALua/OneLua/MostLua等</p>
</blockquote>
<ul>
<li><p>这些衍生物不乏有优秀的甚至有可以替代本体的的作品, 操作方便/界面美观, 但是有一些版本较旧, 功能最快更新的往往是AndroLua_Pro, 因为其他的都是依赖于nirenr的源代码的二次加工, 当然你也可以自己动手, 自己封装.</p>
</li>
<li><p>你只需要明白, 语法都一样就可以啦.</p>
</li>
</ul>
<p><img src="https://upload-images.jianshu.io/upload_images/17032228-b088015359b431bf.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt="图1"/></p>
<center>图1 我正在使用的软件</center>

<p><br/></p>
<h3 id="安装AndroLua-Pro"><a href="#安装AndroLua-Pro" class="headerlink" title="安装AndroLua_Pro"></a>安装AndroLua_Pro</h3><blockquote>
<ul>
<li><a href="https://www.coolapk.com/apk/com.androlua" target="_blank" rel="noopener noreferrer">AndroLua+</a></li>
<li><a href="https://www.coolapk.com/apk/com.One.androlua" target="_blank" rel="noopener noreferrer">OneLua</a></li>
<li>有一些ide在群内优先更新, 需要自己动手去找了</li>
</ul>
</blockquote>
<h3 id="学习网页推荐"><a href="#学习网页推荐" class="headerlink" title="学习网页推荐"></a>学习网页推荐</h3><p>有一些方法已经为你准备好了, 在不侵犯作者权益的情况下你只需要<strong>复制粘贴</strong>就可以了实现, 当然最好要优化一下, 说不定还可以提高效率.</p>
<blockquote>
<ul>
<li><a href="http://www.androlua.cn/" target="_blank" rel="noopener noreferrer">http://www.androlua.cn</a></li>
<li><a href="http://androlua.com/" target="_blank" rel="noopener noreferrer">http://androlua.com</a>(需要注册)</li>
</ul>
</blockquote>
<h3 id="学习软件推荐"><a href="#学习软件推荐" class="headerlink" title="学习软件推荐"></a>学习软件推荐</h3><blockquote>
<ul>
<li>烧风的<a href="https://www.coolapk.com/apk/com.sf.ALuaGuide" target="_blank" rel="noopener noreferrer">ALUA助手</a>补位.</li>
</ul>
</blockquote>
<hr/>
<h2 id="gt-2-上手"><a href="#gt-2-上手" class="headerlink" title="&gt;2. 上手"></a>&gt;2. 上手</h2><p>你需要知道, 其实你还是在开发Android 软件, Android有的特性ALua基本也有.</p>