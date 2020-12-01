---
layout: post
title: Rstudio编译Rmd报错：the fontspec package requires either XeTeX or LuaTeX. 
tags: [lua文章]
categories: [topic]
---
对于一些常规的报告，我一般是会写一个脚本来自动运行、生成格式化的模板。然后通过 `R` 的软件包 `knitr`
来实现自动化报告效果。这样可以十分方便的把数据分析与数据报告结合在一起，如果在分析端的脚本有变化，我们只需要把主要精力放在修改 `R`
脚本就可以了，而在报告端，在每次生成报告时会自动根据分析脚本生成的数据进行更新，从而完成「时时更新与可重复研究」的伟大目标。

比如，我们公司目前要求每周五收盘会，出具一份所有基金的净值报告文档。这个一方面要求数据保持与数据库的同步更新，二来还要求文档有一定的「逼格」。针对第一条，我本身在日常工作中就是使用
`R` 来处理各项数据分析，而对于第二条，以前我也系统性的学习过 `LaTeX` 这个排版届的「一股清流」。因此，自然而然的，我就使用 `Rstudio`
来完成这个常规化的工作了。

# 问题描述

可是，最近在新安装的 `Rstudio` 中运行原来的脚本，却出现一个奇怪的报错。我怀疑是 `Rstudio` 因为版本更新的问题，不再使用原来的部分
`yaml` 参数。下面我们来看看具体的报错：

    
    
    /usr/bin/pandoc +RTS -K512m -RTS ReportingTemplate.utf8.md --to beamer --from markdown+autolink_bare_uris+ascii_identifiers+tex_math_single_backslash --output ReportingTemplate.tex --template backtesting_template.tex --table-of-contents --variable theme=SOCIB --highlight-style espresso --latex-engine pdflatex --self-contained --latex-engine=xelatex 
    output file: ReportingTemplate.knit.md
    
    ! Fatal fontspec error: "cannot-use-pdftex"
    ! 
    ! The fontspec package requires either XeTeX or LuaTeX.
    ! 
    ! You must change your typesetting engine to, e.g., "xelatex" or
    ! "lualatex"instead of "latex" or "pdflatex".
    ! 
    ! See the fontspec documentation for further information.
    ! 
    ! For immediate help type H <return>.
    !...............................................  
    
    ! 
    ! The fontspec package requires either XeTeX or LuaTeX.
    ! 
    ! You must change your typesetting engine to, e.g., "xelatex" or
    ! "lualatex"instead of "latex" or "pdflatex".
    ! 
    ! See the fontspec documentation for further information.
    ! 
    ! For immediate help type H <return>.
    !...............................................  
    
    ! The fontspec package requires either XeTeX or LuaTeX.
    ! 
    ! You must change your typesetting engine to, e.g., "xelatex" or
    ! "lualatex"instead of "latex" or "pdflatex".
    ! 
    ! See the fontspec documentation for further information.
    ! 
    ! For immediate help type H <return>.
    !...............................................  
    
    ! 
    ! You must change your typesetting engine to, e.g., "xelatex" or
    ! "lualatex"instead of "latex" or "pdflatex".
    ! 
    ! See the fontspec documentation for further information.
    ! 
    ! For immediate help type H <return>.
    !...............................................  
    
    ! You must change your typesetting engine to, e.g., "xelatex" or
    ! "lualatex"instead of "latex" or "pdflatex".
    ! 
    ! See the fontspec documentation for further information.
    ! 
    ! For immediate help type H <return>.
    !...............................................  
    
    ! "lualatex"instead of "latex" or "pdflatex".
    ! 
    ! See the fontspec documentation for further information.
    ! 
    ! For immediate help type H <return>.
    !...............................................  
    
    ! 
    ! See the fontspec documentation for further information.
    ! 
    ! For immediate help type H <return>.
    !...............................................  
    
    ! See the fontspec documentation for further information.
    ! 
    ! For immediate help type H <return>.
    !...............................................  
    
    ! 
    ! For immediate help type H <return>.
    !...............................................  
    
    ! For immediate help type H <return>.
    !...............................................  
    
    Error: Failed to compile ReportingTemplate.tex. See ReportingTemplate.log for more info.
    In addition: There were 50 or more warnings (use warnings() to see the first 50)
    Execution halted

# 解决思路

## `LaTeX` 编译引擎

细看这个错误，应该是提示我们需要使用 `XeTeX` 这个新的编译引擎来执行 `fontspect` 包，也就是可以允许我们调用操作系统的字体。而现在
`Rstudio` 无法找到 `XeTeX`。因此，很直观的想法就是去设置一下这个参数。

## SO 的参考

然后还是用 `google` 去搜索了一下其他人是否也遇到类似的问题。果真如此。在 `StackOverFlow`
上面的链接，[https://stackoverflow.com/questions/36806255/rstudio-on-windows-not-
using-
xelatex](Rmarkdown%20The%20fontspec%20package%20requires%20either%20XeTeX%20or%20LuaTeX.)。有个人也是遇到相同的问题，然后后面的提示也是十分给力，验证了我的想法。

## 原来的 `yaml`

这里我们来看一下原来的 `yaml` 表头，虽然也指定了 `pandoc_args` 为 `xelatex`，但是可能是由于 `Rstudio`
在新版本里去掉了对这个参数的引用，才导致我们在编译 `.tex`
文件的时候无法使用该编译引擎。而新的参数列表方式与其他的参数选项一致，即作为一个参数值来传递。这也更加符合 `knitr` 的设计哲学。

    
    
    ---
    title: "汉云投资"
    subtitle: "基金报告"
    output: 
      beamer_presentation:
        theme: SOCIB      ## sthlm, SOCIB
        template: backtesting_template.tex
        pandoc_args: '--latex-engine=xelatex'
        keep_tex: false
        toc: true
        highlight: espresso  ## "default", "tango", "pygments", "kate", "monochrome", 
                            ## "espresso", "zenburn", "haddock"
    date: cntoday
    print: no
    ---

## 修改 `yaml` 解决问题

既然现在问题解决了，那就修改一下。

    
    
    ---
    title: "汉云投资"
    subtitle: "基金报告"
    output: 
      beamer_presentation:
        theme: SOCIB      ## sthlm, SOCIB
        template: backtesting_template.tex
        # pandoc_args: '--latex-engine=xelatex'
        latex_engine: xelatex
        keep_tex: false
        toc: true
        highlight: espresso  ## "default", "tango", "pygments", "kate", "monochrome", 
                            ## "espresso", "zenburn", "haddock"
    date: cntoday
    print: no
    ---

注意，这里为了兼容旧版本，我并没有删除 `pandoc_args: '--latex-
engine=xelatex'`，而是把这个先注释掉。这一点在写代码的时候算是一个小技术，即一次只修改一个地方，等到程序运行稳定后，再对修改过的代码进行删除或移到备用文件，以防止后来又要使用之前的就代码。

好了，这样我又可以愉快的「偷懒」了。让计算机去做「boring stuff」吧。