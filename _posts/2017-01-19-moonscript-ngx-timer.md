---
layout: post
title: MoonScript版的Ngx.Timer
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2017-01-19
tags: [OpenResty Timer]
categories: [OPENRESTY]
---


作者：糖果

实际如果是直接调用ngx.timer的API，moonscript和lua的ngx.timer的函数写法差别不是很大。

candylab.moon

```lua
handler = (fill, params)->
    ok, err = ngx.timer.at(1, handler, "params-data")
    ngx.log(ngx.DEBUG, "ok:", ok, " err:", err)
         
ok, err = ngx.timer.at(6, handler, "params-data")
   
            
if not ok then
    ngx.log(ngx.ERR, "err:", err)
```

candylab.lua


```lua
local http = require("lapis.nginx.http")

local handler
handler = function(fill, params)
  local ok, err = ngx.timer.at(1, handler, "params-data")
  return ngx.log(ngx.DEBUG, "ok:", ok, " err:", err)
end

local ok, err = ngx.timer.at(6, handler, "params-data")

if not ok then
  return ngx.log(ngx.ERR, "err:", err)
end
```