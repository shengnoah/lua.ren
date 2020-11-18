---
layout: post
title: 欢迎使用LuaPerfect! 
tags: [lua文章]
categories: [lua文章]
---
### 简介

LuaPerfect是腾讯公司开发的专业级的Lua集成开发环境，致力于为广大Lua开发人员提供免费的专业的Lua编辑调试工具。

LuaPerfect基于纯C++实现了独立的Lua编辑和调试工具：  
1、接入方便：接口风格类似VS，符合VS用户习惯，非插件，接入自动化程度高，无需配置。  
2、调试功能强大：稳定的基础调试功能，强大的表达式监视，悬浮监视，日志跳转，条件断点，Lua异常捕获，Lua反汇编等功能。还可以直接查看C#对象的各种成员，在Unity下还能列出组件列表和子物体列表。  
3、调试性能高：调试密集Lua运算的游戏也不掉帧，因此特别适合调试游戏。  
4、编辑功能强大：支持语法/语义代码高亮，自动API生成，语法检查，单词/语句自动完成，按语义跳转符号，代码格式化，类型推导，类型注解，全工程符号搜索，按语义重构等功能。  
5、自带性能测试功能，测试密集Lua运算游戏的性能也不掉帧，因此结果更精确。  
6、资源占用少：相对脚本化插件化的方案(IDEA,VSCode等)，同等功能下内存等资源仅同类软件的一半左右。  
7、稳定流畅：运行稳定流畅，经过内部外部大型项目重度使用验证，得到非常高评价。

以下为软件运行截图，图中为调试SLua密集Lua运算的性能测试例子circle.txt，截图显示调试时仍保持59帧未掉帧：

![使用截图](http://154.8.233.13/LuaPerfectScreenshot.png)

### 下载LuaPerfect

#### 官方QQ群下载

LuaPerfect官方群([932801740](https://jq.qq.com/?_wv=1027&k=54bnLYF))下载(速度快，推荐)

#### GitHub下载

LuaPerfect GitHub仓库(<https://github.com/jiangzheng1986/LuaPerfect>)下载

#### 腾讯云下载

腾讯云下载(<http://154.8.233.13/Versions/LuaPerfect_V1.023.zip>)下载

### 贡献墙

感谢你们的支持、反馈和贡献：john，siney，ares，ce，mony，ansen，fidel，cooper，saul，vader，walle，william，tock，glad，yule，天地一MADAO，Jay，Wisdom，未曾，timmy，flash，克克狼，★Smartly☆，frank，qigao，Ray，ud，有你好看，ShaopingCui，Von
Neamann，夜，♥ Virgo丶，…，马劲松，轩，SoMe，随意鸥，Anyq，市井
'，扣扣小妖，kenny，1023094781，心梦无痕，长弓海，ike，天涯，3°，kramer，大魔王有木桑，ZeaLotSean(官网框架)，HHH，HIQ，hop，Я7(R7
Theme代码配色配置)，墟烟，dll，johnd，古月良云，起跑线To，肉松可爱多，ZensYue，小二郎，joary，呵呵哒，ccgo，风哥

### 更新说明

#### 2019.09.01，版本1.024：

1、支持正则表达式(感谢天涯，风哥反馈)  
2、修复某些情况下不同目录下的同名文档其中一个无法断点的问题(感谢Ray反馈)  
3、标题栏带绝对路径,方便区分同一个项目的不同分支(感谢ZensYue反馈)  
4、增加Options-Default Lua Extension菜单  
5、调整工程列表接口  
6、调整工程视图右键菜单顺序(感谢Я7反馈)  
7、修复了Translate Tabs To Spaces相关的4个问题(感谢Я7反馈)  
8、修复了指定查找CurrentFile无效的问题  
9、修复了CurrentFile选项会变成CurrentProject的问题

#### 2019.08.25，版本1.023：

1、修复输入法不跟随光标的问题(感谢古月良云，ccgo，Я7反馈)  
2、支持连续的快捷键，比如Ctrl+K,C(感谢Я7反馈)  
3、缩进都是用的t，能不能搞个配置用4个空格代替(感谢Я7反馈)  
4、修复了LuaDebuggee.dll有些情况下崩溃的问题(感谢hop，天涯，Я7反馈)  
5、修复Cocos调试时的bug，支持Cocos调试!（感谢HHH，Ray，3°反馈）  
6、新增被调试进程命令行参数可配置  
7、修复pdb加载缓慢的问题（感谢Ray反馈）

#### 2019.08.19，版本1.022：

1、Theme中增加自定义快捷键支持(Key的取值: A-Z 0-9 F1-F12 [ ] / . - = ; ’ ,
以及Left,Right,Up,Down,PageUp,PageDown,Home,End,Backspace,Delete,Tab,Enter,Break,Escape)(感谢ares，glad，肉松可爱多，ZensYue，hop反馈)  
2、增加Options-Edit Theme Mode,用于快速自动预览Theme  
3、Lua文档模板(Data/Config/Template.lua),支持Date,Time,FileName,ProjectName等标签(感谢小二郎，古月良云，joary反馈)  
4、提供一个菜单打开配置文档夹:Options-Open Config Directory  
5、支持配置是否自动拷贝ObjectFormater.cs文档的菜单(Options-Skip Synchronizing Code)(感谢hop反馈)  
6、调整Project视图右键菜单顺序，Find In Folder放第一位(感谢呵呵哒反馈)  
7、Project视图LuaFolders增加Find In Project菜单项(感谢呵呵哒反馈)  
8、代码菜单右键菜单增加Find In Folder菜单项(感谢呵呵哒反馈)  
9、Find In Foler增加Ctrl+F3的快捷键(感谢呵呵哒反馈)

#### 2019.08.11，版本1.021：

1、修复单行过长时会导致崩溃的问题。（感谢HIQ反馈）  
2、xlua.dll为特定名称时的调试支持。（感谢hop反馈）  
3、函数里的局部变量被赋值成员，不再被识别为类。（感谢hop反馈）  
4、修复重新加载Theme时没有重建里面的某些颜色表的问题。（感谢Я7反馈）  
5、修复ToLua执行print()会报错的问题。（感谢墟烟反馈）  
6、智能提示新增按tab来完成。（感谢dll，hop，johnd反馈）  
7、修复修改时会自动展开所有折叠的问题。(感谢扣扣小妖反馈)  
8、修复子函数里的括号开始和结束的标志在全部折叠时不会消失的问题。  
9、自定义背景图片(Data/Config/Background.png)。(感谢古月良云反馈)  
10、Я7提供了R7 Theme代码配色配置。(感谢Я7提供配置)(建议用新版本加载配置)  
11、日志的功能，能否允许跳转往上一级的，希望可以自定义向上跳过的函数名列表。(感谢frank，起跑线To，hop反馈)

#### 2019.07.21，版本1.020：

1、新增有pdb情况下，lua是静态库的时候的调试。（感谢HHH，3°反馈）

#### 2019.07.16，版本1.019：

1、Ctrl+鼠标滚动时字体大小的步长从2修改为1。（感谢大魔王有木桑反馈）

#### 2019.07.13，版本1.018：

1、修复Cocos在General模式下提示Project not matched的bug。（感谢3°反馈）  
2、整合General模式和Cocos/Quick模式。（感谢3°反馈）  
3、修复Pandora在General模式下提示Project not matched的bug。（感谢kramer反馈）  
4、在尝试用General模式打开Unity.exe或者UE4XXX.exe时进行错误提示。  
5、修改工具内官网链接地址。  
6、修改版本号升级规则(如不再以1.18.0之类命名，而直接使用1.018命名)。

#### 2019.04.21，版本1.17.0：

1、增加ProjectFilename,CodeFilename及LineNumber命令行参数，用于在命令行直接打开工程/文档。(感谢天涯反馈)  
2、—@field注解允许不带类型。(感谢天涯反馈)  
3、EditorConfig.lpxml增加SkipSynchronizingCode属性，用于控制是否复制各种C#文档到工程。(感谢frank,ares反馈)  
4、ApiGenerator.cs中部分函数改成public,方便定义具体项目自己的ApiGenerator。(感谢ares反馈)  
5、修复streamWriter的Close()放入finally块。(感谢frank反馈)

#### 2019.04.08，版本1.16.3：

1、Windows下打包，LuaDebuggeeLoader.cs会导致提示找不到LuaDebuggee.dll，暂时删除LuaDebuggeeLoader.cs。(感谢扣扣小妖反馈)  
2、修改Paste StartDebug()的内容，加入查看faq的提示。(感谢ike反馈)  
3、修复General模式下,无法正常复制LuaDebuggee.dll的问题。（感谢马劲松反馈）  
4、新增测试General模式的测试用例。（感谢马劲松反馈）

#### 2019.04.03，版本1.16.2：

1、修复了Unity 4.x下32位Unity.exe复制64位LuaDebuggee.dll的问题。（感谢frank反馈）

#### 2019.04.02，版本1.16.1：

1、修复了监视长度超过1000的字符串会出现缓冲区溢出导致Unity挂掉的问题。（感谢kenny,马劲松反馈）

#### 2019.04.01，版本1.16.0：

1、兼容EmmyLua的注解的写法。（感谢Ray,天涯反馈）  
2、LuaDebuggee.dl崩溃时保存Dump到Unity.exe同级Data/Dump目录。（感谢kenny,马劲松反馈）  
3、优化try-catch的性能，并优化Dump后的行为，尽量不导致Unity关闭。（感谢kenny,马劲松反馈）  
4、修复先输入一个很长的字符串，然后选中一个短的字符串，按Ctrl+F不显示的问题。（感谢夜反馈）  
5、修复保存操作会导致显示函数签名提示的问题。  
6、修复保存操作会无法自动关闭函数签名提示的问题。  
7、整合Ctrl+P,Ctrl+Shift+P,Ctrl+Alt+P，增加页签。（SoMe反馈）

#### 2019.03.25，版本1.15.0：

1、处理Pandora里dll要用pua_luaopen开头的问题。（感谢kennyge反馈）  
2、修改自动完成的for k, v in ipairs(t)为for i, v in ipairs(t)。(感谢心梦无痕反馈)  
3、不是默认端口号时增加一个输出提示。（感谢马劲松反馈）  
4、处理Pandora里C#对象查看的问题。（感谢kennyge反馈）  
5、修复非exe目录启动时会报shader错误的问题。(感谢…反馈)  
6、修复.Lua.txt在有词法错误的情况下加载时无法显示语法高亮的问题。(感谢长弓海反馈)  
7、询问接口仍然提示"Remove Source Folder"的问题。(感谢…反馈)  
8、Remove Folder改为Delete Folder。(感谢…反馈)  
9、只有开启调试核心的调试模式才创建Data/Log/Debuggee.log。  
10、修改右键菜单弹出位置的规则。  
11、修改配色和图标。  
12、修复General模式启动调试时不保存文档的问题。  
13、修改Ctrl+P接口评分算法，平衡长度和连续度。(感谢马劲松反馈)

#### 2019.03.19，版本1.14.1：

1、Resources和AB包里都有lua文档的情况导致无法断点的问题。(感谢…,未曾反馈)  
2、修复Assets文档夹被忽略导致无法断点的问题。(感谢1023094781和…反馈)  
3、修复General无法调试的问题。  
4、接管调试把sethook函数替换掉后，主动报错改成仅打印提示。（感谢马劲松反馈）

#### 2019.03.18，版本1.14.0：

1、Add Source Folder改成Open Source Folder，Remove Source Folder改成Close Source
Folder。(感谢市井 '，扣扣小妖，随意鸥反馈)  
2、修复常规输入框里文本颜色变成紫色的问题。  
3、安装到某些文档夹中无法写入的文档时进行提示。(感谢kenny反馈)  
4、默认也打开.lua.bytes后缀的lua文档。(感谢kenny反馈)  
5、Paste StartDebug()菜单改成粘贴会判断加载成功的代码。(感谢ud反馈)  
6、整理一下dll被占用时的提示的交互。(感谢随意鸥反馈)  
7、修复切换进程时非客户区有白色边框的问题。  
8、提供Options-Default File Format和Options-Default Line Ending。(感谢克克狼反馈)

#### 2019.03.14，版本1.13.7：

1、如果是浏览XLua工程目录的话，就全Assets搜索一下Plugins下面的xlua.dll。(感谢Anyq反馈)  
2、接管调试后把sethook函数替换掉，改成仅打印提示的函数。（感谢马劲松反馈）

#### 2019.03.14，版本1.13.6：

1、修复调试时会报错attempt to index a nil value的问题。(感谢马劲松反馈)

#### 2019.03.12，版本1.13.5：

1、支持拖动到屏幕顶部最大化，支持最大化时拖动标题还原。(感谢马劲松反馈)  
2、提供菜单直接配置哪些文档扩展名的文档需要显示在ProjectUI中/解析/检查。(感谢随意鸥反馈)

#### 2019.03.11，版本1.13.4：

1、重新编译，查Dump的问题。(感谢SoMe反馈)

#### 2019.03.11，版本1.13.3：

1、修复因Plugins目录位置不在根目录导致Projects are not matched的报错。(感谢马劲松反馈)

#### 2019.03.11，版本1.13.2：

1、访问__DebuggeeTempVar被用户代码拦截的问题，改为访问LUA_REGISTRYINDEX表(感谢轩反馈)  
2、View-Switch File菜单加入Go To File菜单项。

#### 2019.03.11，版本1.13.1：

1、支持调试由dostring()执行但是文档本身就是存在且传递的是代码的情况(需要开启Options-Debuggee-Search Source In
DoString)。(感谢夜，♥ Virgo丶反馈)

#### 2019.03.11，版本1.13.0：

1、直接在Options-Debuggee-Debug Mode菜单中开启调试器调试模式。(感谢未曾反馈)  
2、Open Containing Folder改成Show In Explorer。(感谢未曾反馈)  
3、代码颜色区分全局变量，局部变量和参数变量。(感谢…反馈)  
4、新增查找Plugins目录列表(用于判断是否XLua等解决方案的工程)。(感谢马劲松反馈)

#### 2019.03.8，版本1.12.4：

1、改为通过在Plugins目录去搜索xlua.dll来判断是否XLua工程，其他解决方案同理。(感谢Von Neamann反馈)  
2、修复命名是xx.xx.lua的文档在编辑器中就显示不出来的问题。(感谢夜，♥ Virgo丶反馈)

#### 2019.03.8，版本1.12.3：

1、类似VS里的Ctrl+Tab和Ctrl+Shift-Tab的Active Files接口。(感谢ShaopingCui反馈)

#### 2019.03.7，版本1.12.2：

1、文档夹层次过深时Track Active效果不好，暂时屏蔽默认开启Track Active。（感谢未曾反馈）

#### 2019.03.6，版本1.12.1：

1、增加常见问题FAQ文档。（感谢ud反馈）  
2、代码格式化快捷键从F8修改为F8或Alt+Shift+F。(感谢有你好看反馈)  
3、由于调试按钮上绑定调试帮助的功能会影响调试体验，取消此引导方法。(感谢tock反馈)

#### 2019.03.4，版本1.12.0：

1、支持多键鼠标的后退/前进键。(感谢ares,flash反馈)  
2、函数调用也用特定颜色标记。(感谢flash,anson反馈)  
3、File菜单增加Add Source Folder菜单项。  
4、不是焦点时不显示括号对,不是Lua文档不显示括号对,两个括号在一起的情况，只显示一个大的。  
5、提供LuaDebuggeeLoader.cs加载LuaDebuggee.dll的方法。(感谢未曾反馈)  
6、复制cs文档时根据优先列表进行配置Data/Config/CandidateFolders.lpxml复制到指定目录(感谢Jay,ares反馈)  
7、支持ULua调试。(感谢frank反馈)  
8、支持ULua-查看C#对象。(感谢frank反馈)  
9、处理Cocos 3.13无法调试的问题。(感谢Ray反馈)  
10、使用代理打印函数处理调试tolua需要修改源码的问题。  
11、优化并统一调试路径匹配算法。  
12、修复调试菜单全部是灰色会导致用户误以为无法调试的问题！(感谢qigao反馈)  
13、支持任意使用Lua的dll的exe的Lua调试(General调试)。(感谢Ray反馈)  
14、默认开启Track Active Documents。(感谢未曾反馈)  
15、提高与选中符号相同的符号的颜色的对比度。(感谢flash反馈)  
16、文档一片白色,最好能上色的都上色。(感谢flash,未曾反馈)  
17、光标去哪就选中所在符号。  
18、查找时没有实时的黄色框显示。(感谢未曾反馈)  
19、成员字段的显示换一种颜色。(感谢未曾反馈)  
20、自定义代码颜色主题。(感谢yule反馈)

#### 2019.02.24，版本1.11.0：

1、粘贴代码时自动缩进的功能。（感谢ares,flash,克克狼反馈）  
2、代码自动格式化的功能。（感谢ares,flash,克克狼反馈）  
3、修复从智能提示中选中时，强制替换了之后的一项的问题。（感谢ares,flash反馈）  
4、新增显示匹配的括号对的功能。（感谢ares,flash反馈）  
5、修复换行后Tab数量不对的问题。（感谢ce反馈）  
6、从智能提示中选中时，强制替换了之后的一项。（感谢ares,flash反馈）  
7、选中的情况下输入"([{，在选中的字符串两边加上"",(),[],{}。  
8、排查没有显卡驱动时黑屏的问题，增加没有驱动时的提示。（感谢未曾反馈）  
9、修复某些机器上使用OpenGLES驱动导致长时间运行会变慢/崩溃的问题。（感谢ares反馈）  
10、修复某些机器上使用OpenGLES驱动导致启动时崩溃的问题。（感谢★Smartly☆反馈）

#### 2019.02.15，版本1.10.0：

1、增加ToLua支持。（感谢timmy反馈）  
2、注释操作同时支持Ctrl±和Ctrl+/快捷键。（感谢ares反馈）

#### 2019.02.15，版本1.9.5：

1、修复生成的api里有与最终lua同名的情况下断点失败的问题。

#### 2019.02.14，版本1.9.4：

1、修复新版本特定情况下项目无法调试的问题。（感谢tock反馈）  
2、保守处理某些情况下窗口配置文档Width=1,Height=1的情况。（感谢ares反馈）  
3、修复ApiGenerator.cs中处理无命名空间的类时会生成有Lua错误的代码的问题。

#### 2019.02.13，版本1.9.3：

1、修复一个CodeInfo为空指针导致崩溃的问题。（感谢Jay反馈）  
2、ApiGenerate类整合Clear操作到生成操作内。  
3、ApiGenerate类处理类名时处理操作更通用。

#### 2019.02.12，版本1.9.2：

1、修改为.lpproj时兼容旧的工程文档/用户配置文档。

#### 2019.02.12，版本1.9.1：

1、通过修改为.lpproj,修复lpproj文档夹会被打包进去最终包里的问题。（感谢Jay,ares,walle反馈）  
2、通过修改为.lpproj,修复lpproj文档夹中小文档会在Unity中进行刷新的问题。（感谢Jay,ares,walle反馈）  
3、ApiGenerator.cs改为复制到ThirdParty/LuaPerfect/Editor中。（感谢Jay,ares,walle反馈）  
4、lpproj/Apis/里的.lua代码无论如何不应该被过滤掉。（感谢Jay反馈）  
5、修复Api生成时会生成类似CS__StaticArrayInitTypeSize=24这样的类名的问题。（感谢Jay反馈）

#### 2019.02.11，版本1.9.0：

新增功能:  
1、新的文档查找/替换接口。  
2、if,ifelse,ifelseif,repeat,while,for,fori,forp,do,returnm,function等自动补全语句。(感谢ce反馈)  
3、".if",".ifelse",".ifelseif",".repeat",".while",".for",".fori",".forp",“return”,".function"等后缀自动补全。(感谢ce反馈)  
4、() {} [] ‘’ ""五种符号都可以成对，然后光标定位在中间。(感谢ce反馈)  
5、修改对话框，使得可以在窗口内拖动。(感谢glad反馈)  
6、通过状态栏显示Parsing…来表示当前正在执行解析操作。  
7、支持直接拖文档夹到ProjectUI。  
8、拖工程里有的文档到ProjectUI，则直接打开该文档。  
新增菜单:  
1、新增切换上一个下一个函数的菜单功能(快捷键Alt+Up,Alt+Down)。  
2、选中单词的菜单功能(快捷键Ctrl+W)。  
3、选中整行的菜单功能(快捷键Ctrl+L)。(感谢ce反馈)  
4、复制当前选中或当前行的菜单功能(快捷键Ctrl+D)。  
5、删除当前行的菜单功能(快捷键Ctrl+K)。  
6、增加Options-Fonts-Largr/Smaller/Reset菜单。  
快捷键修改:  
1、前向和后向导航快捷键修改为Alt+Left及Alt+Right。  
2、注释/取消注释快捷键修改成Ctrl±。  
3、Go To Symbol In File的快捷键改为Ctrl+Shift+P。  
4、Go To Symbol In Project的快捷键改为Ctrl+Alt+P。  
Bug修复:  
1、修复SLua里下了断点的情况下StepOver也在原始行断点的问题。  
2、修复调试状态时无法刷新文档的问题。(感谢ares反馈)  
3、修复启动时就直接注释会出问题的问题。

#### 2019.02.01，版本1.8.1：

1、修复02_U3DScripting无法调试的问题。(感谢未曾反馈)  
2、增加00_LuaPerfectTest的XLua测试例子。  
3、主菜单调试菜单增加调试说明菜单项。

#### 2019.01.31，版本1.8.0：

1、修复双屏下副屏中最大化会消失的问题。(感谢Wisdom反馈)  
2、在文档里标明一下XLua的例子只有07_AsyncTest适合调试。(感谢未曾反馈)  
3、修复StartProfile时无法启动Profile的问题。(感谢未曾反馈)  
4、优化StopProfiling时卡住的问题，修改为异步等待调试钩子结果返回。(感谢Wisdom，未曾反馈)  
5、在StopProfiliing之后输出提示信息，提示用户触发lua操作。(感谢未曾反馈)

#### 2019.01.30，版本1.7.3：

1、修复无法正常检测某些版本的XLua工程的问题。(感谢Jay反馈)  
2、修复任务栏在左边和上边时最大化时显示有问题的问题。(感谢Jay反馈)  
3、优化txt和bytes文档是否是lua文档的判断策略。(感谢Jay反馈)  
4、修复有module(“xxx”, package.seeall)语句的文档无法下断点。(周林ansen反馈)  
5、优化LuaDebugee.dll被占用无法复制时的提示。

#### 2019.01.28，版本1.7.2：

1、修复如果传递给xluaL_loadbuffer的文档名字符串是原始require()的字符串时调试不了的问题。(感谢天地一MADAO反馈)

#### 2019.01.27，版本1.7.1：

1、修复断点列表在文档关闭后内不显示的问题。(感谢天地一MADAO反馈)

#### 2019.01.21，版本1.7.0：

1、增加类似Sublime里面Ctrl+R智能查找当前文档函数列表的功能。(感谢ce反馈)  
2、增加类似Sublime里面Ctrl+Shift+R智能查找当前工程函数列表的功能。(感谢ce反馈)  
3、修复代码编辑时Shift+Tab与预期不符的问题。(感谢ce反馈)  
4、修复编辑器里双击中文字符串崩溃的问题。(感谢yule反馈)  
5、修复调试时被Lua保存一些调试时监视到的C#对象导致切换场景时被工具检测为内存泄漏的问题。(感谢ares反馈)  
6、全部文档整合为一份。(感谢ce反馈)

#### 2019.01.21，版本1.6.1：

1、优化调试器匹配Lua文档的算法,增强算法适应性。(感谢glad反馈)

#### 2019.01.18，版本1.6.0：

1、修改目录结构，精简大小，且更容易找到执行档。  
2、优化自动更新进程，优化为增量更新。  
3、支持笔记本触摸板左右滑动事件。  
4、优化工程列表菜单为工程列表接口。

#### 2019.01.11，版本1.5.1：

1、修复读取txt文档会崩溃的问题。(感谢tock反馈)  
2、修复某些写法的LuaLoader下无法调试的问题。(感谢tock反馈)  
3、优化Lua api的生成。(感谢tock反馈)

#### 2019.01.09，版本1.5.0：

1、修复键盘上下移动后再注释时注释错误的行的问题。(感谢ares反馈)  
2、把启动时故意停下来的代码去掉。(感谢walle、ares反馈)  
3、查看性能测试数据时可以选择按代码顺序排序或者耗时排序。(感谢ares反馈)  
4、增加工程树形视图项跟随代码选项卡的激活而改变的TrackActive功能。(感谢walle反馈)  
5、修复自动更新失败导致屏蔽主进程启动的问题。(感谢ce反馈)  
6、修复添加了Assets内的Lua文档夹的工程断点失败的问题。(感谢ce反馈)  
7、整理了判断是否Lua文档的判断逻辑，新增识别bytes类型Lua文档。(感谢tock反馈)  
8、优化性能测试功能的交互，可以直接在编辑器中启动Profile/停止Profile。(感谢ares反馈)  
9、修复点击创建新文档后，焦点不在被创建文档的Edit上的问题。

#### 2019.01.08，版本1.4.3：

1、修改XLua调试文档，提示需要主动Generate Code。(感谢ares反馈)  
2、在XLua下判断到未导出的情况下主动提示Generate Code。(感谢ares反馈)  
3、将ObjectFormater.cs从lpproj目录移动至ThirdParty/LuaPerfect目录。(感谢ares反馈)  
4、修复ObjectFormater.cs里字符串格式化用了C++的%d的问题。(感谢ares反馈)

#### 2019.01.07，版本1.4.2：

1、支持在工程视图右键菜单中直接新增额外的Lua源代码目录。(感谢ares反馈)  
2、支持直接浏览指定XLua工程目录。(感谢ares反馈)

#### 2019.01.04，版本1.4.1：

1、对接入流程中的提示进行修改。（感谢walle反馈）

#### 2019.01.02，版本1.4.0：

1、新增显示函数签名功能。  
2、大幅度增强智能提示功能。

#### 2018.12.28，版本1.3.1：

1、消除泛型原型导致GetComponent()推导失败的问题。  
2、LuaDebuggee自身增加注解。

#### 2018.12.21，版本 1.3.0：

1、大幅度增强F12跳转定义功能。  
2、修复module()定义的模块中无法下断点的问题。(感谢ansen反馈)  
3、修复下了断点的文档被删除，重新启动时崩溃的问题。

#### 2018.12.19，版本 1.2.2：

1、提供可配置筛选哪些文档。(感谢ansen反馈)  
2、智能查找接口里对特定后缀名文档的过滤。(感谢ansen反馈)  
3、自动打开上次打开的工程。

#### 2018.12.18，版本 1.2.1：

1、Unreal及Cocos工程增加打开文档对话框方式打开工程。(感谢vader反馈)

#### 2018.12.14，版本 1.2.0：

1、实现在Unity内监视C#对象的功能。  
2、实现在Unity内可导出lua api。(感谢john反馈)

#### 2018.12.05，版本 1.1.0：

1、优化系统在大规模项目中的性能。(感谢ansen反馈)

#### 2018.11.14，版本 1.0.0：

1、首次正式发布。