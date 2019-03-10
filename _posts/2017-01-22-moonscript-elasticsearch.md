---
layout: post
title: MoonScript与ElasticSearch客户端
description: "关于如何通过Lapis的ElasticSearch客户端的库来访问ES数据。"
modified: 2017-01-22
tags: [ES]
categories: [MoonScript项目]
---


作者：糖果

从Github上看，有一位叫做Dhaval Kapil老师完成了ElasticSearch for Lua的工作，另一位来自阿根廷的叫做Cristian Haunsen的老师，完成了ElasticSearch for Lapis的客户端程序写作工作,dhavalkapil还有一个博客可以访问：dhavalkapil.com

这次实验的目标，是测试一下本地直接运行ES for Lua，然后在Lapis中访问ES，我们的日志在ES，可以用Lua，也可以用Python完成ES的访问工作， Python没有问题，Lua就看这个实验了。


测试代码，如下：


```lua
local elasticsearch = require "elasticsearch"

local client = elasticsearch.client{
    hosts = {
      { -- Ignoring any of the following hosts parameters is allowed.
        -- The default shall be set
        protocol = "http",
        host = "localhost",
        port = 9200
      }
    },
    -- Optional parameters
    params = {
      pingTimeout = 2
    }
}

-- Will connect to default host/port
local client = elasticsearch.client()

local data, err = client:info()
```


Full list of params（全参数列表）:

1. pingTimeout : The timeout of a connection for ping and sniff request. Default is 1.
2. selector : The type of selection strategy to be used. Default is RoundRobinSelector.
3. connectionPool : The type of connection pool to be used. Default is StaticConnectionPool.
4. connectionPoolSettings : The connection pool settings,
5. maxRetryCount : The maximum times to retry if a particular connection fails.
6. logLevel : The level of logging to be done. Default is warning.



#### Getting info of elasticsearch server（取得ES服务器信息）

```lua
local data, err = client:info()
```


#### Index a document（创建索引文档）

Everything is represented as a lua table.(一切皆为Lua Table)

```lua
local data, err = client:index{
  index = "my_index",
  type = "my_type",
  id = "my_doc",
  body = {
    my_key = "my_param"
  }
}
```

#### Get a document（取得文档）

```lua
data, err = client:get{
  index = "my_index",
  type = "my_type",
  id = "my_doc"
}
```

#### Delete a document（删除文档）

```lua
data, err = client:delete{
  index = "my_index",
  type = "my_type",
  id = "my_doc"
}
```

#### Searching a document （检索文档）

You can search a document using either query string:（使有查询字符进行检索）

```lua
data, err = client:search{
  index = "my_index",
  type = "my_type",
  q = "my_key:my_param"
}
```

#### Or either a request body:(或是请求体)

```lua
data, err = client:search{
  index = "my_index",
  type = "my_type",
  body = {
    query = {
      match = {
        my_key = "my_param"
      }
    }
  }
}
```

#### Update a document（更新文档）

```lua
data, err = client:update{
  index = "my_index",
  type = "my_type",
  id = "my_doc",
  body = {
    doc = {
      my_key = "new_param"
    }
  }
}
```

其实MoonScript for ElasticSearch的库是一个纯正的MoonScript实现，但从Readme中描述
是没显明的看是用MoonScript实现，不防抽出的MoonScript核心实现，贴出来看一下：


```lua
client = elasticsearch.client {
    hosts: config.elasticsearch.hosts and parse_hosts_config(config.elasticsearch.hosts) or {
        {
            protocol: "http"
            host: "127.0.0.1"
            port: 9200
        }
    },
    params: {
        -- Should return a table with { status, statusCode, body }
        preferred_engine: (method, uri, params, body, timeout) ->
            http = require "resty.http"
            httpc = http.new!
            args = { :method, :body, headers: { ["Content-Type"]: "application/json" } }
            res, err = httpc\request_uri(uri, args)

            if not res
                ngx.say("failed to request: ", err)
                return

            response = {}
            response.code = res.status
            response.statusCode = res.status
            response.body = res.body
            return response
    }
}
{
  :interpolate_query, :client
}
```

API封装以外，用的还是"resty.http"调用，这里有一个疑问，resty.http和simple.http是
不是同一个部件？




[原文](http://lua.ren/topic/270/elasticsearch%E7%9A%84lua%E5%AE%A2%E6%88%B7%E7%AB%AF)


[lapis-elasticsearch](https://github.com/CriztianiX/lapis-elasticsearch)

PS：本文测式用代码都来自至官方ES-LUA的Github的Readme。
