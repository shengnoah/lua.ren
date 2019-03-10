---
layout: post
title: MoonScript如何使用RESTY-HTTP
description: "MoonScript如何使用RESTY-HTTP"
modified: 2017-02-21
tags: [RESTY-HTTP]
categories: [OPENRESTY]
---

作者：糖果

在OpenResty中发起HTTP请求，一般情况下，有两种方式：
1.通过内部Proxy。
2.使用RESTY-HTTP库发起访问。

Lapis使用的是interal proxy,之前文章有提到，下面提到的是RESTY-HTTP的MoonScript调用
实现。


## RESTY-HTTP安装
实际RESTY-HTTP的主要实现就是两个lua文件， http_headers.lua和http.lua这两个文件。
将文件复制到/usr/local/openresty/lualib/resty下即可使用，再引用http.lua时注意一下
的是Lapis也有一个同名文件，需要注意一下冲突。



## MoonScript代码：

```lua
    http = require "resty.http"
    httpc = http.new()
    res, err = httpc\request_uri("http://www.baidu.com")
    if res.status == ngx.HTTP_OK 
      return res.body
    else 
      return "nil"   
```

## LUA代码：

```lua
    ["/testcase"] = function(self)
      local restyhttp = require("resty.http")
      local httpc = restyhttp.new()
      local res, err = httpc:request_uri("http://www.baidu.com")
      if res.status == ngx.HTTP_OK then
        return res.body
      else
        return "nil"
      end
    end
```      


MoonScript和LUA代码几乎没太大区别，因为request_uri请求中使用的是域名，所以需要
修改conf文件。

## nginx.conf配置

```
    location / {
            resolver 8.8.8.8;
    }            
```    

RESTY-HTTP与CJSON不同，并没有涉及到任何so库的生成，http_headers.lua和http.lua这
两个文件也是在Makefile来实现的，使用的是install命令-d参数，相当于在cp过程中，如果目标位置没
有相应的文件夹就创建一个文件夹。


## Makefile

```
OPENRESTY_PREFIX=/usr/local/openresty

PREFIX ?=          /usr/local
LUA_INCLUDE_DIR ?= $(PREFIX)/include
LUA_LIB_DIR ?=     $(PREFIX)/lib/lua/$(LUA_VERSION)
INSTALL ?= install
TEST_FILE ?= t

.PHONY: all test install

all: ;

install: all
        $(INSTALL) -d $(DESTDIR)/$(LUA_LIB_DIR)/resty
        $(INSTALL) lib/resty/*.lua $(DESTDIR)/$(LUA_LIB_DIR)/resty/


```






