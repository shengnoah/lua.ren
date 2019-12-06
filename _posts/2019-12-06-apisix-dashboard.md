---
layout: post
title: APISIX后台管理路由创建接口
description: APISIX后台管理路由创建接口
date: 2019-12-06 8:50:18 +0800 
tags: [openresty,apisix,gateway,dashboard]
categories: [网关]
image:
    feature: feature.jpg
    creditlink: https://lua.ren 
---


# APISIX后台管理路由创建接口
```shell
curl  -H "Content-Type: application/json"   -X POST -d '{"uris":["asdf"],"plugins":{},"desc":"asdf"}'   0.0.0.0:5050/apisix/admin/routes
```
```shell
{"error_msg":"invalid configuration: object matches none of the alternatives"}
```

```shell
127.0.0.1 - - [06/Dec/2019:04:07:32 +0000] 0.0.0.0:5050 "POST /apisix/admin/routes HTTP/1.1" 400 90 0.000 "-" "curl/7.29.0" - - -
```

```shell
curl 0.0.0.0:5050/apisix/admin/routes
```

```shell
127.0.0.1 - - [06/Dec/2019:04:05:55 +0000] 0.0.0.0:5050 "GET /apisix/admin/routes HTTP/1.1" 200 108 0.002 "-" "curl/7.29.0" - - -
```