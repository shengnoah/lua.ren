---
layout: post
title: LuaInterface简介 
tags: [lua文章]
categories: [topic]
---
LuaInterface

<https://www.cnblogs.com/sifenkesi/p/3901831.html>

# 二 使用

## 1.C#中调用Lua

下载LuaInterface。[下载地址](http://luaforge.net/projects/luainterface/)

里面有两个文件：`lua51.dll`，`LuaInterface.dll`

新建c#控制台，添加引用（引用右键-添加引用）：

![Image 0011557124613](https://qihr.github.io//2019/04/23/C和Lua交互/Image
0011557124613.png)

c#：

    
    
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
    30  
    31  
    32  
    33  
    34  
    35  
    36  
    37  
    38  
    39  
    40  
    

|

    
    
    using System;  
    using System.Collections.Generic;  
    using System.Linq;  
    using System.Text;  
    using System.Threading.Tasks;  
    using LuaInterface;  
      
    namespace   
    {  
        class Program  
        {  
            static void Main(string[] args)  
            {  
                  
                Lua lua = new Lua();  
                  
                // Lua的索引操作[]可以创建、访问、修改global域，括号里面是变量名  
                // 创建global域num和str  
                lua["num"] = 2;  
                lua["str"] = "a string";  
      
                // 创建空table  
                lua.NewTable("tab");  
      
                // 执行lua脚本，着两个方法都会返回object[]记录脚本的执行结果  
                lua.DoString("num = 100; print("i am a lua string")");  
                object[] retVals = lua.DoString("return num,str");  
      
                // 访问global域num和str  
                double num = (double)lua["num"];  
                string str = (string)lua["str"];  
                lua.DoFile("E:\Personal\Test_Personal\LuainterfaceTest\LuajinterfaceTest\LuajinterfaceTest\testLuaInterface.lua");  
                Console.WriteLine("num = {0}", num);  
                Console.WriteLine("str = {0}", str);  
                Console.WriteLine("width = {0}", lua["width"]);  
                Console.WriteLine("height = {0}", lua["height"]);  
                Console.ReadLine();  
            }  
        }  
    }  
      
  
---|---  
  
Lua：

    
    
    1  
    2  
    3  
    

|

    
    
    local width = 99999  
    local height = 888888  
    print("fuck")  
      
  
---|---  
  
dofile使用相对路径？？

### 1.1 LuaInterface与CLR类型对应

LuaInterface | CSharp  
---|---  
nil | null  
string | System.String  
number | System.Double  
boolean | System.Boolean  
table | LuaInterface.LuaTable  
function | LuaInterface.LuaFunction  
  
其他类型传给lua会被视为是userdata。lua将userdata传给c#时还会是原来的数据结构。

* _LuaTable和LuaUserData都有索引操作[]，用来访问或修改域值，索引可以为string或number。_  
LuaFunction和LuaUserData都有call方法用来执行函数，可以传入任意多个参数并返回多个值。*

### 1.2 使用Luainterface的一些问题

（1）异常：混合模式程序集是针对“v2.0.50727”版的运行时生成的，在没有配置其他信息的情况下，无法在 4.0 运行时中加载该程序集。

解决办法：

在App.config文件中添加如下配置节：

    
    
    1  
    2  
    3  
    

|

    
    
    <startup useLegacyV2RuntimeActivationPolicy="true">  
      <supportedRuntime version="v4.0"/>  
    </startup>  
      
  
---|---  
  
## 2.Lua中调用C

 _第一种是纯lua中进行测试：_

### 2.1 获取类，访问构造函数

在c#工程中测试：

luanet.load_assembly函数：加载CLR程序集（程序集的名字在工程右键属性可以看到）；

luanet.import_type函数：加载程序集中的类；

luanet.get_constructor_bysig函数：手动匹配某个构造函数

Lua代码：

    
    
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
    

|

    
    
    -- 加载自定义类型，先加载程序集，在加载类型  
    luanet.load_assembly("LuajinterfaceTest")  
    TestClass = luanet.import_type("LuajinterfaceTest.TestClass2")  
      
      
    obj1 = TestClass(2, 3)    -- 匹配public TestClass2(int n1, int n2)  
    obj1:PrintSomthing()  
    obj2 = TestClass("x")    -- 匹配public TestClass2(string str)  
    obj3 = TestClass(3)        -- 匹public TestClass2(string str)  
      
      
    TestClass_cons2 = luanet.get_constructor_bysig(TestClass, 'System.Int32')  
    obj3 = TestClass_cons2(3)    -- 匹配public TestClass2(int n)  
      
  
---|---  
  
_注意先执行构造函数再进行方法调用_

### 2.2 访问对象的字段和方法

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    -- 加载自定义类型，先加载程序集，在加载类型  
    luanet.load_assembly("LuajinterfaceTest")  
    TestClass = luanet.import_type("LuajinterfaceTest.TestClass2")  
      
    obj1 = TestClass(2, 3)    -- 匹配public TestClass2(int n1, int n2)  
    obj1:PrintSomthing()  
      
  
---|---  
  
访问对象的字段和table一样：button.Text… button[“Text”]

访问对象就是obj1:PrintSomthing()

### 2.3 重载方法的匹配

luanet.get_method_bysig

    
    
    1  
    2  
    

|

    
    
    　　setMethod=luanet.get_method_bysig(obj,'setValue','System.String')"  
    　　setMethod('str')  
      
  
---|---  
  
Luainterface匹配重载方法（包括构造函数）的规律是自动匹配第一个能够匹配的方法（构造函数）

LuaInterface匹配第一个能够匹配的构造函数，在这个过程中，numerical
string（数字字符串）会自动匹配number，而number可以自动匹配string，所以TestClass(3)匹配到了参数为string的构造函数。

out和ref参数的方法，参数和方法返回值同时返回。out参数不需要传入。

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    -- calling int obj::OutMethod1(int,out int,out int)  
      retVal,out1,out2 = obj:OutMethod1(inVal)  
      -- calling void obj::OutMethod2(int,out int)  
      retVal,out1 = obj:OutMethod2(inVal) -- retVal ser´a nil  
      -- calling int obj::RefMethod(int,ref int)  
      
  
---|---  
  
### 2.4 接口

两个接口：

IFoo.method()和IBar.method()，这种情况下obj[“IFoo.method”]

### 2.7事件，委托

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    function handle_mouseup(sender,args)  
    　　print(sender:ToString() .. ’ MouseUp!’)  
    　　button.MouseUp:Remove(handler)  
    end  
    handler = button.MouseUp:Add(handle_mouseup)  
      
  
---|---  
  
add（）会将lua方法转换为CLR委托，并会返回这个委托。

### 2.6 扩展interface的方法

太难了 等会再看吧

差一个c#调lua方法

lua报错输出

_Lua中button.Text… button[“Text”]的区别_

vscode控制台中文乱码<https://blog.csdn.net/xjk2017/article/details/81388493>

CLR

构造方法

![Image 0011555992894](https://qihr.github.io//2019/04/23/C和Lua交互/Image
0011555992894.png)