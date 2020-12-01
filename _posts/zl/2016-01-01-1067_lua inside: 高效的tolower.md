---
layout: post
title: lua inside: 高效的tolower 
tags: [lua文章]
categories: [topic]
---

    #define ltolower(c)	((c) | ('A' ^ 'a'))

`Lua`源码中的`ltolower`是这样实现的, 为什么能管用呢？因为：  
`'A'`的二进制表示是：‭01000001‬,`'Z'`的二进制表示是：‭01011010‬‬  
`'a'`的二进制表示是：‭01100001‬,`'z'`的二进制表示是：‭01111010‬  
这个方法真是clever啊!但是有两个限制条件：

  1. `ASCII`编码
  2. 有外部的代码确保`c`是`alphabetic character`

按照同样的思路，`toupper`可以写成这样：

    
    
    #define ltoupper(c)	((c) & (~('A' ^ 'a')))