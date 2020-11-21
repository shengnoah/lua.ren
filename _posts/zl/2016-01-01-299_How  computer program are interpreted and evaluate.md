---
layout: post
title: How  computer program are interpreted and evaluated 
tags: [lua文章]
categories: [topic]
---
![](https://user-
images.githubusercontent.com/12626454/53999102-28a30300-417d-11e9-9cb5-c5cfa2642bc8.png)

花了一些时间重新捋了一遍自己关于解释器的学习笔记：[CS 61A Summer 2018 Lecture 20:
Interpreters](https://github.com/xxleyi/learning_list/issues/28)

稍稍得了一些门道，把自己的想法记录一下。

自己目前的编程以动态语言 Python 为主，想要对动态语言有一个完整的理解和掌握，解释器这部分是必须要掌握的点。

简单来说，解释器就是程序的求值器。当我们花费心思构建起针对一个个功能实现的程序之后，下一步就是把那些代码喂给解释器，然后解释器把我们的代码当作一段文本，当作一些字符串一样的东西，进行解释求值。

本质上就是一句话：我们写的代码之于解释器来说就是一些待求值的数据。正所谓： **code is data, data is code.**

具体来讲，解释器求值有一个通用流程：Read-Eval-Print Loop，读入-求值-打印输出 循环进行。

光是这么讲还是太抽象。还需要继续拆分：

  * 读入 
    * 词法分析生成「token 序列」
    * 语义分析生成「recursive expression representation」
  * 求值 
    * 互递归 
      * eval ：对表达式求值，得到过程和参数，调用 apply
      * apply：将过程应用于操作数，操作数本身可能是又一个表达式，调用 eval
    * 递归到最后，得到「程序的值」
  * 大部分时候，价值发生在副作用上
  * 而我们的程序，就是让副作用可控且符合期待

  
_不知是该恭喜，还是该怎样，总之阅读到该文的，你是第 人。每一次刷新，都是不同的自己。_

* * *