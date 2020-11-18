---
layout: post
title: Mac 上 Sublime Text 3 配置 Lua 环境 
tags: [lua文章]
categories: [lua文章]
---
#### 1.下载安装

* * *

##### 1\. 先去[Lua官网](http://www.lua.org/ftp/)下载最新版 lua

`Rango-MBP:~ rango$ curl -R -O http://www.lua.org/ftp/lua-5.3.3.tar.gz`

##### 2\. 然后解压压缩包

`Rango-MBP:~ rango$ tar zxf lua-5.3.3.tar.gz`

##### 3\. cd 到解压后的文件夹

`Rango-MBP:~ rango$ cd lua-5.3.3`

##### 4\. 然后编译测试

`Rango-MBP:lua-5.3.3 rango$ make macosx test`

###### 正常情况会看到如下信息

    
    
    1
    
    2

|

    
    
    src/lua -v
    
    Lua 5.3.3  Copyright (C) 1994-2016 Lua.org, PUC-Rio`  
  
---|---  
  
##### 上面几步总概况

    
    
    1
    
    2
    
    3
    
    4

|

    
    
    $ curl -R -O http://www.lua.org/ftp/lua-5.3.3.tar.gz
    
    $ tar zxf lua-5.3.3.tar.gz
    
    $ cd lua-5.3.3
    
    $ make macosx test  
  
---|---  
  
##### 5\. 安装 lua 需要输入开机密码

    
    
    1

|

    
    
    Rango-MBP:lua-5.3.3 rango$  sudo make install  
  
---|---  
  
###### 安装完成后输入`lua -v`看到出现如下信息则安装成功

    
    
    1
    
    2

|

    
    
    Rango-MBP:lua-5.3.3 rango$ lua -v
    
    Lua 5.3.3  Copyright (C) 1994-2016 Lua.org, PUC-Rio  
  
---|---  
  
#### 2\. 配置 Sublime Text 3 的 Lua 编译环境

* * *

##### 1\. 下载 Sublime Text 3 安装

##### 2\. 在 Sublime Text 3中选择 Tools/Build System/New Build System

![](http://ww2.sinaimg.cn/large/741b3941jw1f56t0cxgecj21aw0pwqam.jpg)

##### 3\. 将`"shell_cmd": "make"`替换为如下信息

    
    
    1
    
    2
    
    3

|

    
    
    "cmd": ["/usr/local/bin/lua", "$file"],  
    
     	"file_regex": "^(...*?):([0-9]*):?([0-9]*)",  
    
     	"selector": "source.lua"  
  
---|---  
  
##### 4\. 保存为 Lua （或者 myLua）随你心情，然后重启 Sublime Text 3.

##### 5\. 新建测试文件 myLua.lua,注意这里要新建文件，并保存成.lua之后再向里面添加数据。

##### 6\.
经典的测试代码`print("hello".."world");`选择lua编译环境:![](http://ww1.sinaimg.cn/large/741b3941jw1f56t5aqif4j211g0l044k.jpg)

##### 7\. commond + b 编译查看结果`helloworld`这样就配置完成了，可以愉快的敲代码~(≧▽≦)/~啦啦啦