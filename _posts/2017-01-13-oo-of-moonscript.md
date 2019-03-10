---
layout: post
title: MoonScript's Object
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2017-01-13
tags: [MoonScript简介]
categories: [MoonScript语法]
---


作者：糖果

Coffescript是一种中间的脚本，可以把这种脚本翻译成JavaScript。而MoonScript，是可以翻译成lua语言的中间脚本。

本文简单的介绍的：

1. 如何在VIM中，实现MoonScript语法高亮。 
2. 如何简单的编译MoonScript脚本。
3. MoonScript面向对象OO简介。

# 1.安装MoonScript #
```shell
sudo luarocks install moonscript
```

# 2.创建.moon源文件 #
**app.moon**

```lua
lapis = require "lapis"   
class extends lapis.Application   
  "/": => 
"Welcome to Lapis #{require "lapis.version"}!" 
```


# 3.安装MoonScript语法高亮的插件。 #
**3-1.下载vim的bundle插件管理程序。**

```shell
git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
```


**3-2.创建.vimrc配置文件。**

```shell
vim ~/.vimrc
```


**3-3. 编辑.vimrc文件内容。**

```shell
set nocompatible                                                                                                                      
filetype off                                                                                                                          
set rtp+=~/.vim/bundle/vundle/                                                                                                        
call vundle#rc()                                                                                                                                      
Bundle 'leafo/moonscript-vim'                                                                                                         
syntax enable 
```


**3-4.进入VIM，安装bundle插件。**

```shell
vim +BundleInstall
```


**3-5.翻译成lua脚本,并运行。**

```shell
moonc app.moon
lua app.lua
```



**按照如上步骤操作后:**

1. 可以用moonc命令翻译.moon脚本到.lua脚步。
2. vim支持moonscript的语法高亮检查。


**Moonscript与OO面向对象:**

讲MoonScript不能不提她对OO面向对像的支持，下面简单介绍一下MoonScirpt面像对像的
特性。对向函数与变量的封装，MoonScirpt的函数定义有其独特的地方是。

```lua
foo = ->
    print "foo"
    
foo()    --函数调用
```    

上面是无参数函数调用，没有形参，下面是加入形参的函数声明：


```lua    
foo(x, y) =>
   return x + y

ret = foo 1,5
```       

MoonScript的函数调用，可以不使用(),省着去括号。


接下来，用类封装函数和成员表量，用一个单根继承两类做说明。

```lua
class CandyLab 
    @metadata: "Candy Lab"

class Candy extends CandyLab

    @name: "Candy"
    
    @value: "From CandyLab"
    
    @func1: =>
        print @name
        print @value + 6
        print "func1"

    @func2: (x, y) =>
        print x + y
        return x + y

    @func3: =>
        print "ok"
        print "#{@metadata}"
```

MoonScirpt的对象变量和函数可能直接在外部调用：

```lua
Candy\func1!
ret = Candy\func2 1,5 
Candy\func3!
```

在Candy类中的func3，直接引用的父类的变量metadata,另一个比较独特的地方是，用!表
示调用无参函数。MoonScript的类继承关键还是与Java接近extends。


下面是没有继承关系的两个类之间的函数调用与变量引用，而在引用的过程中有一个特别
一点的表量类型声明，就是Table类型。


```lua
class CandyLab 
    endpoints: 1
    tbl_url: {
             url_1:"http://host.com/1",
             url_2:"http://host.com/2",
             url_3:"http://host.com/3"
         }
          
class Candy
    @func1: =>
        print(Common.endpoints)  
        print(type(Common.tbl_url["ur1_1"]))
```


上面的操作是直接在Candy类中调用CandyLab的变量，其实就是没有权限访问。MoonScript
的OO特性比较常用如上，MoonScript这些特性，如果用来做Web开发，效率相对Lua来说还是
高的，而目前用MoonScript实现的最大的一个项目就是Lua框架Lapis，接下来可以看看Lapis
中是如何使用MoonScript的特性的。




**作者：糖果**

**PS:转载到其它平台请注明作者姓名及原文链接，请勿用于商业用途。**

[https://www.candylab.net](https://www.candylab.net)

