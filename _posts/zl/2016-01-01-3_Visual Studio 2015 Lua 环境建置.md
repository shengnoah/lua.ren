---
layout: post
title: Visual Studio 2015 Lua 环境建置 
tags: [lua文章]
categories: [topic]
---

                        <p>Visual Studio 2015 Lua 环境建置
                    <br />
                    2016/05/10 修正内文<br />

第13步骤 由 "选择数据夹" 改为 "类库名称"<br />

新增红色重点并附上范例项目文件(VS 2016 Project)<br />

<p>环境：Visual Studio 2015 UPDATE 1 &amp; Lua 5.3.2 (.Net 4.6.1)</p>

<p>1.下载Lua：Source</p>

<p><img height="657" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555440_83912.png" width="1240"></p>

<p>2.解压缩该压缩档：D:lua-5.3.2</p>

<p><img height="696" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555449_76311.png" width="1045"></p>

<p>3.开启Visual Studio 2015→新增项目→Visual C++→Win32→Win32 主控台应用程序→Lua5.3→确定→下一步→静态程序库→取消勾选"先行编译标头档"→完成</p>

<p><img height="794" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555486_23457.png" width="1240"><img height="882" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555486_71823.png" width="1240"></p>

<p>4.方案总管→标头档→加入→现有项目→D:lua-5.3.2src→选取全部 *.h 文件(所有C/C++ Header文件，可先依文件类型排序后方便选择)→加入</p>

<p><img height="917" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555520_35931.png" width="1142"></p>

上图右边有误：应该是要选取"新增项目"下方的"现有项目"<br />

<p><img height="545" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555528_04548.png" width="1240"></p>

<p>5.方案总管→原始程序档→加入→现有项目→D:lua-5.3.2src→选取全部 *.c 文件(所有C/C++ Header文件，可先依文件类型排序后方便选择)→取消选择 lua.c与luac.c两个文件→加入</p>

<p><img height="545" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555544_10871.png" width="1240"></p>

<p>6.项目→属性→C/C++→一般→其他 Include 目录→编辑→加入目录"D:lua-5.3.2src"→选择数据夹→确定</p>

<p><img height="1359" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555578_27266.png" width="1194"><img height="821" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555578_3986.png" width="1240"><img height="695" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555579_24148.png" width="865"><img height="821" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555579_4915.png" width="1240"></p>

<p>7.项目→属性→C/C++→进阶→编译成→编辑→编译成 C 程序(/TC)→确定</p>

<p><img height="821" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555602_0563.png" width="1240"></p>

<p>8.开始建置(Release编译)→产生lib文件→位于"方案"目录下的Release数据夹(非"项目"目录下的Release数据夹)</p>

<p><img height="1313" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555620_53776.png" width="1240"><img height="619" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555620_20965.png" width="1240"></p>

<p>(D:UserDocumentsvisual studio 2015ProjectsLua5.3ReleaseLua5.3.lib)</p>

<p>9.将编译完成所产生的lib文件(Lua5.3.lib)复制到Lua Source目录D:lua-5.3.2</p>

<p><img height="1081" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555631_74659.png" width="680"><img height="557" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458555638_80252.png" width="1240"></p>

<p>10.开启Visual Studio 2015(或于上面原方案按右键点选加入)→新增项目→Visual C++→Win32→Win32 主控台应用程序→LuaTest→确定→下一步→完成</p>

<p><img height="882" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458613075_53495.png" width="1240"></p>

<p>11.同上述第6步骤：项目→属性→C/C++→一般→其他 Include 目录→编辑→加入目录"D:lua-5.3.2src"→选择数据夹→确定</p>

<p><img height="821" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458613814_61591.png" width="1240"></p>

<p>12.项目→属性→连接器→一般→其他程序库目录→编辑→加入目录"D:lua-5.3.2"→选择数据夹→确定</p>

<p><img height="821" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458613105_77563.png" width="1240"><img height="642" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458613110_94112.png" width="1240"><img height="821" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458613131_11536.png" width="1240"></p>

<p>13.项目→属性→连接器→输入→其他相依性→编辑→加入类库名称"Lua5.3.lib"→确定</p>

<p><img height="821" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458615457_4052.png" width="1240"></p>

<p>14.原始程序档→加入→新增项目→C++ 档(.cpp)→"main.lua"(注意是lua不是原本的.cpp)→新增</p>

<p><img height="794" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458613140_57805.png" width="1240"></p>

<p>15.修改main.lua程序内容 print("Hello World.");</p>

<pre>
<code>print("Hello World.");</code></pre>

<p><img height="405" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458615471_03996.png" width="1240"></p>

<p>16.修改LuaTest.cpp程序内容</p>

<pre>
<code>#include "stdafx.h"
#include <iostream>
using namespace std;
#include <lua.hpp>

int main()
{
	lua_State *l = luaL_newstate();
	luaL_openlibs(l);
	luaL_dofile(l, "main.lua");
	lua_close(l);
	system("pause");
    return 0;
}</code></pre>

<p><img height="400" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458615478_86804.png" width="1240"></p>

<p>17.编译LuaTest项目并执行后显示完成结果</p>

<p><img height="947" src="https://az787680.vo.msecnd.net/user/jakeuj/293febdc-4e32-48b4-915f-33eb58861fe4/1458616577_29333.png" width="1240"></p>

<p>参照：Lua学习笔记</p>

<p>范例文件：下载位置 (懒人包：全部解压缩到D即可，内含Lua Source)</p>
                    <p><img alt="" src="https://img.dazhuanlan.com/2019/11/25/5ddb75c6d785c.png"></p>