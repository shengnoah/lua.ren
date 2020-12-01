---
layout: post
title: lua 编程之 hello world 
tags: [lua文章]
categories: [topic]
---
今天开始我要学 Lua 了，之所以要学习它，确实也是游戏开发目前需要的，涉及到热更新技术，Lua 相对来说还是比较成熟的，而且就目前主流的各种 Lua
框架的出现，学习 Lua 以及使用 Lua 的难度也是大大降低了。

首先来看下 Lua 是什么，Lua 是一种轻量小巧的脚本语言，用标准C语言编写并以源代码形式开放，
其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。Lua还可以很方便的和其他程序进行集成（c++，c#，java，，，，），可以说用处还是挺多的，虽然目前我就知道热更新这块用到了。

既然说起了 Lua 这门语言，那免不了要和 Unity 正统编程语言 C# 比较一番。

Lua可以在几乎所有的操作系统和平台进行编译运行，可以很方便的更新代码，更新了代码后，可以直接在手机上运行，不需要重新安装（这也就是后续的热更新方案的特点）

而相比我们的 C#，C# 只能在特定的操作系统中进行编译成 dll
文件，然后打包进安装包在其他平台（Android、iOS）运行，在移动平台上不能更新替换已有的dll文件，除非重新下载安装包。当然现在也有以 C#
为主的热更新方案，这也是后话了。

介绍了一些 Lua 的基本情况，我们再来看 Lua 的安装。

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    

|

    
    
      
    1、SciTE  
        Window 系统上安装 Lua  
        window下你可以使用一个叫"SciTE"的IDE环境来执行lua程序，下载地址为：  
        本站下载地址：LuaForWindows_v5.1.4-46.exe  
        Github 下载地址：https://github.com/rjpcomputing/luaforwindows/releases  
        Google Code下载地址 :   
        https://code.google.com/p/luaforwindows/downloads/list  
    2、LuaDist(官方推荐)  
    	http://luadist.org/  
      
  
---|---  
  
当然现在开发 Lua 的方式各种各样，选择自己喜欢的方式进行开发就好。而我的 Lua 开发环境就比较直接了，直接基于 VSCode 进行的：

![](https://upload-
images.jianshu.io/upload_images/2413150-0fd5ed097f34832e.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1000/format/webp)

直接在 VSCode 的插件商店搜索 Lua 安装 Lua 支持的插件以及 Lua Debug 插件，然后环境就差不多了。

我们就地写个 Lua 版本的 HelloWorld，如下：

    
    
    1  
    2  
    

|

    
    
      
    print('hello world')  
      
  
---|---  
  
在 VSCode 里按下 F5 就能将 Lua 代码跑起来了，赶紧试试吧~