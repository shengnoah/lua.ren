---
layout: post
title: MoonScript与Redis客户端
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2017-01-19
tags: [RESTY-REDIS]
categories: [OPENRESTY]
---

所谓的Redis LUA客户端有两种版本，一种就是本地可运行版本，还有一个版本是OpenResty的版本，下面介绍的这段Moonscript段代码是[本地版](https://github.com/nrk/redis-lua)的。


作者：糖果

candylab.moon

```lua
redis = require "redis"
client = redis.connect '127.0.0.1', 6379

auth_flg = client\auth "www.moonscript.cn"

if not auth_flg 
    print("Auth NG!!!")

client\publish "chatroom", "www.candylab.net"
```


candylab.lua

```lua
local redis = require("redis")
local client = redis.connect('127.0.0.1', 6379)
local auth_flg = client:auth("www.moonscript.cn")
if not auth_flg then
  print("Auth NG!!!")
end
return client:publish("chatroom", "www.candylab.net")
```