---
layout: post
title: luasocket getaddrinfo nil 问题  
tags: [lua文章]
categories: [lua文章]
---
使用 luarocks 安装 luasocket，在调用 bind 时，报：

> socket.lua:29: attempt to call field ‘getaddrinfo’ (a nil value)

继续执行以下 lua 代码片段：

    
    
    Lua 5.1.4  Copyright (C) 1994-2008 Lua.org, PUC-Rio
    >
    > do
    >>   local socket = require("socket")
    >>   for k,v in pairs(socket.dns) do
    >>     print(k,v)
    >>   end
    >> end
    gethostname	function: 0xc774f0
    toip	function: 0xc77ea0
    tohostname	function: 0xc77f00
    >
    >
    

发现确实没有加载进来，而 `getaddrinfo` 是一个标准的 posix 系统调用，所以基本可以确定是安装的问题了。。

于是从 github 拉下最新的 luasocket v3.0-rc1 手动编译完成后，可以 bind，可以看到加载进来了：

    
    
    >
    >> do
    >>   local socket = require("socket")
    >>   for k,v in pairs(socket.dns) do
    >>     print(k,v)
    >>   end
    >> end
    getnameinfo	function: 0xc77490
    getaddrinfo	function: 0xc77380
    gethostname	function: 0xc774f0
    toip	function: 0xc77ea0
    tohostname	function: 0xc77f00
    >
    >
    

所以结论已经出来了，是 luarocks 安装 luasocket 的问题了。那我们继续追踪问题的根源，使用 `luarocks install
luasocket` 可以看到 luasocket 的编译参数为：

    
    
    gcc -O2 -fPIC -I/usr/include -c src/mime.c -o src/mime.o -DLUA_COMPAT_APIINTCASTS -DLUASOCKET_DEBUG -DLUASOCKET_API=__attribute__((visibility("default"))) -DUNIX_API=__attribute__((visibility("default"))) -DMIME_API=__attribute__((visibility("default")))
    

而我自己手动编译时，默认参数为：

    
    
    gcc -I/usr/local/lua/include/lua/5.1 -DLUASOCKET_NODEBUG -DLUA_NOCOMPAT_MODULE -DLUASOCKET_API='__attribute__((visibility("default")))' -DUNIX_API='__attribute__((visibility("default")))' -DMIME_API='__attribute__((visibility("default")))' -pedantic -Wall -Wshadow -Wextra -Wimplicit -O2 -ggdb3 -fpic -fvisibility=hidden   -c -o mime.o mime.c
    

嗯。。。问题似乎已经很明确了，没错，就是几个宏的引用问题：

    
    
    -DLUA_COMPAT_APIINTCASTS -DLUASOCKET_DEBUG
    

而我们手动编译是启用的这几个宏：

    
    
    -DLUASOCKET_NODEBUG -DLUA_NOCOMPAT_MODULE
    

当然如果你嫌手动编译麻烦，也可以用 luarocks 安装 2.0.2 的版本，这个是没有问题的：

    
    
    luarocks install luasocket 2.0.2-6
    

另，在调查这个问题的原因时，新发现个 luarocks pack 的使用技巧，pack 就是可以将我们安装过后的 rock 重新生成 rock
文件，之后我们将其上传到需要安装的其他机器，执行 `luarocks install ***.rock` 即可：

    
    
    luarocks pack luasocket
    # 接下来拷贝 luasocket-3.0rc1-2.linux-x86_64.rock 至目标机器并执行以下命令
    # 就可以离线安装 luasocket 了
    luarocks install luasocket-3.0rc1-2.linux-x86_64.rock
    

* * *