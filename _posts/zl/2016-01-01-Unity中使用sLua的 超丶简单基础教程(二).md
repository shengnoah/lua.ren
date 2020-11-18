---
layout: post
title: Unity中使用sLua的 超丶简单基础教程(二) 
tags: [lua文章]
categories: [lua文章]
---
![](https://Lafree317.github.io/../../images/Slua-1.png)

### 前言

[Unity中使用sLua的
超丶简单基础教程(一)](https://lafree317.github.io/2018/01/20/2018-01-20%20-%20Unity%E4%B8%AD%E4%BD%BF%E7%94%A8sLua%E7%9A%84%20%E8%B6%85%E4%B8%B6%E7%AE%80%E5%8D%95%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B\(%E4%B8%80\)/)

上一篇博客讲了一下简单调用LuaState读取Lua代码并执行

本篇要讲一下如何更改路径并使得Lua可以调用UnityEngine代码的方法

虽然简短但也是长时间爬坑试验出来的(因为基础教程真的好少啊….)希望对大家有帮助..

### 正文

在创建一个新的场景,一个新的CreateEmpty把C#脚本挂上去代码如下:

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    

|

    
    
    using System;  
    using System.Collections;  
    using System.Collections.Generic;  
    using UnityEngine;  
    using System.IO;  
    using SLua;  
      
    public class TestLua : MonoBehaviour {  
      
    	void Start () {  
    		LuaSvr svr = new LuaSvr();// 如果不先进行某个LuaSvr的初始化的话,下面的mianState会爆一个为null的错误..  
            LuaSvr.mainState.loaderDelegate += LuaReourcesFileLoader;  
    		svr.init(null, () => // 如果不用init方法初始化的话,在Lua中是不能import的  
    		{  
                svr.start("Test");  
    		});  
    	}  
      
        // SLua Loader代理方法  
        private static byte[] LuaReourcesFileLoader(string strFile)  
        {  
            // 这里为了测试就不先判断为空,开发的时候再加上  
            string filename = Application.dataPath + "/Scripts/Lua/" + strFile.Replace('.', '/') + ".txt";  
            return File.ReadAllBytes(filename);  
        }  
    }  
      
  
---|---  
  
然后在Assest/Scripts/Lua/中创建一个Lua.txt脚本 命名路径都可以更改代码如下:

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
    
    import "UnityEngine"  
      
    function main()  
        print("Lua创建了一个Cube")  
        local cube = GameObject.CreatePrimitive(PrimitiveType.Cube)  
    end  
      
    main()  
      
  
---|---  
  
#### 效果图

![](https://user-gold-
cdn.xitu.io/2018/2/3/1615b60570c81eb1?w=504&h=324&f=png&s=66637)

#### 其他

上一篇主要说了一下如何引用SLua读取lua脚本  
这一篇主要说了一下Lua如何调用UnityEngine的方法  
下一遍我会说一下开发一个中间层,把Start Awake Update等方法都传递到Lua中  
最终完成一个纯Lua项目

本篇教程很基础,如果有精力会将之后学习到的知识都整理成博客分享给大家~