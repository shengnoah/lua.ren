---
layout: post
title: 基于AndroLua_Pro的Android开发代码示例 
tags: [lua文章]
categories: [lua文章]
---
> 温馨提示: **请使用电脑浏览器打开** ,以确保最佳的阅读体验,谢谢.(￣▽￣)”

我希望你有了一些AndroLua_Pro的基础,
如果还没有可以结合这篇文章([基于AndroLua_Pro的Android开发笔记](https://alittlemc.github.io/2019/05/28/AndroLua_Pro_0/))来看.

其实我代码都放在了GitHub, 可以去看看呗.

  * **缓存**

    * **缓存函数**
        
                 1  
        2  
        3  
        4  
        5  
        6  
        7  
        8  
        9  
        

|

        
                  
        for i=0,10000,1 do  
            print(string.format("%s",tostring(i)))  
        end  
          
        local fun=string.format  
        for i=0,10000,1 do  
            print(format("%s",tostring(i)))  
        end  
          
  
---|---  
  * **缓存定值** 或者 **减少无光运算** 等:

> (这个不举例了, 我相信你懂)

  * **除法换乘法** :

> (你要知道计算机算乘法比乘法高效多了)

  * **用Lua不用Java**
    * **能用Lua语句就不要用Java语句** :

> (统计结果是lua会高效6~30倍)

  * **能用静态/动态库(自带的)方法就不要自写方法** :

> (这个是一个很原则的问题, 因为Lua底层其实就是C, 因为动态库文档的存在.os)

  * **线程操作**

> (部分功能不需要调来或耗时久等, 比如写入文档, 尽量用线程)

# [](https://alittlemc.github.io/#2-%E6%96%87%E4%BB%B6%E7%B1%BB "2. 文档类")2\.
文档类

> 其实对于写入文档用线程来执行就好了.

##
[](https://alittlemc.github.io/#gt-1-fw-%E5%86%99%E5%85%A5%E6%96%87%E4%BB%B6
">1. fw.写入文档")>1\. fw.写入文档

  * fw(string 路径, string 内容) 无返回值 | 线程
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
        function (a,b)  
      local t=os.clock()  
      task(function(a,b)  
        f=File(tostring(File(tostring(a)).getParentFile())).mkdirs()  
        io.open(tostring(a),"w"):write(tostring(b)):close()  
      end,a,b,function() print("写入文档用时"..os.clock()-t.."s")end)  
    end  
      
  
---|---  

##
[](https://alittlemc.github.io/#gt-2-fr-%E8%AF%BB%E5%8F%96%E6%96%87%E4%BB%B6
">2. fr.读取文档")>2\. fr.读取文档

  * fr(string 路径) 返回文档内容
  * 这里我用了几个的 **三目运算** , 啦啦.
    
        1  
    2  
    3  
    4  
    5  
    6  
    

|

    
        function fr(f)  
      F,n=io.open(f)--文档不存在or是否为文档夹  
      return (n and {false} or   
      {(not File(f).isDirectory() and   
      {F:read("*a")} or {false})[1]})[1]  
    end  
      
  
---|---  

##
[](https://alittlemc.github.io/#gt-3-fd-%E5%88%A0%E9%99%A4%E6%96%87%E4%BB%B6
">3. fd.删除文档")>3\. fd.删除文档

  * fd(string 路径) 无返回值 | 线程
    
        1  
    2  
    3  
    4  
    5  
    6  
    

|

    
        function fd(f)  
      local t=os.clock()  
      task(function(f)  
        File(f).delete()  
      end,function() print("删除文档用时"..os.clock()-t.."s")end)  
    end  
      
  
---|---  

##
[](https://alittlemc.github.io/#gt-4-fbool-%E5%88%A4%E6%96%AD%E6%96%87%E4%BB%B6%E5%AD%98%E5%9C%A8
">4. fbool.判断文档存在")>4\. fbool.判断文档存在

  * fbool(string 文档路径) 存在返回true,不存在返回false
    
        1  
    2  
    3  
    

|

    
        function fbool(f)  
      return (f and {File(f).exists()} or {false})[1]  
    end  
      
  
---|---  

##
[](https://alittlemc.github.io/#gt-5-fs-%E6%96%87%E4%BB%B6%E5%A4%A7%E5%B0%8F
">5. fs.文档大小")>5\. fs.文档大小

  * fs(string 路径,bool 模式) 模式决定返回值
  * 模式:
    * true->返回 带有单位, string
    * false->返回 字节数, number
        
                1  
        2  
        3  
        4  
        5  
        

|

        
                function fs(f,mod)  
          size=File(tostring(f)).length()  
          Sizes=Formatter.formatFileSize(activity,size)  
          return (mod and {Sizes} or {size})[1]  
        end  
          
  
---|---  

##
[](https://alittlemc.github.io/#gt-6-ft-%E6%96%87%E4%BB%B6%E6%9C%80%E5%90%8E%E4%BF%AE%E6%94%B9%E6%97%B6%E9%97%B4
">6. ft.文档最后修改时间")>6\. ft.文档最后修改时间

  * ft(string 路径) 返回时间字符串
    
        1  
    2  
    3  
    4  
    5  
    

|

    
        function ft(f)  
      return Calendar.getInstance()  
      .setTimeInMillis(File(f).lastModified())  
      .getTime().toLocaleString()  
    end  
      
  
---|---  

##
[](https://alittlemc.github.io/#gt-7-%E6%96%87%E4%BB%B6%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%9B%BF%E6%8D%A2
">7. 文档字符串替换")>7\. 文档字符串替换

  * fchange(sting )