---
layout: post
title: lua_regex_cache_max_entries 
tags: [lua文章]
categories: [topic]
---
## lua_regex_cache_max_entries

**syntax:** _lua_regex_cache_max_entries <num>_

**default:** _lua_regex_cache_max_entries 1024_

**context:** _http_

Specifies the maximum number of entries allowed in the worker process level
compiled regex cache.

The regular expressions used in ngx.re.match, ngx.re.gmatch, ngx.re.sub, and
ngx.re.gsub will be cached within this cache if the regex option `o` (i.e.,
compile-once flag) is specified.

The default number of entries allowed is 1024 and when this limit is reached,
new regular expressions will not be cached (as if the `o` option was not
specified) and there will be one, and only one, warning in the `error.log`
file:

    
    
    2011/08/27 23:18:26 [warn] 31997#0: *1 lua exceeding regex cache max entries (1024), ...

Do not activate the `o` option for regular expressions (and/or `replace`
string arguments for ngx.re.sub and ngx.re.gsub that are generated _on the
fly_ and give rise to infinite variations to avoid hitting the specified
limit.

## 中文

**语法:** _lua_regex_cache_max_entries <num>_

**默认值:** _lua_regex_cache_max_entries 1024_

**上下文:** _http_

指定在worker进程级别编译的正则表达式缓存结果的最大数量。

当正则选项o被指定的时候，ngx.re.match，ngx.re.gmatch，ngx.re.sub，ngx.re.gsub使用的正则表达式会被缓存在缓存中。

缓存数量的最大值默认是1024，如果数量达到最大值限定值，新的正则表达式将不会被缓存（就像o选项没有指定一样），但是会写一条，仅仅一条警告日志到error.log文件中。

    
    
    2011/08/27 23:18:26 [warn] 31997#0: *1 lua exceeding regex cache max entries (1024), ...

注意不要激活那些会很快的产生和引起无数的变化的正则表达式的o选项（或者为ngx.re.sub和ngx.re.gsub替换字符串参数），以避免达到指定的限制。

#### 扩展

o 选项参数用于提高性能，指明该参数之后，被编译的 Pattern 将会在 worker 进程中缓存，并且被当前 worker 进程的每次请求所共享。
Pattern 缓存的上限值通过 lua_regex_cache_max_entries 来修改

    
    
    # nginx.conf
    location /test {
        content_by_lua '
            local regex = [[\d+]]
    
            -- 参数"o"是开启缓存必须的
            local m = ngx.re.match("hello, 1234", regex, "o")  
            if m then
                ngx.say(m[0])
            else
                ngx.say("not matched!")
            end
        ';
    }
    # 在网址中输入"yourURL/test"，即会在网页中显示 1234 。