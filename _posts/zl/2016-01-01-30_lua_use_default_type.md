---
layout: post
title: lua_use_default_type 
tags: [lua文章]
categories: [topic]
---
<p>
		<!--
author: wngn123
head: head.png
date: 2016-08-24
title: lua_use_default_type
tags: lua
category: Lua
status: publish
summary: 指定是否使用MIME类型（default_type指令指定的类型）作为Content-Type响应头信息。如果你不想使用这个默认的Content-Type响应头信息给你的Lua请求去处理，则将这个指令设置为关闭状态>（off）。
-->
</p><h2>lua_use_default_type</h2>
<p><strong>syntax:</strong> <em>lua_use_default_type on | off</em></p>
<p><strong>default:</strong> <em>lua_use_default_type on</em></p>
<p><strong>context:</strong> <em>http, server, location, location if</em></p>
<p>Specifies whether to use the MIME type specified by the <a href="http://nginx.org/en/docs/http/ngx_http_core_module.html#default_type">default_type</a> directive for the default value of the <code>Content-Type</code> response header. If you do not want a default <code>Content-Type</code> response header for your Lua request handlers, then turn this directive off.</p>
<p>This directive is turned on by default.</p>
<p>This directive was first introduced in the <code>v0.9.1</code> release.</p>
<h2>中文</h2>
<p><strong>语法 :</strong> <em>lua_use_default_type on | off</em></p>
<p><strong>默认值:</strong> <em>lua_use_default_type on</em></p>
<p><strong>上下文:</strong> <em>http, server, location, location if</em></p>
<p>指定是否使用MIME类型（default_type指令指定的类型）作为Content-Type响应头信息。如果你不想使用这个默认的Content-Type响应头信息给你的Lua请求去处理，则将这个指令设置为关闭状态（off）。</p>
<p>这个指令默认是开启状态（on）</p>
<p>这个指令第一次出现是在 v0.9.1 稳定版中。</p>
<p><code>wngn123@163.com on 206-07-12</code></p>
		<p></p>