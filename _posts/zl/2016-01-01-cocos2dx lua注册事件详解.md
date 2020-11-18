---
layout: post
title: cocos2dx lua注册事件详解 
tags: [lua文章]
categories: [lua文章]
---
最近在学习cocos2dx lua的时候，遇到了一些关于事件注册的逻辑！

结合用户实际操作和游戏的真实需求，关于事件在游戏中还是使用非常多的，所以特此记录一下

> 事件(源自网络)
>

>>
事件是可以被控件识别的操作。如按下确定按钮，选择某个单选按钮或者复选框。每一种控件有自己可以识别的事件，如窗体的加载、单击、双击等事件，编辑框（文本框）的文本改变事件，等等。

###### 事件是用户对窗口上各种组件的操作。

  * 事件有系统事件和用户事件。
    * 1.系统事件由系统激发，如时间间隔24小时，银行储蓄的存款日期增加一天。
    * 2.用户事件由用户激发，如用户点击按钮，在文本框中显示特定的文本。事件驱动控件执行某项功能。

###### 触发事件的对象称为事件发送者；接收事件的对象称为事件接受者；

> 注: 这里只针对用户事件！

#### cocos2dx中事件的类型

  * registerScriptTouchHandler 注册触屏事件
  * registerScriptTapHandler 注册点击事件
  * registerScriptHandler 注册基本事件 包括 触屏 层的进入 退出 事件
  * registerScriptKeypadHandler 注册键盘事件
  * registerScriptAccelerateHandler 注册加速事件

在3.x之前事件的注册可以直接使用这些方式来注册，

  * 事件监听器主要有：
    * 触摸事件 : EventListenerTouchOneByOne、EventListenerTouchAllAtOnce
    * 鼠标响应事件 : EventListenerMouse
    * 键盘响应事件 : EventListenerKeyboard
    * 加速计事件 : EventListenerAcceleration
    * 自定义事件 : EventListenerCustom
    * 物理碰撞事件 : EventListenerPhysicsContact
    * 游戏手柄事件 : EventListenerController

而在3.x中由于加入了C++11的特性，而对事件的分发机制通过事件分发器EventDispatcher 来进行统一的管理。

> 官方说明：触摸事件，键盘事件，加速器事件和自定义事件等所有事件都由 EventDispatcher 分发。 TouchDispatcher,
> KeypadDispatcher, KeyboardDispatcher, AccelerometerDispatcher 已经被移除。

### 【事件分发器】

事件分发器EventDispatcher，用于统一管理事件监听器的所有事件的分发。

  * EventDispatcher 的特性主要有:

    * 事件的分发基于渲染顺序
    * 所有的事件都由 EventDispatcher 分发
    * 可以使用 EventDispatcher 来分发自定义事件
    * 可以注册一个 lambda 表达式作为回调函数

##### 1、_eventDispatcher

_eventDispatcher是Node的属性，通过Director::getInstance()->getEventDispatcher() 获得。

  * _eventDispatcher的工作由三部分组成：
    * （1）事件分发器 ：EventDispatcher。
    * （2）事件类型 ：EventTouch, EventKeyboard 等。
    * （3）事件监听器 ：EventListenerTouch, EventListenerKeyboard 等。

监听器实现了各种触发后的逻辑，在适当时候由事件分发器分发事件类型，然后调用相应类型的监听器。

##### 2、添加/删除监听器

  * 添加监听器：

    * addEventListenerWithSceneGraphPriority ，
    * addEventListenerWithFixedPriority 。
  * 删除监听器：

    * removeEventListener ，
    * removeAllEventListeners 。

##### 3、主要函数

包含监听器的添加、删除、暂停、恢复，优先级的设置，手动分发事件等。

    
    
    class EventDispatcher : public Ref
    {
    /**
    * 添加监听器
    *     - addEventListenerWithSceneGraphPriority
    *     - addEventListenerWithFixedPriority
    *     - addCustomEventListener
    */
    //使用 场景图的优先级 为指定事件添加一个监听.
    //listener : 指定要监听的事件.
    //node     : 这个节点的绘制顺序是基于监听优先级.
    //优先级   : 0
    void addEventListenerWithSceneGraphPriority(EventListener* listener, Node* node);
    
    //使用 一定的优先级 为指定事件添加一个监听.
    //listener      : 指定要监听的事件.
    //fixedPriority : 这个监听器的固定优先级.
    //优先级        : fixedPriority。(但是不能为0，因为他是场景图的基本优先级)
    void addEventListenerWithFixedPriority(EventListener* listener, int fixedPriority);
    
    //用户自定义监听器
    EventListenerCustom* addCustomEventListener(const std::string &eventName, const std::function<void(EventCustom*)>& callback);
    
    
    /**
    * 删除监听器
    *     - removeEventListener
    *     - removeEventListenersForType
    *     - removeEventListenersForTarget
    *     - removeCustomEventListeners
    *     - removeAllEventListeners
    */
    //删除指定监听器
    void removeEventListener(EventListener* listener);
    
    //删除某类型对应的所有监听器
    //EventListener::Type::
    //  单点触摸 : TOUCH_ONE_BY_ONE
    //  多点触摸 : TOUCH_ALL_AT_ONCE
    //  键盘     : KEYBOARD
    //  鼠标     : MOUSE
    //  加速计   : ACCELERATION
    //  自定义   : CUSTOM
    void removeEventListenersForType(EventListener::Type listenerType);
    
    //删除绑定在节点target上的所有监听器
    void removeEventListenersForTarget(Node* target, bool recursive = false);
    
    //删除名字为customEventName的所有自定义监听器
    void removeCustomEventListeners(const std::string& customEventName);
    
    //移除所有监听器
    void removeAllEventListeners();
    
    
    /**
    * 暂停、恢复在节点target上的所有监听器
    *     - pauseEventListenersForTarget
    *     - resumeEventListenersForTarget
    */
    void pauseEventListenersForTarget(Node* target, bool recursive = false);
    void resumeEventListenersForTarget(Node* target, bool recursive = false);
    
    
    /**
    * 其他
    *     - setPriority
    *     - setEnabled
    *     - dispatchEvent
    *     - dispatchCustomEvent
    */
    //设置某监听器的优先级
    void setPriority(EventListener* listener, int fixedPriority);
    
    //启用事件分发器
    void setEnabled(bool isEnabled);
    bool isEnabled() const;
    
    //手动派发自定义事件
    void dispatchEvent(Event* event);
    
    //给名字为eventName的自定义监听器, 绑定用户数据
    void dispatchCustomEvent(const std::string &eventName, void *optionalUserData = nullptr);
    }
    

##### 4、关于事件监听器的优先权

通过 addEventListenerWithSceneGraphPriority 添加的监听器，优先权为0。  
通过 addEventListenerWithFixedPriority 添加的监听器，可以自定义优先权，但不能为0。

  * 优先级越低，越先响应事件。
  * 如果优先级相同，则上层的（z轴）先接收触摸事件。

##### 5、使用步骤

  * （1）获取事件分发器：
    * dispatcher = Director::getInstance()->getEventDispatcher();
  * （2）创建监听器：
    * auto listener = EventListenerTouchOneByOne::create();
  * （3）绑定响应事件函数：
    * listener->onTouchBegan = CC_CALLBACK_2(callback, this);
  * （4）将监听器添加到事件分发器dispatcher中：
    * dispatcher->addEventListenerWithSceneGraphPriority(Listener, this);
  * （5）编写回调响应函数：
    * bool callback(Touch _touch, Event_ event) { … }

### 实战案例

先来看看项目用用到的一些简单时间的操作， 两种方式创建使用

##### 触摸事件

根据用户手机在屏幕触摸的位置，对场景或者场景中的精灵，控件的做一些处理，这种类型偏向于触摸屏的设备。

    
    
    local function onTouchBegan(touch, event)
    
    local location = touch:getLocation()
    local visiableSize = cc.Director:getInstance():getVisibleSize()
    local origin = cc.Director:getInstance():getVisibleOrigin()
    
    local finalX = location.x - (origin.x + visiableSize.width/2)
    local finalY = location.y - (origin.y + visiableSize.height/2)
    
    finalX, finalY 根据实际屏幕计算触摸点
    
    end
    
    local listener = cc.EventListenerTouchOneByOne:create()
    listener:registerScriptHandler(onTouchBegan, cc.Handler.EVENT_TOUCH_BEGAN)
    local eventDiapatcher = self:getEventDispatcher()
    eventDiapatcher:addEventListenerWithSceneGraphPriority(listener, self)
    

##### 键盘事件

这里是coco2dx 定义的一套键盘字节码，每一个键盘上的键都对应一个数字，我们可以根据用户按键对精灵和界面做控制，这种偏向于桌面版的游戏！

###### 方法一

    
    
    -- 键盘监听器
    local listener = cc.EventListenerKeyboard:create()
    
    listener:registerScriptHandler(function(keyCode, event)
    if self.tank ~= nil then
    -- w
    if keyCode == 146 then
    self.tank:MoveBegin("up")
    -- s
    elseif keyCode == 142 then
    self.tank:MoveBegin("down")
    -- a
    elseif keyCode == 124 then
    self.tank:MoveBegin("left")
    -- d
    elseif keyCode == 127 then
    self.tank:MoveBegin("right")
    end
    end
    end, cc.Handler.EVENT_KEYBOARD_PRESSED) --- cc.Handler.EVENT_KEYBOARD_RELEASED)
    local eventDiapatcher = self:getEventDispatcher()
    eventDiapatcher:addEventListenerWithSceneGraphPriority(listener, self)
    

###### 方法二

    
    
    local function keyboardPressed(keyCode, event)
    -- up
    if keyCode == 28 then
    self:MoveCursor(0, 1)
    -- down
    elseif keyCode == 29 then
    self:MoveCursor(0, -1)
    -- left
    elseif keyCode == 26 then
    self:MoveCursor(-1, 0)
    -- right
    elseif keyCode == 27 then
    self:MoveCursor(1, 0)
    -- page up
    elseif keyCode == 38 then
    self:SwitchCursor(-1)
    -- page down
    elseif keyCode == 44 then
    self:SwitchCursor(1)
    -- enter
    elseif keyCode == 38 then
    self:Place()
    -- delete
    elseif keyCode == 44 then
    self:Delete()
    -- F3
    elseif keyCode == 49 then
    self:Load()
    -- F4
    elseif keyCode == 50 then
    self:Save()
    end
    print("key board ???????? keyCode", keyCode)
    end
    
    -- 键盘监听器
    local listener = cc.EventListenerKeyboard:create()
    listener:registerScriptHandler(keyboardPressed, cc.Handler.EVENT_KEYBOARD_PRESSED)
    local eventDiapatcher = self:getEventDispatcher()
    eventDiapatcher:addEventListenerWithSceneGraphPriority(listener, self)
    

这里主要说一下codeKey，codeKey是cocos2dx定义的一套键盘的代码，每个平台几乎是通用的

### 【触摸事件】

##### 1、单点触摸：EventListenerTouchOneByOne

单点触摸监听器相关：

    
    
    static EventListenerTouchOneByOne* create();
    
    std::function<bool(Touch*, Event*)> onTouchBegan; //只有这个返回值为 bool
    std::function<void(Touch*, Event*)> onTouchMoved;
    std::function<void(Touch*, Event*)> onTouchEnded;
    std::function<void(Touch*, Event*)> onTouchCancelled;
    

使用举例：

    
    
    //获取事件分发器
    auto dispatcher = Director::getInstance()->getEventDispatcher();
    
    //创建单点触摸监听器 EventListenerTouchOneByOne
    auto touchListener = EventListenerTouchOneByOne::create();
    
    //单点触摸响应事件绑定
    touchListener->onTouchBegan     = CC_CALLBACK_2(HelloWorld::onTouchBegan, this);
    touchListener->onTouchMoved     = CC_CALLBACK_2(HelloWorld::onTouchMoved, this);
    touchListener->onTouchEnded     = CC_CALLBACK_2(HelloWorld::onTouchEnded, this);
    touchListener->onTouchCancelled = CC_CALLBACK_2(HelloWorld::onTouchCancelled, this);
    
    //在事件分发器中，添加触摸监听器，事件响应委托给 this 处理
    dispatcher->addEventListenerWithSceneGraphPriority(touchListener, this);
    
    
    //单点触摸事件响应函数
    bool onTouchBegan(Touch *touch, Event *unused_event)     { CCLOG("began"); return true; }
    void onTouchMoved(Touch *touch, Event *unused_event)     { CCLOG("moved"); }
    void onTouchEnded(Touch *touch, Event *unused_event)     { CCLOG("ended"); }
    void onTouchCancelled(Touch *touch, Event *unused_event) { CCLOG("cancelled"); }
    

##### 2、多点触摸：EventListenerTouchAllAtOnce

多点触摸监听器相关：

    
    
    static EventListenerTouchAllAtOnce* create();
    
    std::function<void(const std::vector<Touch*>&, Event*)> onTouchesBegan;
    std::function<void(const std::vector<Touch*>&, Event*)> onTouchesMoved;
    std::function<void(const std::vector<Touch*>&, Event*)> onTouchesEnded;
    std::function<void(const std::vector<Touch*>&, Event*)> onTouchesCancelled;
    

使用举例：

    
    
    //获取事件分发器
    auto dispatcher = Director::getInstance()->getEventDispatcher();
    
    //创建多点触摸监听器 EventListenerTouchAllAtOnce
    auto touchesListener = EventListenerTouchAllAtOnce::create();
    
    //多点触摸响应事件绑定
    touchesListener->onTouchesBegan     = CC_CALLBACK_2(HelloWorld::onTouchesBegan, this);
    touchesListener->onTouchesMoved     = CC_CALLBACK_2(HelloWorld::onTouchesMoved, this);
    touchesListener->onTouchesEnded     = CC_CALLBACK_2(HelloWorld::onTouchesEnded, this);
    touchesListener->onTouchesCancelled = CC_CALLBACK_2(HelloWorld::onTouchesCancelled, this);
    
    //在事件分发器中，添加触摸监听器，事件响应委托给 this 处理
    dispatcher->addEventListenerWithSceneGraphPriority(touchesListener, this);
    
    //多点触摸事件响应函数
    void onTouchesBegan(const std::vector<Touch*>& touches, Event *unused_event)    { CCLOG("began"); }
    void onTouchesMoved(const std::vector<Touch*>& touches, Event *unused_event)    { CCLOG("moved"); }
    void onTouchesEnded(const std::vector<Touch*>& touches, Event *unused_event)    { CCLOG("ended"); }
    void onTouchesCancelled(const std::vector<Touch*>&touches, Event *unused_event) { CCLOG("cancelled"); }
    

### 【鼠标事件】

EventListenerMouse，主要用于监听鼠标的点击、松开、移动、滚轮的事件。

鼠标事件监听器相关：

    
    
    static EventListenerMouse* create();
    
    std::function<void(Event* event)> onMouseDown;     //按下鼠标, 单击鼠标
    std::function<void(Event* event)> onMouseUp;   //松开鼠标, 按下的状态下松开
    std::function<void(Event* event)> onMouseMove;  //移动鼠标, 在屏幕中移动
    std::function<void(Event* event)> onMouseScroll;//滚动鼠标, 滚动鼠标的滚轮
    

使用举例：

    
    
    //获取事件分发器
    auto dispatcher = Director::getInstance()->getEventDispatcher();
    
    //创建鼠标事件监听器 EventListenerMouse
    EventListenerMouse* mouseListenter = EventListenerMouse::create();
    
    //鼠标事件响应函数
    mouseListenter->onMouseDown   = CC_CALLBACK_1(HelloWorld::onMouseDown,   this);
    mouseListenter->onMouseUp     = CC_CALLBACK_1(HelloWorld::onMouseUp,     this);
    mouseListenter->onMouseMove   = CC_CALLBACK_1(HelloWorld::onMouseMove,   this);
    mouseListenter->onMouseScroll = CC_CALLBACK_1(HelloWorld::onMouseScroll, this);
    
    //添加鼠标事件监听器，事件响应处理委托给this
    dispatcher->addEventListenerWithSceneGraphPriority(mouseListenter, this);
    
    //事件响应函数
    void onMouseDown(Event* event)   { CCLOG("Down"); }
    void onMouseUp(Event* event)     { CCLOG("UP"); }
    void onMouseMove(Event* event)   { CCLOG("MOVE"); }
    void onMouseScroll(Event* event) { CCLOG("Scroll"); }
    

### 【键盘事件】

EventListenerKeyboard，主要用于监听键盘某个键的按下、松开的事件。

键盘事件监听器相关：

    
    
    static EventListenerKeyboard* create();
    
    std::function<void(EventKeyboard::KeyCode, Event*)> onKeyPressed;  //按下某键
    std::function<void(EventKeyboard::KeyCode, Event*)> onKeyReleased; //松开某键
    
    //键盘按键枚举类型 EventKeyboard::KeyCode
    //KeyCode的值对应的不是键盘的键值、也不是ASCII码，只是纯粹的枚举类型
    //如:
    //  EventKeyboard::KeyCode::KEY_A
    //  EventKeyboard::KeyCode::KEY_1
    //  EventKeyboard::KeyCode::KEY_F1
    //  EventKeyboard::KeyCode::KEY_SPACE
    //  EventKeyboard::KeyCode::KEY_ALT
    //  EventKeyboard::KeyCode::KEY_SHIFT
    

使用举例：

    
    
    //
    //获取事件分发器
    auto dispatcher = Director::getInstance()->getEventDispatcher();
    
    //创建键盘按键事件监听器
    EventListenerKeyboard* keyboardListener = EventListenerKeyboard::create();
    
    //绑定事件响应函数
    keyboardListener->onKeyPressed = CC_CALLBACK_2(HelloWorld::onKeyPressed, this);
    keyboardListener->onKeyReleased = CC_CALLBACK_2(HelloWorld::onKeyReleased, this);
    
    //添加监听器
    dispatcher->addEventListenerWithSceneGraphPriority(keyboardListener, this);
    
    
    //事件响应函数
    void onKeyPressed(EventKeyboard::KeyCode keyCode, Event* event) {
    if (EventKeyboard::KeyCode::KEY_J == keyCode) {
    CCLOG("Pressed: J");
    }
    }
    void onKeyReleased(EventKeyboard::KeyCode keyCode, Event* event) {
    if (EventKeyboard::KeyCode::KEY_SPACE == keyCode) {
    CCLOG("Released: SPACE");
    }
    }
    

### 【加速计事件】

EventListenerAcceleration，主要用于监听移动设备的所受重力方向感应事件。

重力感应来自移动设备的加速计，通常支持 (X, Y, Z)
三个方向的加速度感应，所以又称为三向加速计。在实际应用中，可以根据3个方向的力度大小来计算手机倾斜的角度或方向。

##### 1、加速计信息类Acceleration

该类中每个方向的加速度，大小都为一个重力加速度大小。

    
    
    //加速计信息
    class Acceleration
    {
    double x; double y; double z;
    };
    

##### 2、开启加速计感应

在使用加速计事件监听器之前，需要先启用此硬件设备：

    
    
    Device::setAccelerometerEnabled(true);
    

##### 3、加速计监听器相关

    
    
    static EventListenerAcceleration* create(const std::function<void(Acceleration*, Event*)>& callback);
    
    std::function<void(Acceleration*, Event*)> onAccelerationEvent;
    

##### 4、使用举例

    
    
    //标签: 显示加速计信息
    label = Label::createWithTTF("no used", "Marker Felt.ttf", 12);
    label->setPosition(visibleSize / 2);
    this->addChild(label);
    
    
    //小球: 可视化加速计
    ball = Sprite::create("ball.png");
    ball->setPosition(visibleSize / 2);
    this->addChild(ball);
    
    
    //获取事件分发器
    auto dispatcher = Director::getInstance()->getEventDispatcher();
    
    //需要开启移动设备的加速计
    Device::setAccelerometerEnabled(true);
    
    //创建加速计事件监听器
    auto accelerationListener = EventListenerAcceleration::create(CC_CALLBACK_2(HelloWorld::onAccelerationEvent, this));
    
    //添加加速计监听器
    dispatcher->addEventListenerWithSceneGraphPriority(accelerationListener, this);
    
    
    //事件响应函数
    void HelloWorld::onAccelerationEvent(Acceleration* acceleration, Event* event)
    {
    char s[100];
    sprintf(s, "X: %f; Y: %f; Z:%f; ", acceleration->x, acceleration->y, acceleration->z);
    label->setString(s);
    
    //改变小球ball的位置
    float x = ball->getPositionX() + acceleration->x * 10;
    float y = ball->getPositionY() + acceleration->y * 10;
    Vec2 pos = Vec2(x, y);
    pos.clamp(ball->getContentSize() / 2, Vec2(288, 512) - ball->getContentSize() / 2);
    ball->setPosition(pos); //设置位置
    }
    

### 【自定义事件】

以上是系统自带的事件类型，事件由系统内部自动触发，如 触摸屏幕，键盘响应等。

EventListenerCustom 自定义事件，它不是由系统自动触发，而是人为的干涉。

它的出现，使得2.x中的 观察者模式 NotificationCenter（订阅发布消息） 被无情的遗弃了。

> 在 3.x 中，使用EventListenerCustom来实现消息的订阅与发布。

学习它之前，最好了解一下 NotificationCenter 这个类的用法。

> NotificationCenter 的用法参见：<http://shahdza.blog.51cto.com/2410787/1611575>

##### 1、创建自定义监听器

该监听器，就相当于是订阅消息。即与NotificationCenter的 addObserver 类似。

    
    
    //eventName : 监听器名字，即消息的名称
    //callback  : 监听器函数，即消息的回调函数
    static EventListenerCustom* create(const std::string& eventName, const std::function<void(EventCustom*)>& callback);
    

##### 2、分发自定义事件

自定义的事件监听器，需要通过手动的方式，将事件分发出去。

> 通过 EventCustom(string eventName); 来设置需要发布消息的数据信息，eventName为消息名称。

其中EventCustom可以通过setUserData来绑定想要传递的消息数据。

> 通过 dispatcher->dispatchEvent(&event); 来手动将事件分发出去。即发布消息。

这与NotificationCenter的 postNotification 类似。

    
    
    EventCustom event("custom_event");
    event->setUserData((void*)123); // 绑定消息传递的数据，可以为任意类型void。
    dispatcher->dispatchEvent(&event); // 发布名称为"custom_event"的消息。
    

##### 3、使用举例

    
    
    //获取事件分发器
    auto dispatcher = Director::getInstance()->getEventDispatcher();
    
    //创建自定义事件监听器
    //监听器名字  : "custom_event"
    //事件响应函数: HelloWorld::onCustomEvent
    auto customListener = EventListenerCustom::create("custom_event", CC_CALLBACK_1(HelloWorld::onCustomEvent, this));
    
    //添加自定义事件监听器，优先权为1
    dispatcher->addEventListenerWithFixedPriority(customListener, 1);
    
    
    //手动分发监听器的事件，通过dispatchEvent发布名称为custom_event的消息。
    EventCustom event = EventCustom("custom_event");
    event->setUserData((void*)123); // 绑定消息传递的数据，可以为任意类型void。
    dispatcher->dispatchEvent(&event);
    
    
    //消息事件回调函数
    void HelloWorld::onCustomEvent(EventCustom* event)
    {
    // 获取消息传递的数据
    int* data = (int*)event->getUserData()
    CCLOG("onCustomEvent data = %d", data);
    }
    

##### 4、说明

> 每个自定义的事件监听器，都有一个监听器名字eventName。即为订阅的消息名称。

> 需要通过 dispatcher->dispatchEvent(&event); 来手动将事件分发出去。即为发布消息。

> 可以通过 dispatcher->dispatchCustomEvent(,); 来给自定义事件监听器绑定一个用户数据。