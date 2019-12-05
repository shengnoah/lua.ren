---
layout: post
title: Openresty Nginx Tengine添加动态so库
description:  Openresty Nginx Tengine添加动态so库 
date: 2019-10-11 8:50:18 +0800 
tags: [openresty,tengine,nginx]
categories: [动态库]
image:
    feature: feature.jpg
    creditlink: https://lua.ren 
---



作者：糖果

有一个.so的库，需要在这三个平台上都能部署，下面是部署的三种方式：


## 1：Nginx

对于Nginx来说，安装很正常，直接就是配置configure安装。 编译的方式有两种，一种是静态房的方式安装。

1.1.静态方式的.o文件：

```shell
./configure --add-module=/root/candylab/ngx-http-candylab-module
```

1.2.动态方式的.so文件：

```shell
./configure --add-dynamic-module=/root/candylab/ngx-http-candylab-module
```

## 2：Openresty

Openresty是Nginx的基础上加入了Nginx lua模块，并加入了自己独有其它功能特性。但对于modue的安装方式来说和nginx基本上保持一致。

2.1.静态方式的.o文件：

```shell
./configure --add-module=/root/candylab/ngx-http-candylab-module
```

3.2.2.动态方式的.so文件：

```shell
./configure --add-dynamic-module=/root/candylab/ngx-http-candylab-module
```


## 3：Tengine
Tengine应该是春哥离开阿里之前写的，其静态库的编译方式是一样，需要make和make install，但针对.so文件tengine有dso管理，module变更可以不用make整个工程。

 

2.1.静态方式的.o文件：

```shell
./configure --add-module=/root/candylab/ngx-http-candylab-module
```

2.2.动态方式的.so文件：

```shell
sudo ./dso_tool -a=/root/candylab/ngx-http-candylab-module
```

总结：对于动态库来说，直接会生成到安装目录的modules文件夹中，而静态库.o用nginx -V可以显示出来举例有那些。

## Troubleshooting：
这里对源代码编译可能有一个坑，就是有人写代码变量，不初始化，造成编译不过，因为历史遗留问题，有时没时间调试。就直接在编译选项里改。


编译参数如下：
```shell
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Wno-unused-but-set-variable  -g 
```
这里最常见的一个参数就是-Wno-unused-but-set-variable，忽略变量声明又不用这种问题。

对于.so的使用方式，tengine和其它两种的方式也不一样，tengine使用.so是如下：

```shell
dso { 
    load ngx_http_candylab_module.so; 
} 
```

而在openrest和nginx里用的是： 

```shell
load_module modules/ngx_http_candylab_module.so; 
```

需要注意的是，如果事先已经编译了同名.o静态库，就不要再用dso工具生成动态的.so库了，这样混合使用会提示出现错误的。



## 其它：

如何解决动态库依赖其它的库的问题。

 两种方式：

第一种：是在.configure阶段添加 --with-ld-opt='-lrt'  选项，=等号后面是要链接的库选项-lrt。

第二种：在模块的config文章中设置。


```shell

ngx_feature_libs="-lm"

if test -n "$ngx_module_link"; then
    ngx_module_type=HTTP
    ngx_module_name=$ngx_addon_name
    ngx_module_srcs="$_HTTP_TEST_SRCS"
    ngx_module_libs=$ngx_feature_libs
    . auto/module
else
    NGX_ADDON_SRCS="$NGX_ADDON_SRCS $_HTTP_WAF_SRCS"
    CORE_LIBS="$CORE_LIBS $ngx_feature_libs"
    CORE_INCS="$CORE_INCS $ngx_module_incs"
    HTTP_MODULES="$HTTP_MODULES $ngx_addon_name"
fi
```

这里有一个问题，如果现在的二进制安装文件和安装的源文件不一致，版本号相同，可能会出二制不兼容，这种情况需要make install复盖原先的二进制文件。




