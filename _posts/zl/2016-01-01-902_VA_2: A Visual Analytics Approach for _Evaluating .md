---
layout: post
title: VA_2: A Visual Analytics Approach for _Evaluating Visual Analytics Applications 
tags: [lua文章]
categories: [topic]
---
论文：VA^2: A Visual Analytics Approach for // Evaluating Visual Analytics
Applications

作者：Tanja Blascheck, Markus John, Kuno Kurzhals, Steffen Koch, Thomas Ertl

发表会议：VAST 2015  
本文提出一个用于展示和分析“用户如何使用可视化分析系统”的高度交互可视化的环境，途径是分析用户在可视分析过程中产生的 thinking aloud,
interaction, and eye movement 数据。

[![img](http://www.cad.zju.edu.cn/home/vagblog/wp-
content/uploads/2015/12/VA1.png)](http://www.cad.zju.edu.cn/home/vagblog/wp-
content/uploads/2015/12/VA1.png)

评估可视分析的技术在探索用户如何理解一个新方法的过程中起着相当重要的作用，目前有一些评估可视分析的技术，但却很少有人做一个并列的综合分析评估程序。本文采取了综合的可视分析技术，对不同的数据类别进行分析，以评估可视化和可视化系统。

评估可视化系统主要收集三种数据：

  1. Interaction Logs  
用户在哪些地方进行交互，文章根据参考文献中对 Interaction logs 分成了十一类

  2. Thinking aloud protocols  
一般用户在获得信息时会有所表现

  3. Eye tracking

找到哪些内容是用户所聚焦的

[![img](http://www.cad.zju.edu.cn/home/vagblog/wp-
content/uploads/2015/12/VA2.png)](http://www.cad.zju.edu.cn/home/vagblog/wp-
content/uploads/2015/12/VA2.png)

文章通过对 thinking aloud, interaction, and eye movement 数据的处理来同步展示用户的思维过程，有助于分析师分析
patterns.

用户分析：  
最终用户认为该方法的优点在于编码是颜色和形状减少视觉的混乱，且分析师不必需要太多的专业知识就可以上手。但用户也认为有一些缺点，比如说：interaction
logs 在编码时十一种颜色很难记住，自言自语的数据不能被隐藏，分析师在分析的时候需要大量的滚动操作等等。  
本文的最大贡献就是评估 VA 系统时分析了 Eye tracking、Thinking aloud、Interaction logs
三种数据。且有很强的普适性，除了 VarifocalReader 文章还讨论了 Word Cloud Explorer 的使用情况。

总结文章的优点有如下：  
框架明了，介绍清晰，想法新颖，编码科学。