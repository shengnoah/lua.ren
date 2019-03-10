---
layout: post
title: MoonScript与Simple.HTTP
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2017-01-19
tags: [RESTY-HTTP]
categories: [OPENRESTY]
---


作者：糖果

MoonScript调用Lapis的Simple.http，其实调用的就是OpenResty的http的接口。

candylab.moon
```lua
http = require "lapis.nginx.http"
body, status_code, headers = http.simple {
    url: 'http://moonscript.cn' 
    method: "GET"
    headers: {} 
}
```

candylab.lua
```lua
local http = require("lapis.nginx.http")
local body, status_code, headers = http.simple({
    url = 'http://moonscript.cn',
    method = "GET",
    headers = { }
})
```

