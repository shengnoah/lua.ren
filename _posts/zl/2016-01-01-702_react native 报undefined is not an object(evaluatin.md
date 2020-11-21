---
layout: post
title: react native 报undefined is not an object(evaluating this.state.currentPage) 
tags: [lua文章]
categories: [topic]
---
**之前由于项目不忙，学习了一个多月的React Native，现将遇到的问题及解决方案记录一下**

* * *

在方法中使用state中的属性的时候出现这样的错误，原因是此时的this指向的对象是当前的方法而不是这个对象类

解决方法有两个：

1.绑定这个对象类

    
    
    1  
    

|

    
    
    onPress={this.onClick.bind(this)}  
      
  
---|---  
  
2.使用箭头函数

    
    
    1  
    

|

    
    
    onPress={()=>this.onClick();}  
      
  
---|---  
  
* * *
    
    
    个人比较推荐第二种方式