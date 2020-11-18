---
layout: post
title: Redisson lua脚本执行踩坑之神秘的引号 
tags: [lua文章]
categories: [lua文章]
---
最近项目需求，在redis中需要执行批量删除指定key，并且要支持原子操作，那么当然只有自己写lua脚本了。

项目中使用的是redisson作为redis连接工具，首先先定义好lua脚本：

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    private static final String BATCH_DEL_SCRIPT = "for k,v in  pairs(ARGV) do n " +  
             "redis.call('del',v) n" +  
             "end n" +  
             "return #ARGV";  
      
  
---|---  
  
脚本很简单，根据参数传入的key，进行循环删除，然后返回参数的数量，当然这里也可以改为返回删除的数量总和。  
redisson使用：

    
    
    1  
    

|

    
    
    redissonClient.getScript().evalAsync(RScript.Mode.READ_WRITE, BATCH_DEL_SCRIPT, RScript.ReturnType.INTEGER, keys, keys.toArray());  
      
  
---|---  
  
然后开始执行以上代码，就是死活删除不了指定的key，比如redis中存在key：aaaa，执行命令后aaaa还在。各种调查和学习lua语法后还是不行。于是转换思路，不执行删除，而是将删除操作修改为赋值操作，只要将数据置为空也可以达到效果。于是脚本变为：

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    private static final String BATCH_DEL_SCRIPT = "for k,v in  pairs(ARGV) do n " +  
             "redis.call('set',v,'') n" +  
             "end n" +  
             "return #ARGV";  
      
  
---|---  
  
通过以上脚本执行后，还是不行，aaaa还存在，而且刷新后多了key：”aaaa”。什么鬼!!  
怎么多了个key值，而且是加了引号的。一脸闷逼中。

知道了设置的规则后，那么解决办法就简单了，只需要在设置或删除key之前去除多余的引号就好了，于是脚本改为：

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    private static final String BATCH_DEL_SCRIPT = "for k,v in  pairs(ARGV) do n " +  
             "redis.call('del',string.gsub(v,'"',''),'') n" +  
             "end n" +  
             "return #ARGV";  
      
  
---|---  
  
这样完美解决。

至于为什么lua脚本执行后，字符串对象为什么会多了双引号，这个还在调查中，如果你知道，欢迎评论告知，不胜感激。