---
layout: post
title: cocos2dx lua cc & ccs & ccui区别和使用 
tags: [lua文章]
categories: [lua文章]
---
最近遇到一个问题，网上一些教程，或者官方的一些教程，在对应使用cc，ccs
,ccui的时候，导致我这边没有提示，开始以为是编辑器的问题，然后硬写上去之后，在调试的时候直接报错，经过多次查看Api文档发现，我使用的Api中，并没有这个方法，那么问题来了！

  * 我应该怎么去调用这个对应的Api呢？
  * 如何验证使用cocos2dx lua调用Api是正常的呢？
  * 遇到一个功能或者一个需求，应该如何查找对应的Api呢？

##### 网上有个朋友说了这么一段原话：

>
> 同一个框架里Api名字大小不统一我也就忍了（setBackGround…和setBackground…），可是同一个类名却表达两个不同的东西这个实在让我非常气愤，刚才瞎研究半天ScrollView才发现我程序里用的是ccui.ScrollView对象，而我盯着cc.ScrollView看了半天，我说怎么Api对不上号！

其实大概的意思就是cocos2dx
lua对应Api版本号的问题,因为以前cocos用cocos2d-lua写，后来带领大家往quick转，现在合并之后，又带领大家回到cocos2d-lua，所以必定会产生一些规范和版本号的区别。

> 尤其在2.x和3.x之间的变化比较大，或者是[quick版本和整合版](https://github.com/fusijie/Cocos-
> Resource)的多种调整。

  * 这里主要介绍关于cc，ccs ,ccui的区别，使用和注意点，如果要了解更多2.x和3.x版本之前的问题和区别，可以参考这里：

    * [cocos2d-x v3.0 Release Notes](https://github.com/cocos2d/cocos2d-x/blob/cocos2d-x-3.0/docs/RELEASE_NOTES.md#user-content-use-ccccsccui-gl-and-sp-as-module-name)

## cc & ccs & ccui

经过可靠资料和官方提供的信息我们可以得出一个大概的结论：

  * cc代表Cocos核心: Cocos2DConstants.lua 储存
  * ccs代表CocoStudio: StudioConstants.lua 储存
  * ccui代表CocoStudio的UI控件: GuiConstants.lua 储存

cc和ccui倒是很好理解，那么ccs这么理解呢？

> 其实应该说相当于C++的命名空间，ccs是cocostudio的缩写，代表在CocoStudio的命名空间（只是类似）。

##### 比如我们发现

  * 之前的cc.ui.xxx是quick自己封装过的控件，
  * 而ccs里面的对象是c++里面的原生控件
  * 若使用cc.ui.xxx将出现在ccs里面有交互的时候无法响应touch事件的问题
  * 必须用ccui.xxx，这个是c++里面的控件，touchEvent也是同一体系的。
  * 并且ccui.ScrollView 对应 cocos2d::ui::ScrollView，cc.ScrollView 对应 cocos2d::extension::ScrollView

### 我们来看看，最简单而且最常用的Button按钮的创建：

#### 这是老版的(quick)

    
    
    cc.ui.UIPushButton.new({ normal = "comm_btnGreenBackBack.png", pressed = "comm_btnGreenBackBack_sel.png" })
        :onButtonClicked(function()
            print("start")
        end)
        :pos( display.cx / 2, display.cy )
        :addTo(self)
    

  * 在Quick中有三种不同的Button控件，分别是：UIPushButton (按钮控件)、UICheckBoxButton ( CheckButton 控件)和 UICheckBoxButtonGroup ( CheckButton 组控件)。

    * 其中 UIPushButton 是最常用的按钮控件，它继承自UIButton，我们可以通过 cc.ui.UIPushButton.new(images, options) 方法来创建 UIPushButton。
    * 参数 images 是 table 类型，它代表各个按钮状态（正常、按下、禁用）下的图片；options 为可选参数，也是 tabl e类型，包含了是否scale9缩放，偏移flipX、flipY值等设置。

onButtonClicked
方法用于监听按钮的点击事件，当点击按钮时，将调用该方法中的代码。如上例中，当我们点击按钮时，会在控制台窗口中打印“start”的字段。同
onButtonClicked 方法类似的还有：

    
    
    onButtonPressed(callback)：用于监听按钮的按下事件
    onButtonRelease(callback)：用于监听按钮的释放事件
    onButtonStateChanged(callback)：用于监听按钮的状态改变事件
    

#### 新版本Button

    
    
        --这是一个按钮
    local btn = ccui.Button:create("button/btnDog_N.png", "button/btnDog_P.png", "button/btnDog_D.png", 0)
        :pos(display.cx, 100)
        :addTo(self)
        --按钮文字
        btn:setTitleText("按钮")
        --字体大小
        btn:setTitleFontSize(25)
        --偏移
        btn:setTitleOffset(20, 100)
        --字体颜色
        btn:setTitleColor(cc.c3b(255, 255, 255))
        --按钮的回调函数
        btn:addTouchEventListener(function(sender, eventType)
        if (0 == eventType)  then
            print("pressed")
        elseif (1 == eventType)  then
              print("move")
        elseif  (2== eventType) then
              print("up")
        elseif  (3== eventType) then
            print("cancel")
        end
    end)
    
    --按钮无效
    --btn:setEnabled(false)
    

以上就是基本按钮的创建，所以平时在开发中要注意一下API的问题，不然很多错误都不知道问题出在哪里！

##### 在我们平时使用控件的时候，也同样会遇到一些问题，比如

  * quick中的cc.ui.UIPushButton和ccui.Layout是不能共存的，在layout存在的情况下UIPushButton是不能被监听到的
  * 如果在quick中使用ccui.Button，那么ccui.Button是不能被屏蔽的，也就是说，无论quick中怎么处理不让ccui.Button被点击都无效，只要他存在，就会被点击。
  * 同时cc.ui.UIScrollView也存在问题，其不能被缩放，但是其父节点可以缩放。
  * cc.ui.UIListView也存在不能缩放的问题。

> 也就是说，以上API中，一旦按设计的分辨率来设计，就不能再被缩放。

### Api官方文档验证

为了验证这个问题我们查看官方文档发现这样的内容：

    
    
    Misc Api changes
    Use cc、ccs、ccui gl and sp as module name
    
    Now classes are bound into different modules instead of using global module. This will avoid conflicts with other codes.
    
        classes in cocos2d、cocos2d::extension、CocosDenshion and cocosbuilder were bound to cc module
        classes in cocos2d::ui were bound to ccui module
        classes in spine were bound to sp module
        classes in cocostudio were bound to ccs module
        global variables are bound to corresponding modules
        all funcionts and constants about openGl were bound to gl module
    
    Examples:
    
        | v2.1                    | v3.0                    |
        | CCDirector              | cc.Director             |
        | CCArmature              | ccs.Armature            |
        | kCCTextAlignmentLeft    | cc.kCCTextAlignmentLeft |
    

最终所表达的意思其实就是

  * 现在，类被绑定到不同的模块中，而不是使用全局模块。这将避免与其他代码发生冲突。
  * CCOS2D、COCOS2D:：扩展、COCOSDENSHION和COCOSUBIDER绑定到CC模块
  * CCOS2D:：UI绑定到CCUI模块
  * 脊柱类与SP模块结合
  * COStudio中的类绑定到CCS模块
  * 全局变量绑定到相应模块。
  * OpenGL的所有函数和常数都绑定到GL模块。

> OpenGL和SP可以忽略

### 实战与代码验证

最后，我们为了cc，ccs，ccui专门去寻找了一些关于Api所提供的支持和规律。

##### cc在Cocos2dConstants.lua中,用来存储cc 模块的常量

    
    
    cc = cc or {}
    

  * 并且每个可调用的文件对应的最前面也有同样的定义,比如有
    * Action.lua
    * Animation.lua
    * Director.lua
    * EventDispatcher.lua
    * GLView.lua
    * Hide.lua
    * Image.lua
    * Node.lua
    * Scene.lua
    * Scheduler.lua
    * SpriteFrame.lua
    * SpriteFrameCache.lua
    * TextureCache.lua
    * Timer.lua
    * Touch.lua
    * UserDefault.lua
    * src/cocos/cocos2d/Cocos2d.lua
    * src/cocos/cocos2d/Cocos2dConstants.lua
    * src/cocos/controller/ControllerConstants.lua
    * ……

##### ccs在StudioConstants.lua中,用来存储ccs模块的常量

    
    
    ccs = ccs or {}
    

  * 并且每个可调用的文件对应的最前面也有同样的定义,比如有
    * ActionFrame.lua
    * ActionObject.lua
    * Armature.lua
    * Bone.lua
    * ColorFrame.lua
    * ComAttribute.lua
    * ComController.lua
    * Frame.lua
    * Skin.lua
    * TextureFrame.lua
    * Timeline.lua
    * VisibleFrame.lua
    * ……

##### ccui在GuiConstants.lua中,用来存储ccui模块的常量

    
    
    ccui = ccui or {}
    

  * 并且每个可调用的文件对应的最前面也有同样的定义,比如有
    * Button.lua
    * CheckBox.lua
    * EditBox.lua
    * Helper.lua
    * ImageView.lua
    * Layout.lua
    * ListView.lua
    * LoadingBar.lua
    * PageView.lua
    * ScrollView.lua
    * Slider.lua
    * Text.lua
    * TextField.lua
    * ……

###### 推荐

  * <https://blog.csdn.net/blackzhangwei/article/details/80088314?utm_source=blogxgwz0>
    * [cocos2d-x_v3.0_release_notes.md](https://github.com/fusijie/Cocos2dx-Release-Note/blob/master/cocos2d-x_v3.0_release_notes.md)
    * [C++11 新特性](http://blog.csdn.net/ls1122/article/details/38339851)