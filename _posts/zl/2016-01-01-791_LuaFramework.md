---
layout: post
title: LuaFramework 
tags: [lua文章]
categories: [topic]
---
思考并回答以下问题：

在上一篇文章[LuaFramework-1](/2019/10/16/LuaFramework-1/
"LuaFramework-1")中，介绍了LuaFramework加载面板的方法，但这个方法并不适用于其它Prefab资源，在这套框架中非面板型资源的加载方法另有套路。

## 创建一个预制体

打开上次使用的工程，打开Main场景，创建一个名为ImgOrc的Image，图片就选例子用的兽人头像。在Assets/LuaFramework/CustomPrj目录下新建一个Prefabs目录，然后拖动ImgOrc到该目录下做成预制体，如图1-1

2、将预制体打成AssetBundle包  
打开Assets/ LuaFramework/Editor/Packager.cs文件（用VS或Mono
Develop编辑器打开），找到HandleExampleBundle方法，添加对ImgOrc预制体的打包代码，如图1-2所示

图1-2

    
    
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
    

|

    
    
      
    /// 处理框架实例包  
    /// </summary>  
    static void ()   
    {  
        string resPath = AppDataPath + "/" + AppConst.AssetDir + "/";  
        if (!Directory.Exists(resPath)) Directory.CreateDirectory(resPath);  
      
        AddBuildMap("prompt" + AppConst.ExtName, "*.prefab", "Assets/LuaFramework/Examples/Builds/Prompt");  
        AddBuildMap("message" + AppConst.ExtName, "*.prefab", "Assets/LuaFramework/Examples/Builds/Message");  
      
        //打包我们新加的FirstPanel预制体  
        AddBuildMap("first" + AppConst.ExtName, "*.prefab", "Assets/LuaFramework/CustomPrj/FirstTest");  
        //打包我们新加的ImgOrc预制体  
        AddBuildMap("prefabs" + AppConst.ExtName, "*.prefab", "Assets/LuaFramework/CustomPrj/Prefabs");  
      
        AddBuildMap("prompt_asset" + AppConst.ExtName, "*.png", "Assets/LuaFramework/Examples/Textures/Prompt");  
        AddBuildMap("shared_asset" + AppConst.ExtName, "*.png", "Assets/LuaFramework/Examples/Textures/Shared");  
    }  
      
  
---|---  
  
这里要对AddBuildMap方法的参数加以说明

第一个参数是包名，即打包后在StreamingAssets目录中显示的包的名称，AppConst.ExtName是扩展名，框架默认为”.unity3d”，可以自行修改；  
第三个参数是要打包的资源的目录，暂且称为源目录；  
第二个参数是打包模式，以字符串形式过滤出要打包的文件名，比如这里*.prefab就表示源目录下的所有prefab文件。  
之所以将ImgOrc预制体打包的包名定义为prefabs而不是ImgOrc，是因为这个打包是针对整个目录的而不是单个资源。为了说明这一情况，我会在CustomPrj/Prefabs目录下再加一个预制体ButtonPrefab。见图1-3

见图1-3

这次不修改打包文件。

找到LuaFramework菜单，点击Build Windows Resources菜单项，开始自动打包操作，中间无需干预。

打包结束，查看StreamingAssets目录，能看到与刚刚给定的包名相应的文件：prefabs.unity3d，见图1-4

图1-4

图中红框标识的还有一个名为prefabs的文件，这是没有显示后缀（其实有后缀，在unity中未显示），这个是prefabs.unity3d包的清单文件，在windows下打开看以看到其全名为prefabs.unity.manifest。

用Notepad++打开这个清单文件，可以看到在37、38行列出一这个AB包包含的两个资源：ButtonPrefab.prefab和ImgOrc.Prefab。见图1-5：

图1-5

此框架的资源打包方式就是这样的，所有资源都需要在代码中添加相应包名、过滤模式、目录。游戏开发前期和资源变动较大时，会频繁改动这个脚本的内容，用起来并不是很方便。

3、在代码中加载预制体  
资源包有了，现在需要在代码中加载这个包。

打开FirstCtrl.lua文件，在FirstCtrl.OnCreate方法中添加读取资源包的方法，见代码：

复制代码  
—启动事件—  
function FirstCtrl.OnCreate(obj)  
gameObject = obj;  
message = gameObject:GetComponent(‘LuaBehaviour’);

    
    
    --加载prefabs.unity3d资源包
    resMgr:LoadPrefab("prefabs.unity3d", {"ImgOrc"}, function (prefabs)
    
    end);
    

end  
复制代码  
resMgr是框架封装好的资源管理工具，LoadPrefab函数用来读取资源包。第一个参数是包名，第二个参数是包里的资源名（Prefab名称），第三个参数是包读取成功的回调，参数prefabs为读出来的资源。

..

继续完成代码，进行实例操作等操作，代码如下：

说明点：

回调函数的参数prefabs是一个userdata类型的数据（userdata一般是C#中的部分引用类型在Lua中的表示），这里猜测是一个数组，因为通过.Length可以取到长度，能过[0]能取到第一个元素。  
newObject是框架封装好的实例化函数，猜测本质就是c#中的GameObject.Instantiate方法。  
设置父对象、位置、缩放这几步操作和c#中的操作差不多。因为本身调用的就是c#中的方法，即transform的方法。  
FirstCtrl.lua文件中的gameObject，transform代表的就是FirstPanel面板及面板上transform组件，是在面板创建时（OnCreate方法前两行）注入进来的。

代码完成后，运行Unity，能看到ImgOrc已经加载到了FirstPanel对象下，见图1-6

图1-6

非Panel的预制体加载流程就是这样，加载方法是参照例子的PromptCtrl.lua写的。

resMgr中还有放多其它加载资源的方法，留待以后再探究。

2、怎么给按钮添加监听  
在用c#写代码的时候，给Button添加监听有两种方法，一是将脚本绑在Button组件上，通过面板选择脚本中的方法来添加做；二是在代码中通过Button.onClick.AddListener方法添加。

那么在Lua应该怎么做呢？

还是以FirstPanel为例，给FirstPanel右上角添加一个关闭按钮，应用预制体然后重新打包，

然后：

1、在FirstPanel.lua文件中引用按钮  
打开FirstPanel.lua文件，在InitPanel函数中添加查找按钮的代码：

—初始化面板—  
function FirstPanel.InitPanel()  
—查找关闭按钮  
this.btnClose = transform:FindChild(“CloseButton”).gameObject;  
end

2、在FirstCtrl.lua文件中添加监听

打开FirstCtrl.lua文件，找到OnCreate方法，然后通过FirstPanel所挂的LuaBehaviour脚本来添加监听事件，见图2-1

图2-1

AddClick方法有两个参数，第一个是按钮本身（上一步才引用过的），第二个是点击后的回调函数。

AddClick的具体实现可以可以在LuaBehaviour.cs中找到。

运行Unity，点击关闭按钮，能看到打印了期望中的日志，见图2-2

图2-2

给按钮添加监听就是这么简单，不过里边还藏着一些坑，以后的文章再细讲。

总结一点，用Lua做逻辑的话，所有UI元素的使用都需要先在相应的XxxPanel中引用
，然后到XxxCtrl中添加事件，对于结构复杂的UI，做起来非常耗时间。

文中多次操作了FirstCtrl.lua和FirstPanel.lua文件，为了方便参阅，现将两个脚本完整的贴出来：

FirstPanel

FirstCtrl