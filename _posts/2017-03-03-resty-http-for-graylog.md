---
layout: post
title: 用RESTY-HTTP实现Graylog的Widget更新与查询
description: "用RESTY-HTTP实现Graylog的Widget更新与查询"
modified: 2017-03-03
tags: [RESTY-HTTP,Graylog]
categories: [OPENRESTY]
image:
    feature: feature.jpg
    credit: dargadgetz
    creditlink: http://www.dargadgetz.com/ios-8-abstract-wallpaper-pack-for-iphone-5s-5c-and-ipod-touch-retina/
---

作者：糖果

MoonScript for GrayLog是之前写的一个基于Lapis与Simple HTTP的Graylog日志查询SDK，
支持Stream查询，最近为了做自动化分析，加入了新的接口中调用功能，加入了对Dashboard
widgets和更新与查询，通过这个程序，实现一些反扫逻辑。


```lua

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
        return res.body


    @updateWidget: (dashboardId, widgetId,jsonBody) =>
        errList = {}
        if type(dashboardId) == 'nil'
            table.insert(errList, "dashboard id is nil\n")

        if type(widgetId) == 'nil'
            table.insert(errList, "widget id is nil\n")

        if type(jsonBody) == 'nil'
            table.insert(errList, "json body is nil\n")

        num = table.getn(errList) 
        if num > 0 
            return errList


        url = "http://"..self.host..":"..self.port
        req_url = url..'/dashboards/'..dashboardId..'/widgets/'..widgetId

        self.headers_info = {
            'Authorization': self.auth, 
            'Accept': '*/*',
            'Content-Type':'application/json'
        }

        self\putRequest req_url, jsonBody
        return 1


    @getRequest:(req_url) =>
        http = require "resty.http"
        httpc = http.new()
        metadata = {
          method:"GET",
          headers: self.headers_info
        }

        res, err = httpc\request_uri(req_url, metadata)

        if not res
          ngx.say("failed to request: ", err)
          return

        ngx.status = res.status
        return res.body


    @getWidgetValue: (dashboardId, widgetId) =>
        errList = {}
        if type(dashboardId) == 'nil'
            table.insert(errList, "dashboard id is nil\n")

        if type(widgetId) == 'nil'
            table.insert(errList, "widget id is nil\n")

        num = table.getn(errList) 
        if num > 0 
            return errList

        url = "http://"..self.host..":"..self.port
        req_url = url..'/dashboards/'..dashboardId..'/widgets/'..widgetId..'/value'

        self.headers_info = {
            'Authorization': self.auth, 
            'Accept': 'application/json',
        }

        ret = self\getRequest req_url
        return ret
        
```
这次没有使用过去端末加JSON数据请求的方式，把simple http换成了RESTY-HTTP,项目名改
了，叫“Finder”。


本文请不要用于商业目地，非商业转载请署名原作者与原文链接。
[https://www.moonscript.cn/openresty/resty-http-for-graylog/](https://www.moonscript.cn/openresty/resty-http-for-graylog/)