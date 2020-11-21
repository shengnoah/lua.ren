---
layout: post
title: 基于 UGUI 、 toLua 的 LuaFramework  
tags: [lua文章]
categories: [topic]
---
### 怎么做到热更新的

开始游戏时对比本地和服务器的files.txt,其中包含了每个AssetBundle的名字和MD5码，如果有文件不存在过MD5码不同则进行更新下载，进而实现更新内容。

#### 个人理解

luaframework的好处：

  1. 不需要在Unity内考虑如何调用lua脚本
  2. 只需要完善相应panel界面以及Ctrl和Panel脚本即可

#### 使用过程：

  1. Unity项目标题栏LuaFramework根据项目平台进行Build  
会有错误出现，无关紧要的错误注释即可

  2. 安装资源管理服务器，将StreamingAssets文件夹放入其中
  3. 配置AppConst.cs内容  
UpdateMode（更新模式-默认关闭）、GameFrameRate（游戏帧频）、WebUrl（测试更新地址）

  4. 打开Sences/main场景，正常显示测试场景内容即可
  5. 将场景打包发布对应平台
  6. 在Unity中更改main场景中对应的panel
  7. Unity标题栏LuaFramework选项根据对应项目平台重新进行Build
  8. 将StreamingAssets文件夹重新放入资源管理服务器
  9. 在模拟器中打开相应app查看场景内容是否改变

##### 建议：

  1. 发布windows平台exe应用无法进行内容的更新显示，始终是打包时的内容，建议打包成 Android apk
  2. 打包安卓 apk 时会报错，需要把Plugins文件加下的ios,x86,x64文件夹删除即可正常打包

#### 添加Panel流程：

如果你想要添加一个Panel,你需要一下几个东西：

  1. 在unity中的Panel预制体
  2. 需要有两个lua脚本，  
一个panel.lua，用来控制该界面元素的显隐操作  
一个ctrl.lua，用来编写该界面的逻辑内容  
类似于PromptPanel和PromptCtrl

  3. 把panel.lua放进define脚本中的PanelNames中
  4. 把ctrl.lua放进define脚本中的CtrlNames中
  5. 在CtrlManager中require所有的lua的Ctrl脚本
  6. 在CtrlManager的Init方法中添加相应的数据

##### 注意点：

  1. 在CustomSetting.cs中的customTypeList可添加要导出注册到lua的类型列表
  2. 如果有新的资源需要打包，需要在Packager的HandleExampleBundle()方法中添加，每一行就是一个包

#### 在lua中调用unity的静态类

  1. 在CustomSetting.cs中的customTypeList、staticClassTypes注册类
  2. 在LuaFramework/ToLua/Lua中的tolua.lua中reqire要注册的类
  3. 点击Lua/Generate All Lua
  4. 即可在Lua脚本中直接调用静态类中的方法

##### 注意点：

  1. lua中调用Unity的静态类的属性和方法都只需要一个点 例如PlayerPrefs.GetString(username)

#### 在lua中调用unity的非静态类

  1. 在CustomSetting.cs中的customTypeList中注册类
  2. 类似lua中Ctrl脚本的panelMgr:CreatePanel(‘Login’, this.OnCreate);回调OnCreate把unity中的对象传到lua中，所以OnCreate有一个obj参数，function LoginCtrl.OnCreate(obj)

##### 注意点：

  1. 在lua中调用unity中的非静态类的属性时用一个点，调用方法时用冒号  
例如：transform = obj.transform;  
panel = transform:GetComponent(‘UIPanel’);

#### 常见错误

  1. OnStreamingAssets—»xxxxxx 解决方法：未成功生成相应平台资源包，重新点击Unity标题栏LuaFramework选项根据对应项目平台重新进行Build

  2. 打开一个panel后关闭它，然后再重新打开，可能会出现背景图片消失的情况 解决方法：将预制物打一个包即可，背景图片无需另外打一个包

* * *