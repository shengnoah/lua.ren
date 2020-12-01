---
layout: post
title: TeX的家族：LaTeX， pdfTeX， XeTeX， LuaTeX和ConTeXt 
tags: [lua文章]
categories: [topic]
---
最近这段时间一直在研究LaTeX，但是一直被各种各样的词汇搞到晕头转向，后来找到了两篇文章，文章中对于各种名词的解释比较到位，并从TeX的发展讲解了整个历史，其中一个作者更是绘制出了整个家族树，对于理解TeX的历史很有帮助，奈何两篇文章都是英文，这么好的文章应该分享给国内的朋友们，同时也为了回忆一下自己之前学的英语，这里翻译了其中的一篇文章，如果有错误，还望大家原谅  
  
 _ShareLaTeX_ 刚刚支持了 _pdflatex_ ， _latex_ 和 _xelatex_ 编译环境，你知道这些都是什么意思吗？请往下看。  
LaTeX的故事可以追溯到1978年， **高德纳（ Donald Knuth）**
第一次觉察到需要有一套高质量的排版系统。那时他构建的排版系统为后来LaTeX的质量提供了良好的保障。虽然近几十年来很多的功能被添加了进去，但是直到今天，它仍然是最好的排版工具。事实上，LaTeX就是在高德纳原生系统中添加功能而发展来的，那个原生系统叫做
**TeX** 。

## 原生TeX

原生的TeX衍生出了一个庞大的工具家族，当你第一次去了解这个家族时，你看到的是LaTeX， pdfTeX， XeLaTeX，
LuaTeX和ConTeXt等等这些让人发晕的东西。这些东西的老祖宗就是高德纳发明的TeX。TeX可以使一个普通的文本文档变成美丽的排版文档。高德纳非常注重TeX系统中对于细节的处理，一个简单的TeX例子如下下：  

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    TeX{} is good at typesetting words like `fjord'， `efficiency'，  
    and `fiasco'. It is also good at typesetting math like，  
    $a^2 + b^2 = c^2$.  
    bye  
      
  
---|---  
  
通过TeX程序生成的文档就变成了下面这个样子

![](http://fatmouse.xyz//uploads/tex-example.png)

注意例子当中的 **fi** ， **fj** ， **ff**
是怎样完美的组合到一起的，还有数学公式是怎样完美的被呈现的。从更大的方面来说，TeX非常擅长处理那些特殊位置出现的断行和连字符，并且很好的放置它们。

TeX不仅仅在排版方面表现出众，它还有一堆强大的命令，正如上面例子中的`bye`和`TeX`。这些命令可以做简单的事情例如改变字体大小，或者文档的行文方向，还可以做出一些强大的功能比如保持页面中段落的交叉引用，或者自动生成目录等等。TeX系统大概内建了300条命令，除此之外还支持根据这些命令重新建立新的命令。高德纳教授额外编写了大约600条有用的命令，并将它们放置到叫做
**Plain TeX** 的包里以便使通常的排版方便些。

## LaTeX

TeX和Plain TeX中的命令仍然很基础，而且不是很容易理解和运用。为了解决这个问题， **Leslie Lamport**
教授在19世纪80年代早期创造了一个基于TeX的更高层次的语言，叫做 **LaTeX**
。LeTeX是以TeX底层命令为基础重新定义的命令集，而且被赋予了许多含义。当你使用LaTeX命令的时候，实际上它会被解析成底层的TeX命令，除非你直接只用原有的TeX命令。一些概念像`(usepackage{...})，
environments (begin{environment} ... end{environment})`和`document classes
(documentclass{...})`都被Leslie Lamport教授加入了LaTeX。

Leslie
Lamport教授除了建立了LaTeX标准包之外，还允许开源社区对其进行发展。到目前为止，已经有了成千上万的LaTeX包可以让你实现从画图到编花纹各种各样的排版，还有学学多多的文档类让你应付不同的文档排版，无论你是写一本书，一份实验报告还是一份简历。许多出版商和期刊杂志都有他们自己的模板以满足己方需求。

## pdfTeX

自从TeX系统诞生以来已过20多载，但其表现是相当的稳定，高德纳在1989年宣布所有功能都被定型，以后所做的工作只是BUG得修改。当然，这并没有停止LaTeX不断前进的脚步。事实上，由于TeX系统的异常稳定，使得各种各样像LaTeX一样的衍生版可以不断繁荣发展。

注意，这里并不是说底层TeX系统在这20年里没有一点进步，在TeX系统性能稳定的基础上，所有的衍生工具都是伴随着TeX而一起向前发展的。最重要的一次改进便是在19世纪90年代
**Hàn The Thành** 作为他的攻读博士主题所创作出得 **pdfTeX** 。原生TeX系统所产生的排版文档格式叫做 **DVI(DeVice
Independent format)** ，这个文件可以转换成 **Postscript**
格式最终打印出来。然而自从1993年PDF格式诞生以来我们看到了PDF是比postscript更好的一种文档格式。PDF有很多优点，譬如段落中的超链接，段落的元数据可以让你在PDF浏览器的左边看到文档的整个目录，而且还支持越来越多的现代图片格式。pdfTeX便是对TeX进行了修改使其可以直接输出PDF文档格式。

今天当你安装了一个LaTeX的发行版之后，其里面就包含了两个程序—— **TeX** 和 **pdfTeX** 。同时发行版应该也会包含另外另个程序
**LaTeX** 和 **pdfLaTeX** ，但这两个只不过是 **TeX** 和 **pdfTeX** 分别对应的高级封装而已。如果你运行的是
**TeX** 和 **LaTeX** ，你将得到DVI文件，进一步会处理成postscript或者PDF。而如果你使用了 **pdfTeX** 或
**pdfLaTeX** ，你便可以直接得到PDF文件。

多数情况下，pdfTeX和pdfLaTeX要比TeX和LaTeX有更强大的功能，但只有一个缺点。原生TeX和DVI格式支持 **encapsulated
postscript images (.eps)**
图像格式，并可以将其轻松的嵌入到postscript文件中，pdfTeX却不行，它只能将其转化为PDF格式，再进行嵌入（通常来讲LaTeX的发行版中都会包含
**epstopdf** ，这个程序可以将 **.eps** 文件转换为 **.pdf**
文件），除此之外，pdfTeX可以支持.png，.jpg，.pdf图像格式。

## XeLaTeX和LuaTeX

到目前为止我们看到TeX系统自诞生以来已经发展出两条不同的路线：一条是为了更好地在顶层使用原生TeX系统而建立的LaTeX，另一条路线就是为了支持PDF输出而建立的pdfTeX。故事并没有结束，还有许许多多的功能没有被添加进TeX这个大家族中。2004年
**Jonathon Kew** 创造了 **XeTeX**
，这是另一个以原生TeX引擎为底层的高级语言集，这次不仅使得排版系统支持原有的英文数字和字母，还支持了现代的各国字体。这使得其他非英文文字国家使用TeX系统更加容易方便，同时可以让原本只能在文档处理器中使用的字体应用到LaTeX文档当中。

**LuaTeX**
是为了让TeX系统更能贴近现有的编程语言，其思想就是让TeX可以像编程语言一样可以做任何你想做的事情，对于目前来说LaTeX的内部构造对于不熟悉TeX系统的人是很难理解的。
**Lua** 是一种简单，稳定的脚本语言，对于扩展LaTeX宏包来说比较理想。但是到2012年它的API仍然不是很稳定，所以还不能使用。

## ConTeXt

我们在上文提到过LaTeX作为原生TeX系统的一种命令集扩展，但这不是唯一的一个扩展包。1990年 **Hans Hagen** 建立了另一个扩展系统
**ConTeXt**
。LaTeX的主要目的在于使作者从繁琐的排版中解脱出来，当你使用`section`或者`emph`命令的时候，你不用去在乎它们会怎样做，这些会留给系统处理。另一方面来讲，ConTeXt可以说是提供了更加方便的排版写作方式。不过我对其不是很熟悉，所以这里不便过多介绍。

作为一个历史回顾，我不得不提起 **AMSTeX** ，这是由 **美国数学协会（AMS）**
从1982年到1985年所创建的文本文档宏包，它仍然存在于当前的宏包中，很多文档的开始你都会见到它`usepackage{amsmath}`。

## 未来

TeX和LaTeX的未来会是什么样？我不知道，但是Alan Kay曾经说过

> **“** 预测未来最好的方式就是创造未来！

* * *

以下是另一个作者所总结的家族树，[点击可以查看PDF](http://disco.bu.edu/~tsl/TeX-Family-Tree-
Timeline/tft-portrait.pdf)

![](http://fatmouse.xyz//uploads/latex_family_tree_l.jpg)

原文地址：[The TeX family tree: LaTeX, pdfTeX, XeTeX, LuaTeX and
ConTeXt](https://www.sharelatex.com/blog/2012/12/01/the-tex-family-tree-latex-
pdftex-xelatex-luatex-context.html)

家族树地址：[TeX family tree with
timeline?](http://tex.stackexchange.com/questions/42594/tex-family-tree-with-
timeline)

原作者： James Allen, Todd Lehman

翻译：@胖老鼠

时间：01 Dec 2012