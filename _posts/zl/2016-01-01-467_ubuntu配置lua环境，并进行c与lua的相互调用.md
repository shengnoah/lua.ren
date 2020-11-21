---
layout: post
title: ubuntu配置lua环境，并进行c与lua的相互调用 
tags: [lua文章]
categories: [topic]
---
先查看一下apt可获取的lua版本  
![](http://arrayindex.me//2018/08/29/ubuntu配置lua环境，并进行c与lua的相互调用/1.png)  
  
我们选择lua5.1版本进行安装  

    
    
    1  
    

|

    
    
    sudo apt install lua5.1  
      
  
---|---  
  
安装完之后测试一下是否安装成功，如果可以正常使用，则lua环境已经安装完毕。  
![](http://arrayindex.me//2018/08/29/ubuntu配置lua环境，并进行c与lua的相互调用/2.png)

# 2.安装lua相关的c库

lua环境安装完毕，但是此时在c中还不能对lua进行调用，或者生成供lua调用的c库，因为还没有安装lua的c库，通过下面这条命令安装相应的库文件和头文件  

    
    
    1  
    

|

    
    
    sudo apt-get install lua5.1-0-dev  
      
  
---|---  
  
安装完毕后，我们写代码进行测试

## 2.1生成c的动态库供lua调用

新建一个c文件  

    
    
    1  
    

|

    
    
    vim addlib.c  
      
  
---|---  
  
写一个addc函数供lua调用  

    
    
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
    27  
    28  
    

|

    
    
      
    #include <lua5.1/lualib.h>  
    #include <lua5.1/lauxlib.h>  
      
    static int (lua_State *L)  
    {  
        int a,b,c;  
        a = lua_tonumber(L,1);  
        b = lua_tonumber(L,2);  
        c = a + b;  
        lua_pushnumber(L,c);  
        return 1;  
    }  
      
    static const struct luaL_Reg lib[] =  
    {  
          
        {"addc",addc},  
        {NULL,NULL}  
    };  
    //luaopen_xxx  这个xxx一定要和导出的库名一样，不然lua无法识别这个函数，无法进行函数的注册   
    int luaopen_addlib(lua_State *L)  
    {  
        //这里的"testadd"是在lua中调用库函数的全局变量名，不需要和库名addlib保持一致,但一般会用一样的名字  
        luaL_register(L,"testadd",lib);  
        //luaL_register(L,"addlib",lib);  
        return 1;  
    }  
      
  
---|---  
  
保存后对代码进行编译，生成lua用的so或dll库  

    
    
    1  
    

|

    
    
    gcc addlib.c -fPIC -shared -o addlib.so  
      
  
---|---  
  
![](http://arrayindex.me//2018/08/29/ubuntu配置lua环境，并进行c与lua的相互调用/4.png)  
接下来进行lua对c调用的测试  
![](http://arrayindex.me//2018/08/29/ubuntu配置lua环境，并进行c与lua的相互调用/5.png)  
调用成功

## 2.2在c中调用lua

创建printHello.lua文件  

    
    
    1  
    

|

    
    
    vim printHello.lua  
      
  
---|---  
  
写一个PrintHelloLua函数  

    
    
    1  
    2  
    3  
    

|

    
    
    function PrintHelloLua()  
        print("hello !!!")  
    end  
      
  
---|---  
  
创建luaFunctionTest.c文件  

    
    
    1  
    

|

    
    
    vim luaFunctionTest.c  
      
  
---|---  
      
    
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
    27  
    28  
    29  
    

|

    
    
      
    #include <lua5.1/lualib.h>  
    #include <lua5.1/lauxlib.h>  
      
    int main()  
    {  
        //创建lua运行环境  
        lua_State *luaEnv = lua_open();  
        luaopen_base(luaEnv);  
        luaL_openlibs(luaEnv);  
        if(!luaEnv)  
        {  
            return -1;  
        }  
      
        //载入lua文件  
        int loadInfo = luaL_loadfile(luaEnv,"printHello.lua");  
        if(loadInfo)  
        {  
            return -1;  
        }  
        //执行lua文件  
        lua_pcall(luaEnv,0,0,0);  
      
        //调用PrintHelloLua函数  
        lua_getglobal(luaEnv,"PrintHelloLua");  
        lua_pcall(luaEnv,0,0,0);  
        return 0;  
    }  
      
  
---|---  
  
生成可执行文件,需要通过 -llua5.1指明使用的库文件  

    
    
    1  
    

|

    
    
    gcc -o luaFunctionTest luaFunctionTest.c -llua5.1  
      
  
---|---  
  
运行可执行文件,成功输出 hello !!!  

    
    
    1  
    

|

    
    
    ./luaFunctionTest  
      
  
---|---  
  
![](http://arrayindex.me//2018/08/29/ubuntu配置lua环境，并进行c与lua的相互调用/6.png)