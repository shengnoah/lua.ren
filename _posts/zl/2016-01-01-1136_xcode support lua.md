---
layout: post
title: xcode support lua 
tags: [lua文章]
categories: [topic]
---
<blockquote>
<p>在lua开发时，我是用mac开发的，需要用Xcode进行调试，这就需要Xcode支持lua,并代码高亮。但默认是不支持的，这就需要我们设置了。</p>
</blockquote>

<p>####最简单的办法:</p>
<ul>
<li>下载<a href="https://github.com/breinhart/Lua-In-Xcode" target="_blank" rel="external noopener noreferrer">Lua-In-Xcode</a></li>
<li>按照说明在shll中执行<br/><code>sudo ./Add-Lua.sh --beta</code></li>
<li>重启Xcode,选菜单<code>Editor&gt;SynTax Coloring&gt;Lua</code></li>
</ul>