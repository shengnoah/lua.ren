---
layout: post
title: lua_regex_cache_max_entries 
tags: [lua文章]
categories: [topic]
---
<p>
		<!--
author: wngn123
head: head.png
date: 2016-08-24
title: lua_regex_cache_max_entries
tags: lua
category: Lua
status: publish
summary: 指定在worker进程级别编译的正则表达式缓存结果的最大数量
-->
</p><h2>lua_regex_cache_max_entries</h2>
<p><strong>syntax:</strong> <em>lua_regex_cache_max_entries &lt;num&gt;</em></p>
<p><strong>default:</strong> <em>lua_regex_cache_max_entries 1024</em></p>
<p><strong>context:</strong> <em>http</em></p>
<p>Specifies the maximum number of entries allowed in the worker process level compiled regex cache.</p>
<p>The regular expressions used in <a href="#ngxrematch">ngx.re.match</a>, <a href="#ngxregmatch">ngx.re.gmatch</a>, <a href="#ngxresub">ngx.re.sub</a>, and <a href="#ngxregsub">ngx.re.gsub</a> will be cached within this cache if the regex option <code>o</code> (i.e., compile-once flag) is specified.</p>
<p>The default number of entries allowed is 1024 and when this limit is reached, new regular expressions will not be cached (as if the <code>o</code> option was not specified) and there will be one, and only one, warning in the <code>error.log</code> file:</p>
<pre><code>2011/08/27 23:18:26 [warn] 31997#0: *1 lua exceeding regex cache max entries (1024), ...</code></pre>
<p>Do not activate the <code>o</code> option for regular expressions (and/or <code>replace</code> string arguments for <a href="#ngxresub">ngx.re.sub</a> and <a href="#ngxregsub)">ngx.re.gsub</a> that are generated <em>on the fly</em> and give rise to infinite variations to avoid hitting the specified limit.</p>
<h2>中文</h2>
<p><strong>语法:</strong> <em>lua_regex_cache_max_entries &lt;num&gt;</em></p>
<p><strong>默认值:</strong> <em>lua_regex_cache_max_entries 1024</em></p>
<p><strong>上下文:</strong> <em>http</em></p>
<p>指定在worker进程级别编译的正则表达式缓存结果的最大数量。</p>
<p>当正则选项o被指定的时候，ngx.re.match，ngx.re.gmatch，ngx.re.sub，ngx.re.gsub使用的正则表达式会被缓存在缓存中。</p>
<p>缓存数量的最大值默认是1024，如果数量达到最大值限定值，新的正则表达式将不会被缓存（就像o选项没有指定一样），但是会写一条，仅仅一条警告日志到error.log文件中。</p>
<pre><code>2011/08/27 23:18:26 [warn] 31997#0: *1 lua exceeding regex cache max entries (1024), ...</code></pre>
<p>注意不要激活那些会很快的产生和引起无数的变化的正则表达式的o选项（或者为ngx.re.sub和ngx.re.gsub替换字符串参数），以避免达到指定的限制。</p>
<h4>扩展</h4>
<p>o 选项参数用于提高性能，指明该参数之后，被编译的 Pattern 将会在 worker 进程中缓存，并且被当前 worker 进程的每次请求所共享。 Pattern 缓存的上限值通过 lua_regex_cache_max_entries 来修改</p>
<pre><code># nginx.conf
location /test {
    content_by_lua &#39;
        local regex = [[\d+]]

        -- 参数&#34;o&#34;是开启缓存必须的
        local m = ngx.re.match(&#34;hello, 1234&#34;, regex, &#34;o&#34;)  
        if m then
            ngx.say(m[0])
        else
            ngx.say(&#34;not matched!&#34;)
        end
    &#39;;
}
# 在网址中输入&#34;yourURL/test&#34;，即会在网页中显示 1234 。</code></pre>
		<p></p>