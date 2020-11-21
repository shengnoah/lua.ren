---
layout: post
title: xcode support lua 
tags: [lua文章]
categories: [topic]
---
> 在lua开发时，我是用mac开发的，需要用Xcode进行调试，这就需要Xcode支持lua,并代码高亮。但默认是不支持的，这就需要我们设置了。

####最简单的办法:

  * 下载[Lua-In-Xcode](https://github.com/breinhart/Lua-In-Xcode)
  * 按照说明在shll中执行  
`sudo ./Add-Lua.sh --beta`

  * 重启Xcode,选菜单`Editor>SynTax Coloring>Lua`