---
layout: post
title: Vanilla （lua web framework）中文文档 [2018.09.19] 
tags: [lua文章]
categories: [topic]
---
## 0、前言

 _香草/Vanilla是一个基于Openresty实现的高性能Web应用开发框架._

![Vanilla](http://m1.sinaimg.cn/maxwidth.300/m1.sinaimg.cn/120d7329960e19cf073f264751e8d959_2043_2241.png)

**邮件列表**

  * vanilla-en [vanilla-en@googlegroups.com](mailto:vanilla-en@googlegroups.com)
  * vanilla-devel [vanilla-devel@googlegroups.com](mailto:vanilla-devel@googlegroups.com)
  * vanilla中文邮件列表 [vanilla@googlegroups.com](mailto:vanilla@googlegroups.com)

**推荐始终使用最新版的Vanilla**

_当前Vanilla最新版本0.1.0.rc6，支持命令：_

  * vanilla-0.1.0.rc6（ _你没看错，自0.1.0.rc5起，vanilla的命令行和框架代码都带着版本号，方便多版本共存，也方便框架升级_ ）
  * v-console-0.1.0.rc6

**特性**

  * 提供很多优良组件诸如：bootstrap、 router、 controllers、 models、 views。
  * 强劲的插件体系。
  * 多 Application 部署。
  * 多版本框架共存，支持便捷的框架升级。
  * 一键 nginx 配置、 应用部署。
  * 便捷的服务批量管理。
  * 你只需关注自身业务逻辑。

### 0.1、安装

    
    
    1  
    

|

    
    
    $ ./setup-framework -v $VANILLA_PROJ_ROOT -o $OPENRESTY_ROOT          
      
  
---|---  
  
### 0.2、快速开始

 **部署你的第一个Vanilla Application**

    
    
    1  
    

|

    
    
    $ ./setup-vanilal-demoapp  [-a $VANILLA_APP_ROOT -u $VANILLA_APP_USER -g $VANILLA_APP_GROUP -e $VANILLA_RUNNING_ENV]    #运行 ./setup-vanilal-demoapp -h 查看更多参数细节  
      
  
---|---  
  
**启动你的 Vanilla 服务**

    
    
    1  
    

|

    
    
    $ ./$VANILLA_APP_ROOT/va-appname-service start  
      
  
---|---  
  
**社区组织**

**QQ群 &&微信公众号**

  * _Openresty/Vanilla 开发 1 群：205773855_
  * _Openresty/Vanilla 开发 2 群：419191655_
  * _Openresty 技术交流 1 群：34782325_
  * _Openresty 技术交流 2 群：481213820_
  * _Openresty 技术交流 3 群：124613000_
  * _Vanilla开发微信公众号:Vanilla-OpenResty(Vanilla相关资讯、文档推送)_

## 1、快速上手

### 1.1、Hello World

#### 1.1.1、Vanilla 的安装

 **安装准备**

  1. 安装好 OpenResty
  2. Vanilla Github 地址：<https://github.com/idevz/vanilla>

**安装**

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    

|

    
    
    # 1.git clone 最新 Vanilla 版本（或者下载相应的 Vanilla release 版本）  
    git clone https://github.com/idevz/vanilla.git  
      
    # 2. 切换到 Vanilla 文件夹  
    cd vanilla  
      
    # 3.编译 vanilla： ./setup-framework -v $VANILLA_PROJ_ROOT -o $OPENRESTY_ROOT 其中 $VANILLA_PROJ_ROOT 为 vanilla 框架安装目录。 -o 为 openresty 安装目录  
      
    ./setup-framework -v /application/vanilla -o /application/openresty  
      
  
---|---  
  
_经过这 3 步如果没有报错，则安装 vanilla 成功_

**创建 vanilla 项目**

    
    
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
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    

|

    
    
    #1. 创建 vanilla 的运行用户  
      
    useradd -s /sbin/nologin -M nginx  
      
    id nginx # 可以查看到创建的用户  
      
    # 2、创建 vanilla 项目, -a 为 项目路径，-u 为执行用户 -g 为用户组 （在根目录 /home/webserver 下创建名为 cms 的项目）  
      
    ./setup-vanilla-demoapp -a /home/webserver/cms -u nginx -g nginx  
      
    # 3、删掉默认 Nginx 服务  
      
    pkill -9 nginx  
      
    # 4、切换到项目文件夹 编辑项目配置文件，改成你要的  
      
    cd /home/webserver/cms  
    cd nginx_conf  
    vim va-nginx.conf  
    vim va-nginx-development.conf  
      
    # 5、同步配置文件到运行目录  
      
    ./va-cms-service initconf dev -f #开发模式  
    ./va-cms-service initconf -f #生产模式  
      
      
    # 6、启动项目（2选1）  
      
    ./va-cms-service start dev  # 启动开发模式  
    ./va-cms-service start  # 启动生产模式  
      
  
---|---  
  
_服务启动后，开发环境默认启动在 9110 端口，<http://localhost:9110> 即可访问_

**vanilla 常用命令**

  1. 启动项目： `./va-cms-service start` 或者 `./va-orcms-service start dev`

  2. 重启项目 `./va-cms-service restart` 或者 `./va-orcms-service restart dev`

  3. 停止项目： `./va-cms-service stop` 或者 `./va-orcms-service stop dev`

  4. 创建配置文件 `./va-cms-service initconf dev -f`

### 1.2、如何调试

#### 1.2.1、Vanilla 的 调试

 _除了查看 nginx 错误日志辅助开发外，为了方便 Vanilla 项目的开发和调试，Vanilla 提供了诸如`print_r`
之类的对象输出方法，以及详细友好的页面报错输出，你不需要到服务器日志去查看，就能所见即所得的开发调试代码._

**sprint_r，print_r，lprint_r，err_log**

  * **sprint_r**

_将 LUA 对象等格式化为易读的字符串返回_

  * **print_r**

_类似`ngx.say` 的效果，将对象、变量等以易读的格式进行输出，适用于 Vanilla 开发的 Web 服务_

  * **lprint_r**

_`print_r` 的 CLI 版本，适用于 v-console 命令行环境_

    
    
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

    
    
    ╰─○ v-console-0.1.0.rc6  
    Lua 5.1.4  Copyright (C) 1994-2008 Lua.org, PUC-Rio  
    v-console>a={}  
    v-console>a.v1='a_v1'  
    v-console>a.v2='a_v2'  
    v-console>lprint_r(a)  
    {  
      v2 = "a_v2",  
      v1 = "a_v1"  
    }  
    v-console>  
      
  
---|---  
  
  * **err_log**

_err_log 方法是对`ngx.ERR` 的封装，将 `msg` 记录到 nginx 错误日志_

### 1.3、如何新增一个Controller

#### 1.3.1、Vanilla 的 controller

 _vanilla 的 controller 是业务处理的关键，vanilla 通过对 URI 的路由，找到本次请求对应的 controller 和
action。_

  * **最简单的 Controller**

_自动生成的 demo 中默认生成了 IndexController 和 index action（`function
IndexController:index()`），默认使用简单路由协议（`vanilla.v.routes.simple`）对 URI 进行路由_

    
    
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

    
    
    local IndexController = {}  
      
    -- curl http://localhost:9110  
    function ()  
        local view = self:getView()  
        local p = {}  
        p['vanilla'] = 'Welcome To Vanilla...' .. user_service:get()  
        p['zhoujing'] = 'Power by Openresty'  
        -- view:assign(p)  
        do return view:render('index/index.html', p) end  
        return view:display()  
    end  
      
    -- curl http://localhost:9110/index/action_b  
    function IndexController:action_b()  
        return 'index->action_b'  
    end  
      
    return IndexController  
      
  
---|---  
  
**以上代码解释**

_关于上面的 controller 实例代码，我们只需关注下面几点_

  * IndexController:index （index Controller 中的 index Action），通过 `self:getView()` 方法获取视图实例
  * 可以通过先调用 `view:assign(p)` 将所需要的参数传入视图，再调用 `view:display()` 进行模板渲染，或者可以直接调用 `view:render('index/index.html', p)` 方法，指定需要渲染的模板，并同时传入相应的参数
  * 模板参数都是与 LUA 数组的形式进行传递
  * 每个 action 的返回值都必须是字符串，所以可以知道 `view:display()` 和 `view:render()` 方法都是返回字符串
  * IndexController:action_b （index Controller 中的 action_b Action，这里注意，action 的方法名必须小写），使用默认的简单路由协议，访问 URI 为 `curl http://localhost:9110/index/action_b`

_注：目前 vanilla 所默认使用的模板引擎是 appo 老师开发的[resty-
template](https://github.com/bungle/lua-resty-template)，模板详细的使用文档请移步 appo
老师处参阅。_

#### 1.3.2、新添加一个 Controller

 _给 Vanilla 添加一个新的 Controller 非常简单，只需要在项目的 controllers 目录，实现一个 LUA
包，包导入的函数即为各个 action， 文件名与 controller 同名。例如添加一个名为 idevz 的 controller， 且实现一个名为
dohello 的 action（）。_

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    local IdevzController = {}  
    -- curl http://localhost:9110/idevz/dohello  
    function IdevzController:dohello()  
        return 'do-hello-action.'  
    end  
    return IdevzController  
      
  
---|---  
  
### 1.4、如何使用Models/Dao

#### 1.4.1、Vanilla 的 DAO

 _vanilla 的 DAO 预设为项目对数据源的封装，一切对数据源的操作都可以封装成 DAO，方便维护、管理、缓存等。 Vanilla 的 DAO
在项目的 models/dao 路径下，一般使用`LoadModel` 方法进行加载_

**最简单的 DAO**

_由自动生成的 demo 中默认生成了 TableDao，可以看出 TableDao 只是一个普通的 LUA 包。_

    
    
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
    20  
    21  
    

|

    
    
    local TableDao = {}  
      
    function TableDao:set(key, value)  
        self.__cache[key] = value  
        return true  
    end  
      
    function TableDao:new()  
        local instance = {  
            set = self.set,  
            __cache = {}  
        }  
        setmetatable(instance, TableDao)  
        return instance  
    end  
      
    function TableDao:__index(key)  
        local out = rawget(rawget(self, '__cache'), key)  
        if out then return out else return false end  
    end  
    return TableDao  
      
  
---|---  
  
**以上代码解释**

_DAO 可以是任何对数据层访问封装的 LUA 包，实现方式非常自由。_

### 1.5、如何使用Models/Service

#### 1.5.1、Vanilla 的 Service

 _vanilla 的 Service 预设为项目对某些通用业务逻辑封装为独立的 Service，方便维护、管理、缓存等。 Vanilla 的
Service 在项目的 models/service 路径下，一般使用`LoadModel` 方法进行加载_

**最简单的 Service**

_由自动生成的 demo 中默认生成了 UserService，可以看出 UserService 也只是一个普通的 LUA 包。不过 Service
一般调用更底层的 DAO ，并对之做必要封装，并将相关的 Service 暴露给 Controller 使用_

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    

|

    
    
    local table_dao = LoadApplication('models.dao.table'):new()  
    local UserService = {}  
      
    function UserService:get()  
        table_dao:set('zhou', 'UserService res')  
        return table_dao.zhou  
    end  
      
    return UserService  
      
  
---|---  
  
**以上代码解释**

_Service 可以是任何对数据层访问封装的 LUA 包_

## 2、APIs

### 2.1、配置

 _香草/Vanilla的配置由以下三个部分组成._

  * _App配置_
  * _Nginx配置_
  * _WAF配置_

#### 2.1.1、App配置

 **应用基础配置（config/application.lua）**

    
    
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

    
    
    Appconf.sysconf = {				--系统预加载配置文件  
        'v_resource',  
    }  
    Appconf.name = 'app_name'		--app名称，执行vanilla new命令时给定的应用名  
      
    Appconf.route='vanilla.v.routes.simple'		--路由器，指定URL路由方式，目的解析出需要执行的controller与action  
    Appconf.bootstrap='application.bootstrap'		--初始化bootstrap（用来对应用进行初始化操作）  
    Appconf.app={}		--app相关配置  
    Appconf.app.root='./'		--当前vanilla start命令执行路径  
      
    Appconf.controller={}		--当前app的controller相关配置  
    Appconf.controller.path=Appconf.app.root .. 'application/controllers/'		--controller文件所在路径（使用默认生成路径即可）  
      
    Appconf.view={}		--当前app的视图层相关配置  
    Appconf.view.path=Appconf.app.root .. 'application/views/'		--模板路径  
    Appconf.view.suffix='.html'		--模板后缀  
    Appconf.view.auto_render=true		--是否开启自动渲染  
      
  
---|---  
  
**应用基础配置的引用**

    
    
    1  
    2  
    

|

    
    
    -- 如上的配置，可以在代码中通过 Registry['APP_CONF'] 表来进行获取，比如获取 APP_NAME  
    local app_name = Registry['APP_CONF']['name']  
      
  
---|---  
  
**错误处理配置（config/errors.lua）**

_根据errors.lua文件中实例，配置用户级别错误码._

    
    
    1  
    2  
    3  
    

|

    
    
    local Errors = {  
        [1000] = { status = 500, message = "Controller Err." },  
    }  
      
  
---|---  
  
**Restful 路由协议配置（config/restful.lua）**

_根据 URI 需要来自定义路由协议的配置_

    
    
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

    
    
    local restful = {  
        v1={},  
        v={}  
    }  
      
    restful.v.GET = {  
        {pattern = '/', controller = 'index', action = 'index'},  
        {pattern = '/:category', controller = 'index', action = 'list'}  
    }  
      
    restful.v.POST = {  
        {pattern = '/post', controller = 'index', action = 'post'},  
    }  
      
    restful.v1.GET = {  
        {pattern = '/api', controller = 'index', action = 'api_get'},  
    }  
      
    return restful  
      
  
---|---  
  
**系统相关配置（sys/*）**

_比如DB、MC等资源配置，系统相关的分机房配置等（在某些大公司，这部分配置又运维人员统一管理和下发），文件格式目前使用相对更运维友好的 ini
文件，开发中可以方便的在 Registry[‘sys_conf’]
中获取相关数据，如`Registry['sys_conf']['cache']['lrucache']` 获取 lrucache 相关配置_

**系统缓存相关配置 （sys/cache）**

    
    
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
    20  
    21  
    22  
    

|

    
    
    [shared_dict]  
    dict=idevz  
    exptime=100  
      
    [memcached]  
    instances=127.0.0.1:11211 127.0.0.1:11211  
    exptime=60  
    timeout=100  
    poolsize=100  
    idletimeout=10000  
      
    [redis]  
    instances=127.0.0.1:6379 127.0.0.1:6379  
    exptime=60  
    timeout=100  
    poolsize=100  
    idletimeout=10000  
      
    [lrucache]  
    items=200  
    exptime=60  
    useffi=false  
      
  
---|---  
  
  * 目前这部分配置一般由 vanilla.v.libs.cache 来使用
  * 目前支持的配置项如 poolsize（连接池大小）、timeout（数据获取超时等）

**系统缓存相关配置 （sys/v_resource）**

    
    
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
    20  
    21  
    22  
    

|

    
    
    [mc]  
    conf=127.0.0.1:7348 127.0.0.1:11211  
      
    [redis]  
    conf=127.0.0.1:7348 127.0.0.1:<span cl