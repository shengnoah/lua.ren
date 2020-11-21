---
layout: post
title: LuaFramework 
tags: [lua文章]
categories: [topic]
---
思考并回答以下问题：

  * 用Lua写逻辑时，可以先用C#实现，再转换成Lua语言。这样好吗？

![](https://chebincarl.github.io//2019/10/16/LuaFramework-1/100.jpg)

Lua的几个变种：XLua、ToLua（原uLua）和Slua都可以做Unity热更，而ToLua更是提供了一个简易的热更框架--
LuaFramework_UGUI，使得上手变得容易，因此选定LuaFramework_UGUI框架来实现项目的热更功能。

问：ToLua、XLua以及SLua，它们之间是什么关系？  
答：个人理解，Lua定义了一种语言规范，而ToLua、Xlua、Slua都是这种规范的一种实现。

问：Unity、ToLua、LuaFramework_UGUI，它们之间有什么联系？  
答：ToLua搭建了一个Lua语言与Unity中c#语言沟通的桥梁，借助ToLua，你可以在C#语言中调用Lua方法，也可以在Lua语言中调用C#方法。

而LuaFramework_UGUI则是基于ToLua的这种能力实现的一个热更新方案（提供包括资源包管理、下载、加载等一系列功能）。

tolua是用纯lua开发游戏逻辑，不能对C#中的代码进行修改。  
xlua可以通过hotfix对C#中的代码进行修改，更加方便修改。

# 配置Lua开发环境

有一点要说明的是，使用此种方式（ToLua+LuaFramework）做热更新，则意味着你的大部分逻辑都需要改用Lua语言来编写。

因此，开发前得先得配置好Lua开发环境。毕竟，工欲善其事，必先利其器。

步骤如下：

  * 1、下载并安装IntelliJ IDEA Community Edition。 
  * 2、在EmmyLua的群里找到IntelliJ-EmmyLua-版本号.zip、EmmyLua-Unity.zip两个插件文件，然后安装插件。
  * 3、把EmmyLuaService.cs文件放到Unity项目下的Editor文件夹下，并点击Enable。
  * 4、然后在IDEA里就可以有Unity函数的提示了。

> 把.txt关联，去掉.meta。

![](https://chebincarl.github.io//2019/10/16/LuaFramework-1/0.png)

# Lua中是怎么加载一个面板的

运行框架，显示了一个Lua脚本动态创建的面板，即PromptPanel，如图1所示。

![](https://chebincarl.github.io//2019/10/16/LuaFramework-1/1.png)

翻看框架的目录结构，会在Assets/LuaFrame/Examples/Builds/Prompt目录找到两个预制体，PromptPanel和PromptItem，也就是这个面板的主体和兽人头像，如图2所示。

![](https://chebincarl.github.io//2019/10/16/LuaFramework-1/2.png)

用IntelliJ
IDEA打开工程目录，在Controller目录和View目录会找到与PromptPanel密切相关的两个文件PromptCtrl.lua、PromptPanel.lua，如图3所示

![](https://chebincarl.github.io//2019/10/16/LuaFramework-1/3.png)

由目录名称可知，此框架采用了一种MVC结构，用以对代码功能做区分。xxxPanel负责页面显示逻辑，xxxCtrl负责事件处理，示例没有给出明显的Model层，读者可以根据自身项目酌情添加。

继续查看框架代码，会在Logic/Game.lua中找到游戏的入口：Game.OnInitOK函数，见图4。

![](https://chebincarl.github.io//2019/10/16/LuaFramework-1/4.png)

在这个函数中，有3个重要逻辑：

  * 1、初始化View
  * 2、初始化Ctrl
  * 3、启动Ctrl

**1、初始化View**

初始化View就是调用InitViewPanels这个函数，InitViewPanels函数用于加载View目录下定义的xxxPanel。  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    function ()  
       for i = 1, #PanelNames do  
          require ("View/"..tostring(PanelNames[i]))  
       end  
    end  
      
  
---|---  
  
PanelName则是在LuaFramwwork/Lua/Common/define.lua中定义的，对应面板的名称。

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    PanelNames = {  
        "PromptPanel",      
        "MessagePanel",  
    }  
      
  
---|---  
  
**2、初始化Ctrl**

初始化Ctrl是指CtrlManager.Init();这句，可以在LuaFramwwork/Lua/Logic/CtrlManager.lua中看到相关定义。这个函数中通过调用New函数创建了Ctrl的实例。

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    function CtrlManager.Init()  
        logWarn("CtrlManager.Init----->>>");  
        ctrlList[CtrlNames.Prompt] = PromptCtrl.New();  
        ctrlList[CtrlNames.Message] = MessageCtrl.New();  
        return this;  
    end  
      
  
---|---  
  
**3、启动Ctrl**

启动就是根据CtrlNames找到对应的Ctrl的实例，然后调用其Awake方法：

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    local ctrl = CtrlManager.GetCtrl(CtrlNames.Prompt);  
    if ctrl ~= nil and AppConst.ExampleMode == 1 then  
        ctrl:Awake();  
    end  
      
  
---|---  
  
以上都是推测，

为了验证猜测的对不对，我把CtrlManager.GetCtrl(CtrlNames.Prompt)这一句改为CtrlManager.GetCtrl(CtrlNames.Message)，如果这次加载出来的是MessagePanel，则说明上述过程推断正确。

….

改完后运行，发现加载的还是PromptPanel，难道确实是找错地方了？

别急，这里还涉及另一个概念。

在热更框架中，程序运行的并不是我们在LuaFramework/lua目录下编写的代码，而是在Assets/StreamingAssets目录下的打包后的代码，见图5。

![](https://chebincarl.github.io//2019/10/16/LuaFramework-1/5.png)

那么有什么办法让我们刚刚改的代码生效呢？

思路有两个：

  * 1、将写的代码打包到StreamAssets中；
  * 2、让程序直接运行打包前的代码；

思路1的操作方法是：执行LuaFramework菜单下的Build XXX
Resources菜单（见图2-6），因为我现在的程序是运行在Windows平台，所以选择Build Windows Resource。

![](https://chebincarl.github.io//2019/10/16/LuaFramework-1/6.png)

点击菜单，等待重新打包完成。打包结束后，能看到整个StreamingAssets目录中的内容都更新了，在里边可以找到message和prompt相关的资源，见图7。

![](https://chebincarl.github.io//2019/10/16/LuaFramework-1/7.png)

重新运行后，得到了想的结果，程序直接加载了MessagePanel面板，见图8。

![](https://chebincarl.github.io/https://chebincarl.github.io//2019/10/16/LuaFramework-1/8.png)

由此印证我们对整个面板流程的加载的推测分析。

关于思路2让程序直接运行打包前的代码，只需要关闭Lua的AssetBundle模式就好了。

找到LuaFramework/Scripts/ConstDefine/AppConst.cs文件，将LuaBundleMode =
true;改为LuaBundleMode = false;即可，见图8，图中是改过之后的。

![](/2019/10/16/LuaFramework-1/8.png)

LuaBundleMode 改为false之后，Lua代码修改后就无需重新Build xxx Resources就能直接看到效果。

尽管思路1和思路2是二选一即可的，但为方便后边的示例，这里要统一修改为false。

## 3、如何创建自己的面板

在上一步的分析中，我们得知创建一个面板需要先初始化View，再实例化Ctrl，然后调用Ctrl的Awake。这些都是代码层面的，前提还有一个，我们需要一个XxxPanel预制体。

总结一下，如果要创建一个我们自己的面板，则需要如下步骤：

1、创建一个XxxPanel预制体

2、创建对应的XxxView

3、创建对应的XxxCtrl

4、添加CtrlNames及PanelNames

5、加载XxxCtrl

下面我将以FirstPanel为例进行演示。

1、创建FirstPanel预制体。  
在Hierarchy面板中创建一个FirstPanel，并在LuaFramework目录下新建CustomPrj/FirstTest目录，将FirstPanel拖到此做成预制体，见图3-1。

图3-1

然后删掉Hierarchy面板中的FirstPanel，因为后面我们会动态加载它。

2、创建FirstView.lua脚本。  
在Lua/View目录下创建一个FirstView的lua脚本，脚本结构参照MessageView编写，如下：

View Code

注：lua脚本的创建方法是在IDEA中，选中目录，右键->New->Lua File。

3、创建FirstCtrl.lua脚本。  
在Lua/Controller目录下创建一个FirsCtrl的lua脚本，脚本结构参照MessagCtrl编写，如下：

View Code

4、添加CtrlNames及PanelNames  
在Lua/Common找到define.lua，在CtrlNames中添加 First =
“FirstCtrl”,在PanelNames中添加”FirstPanel”，如下：

    
    
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
    

|

    
    
    CtrlNames = {  
        Prompt = "PromptCtrl",  
        Message = "MessageCtrl",  
        First = "FirstCtrl"  
    }  
      
    PanelNames = {  
        "PromptPanel",      
        "MessagePanel",  
        "FirstPanel"  
    }  
      
  
---|---  
  
5、加载FirstCtrl  
在Lua/Logic/Game.lua文件的Game.OnInitOK函数中，将CtrlManager.GetCtrl()的参数修改为我们刚刚添加的CtrlNames.First，如下所示：  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    CtrlManager.Init();  
    local ctrl = CtrlManager.GetCtrl(CtrlNames.First);  
    if ctrl ~= nil and AppConst.ExampleMode == 1 then  
        ctrl:Awake();  
    end  
      
  
---|---  
  
保存代码并运行

…………..

嗯，什么都没加载出来。

好吧，我得承认，在学习这个框架的过程中，每走一步都是坑。

我就是在艰难的趟过这些坑来之后，才觉得有必要将这个过程记录下来，才有了这一系列文章，希望对后来人有所帮助。

………….

为什么我们自己的创建的面板没有加载呢？

查看日志发现，在”LuaFramework
InitOK—->>>”日志输出之前，PromptCtrl.New和MessageCtrl.New都被调用了一次，而我们新加的FirstCtrl却没有，见图3-2。

图3-2

应该是我们某些地方少加了调用。

查找后发现，确实有这样一个地方。在Lua/Logic/CtrlManager.lua脚本的Init方法，对所有Ctrl的New方法进行了调用。

我们添加对FirstCtrl.New的调用，如下：  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    function CtrlManager.Init()  
        logWarn("CtrlManager.Init----->>>");  
        ctrlList[CtrlNames.Prompt] = PromptCtrl.New();  
        ctrlList[CtrlNames.Message] = MessageCtrl.New();  
        ctrlList[CtrlNames.First] = FirstCtrl.New();  
        return this;  
    end  
      
  
---|---  
  
（其实第二节中我们发现了这个地方，本节中忘了将自己的代码加进去）

然后再运行

…..

报错了,说我们的FirstCtrl是一个nil value， 见图3-3

图3-3

经查，是在CtrlManager中，我们没有加载对应的脚本，见图3-4（图中是已添加之后的）

图3-4

再次运行

出现了更多的错误，见图3-5

图3-5

……

有没有想崩溃的感觉，唉，我当初就是这么一步步过来的。

这次的错误是缺少first.unity3d.

这里的原因是，我们之前刚把Lua代码AssetBundle模式关掉（设置为false），lua代码不用AssetBundle模式了，但我们的资源（FirstPanel预制体）还
是使用的AssetBundle模式。

并且资源的AssetBundle模式好像无法关闭，因此需要对FirstPanel预制体进行打包操作。

操作如下：

1、找到LuaFramework/Editor/Packager.cs文件中的HandleExampleBundle方法（约160行左右），添加对FirstPanel预制体打包的代码，包名为”first”，如下所示：

    
    
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
    

|

    
    
      
    /// 处理框架实例包  
    /// </summary>  
    static void HandleExampleBundle()   
    {  
        string resPath = AppDataPath + "/" + AppConst.AssetDir + "/";  
        if (!Directory.Exists(resPath)) Directory.CreateDirectory(resPath);  
      
        AddBuildMap("prompt" + AppConst.ExtName, "*.prefab", "Assets/LuaFramework/Examples/Builds/Prompt");  
        AddBuildMap("message" + AppConst.ExtName, "*.prefab", "Assets/LuaFramework/Examples/Builds/Message");  
      
        //打包我们新加的FirstPanel预制体  
        AddBuildMap("first" + AppConst.ExtName, "*.prefab", "Assets/LuaFramework/CustomPrj/FirstTest");  
      
        AddBuildMap("prompt_asset" + AppConst.ExtName, "*.png", "Assets/LuaFramework/Examples/Textures/Prompt");  
        AddBuildMap("shared_asset" + AppConst.ExtName, "*.png", "Assets/LuaFramework/Examples/Textures/Shared");  
    }  
      
  
---|---  
  
2、执行unity编辑器上方LuaFramework菜单中的Build Windows
Resources菜单项，进行打包操作。打包完成后，可以在StreamingAssets目录中看到first.unity3d文件。见图3-6

图3-6

再次运行，

这次终于得到了我们想要的结果，我们自己创建的面板FirstPanel，就这么加载出来了。

见图3-7

图3-7

真是太不容易了！

现在，将我们改错的经过都加入到完整的步骤中，那么，加载一个我们自己创建的面板的完整步骤如下：

1、创建一个XxxPanel预制体

2、创建对应的XxxView

3、创建对应的XxxCtrl

4、添加CtrlNames及PanelNames

5、在CtrlManager中加入对XxxCtrl.New的调用，并在头部require “XxxCtrl”

6、在Packager.cs文件中对XxxPanel预制体进行打包

7、在Game.lua加载XxxCtrl

后续写模块的时候都会按这个流程来。

# 后记

在本篇文章的第二节的写作过程中，为什么我会用推测并验证的写法，而不是直接给出一个正确结论？第三节中，我为什么没有直接给出正确的操作步骤，而是边走边改错？

因为我希望本文能如实还原我学LuaFramework的过程，记录每一个问题的发生条件，以及我解决问题的思路。

# 命令模式走迷宫

第一节 搭建游戏场景和UI

试玩。

第二节 详解命令模式

行为型模式 面向对象编程思想

第三节 应用命令模式到游戏中

总结

# StrangeIOC的使用

第一节 基本了解

第二节 案例

第三节 和PureMVC的比较

# Unity实现贪吃蛇游戏