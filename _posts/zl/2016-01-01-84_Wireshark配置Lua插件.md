---
layout: post
title: Wireshark配置Lua插件 
tags: [lua文章]
categories: [topic]
---
版权声明：本文为 Jiawei Xu 于2019年4月28日所写，未经允许不得转载。

### 配置方法

在wireshark的安装目录下面编辑init.lua文件，例如mac上：

    
    
    1

|

    
    
    vim /Applications/Wireshark.app/Contents/Resources/share/wireshark/init.lua  
  
---|---  
  
最后一行如果没有加添加，否则修改：`dofile("YOURFILEPATH.lua")`，里面写绝对路径。

然后重启wireshark。

注：DATA_DIR表示全局配置路径，USER_DIR表示用户配置路径。可以通过菜单`About Wireshark`->`Folders`查看路径。

### 相关链接

<https://wiki.wireshark.org/Lua/>

<https://www.wireshark.org/docs/wsdg_html_chunked/wsluarm.html>