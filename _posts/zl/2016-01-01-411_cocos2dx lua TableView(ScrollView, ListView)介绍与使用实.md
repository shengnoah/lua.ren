---
layout: post
title: cocos2dx lua TableView(ScrollView, ListView)介绍与使用实战 
tags: [lua文章]
categories: [topic]
---
在众多移动应用中，能看到各式各样的列表/表格数据

>
> 不管是iOS中的UITableView/UICollectionView/UIScrollView,还是Android中的ListView/CircleView，都是实际项目开发中使用最平凡，也是最重要的功能。

为了用户考虑，也为了性能考虑， 我们一般都会重复利用所创建的列表项，这样就避免了界面卡顿。cocos2dx lua
3.x中有一个TabeView，效果和上面列举的那些做列表的一样，尤其与iOS中UITableView方法属性，使用方式都有很多相似的地方。

>
> 游戏开发中，虽然没有和普通应用那么多列表，但是也会有些消息列表，用户，排行榜等，所以，这篇我们就来看看如何用TableView以及解决在实际开发中的一些问题。

### TableView使用

直接上代码，这里我们使用cocos2dx
lua提供TableView实现水平和垂直的列表，基本满足常见功能，具体细节，可以根据注释或者代码逻辑，结合实际需求进行调整和优化

    
    
    local TableScene = class("TableScene")
    TableScene.__index = TableScene
    
    --这里是为了让layer能调用TableViewTestLayer的方法
    function TableScene.extend(target)
        local t = tolua.getpeer(target)
        if not t then
            t = {}
            tolua.setpeer(target, t)
        end
        setmetatable(t, TableScene)
        return target
    end
    
    --滚动事件
    function TableScene.scrollViewDidScroll(view)
        --print("滚动事件")
    end
    
    function TableScene.scrollViewDidZoom(view)
        print("scrollViewDidZoom")
    end
    
    --cell点击事件
    function TableScene.tableCellTouched(table,cell)
        print("点击了cell：" .. cell:getIdx())
    end
    
    --cell的大小，注册事件就能直接影响界面，不需要主动调用
    function TableScene.cellSizeForTable(table,idx)
        return 150,150
    end
    
    --显示出可视部分的界面，出了裁剪区域的cell就会被复用
    function TableScene.tableCellAtIndex(table, idx)
        local strValue = string.format("%d",idx)
        print("数据加载"..strValue)
        local cell = table:dequeueCell()
        local label = nil
        if nil == cell then
            print("创建了新的cell")
            cell = cc.TableViewCell:new()
    
            --添加cell内容
            local sprite = display.newSprite("res/apple.png")
            sprite:setAnchorPoint(cc.p(0,0))
            sprite:setPosition(cc.p(0, 0))
            cell:addChild(sprite)
    
            label = cc.Label:createWithSystemFont(strValue, "Helvetica", 40)
            label:setPosition(cc.p(0,0))
            label:setAnchorPoint(cc.p(0,0))
            label:setColor(cc.c3b(255,0,0))
            label:setTag(123)
            cell:addChild(label)
        else
            print("使用已经创建过的cell")
            label = cell:getChildByTag(123)
            if nil ~= label then
                label:setString(strValue)
            end
        end
    
        return cell
    end
    
    --设置cell个数，注册就能生效，不用主动调用
    function TableScene.numberOfCellsInTableView(table)
        return 100
    end
    
    function TableScene:init()
    
        local visiableSize = cc.Director:getInstance():getVisibleSize()
        local origin = cc.Director:getInstance():getVisibleOrigin()
    
        local winSize = cc.Director:getInstance():getWinSize()
    
        local isVERTICAL = false
    
        if isVERTICAL then
    
    
            -----------------------------------------------------------
            --创建TableView
            local tableView = cc.TableView:create(cc.size(winSize.width - 20,150))
            --设置滚动方向  水平滚动
            tableView:setDirection(cc.SCROLLVIEW_DIRECTION_HORIZONTAL)
            tableView:setPosition(cc.p(10, winSize.height / 2))
            tableView:setDelegate()
            self:addChild(tableView)
            --registerScriptHandler functions must be before the reloadData funtion
            --注册脚本编写器函数必须在reloadData函数之前（有道自动翻译）
    
            --cell个数
            tableView:registerScriptHandler(TableScene.numberOfCellsInTableView,cc.NUMBER_OF_CELLS_IN_TABLEVIEW)
            --滚动事件
            tableView:registerScriptHandler(TableScene.scrollViewDidScroll,cc.SCROLLVIEW_SCRIPT_SCROLL)
            tableView:registerScriptHandler(TableScene.scrollViewDidZoom,cc.SCROLLVIEW_SCRIPT_ZOOM)
            --cell点击事件
            tableView:registerScriptHandler(TableScene.tableCellTouched,cc.TABLECELL_TOUCHED)
            --cell尺寸、大小
            tableView:registerScriptHandler(TableScene.cellSizeForTable,cc.TABLECELL_SIZE_FOR_INDEX)
            --显示出可视部分的cell
            tableView:registerScriptHandler(TableScene.tableCellAtIndex,cc.TABLECELL_SIZE_AT_INDEX)
            --调用这个才会显示界面
            tableView:reloadData()
            -----------------------------------------------------------
    
        else
    
            -----------------------------------------------------------
            --跟上面差不多，这里是创建一个“垂直滚动”的TableView
            tableView = cc.TableView:create(cc.size(200, winSize.height - 20))
            tableView:setDirection(cc.SCROLLVIEW_DIRECTION_VERTICAL)
            tableView:setPosition(cc.p(winSize.width / 2, 10))
            tableView:setDelegate()
            tableView:setVerticalFillOrder(cc.TABLEVIEW_FILL_TOPDOWN)
            self:addChild(tableView)
            --registerScriptHandler functions must be before the reloadData funtion
            --注册脚本编写器函数必须在reloadData函数之前（有道自动翻译）
    
            --cell个数
            tableView:registerScriptHandler(TableScene.numberOfCellsInTableView,cc.NUMBER_OF_CELLS_IN_TABLEVIEW)
            --滚动事件
            tableView:registerScriptHandler(TableScene.scrollViewDidScroll,cc.SCROLLVIEW_SCRIPT_SCROLL)
            tableView:registerScriptHandler(TableScene.scrollViewDidZoom,cc.SCROLLVIEW_SCRIPT_ZOOM)
            --cell点击事件
            tableView:registerScriptHandler(TableScene.tableCellTouched,cc.TABLECELL_TOUCHED)
            --cell尺寸、大小
            tableView:registerScriptHandler(TableScene.cellSizeForTable,cc.TABLECELL_SIZE_FOR_INDEX)
            --显示出可视部分的cell
            tableView:registerScriptHandler(TableScene.tableCellAtIndex,cc.TABLECELL_SIZE_AT_INDEX)
            --调用这个才会显示界面
            tableView:reloadData()
            -----------------------------------------------------------
    
        end
    
        return true
    end
    
    --这里是为了让layer能调用TableViewTestLayer的方法
    function TableScene.create()
        local layer = TableScene.extend(cc.Layer:create())
        if nil ~= layer then
            layer:init()
        end
    
        return layer
    end
    
    return TableScene
    

### 测试验证

    
    
    --[LUA-print] 数据加载0
    --[LUA-print] 创建了新的cell
    --[LUA-print] 数据加载1
    --[LUA-print] 创建了新的cell
    --[LUA-print] 数据加载2
    --[LUA-print] 创建了新的cell
    --[LUA-print] 数据加载3
    --[LUA-print] 创建了新的cell
    --[LUA-print] 数据加载4
    --[LUA-print] 创建了新的cell
    --[LUA-print] 数据加载5
    --[LUA-print] 创建了新的cell
    --[LUA-print] 数据加载6
    --[LUA-print] 创建了新的cell
    --[LUA-print] 数据加载7
    --[LUA-print] 创建了新的cell
    --[LUA-print] 数据加载8
    --[LUA-print] 创建了新的cell
    --[LUA-print] 数据加载9
    --[LUA-print] 使用已经创建过的cell
    --[LUA-print] 数据加载10
    --[LUA-print] 使用已经创建过的cell
    --[LUA-print] 数据加载11
    --[LUA-print] 使用已经创建过的cell
    --[LUA-print] 数据加载12
    --[LUA-print] 使用已经创建过的cell
    --[LUA-print] 数据加载13
    --[LUA-print] 使用已经创建过的cell
    --[LUA-print] 数据加载14
    --[LUA-print] 使用已经创建过的cell
    --[LUA-print] 数据加载15
    --[LUA-print] 使用已经创建过的cell
    --[LUA-print] 数据加载16
    --[LUA-print] 使用已经创建过的cell
    --[LUA-print] 数据加载17
    --[LUA-print] 使用已经创建过的cell
    --[LUA-print] 数据加载18
    --[LUA-print] 使用已经创建过的cell
    --[LUA-print] 数据加载19
    --[LUA-print] 使用已经创建过的cell
    --[LUA-print] 数据加载20
    

基本上重用了iOS中的UITableView机制，并且非常流畅的滑动显示一个完整的列表

> 注意点：reloadData()的调用和上面图层之前的关系

美中不足的是cocos创建的这个tableView也是有bug的，如果你的这个tableView有点击事件，不妨你上下滑动几下item，然后在隐藏的上下方点击，是不是仍然有点击事件呢。简单一招，添加Panel遮挡，勾上交互性轻松搞定。

cocos2dx lua中老版的listView也可以实现统一的功能，但是只能加载少量的item,多了就会很卡，所以推荐以后直接使用TableView。

### 其他拓展

网上有其他相关的一些教程，可以省去UI的搭建

  1. 先用CocosStudio或CocosCreator制作UI界面
    * 看这篇文章：<http://blog.csdn.net/fjdmy001/article/details/52982515>

  2. 然后修改config.json,

> 窗口的配置文件，想设置模拟器的大小就在这里设置

竖屏：”isLandscape” = false  
尺寸：”width” = 540, “height” = 960

  3. 修改config.lua

> 游戏的配置文件

开启全局变量：CC_DISABLE_GLOBAL = false  
设计尺寸：width = 1080，height = 1920，autoscale = “FIXED_HEIGHT”

使用cocos2dx的相关方法加载对应的文件，然后类似的方法去加载对应的内容(和iOS中定义tableview与cell类似)

具体可参考这里：

  * 排行榜之ScrollView：<https://blog.csdn.net/fjdmy001/article/details/52997012>
  * 排行榜之TableView：<https://blog.csdn.net/fjdmy001/article/details/52998376>