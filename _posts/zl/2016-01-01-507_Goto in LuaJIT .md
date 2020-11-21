---
layout: post
title: Goto in LuaJIT  
tags: [lua文章]
categories: [topic]
---
Lua 在 5.2 之后的版本，加入了 `goto` 这个关键字，用来控制程序跳转到指定 `label`。我们可以利用这个特性，来模拟 continue
的实现。需要注意的是 `goto` 只能跳转到 `label`，而 `::name::` 的格式就可以设置一个 `label`。

    
    
    for i=1,5 do
        if i == 3 then
            goto continue
        end
        print(i)
        ::continue::
    end
    

这样就简单实现了 continue。但是有些同学可能会有疑问，这是 Lua 5.2 的特性，我们都知道 OR 中使用的 Lua 5.1，那如何使用？

其实我们只需要使用 LuaJIT 就可以解决这个问题了。在 LuaJIT 的主页上有这个介绍：

> LuaJIT supports some language and library extensions from Lua 5.2. Features
> that are unlikely to break existing code are unconditionally enabled:  
>  
>
>
>   * goto and ::labels::.
>

也就是说 LuaJIT 把 5.2 的这个特性拿过来了。那我们来验证下：

    
    
    [root@master:~]# cd /usr/local/openresty/luajit/bin/
    [root@master:bin]# ./luajit
    LuaJIT 2.1.0-beta1 -- Copyright (C) 2005-2015 Mike Pall. http://luajit.org/
    JIT: ON SSE2 SSE3 SSE4.1 fold cse dce fwd dse narrow loop abc sink fuse
    >
    > for i=1,5 do
        if i == 3 then
            goto continue
        end
        print(i)
        ::continue::
    end>> >> >> >> >> >>
    1
    2
    4
    5
    >
    

继续验证 OR ：

    
    
    [root@master:demo]# nl app/app.lua
         1	local lor = require("lor.index")
         2	local app = lor()
    
         3	app:get("/index", function(req,res,next)
         4	    res:send("hello")
         5	end)
    
         6	app:get("goto", function(req,res,next)
         7	    for i=1, 5 do
         8	        if i == 3 then
         9	            goto continue
        10	        end
        11	        ngx.say(i)
        12	        ::continue::
        13	    end
        14	end)
    
        15	return app
    
    [root@master:demo]#
    [root@master:demo]# http :8888/goto
    HTTP/1.1 200 OK
    Connection: keep-alive
    Content-Type: text/plain
    Date: Fri, 02 Dec 2016 02:55:43 GMT
    Server: openresty/1.9.7.4
    Transfer-Encoding: chunked
    X-Powered-By: Lor Framework
    
    1
    2
    4
    5
    
    [root@master:demo]#
    

那么 `goto` 还有哪些玩法？尾递归优化使用 `goto` 来实现：

    
    
    do
        local function fact(n)
            return fact_iter(n, 1)
        end
    
        function fact_iter(n, now)
            ::call::
            if n <= 1 then
                return now
            else
                -- return fact_iter(n-1, now*n)
                n, now = n-1, now*n
                goto call
            end
        end
    
        print(fact(5))
        print(fact(10000))
        print(fact(10000000))
    end
    

然而感觉并没什么卵用。其他玩法需要各位同学自己去挖掘了。。不过需要注意的是 Lua 也有对 `goto` 的滥用限制：

  * A label is visible in the entire block where it is defined (including nested blocks, but not nested functions).
  * A goto may jump to any visible label as long as it does not enter into the scope of a local variable.

* * *