---
layout: post
title: cocos2dx lua —— Http请求总结与实战(封装) 
tags: [lua文章]
categories: [topic]
---
今天的主题是关于cocos2dx
lua实现短链接网络请求，使用Http实现基本的服务器网络数据获取，关于长链接（socket后续文件或者遇到需要的时候回特别实现与处理）

> 关于Http这里就不多做介绍了，不过，作为一个程序员，网络请求是开发中最多也是最重要的一环节，这里比较建议，搞懂http的整个请求流程！

推荐：[一次完整的HTTP请求过程 ](https://blog.csdn.net/yezitoo/article/details/78193794)

在有了基本的Lua知识和cocos2dx lua基本的了解和学习之后，我有了一个初步的cocos2dx
lua开发常识，然后就开始在上面实现基本的界面，并根据界面操作请求和响应数据！

### 入口场景

在main中初始化场景中必要的UI.

创建一个背景图片和一个按钮，实现点击按钮跳转到另外一个场景，进行网络请求和数据获取

    
    
    --- @class MainScene
    local MainScene = class("MainScene",cc.load("mvc").ViewBase)
    
    ---onEnter
    function MainScene:onEnter()
        print("onEnter")
    end
    
    ---createStaticButton 通用创建按钮方法
    ---@param node table
    ---@param imageName table
    ---@param x table
    ---@param y table
    ---@param callBack table
    local function createStaticButton(node, imageName, x, y, callBack)
    
        local btn = ccui.Button:create(imageName, imageName)
        btn:move(x, y)
        btn:addClickEventListener(callBack)
        btn:addTo(node)
    
    end
    --
    -----onCreate
    function MainScene:onCreate()
    
        -- 初始化背景
        display.newSprite("HelloWorld.png")
            :move(display.center)
            :addTo(self)
    
        -- 初始化按钮
        createStaticButton(self, "button_start.png", display.cx, display.cy-150, function ()
            self:getApp():enterScene("ApiRequest")
        end)
    
    end
    
    return MainScene
    

### 网络应用场景(ApiRequest)

然后开始处理跳转之后的ApiRequest，和相关请求逻辑，这里主要是使用我们封装好的CocosRequest实现基本上的请求逻辑，然后拿到数据之后我们就可以根据实际UI和具体业务逻辑做处理

> 注意

> 测试的时候，将local url =
> “[https://xxxx?cmd=501001&uid=628941&novelid=3782"中的值换成自己的地址就可以](https://xxxx?cmd=501001&uid=628941&novelid=3782"中的值换成自己的地址就可以)
    
    
    require "json"
    local CocosRequest = require "app.CocosRequest"
    
    --- @class ApiRequest
    local ApiRequest = class("ApiRequest", cc.load("mvc").ViewBase)
    ----local MainScene = class("MainScene", function() return display.newScene("MainScene") end)
    
    ---onEnter
    function ApiRequest:onEnter()
        print("onEnter")
    end
    
    -----onCreate
    function ApiRequest:onCreate()
    
        ----------------------- 创建自定义事件 start
        local function eventCustomListener1(event)
            local str = "response: "..event._usedata
            --labelStatusCode:setString(str)
    
            -- 如果返回的是 json 数据，这里解析
            local data =  json.decode(event._usedata)
            table.foreach(data,
             function(key, var)
                 print("-----"..key)
                 table.foreach(var,
                    function(a, b)
                       print(a.."-"..b)
                    end)
             end)
        end
    
        local listener1 = cc.EventListenerCustom:create("customEvent1",eventCustomListener1)
        cc.Director:getInstance():setNotificationNode(cc.Node:create())
        local eventDispatcher = cc.Director:getInstance():getNotificationNode():getEventDispatcher()
        eventDispatcher:addEventListenerWithFixedPriority(listener1, 6)
    
        -- 将事件分配器赋值到CocosRequest.eventDispatcher
        -- 用来在http请求返回的回调函数中使用，因为回调函数是在异步线程中执行，必须用自定义事件更新ui线程数据
        local tmpHttp = CocosRequest:getInstance()
        tmpHttp.eventDispatcher = eventDispatcher
        ----------------------- 创建自定义事件 end
    
        local tmp = CocosRequest:getInstance()
    
        local function callback(xhr)
            local event = cc.EventCustom:new("customEvent1")
            event._usedata = xhr.response
            eventDispatcher:dispatchEvent(event)
            print("post callback code = "..xhr.statusText)
        end
    
        local type = tmp.POST
        local url = "https://xxxx?cmd=501001&uid=628941&novelid=3782"
        local dataPost = {}
        dataPost.data = "hello"
        dataPost.aaa = "world"
        dataPost.bbb = "yang"
        tmp:send(type, url, dataPost, callback)
    
    end
    
    return ApiRequest
    

### 网络请求封装(CocosRequest)

最后才是我们的重头戏，CocosRequest是直接使用cocos2dx
lua提供的XMLHttpRequest实现，其实就是做了一套逻辑，具体细节可以根据项目调整(此处已经测试通过，可直接拷贝使用)

    
    
    require "json"
    
    CocosRequest = {}
    CocosRequest.__index = CocosRequest
    CocosRequest.instance = nil
    CocosRequest.callback = nil
    CocosRequest.POST = "POST"
    CocosRequest.GET = "GET"
    
    function CocosRequest:new()
        local self = {}
        setmetatable(self,CocosRequest)
        return self
    end
    
    function CocosRequest:getInstance()
        if nil == self.instance then
            self.instance = self:new()
        end
        return self.instance
    end
    
    -- 数据转换，将请求数据由 table 型转换成 string，参数：table
    function CocosRequest:dataParse(data)
        if "table" ~= type(data) then
            print("data is not a table")
            return nil
        end
    
        local tmp = {}
        for key, value in pairs(data) do
            table.insert(tmp,key.."="..value)
        end
    
        local newData = ""
        for i=1,#tmp do
            newData = newData..tostring(tmp[i])
            if i<#tmp then
                newData = newData.."&&"
            end
        end
        print("------- name is "..newData)
        return newData
    end
    
    -- 发送数据，参数：string，string，table
    function CocosRequest:send(type, url, data, callback)
        local xhr = cc.XMLHttpRequest:new() --new 一个http request 实例
        self.callback = callback    --设置需要执行的函数
    
        local newData = self:dataParse(data)
        if nil == newData or "" == newData then
            return
        end
    
        -- response回调函数
        local function responseCallback()
            print("CocosRequest - "..xhr.response)
            if nil ~= self.callback then
                self.callback(xhr)
            else
                print("callback is nil")
            end
        end
    
        -- 设置返回值类型及回调函数
        xhr.responseType = cc.XMLHTTPREQUEST_RESPONSE_STRING
        xhr:registerScriptHandler(responseCallback)
    
        -- 请求方式判断
        if self.POST == type then
            xhr:open(self.POST, url)
            xhr:registerScriptHandler(responseCallback)
            xhr:send(newData)
        elseif self.GET == type then
            xhr:open(self.GET, url.."?"..newData)
            xhr:send()
        else
            print("ERROR : type only can be "Post" or "GET"")
        end
    end
    
    ---------------------
    
    return CocosRequest
    

由于在Web中使用XMLHTTPRequest对象发出HTTP请求很普遍,Cocos2dx Lua对其进行了移植,可以在 Cocos2 d-x
LumP中使用 XMLHTTPRequest对象

XMLHTTPRequest对象中几个常用的函数和属性如下

  * (1)open(),与服务器连接,创建新的请求
  * (2)send(),向服务器发送请求
  * (3)abort(),退出当前请求
  * (4)readyState属性,提供当前请求的就绪状态,其中4表示准备就绪
  * (5)tatus属性,提供当前HTTP请求状态码,其中200表示成功请求
  * (6)respomseText属性,服务器返回的请求响应
  * (7) onreadystatechange属性。设置回调函数,当服务器处理完请求后就会自动调用该  
函数。

其中open和send函数,以及onreadystatechange属性是HTTP请求的关键。open函  
数有以下5个参数可以使用

  * (1) request-type:发送请求的类型。典型的值是GET或POST,也可以发送HEAD  
请求

  * (2) url:要请求连接的URL
  * (3) asynch:如果希望使用异步连接则为true,否则为 false。该参数是可选的,默认为
  * (4) username:如果需要身份验证,则可以在此指定用户名。该可选参数没有默认值
  * (5) password:如果需要身份验证,则可以在此指定口令。该可选参数没有默认值。

### 服务器响应

下面是我们验证返回后的服务器数据(未做处理)

    
    
    [
        {
            "code":1,
            "msg":"操作成功",
            "data":[
                {
                    "novelid":"3782",
                    "uid":"628875",
                    "cid":"5",
                    "title":"零下记忆",
                    "cover":"http://xxxx/img/cb/2f/f1/cb2ff155bbda8387f4b8efe917c46cd3.jpg",
                    "scover":"http://xxxx/img/cb/2f/f1/cb2ff155bbda8387f4b8efe917c46cd3.jpg",
                    "shareimg":"http://xxxx/img/ed/68/e6/ed68e67f77841ac76be189f0a04aa030.jpg",
                    "intro":"",
                    "tags":"悬疑",
                    "reason":"",
                    "issingle":"1",
                    "sort":"0",
                    "hits":"0",
                    "reads":"18753",
                    "clues":20,
                    "likes":"1",
                    "unlikes":"0",
                    "cmts":"0",
                    "favs":"2",
                    "words":"18420",
                    "chapters":"35",
                    "pub_chapters":"35",
                    "chapter_index":"1",
                    "pubtime":"0",
                    "updatetime":"1539337440",
                    "addtime":"1534222233",
                    "resversion":"30",
                    "isuser":"0",
                    "status":"1",
                    "wstatus":"1",
                    "offsale":"0",
                    "chapterstatus":"-1",
                    "leadrole":{
                        "roleid":"8478",
                        "rolename":"炽念"
                    },
                    "cname":"推理",
                    "isnew":0,
                    "fatime":"8月14日",
                    "isreading":1,
                    "iscomplete":1,
                    "isfav":0,
                    "liketype":0,
                    "shareinfo":{
                        "type":1,
                        "title":"零下记忆",
                        "intro":"",
                        "img":"http://xxxx/img/ed/68/e6/ed68e67f77841ac76be189f0a04aa030.jpg",
                        "url":"http://xxxx/novel/startReading?novelid=3782&chl=jmt"
                    },
                    "userinfo":{
                        "uid":"628875",
                        "username":"writer10",
                        "nickname":"离经易道",
                        "headurl":"http://xxxx/headimg/bb/d3/b6/628875_1_bbd3b631717ad615f38a6670573a375c_300x300.jpg?v=1526708983",
                        "sex":"1",
                        "vtype":"0",
                        "regtype":"2",
                        "vname":"",
                        "vicon":"",
                        "isfollow":0
                    }
                }
            ],
            "cmd":501001,
            "time":1539677040
        }
    ]