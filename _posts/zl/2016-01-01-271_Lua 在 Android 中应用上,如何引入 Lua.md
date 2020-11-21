---
layout: post
title: Lua 在 Android 中应用上,如何引入 Lua 
tags: [lua文章]
categories: [topic]
---
转载请附原文链接：[Lua 在 Android 中应用上,如何引入
Lua](http://yongyu.itscoder.com/2018/04/16/yongyu_20180416_lua_android_one/)

## [](https://yongyu.itscoder.com/#%E4%B8%80%E3%80%81%E6%A6%82%E8%A6%81
"一、概要")一、概要

 _注：该部分适合不熟悉 NDK 编译的新手看，老司机请绕行_

最近公司在做一个项目，利用一份 XML 文档来布局绘制 Android 和 iOS
接口，接口与用户的交互逻辑部分开始是根据自己定义的协议进行手动解析实现，但是这样有两个弊端，第一是每次需要一些特殊功能时候需要事先定义好协议，第二个是自己定义的协议在进行一些复杂的逻辑判断很麻烦，写起来很不方便。所以决定引入脚本来实现逻辑交互功能。说起脚本语言大家应该马上会想起
JavaScript， JavaScript 在前端开发应用最多，而且微信小进程也使用到了 js 脚本，那么我们为什么最终选择使用 Lua 了呢，因为
JavaScript 虽然功能强大，但是引擎使用起来稍微重了一点，而 Lua 是一个功能强大，高效，轻量级的嵌入式脚本语言，使用标准 Lua 库构建的
Lua 解释器需要 246K，Lua 库需要 421K。[Why choose Lua?](https://www.lua.org/about.html)
而且 Android 中嵌入 Lua 优点很多，借助 Lua 脚本语言的优势，可以轻松实现动态逻辑控制，应用可以随时从服务器读取最新 Lua
脚本文档，在不更新应用的情况下修改进程逻辑，算是一种热更新？算吧。

##
[](https://yongyu.itscoder.com/#%E4%BA%8C%E3%80%81Android-%E4%B8%AD%E5%A6%82%E4%BD%95%E5%BC%95%E5%85%A5-Lua
"二、Android 中如何引入 Lua")二、Android 中如何引入 Lua

Lua 解释器是 C 语言写的，而 Android 开发使用的是 Java 语言，所以如果我们不打算用 Java 重写解释器的话，我们需要一种方式使 C 和
Java 能良好的沟通，互相调用。所幸的是 Java 支持本地化编程，能使用 JNI 调用 C，因而让 Lua 嵌入到 Java 中成为可能。但是要将
Lua 大部分需要的函数通过 JNI 转换成对应的 Java 方法实际上也是比较浩大的工程。不过，已经有 LuaJava
这个开源库帮我们完成这个工作，将大部分 Lua 函数封装成堆栈类 LuaState 对应的 Java 方法，我们就可以直接拿来用。

###
[](https://yongyu.itscoder.com/#1%E3%80%81%E5%81%87%E5%A6%82%E4%BD%A0%E7%86%9F%E6%82%89-NDK-%E7%BC%96%E8%AF%91%EF%BC%9A
"1、假如你熟悉 NDK 编译：")1、假如你熟悉 NDK 编译：

 _注意:不熟悉的，请绕行看第二种办法去，笔者就不熟悉，自己好顿折腾_

那么就自己去官网下载源码自己编译 so 库文档再去使用，下面是下载地址:

#### [](https://yongyu.itscoder.com/#1-1-%E8%B5%84%E6%BA%90%E5%87%86%E5%A4%87
"1.1 资源准备")**1.1 资源准备**

1）去[Lua 官网](http://www.lua.org/ftp/) 选择需要版本下载源码

2）去下载 LuaJava 三方裤子源码，这个裤子最新版本是 2007 年最后更新的
[[luajava-1.1](http://files.luaforge.net/releases/luajava/luajava/LuaJava1.1/luajava-1.1.zip)]
版本，当然如果你牛逼，下载下来自己去根据需求改去吧，当然 Gayhub 上也有人改过的，你也可以去搜搜，而且这个裤子里面只提供了 luajava.c
文档没有提供 luajava.h 头文档，这个 luajava.h 文档是根据 LuaState.java 这个类生成的，你可以采用命令行 javac 将
Luajava.java 编程成 Luajava.class 文档，再用 javah 将 Luajava.class 文档编译成 luajava.h
文档，这是 java 函数与 C++
函数对应的静态注册方法，即通过特定的规则来写，此处方法名可以随意起名字，然后还可以用动态注册的方式关联两个方法（显然，静态注册要简单一些）。

3）配置NDK 编译 so 库，编译方式自行选择（ndk build 和 CMake 方式），笔者目前使用的是 Stutio 3.0.1 ，所以采用的是
CMake 编译方式，下面简单介绍下编译流程：

1、在SDK Tools 中勾选安装 CMake、LLDB、NDK

![ndkconfig](https://github.com/yongyu0102/WeeklyBlogImages/blob/master/androidLua/ndkconfig.PNG?raw=true)

2、File -> New -> New Project，在如下接口中勾选`Include C++ Support`，然后一路 Next，直到 Finish
为止即可（图省略）。

3、创建完成项目发现与常规项目比多了.externalNativeBuild文档夹、cpp文档夹、CMakeLists.txt文档。

**.externalNativeBuild文档夹** ：cmake编译好的文档, 显示支持的各种硬件等信息。系统生成。

**cpp文档夹** ：存放C/C++代码文档。

**CMakeLists.txt文档** ：CMake脚本配置的文档。需要自己配置编写。

#### [](https://yongyu.itscoder.com/#1-2-%E7%BC%96%E8%AF%91%E6%AD%A5%E9%AA%A4
"1.2 编译步骤")**1.2 编译步骤**

这里稍微提一句，笔者 菜鸟一枚， c 代码不懂，但是为了学习一下 NDK 编译，所以去官网下载完了，根据按照 CMake
编译规则进行编译，采坑不断。开始下载完 luajava 裤子，发现没有 luajava.h 文档，查了下，这个文档是根据 LuaState.java
编译出来的，于是笔者自己先把下载俩来 luajava 裤子里的 Java 代码放到 工程目录的 java
目录下面，如图：![LuaState](https://github.com/yongyu0102/WeeklyBlogImages/blob/master/androidLua/LuaState.PNG?raw=true)

然后执行 Make Project LuaState ，然后到 appbuildintermediatesclassesdebug 目录下执行：

`javah org.keplerproject.luajava.LuaState` 命令

![javah](https://yongyu.itscoder.com/D:%5CBlog%5CluaBlog%5Cpic%5Cjavah.PNG)

将 LuaState.class 编译出一个 LuaState.h 文档

![h](https://github.com/yongyu0102/WeeklyBlogImages/blob/master/androidLua/h.PNG?raw=true)

然后将文档名字改为 luajava.h 放在 cpp 文档夹下， 并将 lua5.3.3 版本源码和 luajava 的 luajava.c 文档也放在
cpp 文档夹下 ，

![cpp](https://github.com/yongyu0102/WeeklyBlogImages/blob/master/androidLua/cpp.PNG?raw=true)

**配置 app/build.gradle 文档**

    
    
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

|

    
    
    android {
    
        compileSdkVersion 26
    
        defaultConfig {
    
            externalNativeBuild {
    
                cmake {
    
                    arguments "-DANDROID_ARM_NEON=TRUE", "-DCMAKE_BUILD_TYPE=Debug"
    
                    /* cppFlags "-std=c++11 -frtti -fexceptions"*/
    
                    cppFlags "-frtti -fexceptions"
    
                    abiFilters 'armeabi', 'armeabi-v7a', 'x86', 'arm64-v8a', 'mips', 'mips64', 'x86_64'
    
                }
    
            }
    
        }
    
        buildTypes {
    
            release {
    
                minifyEnabled false
    
                proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    
            }
    
        }
    
        externalNativeBuild {
    
            cmake {
    
                path "CMakeLists.txt"
    
            }
    
        }
    
    }  
  
---|---  
  
**配置 CMakeLists.txt 文档** ：

    
    
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

|

    
    
    #设置要编译 c 文档的 路径（多个 c 文档）
    
    aux_source_directory(src/main/cpp SRC_LIST)
    
    add_library( # Sets the name of the library.
    
                 luajava
    
                 # Sets the library as a shared library.
    
                 SHARED
    
                 # Provides a relative path to your source file(s).
    
                 ${SRC_LIST}
    
                 )
    
    find_library( # Sets the name of the path variable.
    
                  log-lib )
    
    target_link_libraries( # Specifies the target library.
    
                           luajava
    
                           # Links the target library to the log library
    
                           # included in the NDK.
    
                           ${log-lib} )
    
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI})  
  
---|---  
  
然后执行 Make Project 进行编译 so 库，报错。查了一下，是因为 luajava1.1 版本当时作者对应的是 lua5.1 版本，而笔者用的是
Lua5.3 ，所以 api 有差异，于是重新下载了一个 lua5.1 版本编译，报错[NDK Clang error: undefined
reference to ‘localeconv’](https://stackoverflow.com/questions/44736135/ndk-
clang-error-undefined-reference-to-localeconv)。查原因 stackoverflow 上面说是 sdk21
之后版本 才实现了 localeconv() 方法，于是直接将 sdk 最小版本改成 21，编译这个错误解决了，然后又有新的报错[Multiple
definitions of main](https://stackoverflow.com/questions/6622007/multiple-
definitions-of-main)。继续查，stackoverflow 上说 lua.c 和 luac.c 两个 main
函数重复了，于是直接粗暴的luac.c 的 main 函数注释掉，一顿折腾终于编译通过，在
appbuildintermediatescmakedebugobj 目录下生产对应 CPU 架构的 so文档

![![img](https://github.com/yongyu0102/WeeklyBlogImages/blob/master/androidLua/so.PNG?raw=true)

#### [](https://yongyu.itscoder.com/#1-3-%E4%BD%BF%E7%94%A8 "1.3 使用")**1.3
使用**

  1. `luajava` 下的 org 文档夹拷贝到工程自己目标工程 `src/main/java` 目录下

  2. 将 `jniLibs/armeabi`下的 `libluajava.so` 重命名为 `libluajava-1.1.so` 或者修改 `org.keplerproject.luajava.LuaState.java` 的 `LUAJAVA_LIB` 常量 为 libluajava 。
    
        1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8

|

    
        public class 
    
    {
    
       
    
      private final static String LUAJAVA_LIB = "luajava-1.1";
    
       
    
         private final static String LUAJAVA_LIB = "luajava";
    
       ...
    
    }  
  
---|---  
  
经过一些列的折腾，最终成功，可以正常使用了。笔者做这些就是为了自己也学习一把 NDK 编译的过程。虽然笔者自己编译的这个 so
库能正常使用，但是还是建议大家使用 gayhub 上别人升级改造过的库，因为 luajava 这个库比较老 支持的是 Lua5.1 ，而且存在小
bug，有人已经把这个库升级到支持 Lua5.3.1 了比如
[**AndroLua**](https://github.com/lendylongli/AndroLua) 这个裤子，大家可以根据需求 gayhub
上找合适自己的裤子吧。反正最后笔者从 gayhub 上找了别人升级过的 c 文档进行编译，比较稳。

###2、假如你不熟悉 NDK 编译，也懒得折腾

直接上 Gayhub 上搜索 androidlua ，然后 clone 到本地，按照人家的 README 文档操作进行，编译出 so 库直接使用即可，
Lua 和 Luajava 源码人家已经帮你集成好了，具体细节你也不必操心，㖏，这里是裤子
[**AndroLua**](https://github.com/lendylongli/AndroLua) 。

到这里本章节结束，下一节介绍，具体使用。