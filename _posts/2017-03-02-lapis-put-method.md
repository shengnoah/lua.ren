---
layout: post
title: Lapis框架的常用处理方法
description: "Lapis框架的常用处理方法"
modified: 2017-03-02
tags: [Lapis]
categories: [Lapis框架]
---

作者：糖果

### 在Lapis中处理GET、POST、PUT。
```lua
import respond_to from require "lapis.application"
class App extends lapis.Application
  @enable "etlua"
  "/login": respond_to {
    GET: =>
      return "login get"
    POST: =>
      return "login post"
    PUT: =>
      return "login put"
  }
```

GET、POST、PUT的使用一目了然。



### 在另一个路由中，调用GraylogSDK访问自己的/login方法，用PUT方法。
```lua

headers_info = {
    'Authorization': auth, 
    'Accept': '*/*',
    'Content-Type':'application/json'
}

class RestyGraylog 
    @putRequest:(req_url, data) =>
        http = require "resty.http"
        httpc = http.new()
        metadata = {
          method:"PUT",
          body: data,
          headers: self.headers_info
        }

        res, err = httpc\request_uri(req_url, metadata)
  
        if not res
          ngx.say("failed to request: ", err)
          return
          
        ngx.status = res.status  
        return res.body
```

在这个版本的Graylog for MoonScript ，没用使用internal proxy的方式，使用的是RESTY-HTTP
来完成这个工作，其达到的效果都是一样的。



本文请不要用于商业目地，非商业转载请署名原作者与原文链接。
[https://www.moonscript.cn/lapis%E6%A1%86%E6%9E%B6/lapis-put-method/](https://www.moonscript.cn/lapis%E6%A1%86%E6%9E%B6/lapis-put-method/)


