---
layout: post
title: cocos2dx lua —— Http请求总结与实战(封装) 
tags: [lua文章]
categories: [topic]
---
<p>今天的主题是关于cocos2dx lua实现短链接网络请求，使用Http实现基本的服务器网络数据获取，关于长链接（socket后续文件或者遇到需要的时候回特别实现与处理）</p>
<blockquote>
<p>关于Http这里就不多做介绍了，不过，作为一个程序员，网络请求是开发中最多也是最重要的一环节，这里比较建议，搞懂http的整个请求流程！</p>
</blockquote>
<p>推荐：<a href="https://blog.csdn.net/yezitoo/article/details/78193794" target="_blank" rel="noopener noreferrer">一次完整的HTTP请求过程
</a></p>

<p>在有了基本的Lua知识和cocos2dx lua基本的了解和学习之后，我有了一个初步的cocos2dx lua开发常识，然后就开始在上面实现基本的界面，并根据界面操作请求和响应数据！</p>
<h3 id="入口场景"><a href="#入口场景" class="headerlink" title="入口场景"></a>入口场景</h3><p>在main中初始化场景中必要的UI.</p>
<p>创建一个背景图片和一个按钮，实现点击按钮跳转到另外一个场景，进行网络请求和数据获取</p>
<pre><code>--- @class MainScene
local MainScene = class(&#34;MainScene&#34;,cc.load(&#34;mvc&#34;).ViewBase)

---onEnter
function MainScene:onEnter()
    print(&#34;onEnter&#34;)
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
    display.newSprite(&#34;HelloWorld.png&#34;)
        :move(display.center)
        :addTo(self)

    -- 初始化按钮
    createStaticButton(self, &#34;button_start.png&#34;, display.cx, display.cy-150, function ()
        self:getApp():enterScene(&#34;ApiRequest&#34;)
    end)

end

return MainScene
</code></pre><h3 id="网络应用场景-ApiRequest"><a href="#网络应用场景-ApiRequest" class="headerlink" title="网络应用场景(ApiRequest)"></a>网络应用场景(ApiRequest)</h3><p>然后开始处理跳转之后的ApiRequest，和相关请求逻辑，这里主要是使用我们封装好的CocosRequest实现基本上的请求逻辑，然后拿到数据之后我们就可以根据实际UI和具体业务逻辑做处理</p>
<blockquote>
<p>注意</p>
</blockquote>
<blockquote>
<p>测试的时候，将local url = “<a href="https://xxxx?cmd=501001&amp;uid=628941&amp;novelid=3782&#34;中的值换成自己的地址就可以" target="_blank" rel="noopener noreferrer">https://xxxx?cmd=501001&amp;uid=628941&amp;novelid=3782&#34;中的值换成自己的地址就可以</a></p>
</blockquote>
<pre><code>require &#34;json&#34;
local CocosRequest = require &#34;app.CocosRequest&#34;

--- @class ApiRequest
local ApiRequest = class(&#34;ApiRequest&#34;, cc.load(&#34;mvc&#34;).ViewBase)
----local MainScene = class(&#34;MainScene&#34;, function() return display.newScene(&#34;MainScene&#34;) end)

---onEnter
function ApiRequest:onEnter()
    print(&#34;onEnter&#34;)
end

-----onCreate
function ApiRequest:onCreate()

    ----------------------- 创建自定义事件 start
    local function eventCustomListener1(event)
        local str = &#34;response: &#34;..event._usedata
        --labelStatusCode:setString(str)

        -- 如果返回的是 json 数据，这里解析
        local data =  json.decode(event._usedata)
        table.foreach(data,
         function(key, var)
             print(&#34;-----&#34;..key)
             table.foreach(var,
                function(a, b)
                   print(a..&#34;-&#34;..b)
                end)
         end)
    end

    local listener1 = cc.EventListenerCustom:create(&#34;customEvent1&#34;,eventCustomListener1)
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
        local event = cc.EventCustom:new(&#34;customEvent1&#34;)
        event._usedata = xhr.response
        eventDispatcher:dispatchEvent(event)
        print(&#34;post callback code = &#34;..xhr.statusText)
    end

    local type = tmp.POST
    local url = &#34;https://xxxx?cmd=501001&amp;uid=628941&amp;novelid=3782&#34;
    local dataPost = {}
    dataPost.data = &#34;hello&#34;
    dataPost.aaa = &#34;world&#34;
    dataPost.bbb = &#34;yang&#34;
    tmp:send(type, url, dataPost, callback)

end

return ApiRequest
</code></pre><h3 id="网络请求封装-CocosRequest"><a href="#网络请求封装-CocosRequest" class="headerlink" title="网络请求封装(CocosRequest)"></a>网络请求封装(CocosRequest)</h3><p>最后才是我们的重头戏，CocosRequest是直接使用cocos2dx lua提供的XMLHttpRequest实现，其实就是做了一套逻辑，具体细节可以根据项目调整(此处已经测试通过，可直接拷贝使用)</p>
<pre><code>require &#34;json&#34;

CocosRequest = {}
CocosRequest.__index = CocosRequest
CocosRequest.instance = nil
CocosRequest.callback = nil
CocosRequest.POST = &#34;POST&#34;
CocosRequest.GET = &#34;GET&#34;

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
    if &#34;table&#34; ~= type(data) then
        print(&#34;data is not a table&#34;)
        return nil
    end

    local tmp = {}
    for key, value in pairs(data) do
        table.insert(tmp,key..&#34;=&#34;..value)
    end

    local newData = &#34;&#34;
    for i=1,#tmp do
        newData = newData..tostring(tmp[i])
        if i&lt;#tmp then
            newData = newData..&#34;&amp;&amp;&#34;
        end
    end
    print(&#34;------- name is &#34;..newData)
    return newData
end

-- 发送数据，参数：string，string，table
function CocosRequest:send(type, url, data, callback)
    local xhr = cc.XMLHttpRequest:new() --new 一个http request 实例
    self.callback = callback    --设置需要执行的函数

    local newData = self:dataParse(data)
    if nil == newData or &#34;&#34; == newData then
        return
    end

    -- response回调函数
    local function responseCallback()
        print(&#34;CocosRequest - &#34;..xhr.response)
        if nil ~= self.callback then
            self.callback(xhr)
        else
            print(&#34;callback is nil&#34;)
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
        xhr:open(self.GET, url..&#34;?&#34;..newData)
        xhr:send()
    else
        print(&#34;ERROR : type only can be &#34;Post&#34; or &#34;GET&#34;&#34;)
    end
end

---------------------

return CocosRequest
</code></pre><p>由于在Web中使用XMLHTTPRequest对象发出HTTP请求很普遍,Cocos2dx Lua对其进行了移植,可以在 Cocos2 d-x LumP中使用 XMLHTTPRequest对象</p>
<p>XMLHTTPRequest对象中几个常用的函数和属性如下</p>
<ul>
<li>(1)open(),与服务器连接,创建新的请求</li>
<li>(2)send(),向服务器发送请求</li>
<li>(3)abort(),退出当前请求</li>
<li>(4)readyState属性,提供当前请求的就绪状态,其中4表示准备就绪</li>
<li>(5)tatus属性,提供当前HTTP请求状态码,其中200表示成功请求</li>
<li>(6)respomseText属性,服务器返回的请求响应</li>
<li>(7) onreadystatechange属性。设置回调函数,当服务器处理完请求后就会自动调用该<br/>函数。</li>
</ul>
<p>其中open和send函数,以及onreadystatechange属性是HTTP请求的关键。open函<br/>数有以下5个参数可以使用</p>
<ul>
<li>(1) request-type:发送请求的类型。典型的值是GET或POST,也可以发送HEAD<br/>请求</li>
<li>(2) url:要请求连接的URL</li>
<li>(3) asynch:如果希望使用异步连接则为true,否则为 false。该参数是可选的,默认为</li>
<li>(4) username:如果需要身份验证,则可以在此指定用户名。该可选参数没有默认值</li>
<li>(5) password:如果需要身份验证,则可以在此指定口令。该可选参数没有默认值。</li>
</ul>
<h3 id="服务器响应"><a href="#服务器响应" class="headerlink" title="服务器响应"></a>服务器响应</h3><p>下面是我们验证返回后的服务器数据(未做处理)</p>
<pre><code>[
    {
        &#34;code&#34;:1,
        &#34;msg&#34;:&#34;操作成功&#34;,
        &#34;data&#34;:[
            {
                &#34;novelid&#34;:&#34;3782&#34;,
                &#34;uid&#34;:&#34;628875&#34;,
                &#34;cid&#34;:&#34;5&#34;,
                &#34;title&#34;:&#34;零下记忆&#34;,
                &#34;cover&#34;:&#34;http://xxxx/img/cb/2f/f1/cb2ff155bbda8387f4b8efe917c46cd3.jpg&#34;,
                &#34;scover&#34;:&#34;http://xxxx/img/cb/2f/f1/cb2ff155bbda8387f4b8efe917c46cd3.jpg&#34;,
                &#34;shareimg&#34;:&#34;http://xxxx/img/ed/68/e6/ed68e67f77841ac76be189f0a04aa030.jpg&#34;,
                &#34;intro&#34;:&#34;&#34;,
                &#34;tags&#34;:&#34;悬疑&#34;,
                &#34;reason&#34;:&#34;&#34;,
                &#34;issingle&#34;:&#34;1&#34;,
                &#34;sort&#34;:&#34;0&#34;,
                &#34;hits&#34;:&#34;0&#34;,
                &#34;reads&#34;:&#34;18753&#34;,
                &#34;clues&#34;:20,
                &#34;likes&#34;:&#34;1&#34;,
                &#34;unlikes&#34;:&#34;0&#34;,
                &#34;cmts&#34;:&#34;0&#34;,
                &#34;favs&#34;:&#34;2&#34;,
                &#34;words&#34;:&#34;18420&#34;,
                &#34;chapters&#34;:&#34;35&#34;,
                &#34;pub_chapters&#34;:&#34;35&#34;,
                &#34;chapter_index&#34;:&#34;1&#34;,
                &#34;pubtime&#34;:&#34;0&#34;,
                &#34;updatetime&#34;:&#34;1539337440&#34;,
                &#34;addtime&#34;:&#34;1534222233&#34;,
                &#34;resversion&#34;:&#34;30&#34;,
                &#34;isuser&#34;:&#34;0&#34;,
                &#34;status&#34;:&#34;1&#34;,
                &#34;wstatus&#34;:&#34;1&#34;,
                &#34;offsale&#34;:&#34;0&#34;,
                &#34;chapterstatus&#34;:&#34;-1&#34;,
                &#34;leadrole&#34;:{
                    &#34;roleid&#34;:&#34;8478&#34;,
                    &#34;rolename&#34;:&#34;炽念&#34;
                },
                &#34;cname&#34;:&#34;推理&#34;,
                &#34;isnew&#34;:0,
                &#34;fatime&#34;:&#34;8月14日&#34;,
                &#34;isreading&#34;:1,
                &#34;iscomplete&#34;:1,
                &#34;isfav&#34;:0,
                &#34;liketype&#34;:0,
                &#34;shareinfo&#34;:{
                    &#34;type&#34;:1,
                    &#34;title&#34;:&#34;零下记忆&#34;,
                    &#34;intro&#34;:&#34;&#34;,
                    &#34;img&#34;:&#34;http://xxxx/img/ed/68/e6/ed68e67f77841ac76be189f0a04aa030.jpg&#34;,
                    &#34;url&#34;:&#34;http://xxxx/novel/startReading?novelid=3782&amp;chl=jmt&#34;
                },
                &#34;userinfo&#34;:{
                    &#34;uid&#34;:&#34;628875&#34;,
                    &#34;username&#34;:&#34;writer10&#34;,
                    &#34;nickname&#34;:&#34;离经易道&#34;,
                    &#34;headurl&#34;:&#34;http://xxxx/headimg/bb/d3/b6/628875_1_bbd3b631717ad615f38a6670573a375c_300x300.jpg?v=1526708983&#34;,
                    &#34;sex&#34;:&#34;1&#34;,
                    &#34;vtype&#34;:&#34;0&#34;,
                    &#34;regtype&#34;:&#34;2&#34;,
                    &#34;vname&#34;:&#34;&#34;,
                    &#34;vicon&#34;:&#34;&#34;,
                    &#34;isfollow&#34;:0
                }
            }
        ],
        &#34;cmd&#34;:501001,
        &#34;time&#34;:1539677040
    }
]
</code></pre>