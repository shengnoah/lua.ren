---
layout: post
title: MoonScript调用LUA-CJSON
description: "LUA CJSON"
modified: 2017-02-23
tags: [LUA-CJSON]
categories: [OPENRESTY]
---

作者：糖果

Lapis的util库有对cjson封装，而我们想更直接的调用CJSON的方法，而不想依赖的封装。


我们首先实现一个MoonScript写文件的代码：

### 写文件：

```lua

write_data = (var, rule)->
    path = "/home/xxx"  
    file = io.open(path,"aw")
    if file==nil then
        return

    ret = file\write(rule)
    file\close()
    return(t)
```

### 访问接口：

```lua
    restyhttp = require "resty.http"
    httpc = restyhttp.new()
    res, err = httpc\request_uri("http://0.0.0.0/getjson")
    jsonbody = ""
    if res.status == ngx.HTTP_OK 
      jsonbody = res.body
    else 
      jsonbody = nil 
```

### 对JSON数据DECODE:

```lua
    write_data(0, jsonbody)
    t = json.decode(jsonbody)
    tjson = json.decode(t['message'])
    ngx.say(util.serialise_value(tjson))
    ""
```







