---
layout: post
title: Nginx+Lua返回JSON类型数据
description: Nginx+Lua返回JSON类型数据
date:   2016-09-17 22:50:18 +0800 
tags: [lua]
categories: [topic]
---
作者：糖果


Nginx返回JSON数据，一种是直接在配置文件里设置，一种是通过Lua代码封装完成，讲Nginx中执行Lua返回JSON的关键，一个用API函数ngx.say，同时配合json.encode对JSON格式的字符串进行编码，然后设定响应头信息的类型。


# Nginx Conf中返回JSON的方式

```lua
location /json/ {

	    default_type application/json;
	    add_header Content-Type 'text/html; charset=utf-8';
	    return 200 '{"about":"糖果的Lua教程,"sites":"lua.ren"}';
}


```


# Nginx Lua返回JSON的方式 

三步操作：

## 1.设置HTTP的响应头信息： 
```lua
ngx.header['Content-Type'] = 'application/json; charset=utf-8'
```
## 2.json.encode(“Lua的Table型变量”)： 
```lua
json = require "cjson" 
res_json_data = json.encode(ret)
```
## 3.用say函数显示，经过encode的JSON数据。
```lua
ngx.say(res_json_data)
```

用Lua实现以上3个步骤，就实同了JSON数据返回。

## 完整代码片段，以下：

```lua
json = require "cjson"
ngx.header['Content-Type'] = 'application/json; charset=utf-8'
ngx.say(json.encode(ret))
```


下面的内容就是用Lua封装了几个函数，通过封装快实现了JSON数据的返回。


一般的Python的WEB框架，都可以的指定返回JSON数据，基本的原理，还是通过指定返回JSON格式的字符串，并且设定HTTP返回时header的Content-Type属性为application/json，来实现返回JSON数据的目地。

而在Openresty+Lua的框架模式下，不用同时指返回的header类，直接在路由对应的匿名函数中，指定返回一个table类型的即可， 在web框架部分区分判断，如果用户返回的是table类型的数据，直接就用cjson这种库，把table数据渲染成JSON返回。


依Blues演示框架为例：

```lua
        app.run = function()
                fun = Route:run(app.router)
                if fun then
                    local ret = fun(app.req, app.id)
                    local rtype = type(ret)
                    if rtype == "table"  then
                        json = require "cjson"
                        ngx.header['Content-Type'] = 'application/json; charset=utf-8'
                        ngx.say(json.encode(ret))
                    end 
                end 

        end 
```

显然，这里只是对返回值的类型是“talbe”的做了处理，也可以对返回类型是“string”或是其它类型的数据做处理。


```lua
        app.run = function()
                fun = Route:run(app.router)
                if fun then
                    local ret = fun(app.req, app.id)
                    local rtype = type(ret)
                    if rtype == "table"  then
                        json = require "cjson"
                        ngx.header['Content-Type'] = 'application/json; charset=utf-8'
                        ngx.say(json.encode(ret))
                    end
                    if rtype == "string"  then
                        ngx.header['Content-Type'] = 'text/plain; charset=UTF-8'
                        ngx.say(ret)
                    end
                end
                le('Application.app.run')
        end
```

没有把这种分类型处理，单独封装成一个方法，简单用这段代码说明问题。

上面是框架中的代码实现，再来看看如何在测试项目中驱动这个功能。


```lua
require "log"
local HiLog = require "HiLog"
local utils = require "utils.utils"
local Application = require "orc"
app = Application.new()


app:get("/json", function(request,id)
    return {k='key', v='value'}    
end)

app:get("/string", function(request,id)
    return "Waterfall"
end)

return app.run()
```
这样以来，我们就可以快速的用Openresty + Lua构建超级微级的路由系统，管理渲染JSON数据，构建一个简单的JSON数据请求服务。

[Blues](https://github.com/shengnoah/Blues)
[Waterfall](https://github.com/shengnoah/Waterfall)