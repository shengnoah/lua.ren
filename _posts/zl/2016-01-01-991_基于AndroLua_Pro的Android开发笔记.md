---
layout: post
title: 基于AndroLua_Pro的Android开发笔记 
tags: [lua文章]
categories: [topic]
---
> 温馨提示: **请使用电脑浏览器打开** ,以确保最佳的阅读体验,谢谢.(￣▽￣)”

> 骚年, 学海无涯, 编程你怕不怕? 啥,你说怕? 那么你还看个P啊.

[0] **英文大小写忽略**  
[1]文章中的 **alua** 和 **AndroLua** 的含义一样  
[2]文章中的 **AndroLua_Pro** 和 **AndroLua+** 的含义一样

  * 我接触androlua时长是两年半了( **cxk警告** ), 因为AndroLua_Pro是真的很冷门, 所以基本上没有什么学习资料, 但是它的优势很好, 很适合用于学习, 我谈不上大神, 但是我可以把我的经验与你们分享.( **话说都开这么多坑了** )

  * 我理解的AndroLua+逻辑应该是, **Java–(jni)–C–(lua栈)–Lua** , 要是想玩得通, 这些小细节必不可少啊.

  * [JNI文章推荐](https://www.jianshu.com/p/87ce6f565d37)

## >1\. AndroLua_Pro开发的优势

  * **简单** :Lua语法简单, 上手 **容易**.
  * **方便** :可以在手机上敲代码, 可以用业余时间来敲敲( **我的大部分代码是打发时间敲的** ).
  * **其他** :效率高, 还可以热更新.

* * *

## >2\. AndroLua_Pro的历史

  * [AndroLua+的百度百科](https://baike.baidu.com/item/AndroLua/17344562?fr=aladdin)
  * GitHub上的[AndroLua_Pro](https://github.com/nirenr/AndroLua_Pro)

是由国内大佬 **nirenr** 基于 **androlua** 优化而来的, 这个是大佬的原话:

> **AndroLua+** 是我基于GitHub开源项目优化增强而来的一个工程，主要是 **效率提高100倍**
> 以上，原来Lua调用Java方法速度大约一秒只能200次左右，经过不断优化，现在大约在10000-30000不等，使其可以在实际项目使用而不明显拖慢程序速度。另外一个就是
> **修复其中关于JNI的局部引用溢出**
> 问题，就如原作者说的，他做这个只是为了练手ndk开发，所以luajava1.1中的各种bug一概没有修复，说到JNI的局部引用溢出，真是一个非常无语的问题，有时间专门写篇博文唠叨下。最后还有是
> **把Lua从5.1.5升级到现在最新的5.3.3** ，貌似目前还没有其他安卓工程使用Lua5.3.3  
> 感谢泥人, 让我开始了掉头发的新生活.

* * *

# 2\. 开始

## >1\. 工具准备

AndroLua_Pro的衍生物 **很多** 比如

> ALua/OneLua/MostLua等

  * 这些衍生物不乏有优秀的甚至有可以替代本体的的作品, 操作方便/界面美观, 但是有一些版本较旧, 功能最快更新的往往是AndroLua_Pro, 因为其他的都是依赖于nirenr的源代码的二次加工, 当然你也可以自己动手, 自己封装.

  * 你只需要明白, 语法都一样就可以啦.

![图1](https://upload-
images.jianshu.io/upload_images/17032228-b088015359b431bf.jpg?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1000/format/webp)

图1 我正在使用的软件

  

### 安装AndroLua_Pro

>   * [AndroLua+](https://www.coolapk.com/apk/com.androlua)
>   * [OneLua](https://www.coolapk.com/apk/com.One.androlua)
>   * 有一些ide在群内优先更新, 需要自己动手去找了
>

### 学习网页推荐

有一些方法已经为你准备好了, 在不侵犯作者权益的情况下你只需要 **复制粘贴** 就可以了实现, 当然最好要优化一下, 说不定还可以提高效率.

>   * [http://www.androlua.cn](http://www.androlua.cn/)
>   * [http://androlua.com](http://androlua.com/)(需要注册)
>

### 学习软件推荐

>   * 烧风的[ALUA助手](https://www.coolapk.com/apk/com.sf.ALuaGuide)补位.
>

* * *

## >2\. 上手

你需要知道, 其实你还是在开发Android 软件, Android有的特性ALua基本也有.