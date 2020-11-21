---
layout: post
title: programming in lua(thrid edition)笔记---10 complete examples 
tags: [lua文章]
categories: [topic]
---
### 10 Complete Examples

本章分析了三个例程：八皇后问题、单词频数统计、马尔可夫链算法，是对之前所学的应用，几乎没有新的东西，所以本章笔记较少。  

  * 用逻辑运算充当三目运算
    
        1  
    

|

    
        io.write(a[i] = j and "X" or "-", " ")  
      
  
---|---  
  * 用`or`实现将table中新元素初始化0
    
        1  
    2  
    3  
    4  
    

|

    
        local counter = {}  
    for w in allwords() do  
    	counter[w] = (counter[w] or 0) + 1  
    end  
      
  
---|---