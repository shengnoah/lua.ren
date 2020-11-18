---
layout: post
title: cocos2dx lua 常见错误汇总与解决方案 
tags: [lua文章]
categories: [lua文章]
---
cocos2dx
lua开比较苦恼的是，没办法断点或者直接一步一步调试处理，但是幸好官方有点良心，给了一个打印的调试Log，我们可以使用它调试和查找对应的错误。

这里就记录一些，个人学习和实战中遇到的一些错误和问题的总结

  * not found view “ApiTest” in search paths “app.views” 
    * ApiTest不在Views里面或者名字写错了，或者没有初始化

  * attempt to index upvalue ‘HttpSingleton’ (a boolean value)
    * 末尾没有返回HttpSingleton

  * attempt to index local ‘self’ (a nil value) 
    * 参数传错了或者将逗号(.)与冒号(:)搞混了

  * attempt to call method ‘schedulerScriptFunc’ (a nil value)
    * 方法写错了

  * attempt to call global ‘getScheduler’ (a nil value)
    * 代码或者语法错误

  * syntax error during pre-compilation
    * 严重的语法错误

  * invalid ‘cobj’ in function ‘lua_cocos2dx_Node_getLocalZOrder’
    * 这个报错是lua的变量还在，但是他底层对应的C++对象已被销毁。

  * InterpolationMissingOptionError: Bad value substitution:
    * 在执行genbindings.py脚本文件时，不要在该文件的外部路径执行，需要CD到该文件目录下执行./genbindings.py

  * TranslationUnitLoadError: Error parsing translation unit.
    * 基本都是.ini文件没有配置正确，仔细检查一下 .ini文件里的 “headers = ”指向的路径是否正确

  * Xcode编译错误，header error
    * 再此外，把.hpp和.cpp加进cocos2d_lua_bindings.xcodeproj时，target需要勾选ios。在设置 UserHeaderSearchPaths 时，注意选择该proj的Ios target进行设置 ，不要选择了mac target 选项，否则ios环境编译不过

  * mportError: No module named yaml
    * 安装了yaml模块，如果还是报错找不到这个模块，这个就是是路径问题，因为我从新安装了python，然而：这里使用的python是系统默认的#!/usr/bin/python，处理好python版本和当前匹配的版本
      * 网上说还有其他方式解决(待验证)
        * import sys
        * sys.path.append(‘/xxx/xxxxx/‘) 加进去也行。

  * attempt to perform arithmetic on local ‘x’ (a nil value)
    * 忘了记录…..

这里专门说需要关于cocos2dx lua开发中的错误，其实cocos2dx lua中也和iOS中一样，分为两种错误：编译时错误和运行时错误

  * 编译错误，一般是语法上存在问题，编译过不去;
  * 运行错误，是指程序在运行过程中出现错误，只能说是程序存在一定的边界bug;

### 编译时错误和运行时错误

#### 编译错误，

比如上面一条

    
    
    error:syntax error during pre-compliation
    

就属于编译语法错误，这里报错其实还会有一些提示信息，如果我们可以通过提示信息，找到LuaStack，在LuaStack中有个LuaStack::luaLoadBuffer(…)，然后查看源码如下：

    
    
    switch (r)
    {
    　　case LUA_ERRSYNTAX:     // 编译出错
    　　CCLOG("[LUA ERROR] load "%s", error: syntax error during pre-compilation.", chunkName);
    　　break;
    　　case LUA_ERRMEM:        // 内存分配错误
    　　CCLOG("[LUA ERROR] load "%s", error: memory allocation error.", chunkName);
    　　break;
    　　case LUA_ERRRUN:        // 运行错误
    　　CCLOG("[LUA ERROR] load "%s", error: run error.", chunkName);
    　　break;
    　　case LUA_YIELD:         // 线程被挂起
    　　CCLOG("[LUA ERROR] load "%s", error: thread has suspended.", chunkName);
    　　break;
    　　case LUA_ERRFILE:
    　　CCLOG("[LUA ERROR] load "%s", error: cannot open/read file.", chunkName);
    　　break;
    　　case LUA_ERRERR:        // 运行错误处理函数时发生错误
    　　CCLOG("[LUA ERROR] load "%s", while running the error handler function.", chunkName);
    　　break;
    　　default:
    　　CCLOG("[LUA ERROR] load "%s", error: unknown.", chunkName);
    }
    

所以，无论怎样，出现错误时，都能将错误信息返回到堆栈的最顶层

##### 如果遇到编译错误是，可以通过如下的代码来打印错误信息：在以上{}之外的后面加

    
    
    const char* error = lua_tostring(L, -1);
    CCLOG("[LUA ERROR] error result: %s",error);
    lua_pop(L, 1);
    

#### 运行错误

而针对于运行错误，一般情况下，你可以参考如下代码(此代码在main.lua中)：

    
    
    -- lua提供，调用其他函数，可以捕捉到错误，第一个参数为要调用的函数， 第二个参数为捕捉到错误时所调用的函数
    -- 返回的参数status为错误状态， msg为错误信息
    local status, msg = xpcall(main, __G__TRACKBACK__)
    if not status then
    print(msg)
    end
    

  * 推荐
    * <http://www.cocoachina.com/bbs/read.php?tid=200145>
    * [https://blog.csdn.net/msdb198901/article/details/52128175 ](https://blog.csdn.net/msdb198901/article/details/52128175)