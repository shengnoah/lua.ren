---
layout: post
title: Unity Excel导表工具，导成Lua，支持嵌套结构 木之子南 
tags: [lua文章]
categories: [topic]
---
<h4 id="为什么要使用导表工具">为什么要使用导表工具？</h4>
<p>几乎所有游戏公司数据由策划来配置，程序负责逻辑，策划看不懂代码，excel是可以相对具象的让策划了解一个模块的数据配置，起到了策划和程序之间的桥梁作用，也可以方便策划对数据的把控。</p>

<p>根据各个公司各个项目的不同，导表工具的输出形式不同，输出形式有：json，sqlite，txt，lua等，根据不同的项目需求，可以选不同的导表工具。本文章主要是介绍Excel导出Lua文件。</p>

<h4 id="此工具的功能">此工具的功能</h4>
<ol>
  <li>支持导出Lua文件，自动换行对齐</li>
  <li>支持自定义字段不导入Lua</li>
  <li>支持无限嵌套的树状结构（table套table）</li>
  <li>支持的Excel格式 .xlsx, .xlsm, .xltx, .xltm</li>
  <li>导出路径如果已存在同名的Lua文件，则会覆盖</li>
</ol>

<h4 id="excel和导出文件的效果">Excel和导出文件的效果</h4>
<h5 id="excel的格式">excel的格式</h5>
<p><img src="https://yiyuan1130.github.io//styles/images/excel2lua/excel2lua.png" alt="在这里插入图片描述"/></p>

<h5 id="导出的lua格式">导出的lua格式</h5>
<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>return {
    [1] = {
        id = 1,
        name = {
            CN = &#34;安娜&#34;,
            EN = &#34;Anna&#34;,
        },
        age = 12,
        isGirl = true,
        hp = 100,
        mp = 100,
        skill = {
            skill1 = {
                name = &#34;神罗天征&#34;,
                attact = 100.3,
            },
            skill2 = {
                name = &#34;绝对防御&#34;,
                attact = 10.5,
                aaa = {
                    test1 = 111,
                    test2 = &#34;111.0&#34;,
                },
            },
        },
    },
    [2] = {
        id = 2,
        name = {
            CN = &#34;雷欧&#34;,
            EN = &#34;Leo&#34;,
        },
        age = 13,
        isGirl = false,
        hp = 100,
        mp = 100,
        skill = {
            skill1 = {
                name = &#34;螺旋丸&#34;,
                attact = 45.9,
            },
            skill2 = {
                name = &#34;23.0&#34;,
                attact = 134.3,
                aaa = {
                    test1 = 222,
                    test2 = &#34;222.0&#34;,
                },
            },
        },
    },
}
</code></pre></div></div>

<h4 id="项目地址">项目地址</h4>
<p><a href="https://github.com/yiyuan1130/tools-excel2lua">点击此处查看 github 工程</a></p>