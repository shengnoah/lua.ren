---
layout: post
title: skynet lua服务 
tags: [lua文章]
categories: [lua文章]
---
## C模块的导出

从skynet核心模块来看，它只认得C服务，每个服务被编译为动态库，在需要时由skynet加载。skynet提供发送消息和注册回调函数的接口，并保证消息的正确到达，并调用目标服务回调函数。其它东西，如消息调度，线程池等，对于用户来说都是透明的。

skynet服务可以由lua编写，因此skynet将C模块核心接口通过skynet/lualib-src/lua-skynet.c导出为
skynet.so提供给lua使用。在lua层，通过skynet/lualib/skynet.lua加载C模块(`require
"skynet.core"`)完成对C API的封装。主要涉及lua服务的加载和退出，消息的发送，回调函数的注册等。用户定义的lua服务通过`require
"skynet"`的接口即可完成服务的注册，启动和退出等。关于skynet lua api可以参见[skynet
wiki](https://github.com/cloudwu/skynet/wiki/LuaAPI "skynet wiki")。

skynet.lua 中，提供的比较重要的接口有：

    
    
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
    

|

    
    
    -- 注册特定类型消息的处理函数  
    function skynet.dispatch(typename, func)  
      
    -- 服务启动函数 在lua服务中调用该函数启动服务 并执行用户定义的start_func  
    function skynet.start(start_func)  
      
    -- 启动一个lua服务，name为lua脚本名字,返回服务地址  
    function skynet.newservice(name, ...)  
      
    -- 启动一个C服务，第一个参数为服务名字，后续为服务参数。返回服务地址  
    function skynet.launch(...)  
      
    -- 为服务地址映射一个全局名字	  
    function skynet.name(name, handle)  
      
    -- 向其它服务发送消息  
    function skynet.send(addr, typename, ...)  
      
    -- 同步发送消息 并阻塞等待回应	  
    function skynet.call(addr, typename, ...)  
      
  
---|---  
  
## lua服务如何关联到C核心层

下面主要提一下skynet是如何在这套C框架上承载lua服务的。

skynet 预置了一个C服务，叫snlua(位于skynet/service-
src/skynet_snlua.c)，这个服务的主要任务就是承载lua服务。一个snlua服务可以承载一个lua服务，可以启动任意份snlua服务。我们直接从snlua这个C服务开始，介绍一个lua服务是如何融合到C框架中的。当需要加载一个名为”console.lua””的服务时，我们将启动一个参数为”console”的snlua服务。主要流程：

  1. 调用skynet.launch(“sunlua”, “console”)
  2. skynet.launch对应C中的cmd_launch，它通过skynet_context_new加载snlua服务：

a.创建服务对应的skynet_context

b.加载snlua.so模块，并调用模块导出的snlua_create创建snlua服务，snlua_create会创建一个lua_State，这样每个lua服务拥有自己的lua_State。

c.创建服务消息队列，并为skynet_context绑定唯一handle，将消息队列放入全局消息队列中

d.调用snlua_init初始化服务，在snlua_init中，完成对snlua回调函数的注册。并且构造一条消息，将要加载的lua服务名(“console”)发给自己。

  1. 在snlua服务的消息回调函数中，先注销回调函数。然后通过加载并运行一个叫loader.lua的脚本，并解析收到的数据(“console”)来完成实际服务的加载。
  2. loader.lua在指定路径(可配置)下找到console.lua脚本，并执行 console.lua 脚本
  3. 此时回调函数就返回了。由于之前已经注销了snlua的回调函数。此时snlua看似”报废”了。而事实在重点在console.lua 当中：

每个skynet lua服务都需要有一个启动函数，通过调用 `skynet.start(function ... end
)`来启动lua服务。在skynet.start中：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    c = require "skynet.core"  
    function skynet.start(start_func)  
    	c.callback(dispatch_message)  
    	skynet.timeout(0, function()  
    		init_service(start_func)  
    	end)  
    end  
      
  
---|---  
  
通过c.callback注册了lua服务的回调函数dispatch_message，c.callback由skynet.so导出，它最终调用skynet_callback这个函数完成对本服务(当前是snlua)的回调函数注册。dispatch_message也定义于skynet.lua中，它主要的功能是根据消息类型(C层定义于skynet.h中，lua层定义于skynet.lua)将消息分发到lua服务指定的回调函数，前面提到过skynet.dispatch可以注册特定类型的处理函数。c.callback将dispatch_message注册为snlua新的回调函数。此时snlua这个服务就承载了lua服务，因为它收到的消息将通过dispath_message转发到lua服务注册的回调函数中。

那么c.callback如何将一个lua函数(dispatch_message)注册为一个C服务(snlua)的回调函数的呢？在skynet/lualib-
src/lua-skynet.c中，c.callback对应的C函数实现如下：

    
    
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

    
    
    static int  
    _callback(lua_State *L) {  
    	struct skynet_context * context = lua_touserdata(L, lua_upvalueindex(1));  
    	int forward = lua_toboolean(L, 2);  
    	luaL_checktype(L,1,LUA_TFUNCTION);  
    	lua_settop(L,1);  
    	lua_rawsetp(L, LUA_REGISTRYINDEX, _cb);  
      
    	lua_rawgeti(L, LUA_REGISTRYINDEX, LUA_RIDX_MAINTHREAD);  
    	lua_State *gL = lua_tothread(L,-1);  
      
    	if (forward) {  
    		skynet_callback(context, gL, forward_cb);  
    	} else {  
    		skynet_callback(context, gL, _cb);  
    	}  
      
    	return 0;  
    }  
      
  
---|---  
  
_callback将lua服务消息回调dispatch_message以_cb函数地址为key保存到lua注册表中。再将_cb函数作为lua服务的”代理回调函数”注册到C核心框架中。这样真正的回调函数_cb就能够满足C服务回调函数形式。这里C中的_cb和lua中的dispatch_message都是预先定义好的，可以通过lua全局注册表做一一映射。

当消息到达snlua时，在_cb中，通过`lua_rawgetp(L, LUA_REGISTRYINDEX,
_cb);`从lua注册表中取出lua服务的真正回调函数dispatch_message，压入消息参数。然后调用dispatch_message。dispatch_message根据消息类型将消息分到到lua服务注册的回调函数中。

总结一下，snlua帮lua服务做了如下工作：

  * 创建服务上下文skynet_context
  * 创建lua_State
  * 分配并绑定唯一handle
  * 创建服务消息队列
  * 执行指定lua服务脚本

在最后一步中，lua服务脚本会通过skynet.start启动服务，后者通过c.callback完成回调函数的替换。之后snlua便成功代理了lua服务，它收到的消息都会转发给lua层的dispatch_message。

## launcher服务

skynet中所有的lua服务都是通过snlua来承载的，skynet提供了一个lua服务launcher.lua(skynet/service/下)专门用来启动其它lua服务，launcer服务本身通过skynet.launch(“snlua”,
“launcher”)来创建，而其它的lua服务则更推荐使用skynet.newservice(“console”)来启动：

    
    
    1  
    2  
    3  
    

|

    
    
    function skynet.newservice(name, ...)  
    	return skynet.call(".launcher", "lua" , "LAUNCH", "snlua", name, ...)  
    end  
      
  
---|---  
  
根据前面skynet.call的原型，skynet.call向名为”.launcher”的服务发送一条类型为”lua”的消息，之后的参数便是消息数据，一般来说，消息的第一个字段代表命令，如这里向”.launcher”服务发送了一个”LAUNCH”命令。launcher.lua的实现比较简单，通过它也能了解lua服务的惯用写法。因此这里我摘录了部分重要代码：

    
    
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
    32  
    33  
    34  
    35  
    36  
    37  
    38  
    39  
    40  
    41  
    42  
    43  
    44  
    45  
    46  
    47  
    48  
    49  
    50  
    51  
    52  
    53  
    

|

    
    
    local services = {} -- 记录各lua服务的启动时参数  
    local command = {} -- 保存各命令对应的处理函数  
    local instance = {} -- for confirm (function command.LAUNCH / command.ERROR / command.LAUNCHOK)  
      
    -- 通过skynet.launch完成服务的加载 并返回服务地址  
    local function launch_service(service, ...)  
    	local param = table.concat({...}, " ")  
    	local inst = skynet.launch(service, param)  
    	local response = skynet.response()  
    	if inst then  
    		services[inst] = service .. " " .. param  
    		instance[inst] = response  
    	else  
    		response(false)  
    		return  
    	end  
    	return inst  
    end  
      
    -- 加载lua服务  
    function command.LAUNCH(_, service, ...)  
    	launch_service(service, ...)  
    	return NORET  
    end  
      
    -- lua服务加载完成 通常在skynet.start完成服务初始化后 发送该命令通知launcher  
    function command.LAUNCHOK(address)  
    	-- init notice  
    	local response = instance[address]  
    	if response then  
    		response(true, address)  
    		instance[address] = nil  
    	end  
      
    	return NORET  
    end  
      
    -- 注册"lua"类型(对应C中的type字段为PTYPE_LUA)消息的回调函数  
    skynet.dispatch("lua", function(session, address, cmd , ...)  
    	cmd = string.upper(cmd)  
    	local f = command[cmd]  
    	if f then  
    		local ret = f(address, ...)  
    		if ret ~= NORET then  
    			skynet.ret(skynet.pack(ret))  
    		end  
    	else  
    		skynet.ret(skynet.pack {"Unknown command"} )  
    	end  
    end)  
      
    -- lua服务启动函数  
    skynet.start(function() end)  
      
  
---|---  
  
这样lua服务的启动通过launcher服务添加一层沙盒，更加安全。launcher还会记录服务的加载状态，输出日志等。launcher一般在bootstrap.lua中创建，并且为其命名”.launcher”。bootstrap.lua是skynet启动执行的第一个lua脚本。