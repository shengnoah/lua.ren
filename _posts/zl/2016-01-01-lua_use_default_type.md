
---
layout: post
title: lua_use_default_type 
tags: [lua文章]
categories: [lua文章]
---
## lua_use_default_type

**syntax:** _lua_use_default_type on | off_

**default:** _lua_use_default_type on_

**context:** _http, server, location, location if_

Specifies whether to use the MIME type specified by the
[default_type](http://nginx.org/en/docs/http/ngx_http_core_module.html#default_type)
directive for the default value of the `Content-Type` response header. If you
do not want a default `Content-Type` response header for your Lua request
handlers, then turn this directive off.

This directive is turned on by default.

This directive was first introduced in the `v0.9.1` release.

## 中文

**语法 :** _lua_use_default_type on | off_

**默认值:** _lua_use_default_type on_

**上下文:** _http, server, location, location if_

指定是否使用MIME类型（default_type指令指定的类型）作为Content-Type响应头信息。如果你不想使用这个默认的Content-
Type响应头信息给你的Lua请求去处理，则将这个指令设置为关闭状态（off）。

这个指令默认是开启状态（on）

这个指令第一次出现是在 v0.9.1 稳定版中。

`wngn123@163.com on 206-07-12`

