---
layout: post
title: 基于AndroLua_Pro的Android开发代码示例 
tags: [lua文章]
categories: [topic]
---
> 温馨提示: **请使用电脑浏览器打开** ,以确保最佳的阅读体验,谢谢.(￣▽￣)”

我希望你有了一些AndroLua_Pro的基础,
如果还没有可以结合这篇文章([基于AndroLua_Pro的Android开发笔记](/2019/05/28/AndroLua_Pro_0/))来看.

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
        --改为  
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

> (这个是一个很原则的问题, 因为Lua底层其实就是C, 因为动态库文件的存在.os)

  * **线程操作**

> (部分功能不需要调来或耗时久等, 比如写入文件, 尽量用线程)

# 2\. 文件类

> 其实对于写入文件用线程来执行就好了.

## >1\. fw.写入文件

  * fw(string 路径, string 内容) 无返回值 | 线程
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
        function (a,b)--j  
      local t=os.clock()  
      task(function(a,b)  
        f=File(tostring(File(tostring(a)).getParentFile())).mkdirs()  
        io.open(tostring(a),"w"):write(tostring(b)):close()  
      end,a,b,function() print("写入文件用时"..os.clock()-t.."s")end)  
    end  
      
  
---|---  

## >2\. fr.读取文件

  * fr(string 路径) 返回文件内容
  * 这里我用了几个的 **三目运算** , 啦啦.
    
        1  
    2  
    3  
    4  
    5  
    6  
    

|

    
        function fr(f)--j  
      F,n=io.open(f)--文件不存在or是否为文件夹  
      return (n and {false} or   
      {(not File(f).isDirectory() and   
      {F:read("*a")} or {false})[1]})[1]  
    end  
      
  
---|---  

## >3\. fd.删除文件

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
      end,function() print("删除文件用时"..os.clock()-t.."s")end)  
    end  
      
  
---|---  

## >4\. fbool.判断文件存在

  * fbool(string 文件路径) 存在返回true,不存在返回false
    
        1  
    2  
    3  
    

|

    
        function fbool(f)  
      return (f and {File(f).exists()} or {false})[1]  
    end  
      
  
---|---  

## >5\. fs.文件大小

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

## >6\. ft.文件最后修改时间

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

## >7\. 文件字符串替换

  * fchange(sting )

> 等待更新