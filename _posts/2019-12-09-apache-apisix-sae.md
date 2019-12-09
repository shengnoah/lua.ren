---
layout: post
title: Apache APISIX在SAE应用市场发布
description: Apache APISIX在SAE应用市场发布
date: 2019-12-09 8:50:18 +0800 
tags: [openresty,apisix,gateway,dashboard]
categories: [网关]
image:
    feature: feature.jpg
    creditlink: https://lua.ren 
---


### 感谢

最近盲装了APISIX的各种版本的代码，主要是装个3-4个版。v0.8 v0.9rc 还有符总的版本，还有温部的一键RPM包。因为是盲装，过程中不断的骚扰了院长和温部，各种打扰，原谅我这个中年人。

最后经过明哥的审核，APISIX 0.8在新浪云市上审核后上架了。

如果那位老师想不折腾，又想部署测试，可以SAE上一键安装测试一下。现在这个版本是v0.8，之后会上架新的版本v0.9版本。


### 部署

安装的过程，主要还是三块：

Operesty安装：rpm安装和源码安装，如果想省事，最好采用RPM安装，除非想单独安一些模块，比如动态静态库这些。

Luarocks安装，这个安装脚本控制，主要先要把Luarocks依赖的基础中间件先安了，还有就是root权限和普通用户Luarocks的配置是不一样的。

APISIX的安装， V0.8可以直接用Luarocks安装，但果可能在Dashboard方面需要注意一下版本的配合。但是v0.8版本的RPM包可能有一些问题,运行起来用不了，如果没有特殊要求，luarocks装也可以，如果UI能用的话。


### 测试

在路由创建的时候需要注意一下， 如果创建的路由不指定任何插件和服务，一定要指定上游，不然请求POST会返回400码，创建不成功。


```shell
curl  -H "Content-Type: application/json"   -X POST -d '{"uris":["asdf"],"plugins":{},"desc":"asdf"}'   0.0.0.0:5050/apisix/admin/routes
```

```shell
{"error_msg":"invalid configuration: object matches none of the alternatives"}
```

```shell
 127.0.0.1 - - [06/Dec/2019:04:07:32 +0000] 0.0.0.0:5050 "POST /apisix/admin/routes HTTP/1.1" 400 90 0.000 "-" "curl/7.29.0" - - -
```



 上面的出错的原因就是没有创建upstream，还有一些插件可能报错，这个到时需要动手亲测。


### 参考链接
[云商店链接](https://market.sinacloud.com/#/dashbord/detail?id=158)

[线上演示版](http://apisix.applinzi.com/apisix/dashboard)

[Lua心经](https://lua.ren)
    

以上略过大量安装细节，有问题大家可以留言，或是在Q群里直接喊，但想想各位老师都是人材， 说话还都好听，估计也不会有啥大问题。

看最新的消息，关注公众号：[糖果的实验室](https://mp.weixin.qq.com/s?__biz=MjM5NjEzNzU5OQ==&mid=2247483961&idx=1&sn=283949ff5a78292c7506fa019a0ebd1f&chksm=a6ec9d29919b143fec6b7b64bcfb7c3589705563fccb16a713b7a6a2d62372fe1099b49e27a1&token=163358827&lang=zh_CN#rd)