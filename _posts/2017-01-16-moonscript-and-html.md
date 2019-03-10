---
layout: post
title: Moonscript与HTML模板
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2017-01-13
tags: [Lapis框架]
categories: [Lapis框架]
---


作者：糖果

Moonscript与HTML模板

Lapis框架可以用Moonscript直接编程，Moonscript可以将需要模板系统对于的网页输出，一揽子的都在Moonscript中用代码实现，可以看leaof.net/lapis下最典型的官方例子，其实DB的相关操作也可以在moonscript中都完成了，对于某种类型的操作来看，这是很酷的。


我们先来看一下模板的例子：

```lua 
import Widget from require "lapis.html"

class Index extends Widget
  content: =>
    h1 class: "header", "Hello"

    @user_panel!
    div class: "body", ->
      text "Welcome to my site!"

  user_panel: =>
    return unless @current_user
    div class: "user_panel", "Welcome back " .. @current_user.name
```


这段mooncript代码会被翻译成下面的HTML
```html
<h1 class="header"><%= "Hello" %></h1>
<% if current_user then %>
  <div class="user_panel">
    Welcome back <%= current_user.name %>
  </div>
<% end %>

<div class="body">
  Welcome to my site
</div>

```


在实际leafo之前的应用代码中，利用的最明显的是下面这段：

```lua
import Widget from require "lapis.html"

class PostLayout extends Widget
  content: =>
    @page.template_stack\push "web"

    div class: "post", ->
      div class: "sidebar", ->
        text @root

      div class: "main_column", ->
        raw @body

```

下面这句话出现了问题：
```lua
      div class: "sidebar", ->
        text @root
```
@root在某些情况下晃是一个”..“, 这个会造成左边栏的导航栏消失了。


这里核心的库就是这个lapis.html
```
import Widget from require "lapis.html"
```

代码太长，就贴个连接过来
[lapis.html.moon](https://github.com/leafo/lapis/blob/master/lapis/html.moon)
,Moonscript如果用来做个人项目还是挺酷的。如何用Moonscript生成Html看下面的连接：[HTML Generation](http://leafo.net/lapis/reference/html_generation.html),
Moonscript生成大量的前端可以维护起来可能有些难度，但做为生成模板描述，还是方便快速有优势。

上面的模板定义代中，使用了一个助手函数raw：

```lua
raw @body
```

还使用了另外的一个助手函数：
```lua
text @root
```


#### raw(str) 
> 引用文本： — outputs the argument, a string, directly to the buffer without escaping.

raw是直接将字符串写入到buffer，并且不经过转意。

#### text(args) 
> 引用文本： — outputs the argument to the buffer, escaping it if it’s a string. If it’s a function, it executes the function in HTML builder environment. If it’s a table, it writes each item in the table

这个函数区别于raws，如果是字符串就转义。是函数就执行。如果是表，把每个元素都写入表。

上面提到的@root是sitegen的内建变量。

> 引用文本： $root — a relative path prefix that points back to the top level directory of your site. For example, if the current page is wwww/hello/world.html, the $root would be ../. This make it easy to built URLs to other pages without being concerned about where your page is being rendered to.

一般的情况下,@root值就取"."、".." 比较常见。不过，此处这种输出会导致左导航栏消失。sitgen, jekyll,还有这个saepy-log，这三个系统，前二者是静态转换Markdown,saepy-log是从Mysql里，读取Markdown（最开始不是，改成了直接支持Markdown。）三个工具分别用三种语言实现。moonscript(lua), ruby, python(tornado)，设计风格也不一样。对SEO支持比较好的是SAEPY_LOG，最酷的还是sitegen。