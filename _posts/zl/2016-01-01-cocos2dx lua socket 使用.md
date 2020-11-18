---
layout: post
title: cocos2dx lua socket 使用 
tags: [lua文章]
categories: [lua文章]
---
从事cocos2dx也有好几年了,从开始的懵懂到现在（不知道如何评价自己，至少能把游戏做出来） 中途磕磕碰碰 想分享下tcp socket在lua中的使用
首先还是引入socket库和定义class和状态码 参考了quick的simple tcp 使用方法

    
    
    	local gameSocket = LuaTcpSocket:new():init()
    	local function onConnectStatus( code, msg )
    	
    	end
    	gameSocket:setConnectCallback(onConnectStatus)
    	gameSocket:connect(ip,port)
    

下面贴代码

    
    
    local scheduler = cc.Director:getInstance():getScheduler()
    local socket = require "socket"
    
    local SOCKET_CONNECT_FAIL_TIMEOUT = 3	-- socket failure timeout
    
    local LuaTcpSocket = class("LuaTcpSocket")
    
    local NET_STATUS = {
    	unconnected = 1,
    	connected = 2,
    	lostConnection = 3,
    }
    
    --初始化
    function LuaTcpSocket:init()
    	self.socket = nil
    	self.session = 0
    	self.tickScheduler = nil
    	self.status = NET_STATUS.unconnected
    	self.lastContent = ""
    	return self
    end
    -- 判断ipv6 苹果强烈要求
    function LuaTcpSocket:isIpv6( host )
    	local result = socket.dns.getaddrinfo(host)
    	local ipv6 = false
    	if result then
    		for k,v in pairs(result) do
    			if v.family == "inet6" then
    				ipv6 = true
    				break
    			end
    		end
    	end
    	return ipv6
    end
    
    -- 连接服务区,ip可以是域名,ipv6环境
    function LuaTcpSocket:connect( ip, port )
    	local v6 = self:isIpv6(ip)
    	self.lastIp = ip
    	self.lastPort = port
    
    	if v6 then
    		self.socket = socket.tcp6()
    	else
    		self.socket = socket.tcp()
    	end
    	self.socket:settimeout(0)--异步模式
    
    	self:checkConnect()
    end
    
    function LuaTcpSocket:realConnect()
    	local succ, status = self.socket:connect(self.lastIp, self.lastPort)
    	-- print("LuaTcpSocket.realConnect:", succ, status)
    	return succ == 1 or status == STATUS_ALREADY_CONNECTED
    end
    
    function LuaTcpSocket:checkConnect()
    	self:stopConnectScheduler()
    	self.waitConnect = 0
    
    	self.connectTimeTickScheduler = scheduler:scheduleScriptFunc(function ( dt )
    		self.waitConnect = self.waitConnect + dt
    		if self.waitConnect >= SOCKET_CONNECT_FAIL_TIMEOUT then
    			self.waitConnect = 0
    			self:close()
    			self:onConnectFail()
    			return
    		end
    		if self:realConnect() then
    			self:stopConnectScheduler()
    			self:onConnected()
    		end
    	end,0,false)
    end
    
    function LuaTcpSocket:stopConnectScheduler()
    	if self.connectTimeTickScheduler then
    		scheduler:unscheduleScriptEntry(self.connectTimeTickScheduler)
    	end
    	self.connectTimeTickScheduler = nil
    end
    
    function LuaTcpSocket:close()
    	self:disconnect(NET_STATUS.unconnected)
    end
    
    function LuaTcpSocket:isConnected()
    	return self.status == NET_STATUS.connected
    end
    
    function LuaTcpSocket:onConnectFail()
    	if self.connectCallback ~= nil then
    		self.connectCallback(-1,"time out")
    	end
    end
    
    function LuaTcpSocket:onConnected()
    	self.status = NET_STATUS.connected
    	self:tick()
    	if self.connectCallback ~= nil then
    		self.connectCallback(0,"sucess")
    	end
    end
    
    function LuaTcpSocket:setConnectCallback( connectCallback )
    	self.connectCallback = connectCallback
    end
    
    
    function LuaTcpSocket:tick()
    	self.tickScheduler = scheduler:scheduleScriptFunc(function ( dt )
    		if self.status == NET_STATUS.connected then
    			local chunck, status, partial = self.socket:receive("*a")
    			-- print("chunck",type(chunck),chunck) --nil	nil
    			-- print("status",type(status),status)  --string	timeout
    			-- print("partial",type(status),partial) --string  len=0
    
    
    			if status and status ~= "timeout" then
    				print("net status",status)
    				print("chunck",chunck)
    				print("partial",partial)
    				self:disconnect(NET_STATUS.lostConnection)
    				return
    			end
    
    			if ( chunck and #chunck == 0 ) or ( partial and #partial == 0 ) then
    				-- print("no data return 111")
    				return
    			end
    
    			if partial and #partial > 0 then
    				self.lastContent = self.lastContent .. partial
    			elseif chunck and #chunck > 0 then
    				self.lastContent = self.lastContent .. chunck
    			end
    			
    
    			local flag, remain, err = self:unpackage(self.lastContent)
    			-- print("flag,remain,err",flag,remain,err)
    			if flag then
    				self.lastContent = ""
    			else
    				if err ~= nil then
    					print("why happen????",err)
    					self:disconnect(NET_STATUS.lostConnection)
    					return
    				end
    				self.lastContent = remain
    			end
    		
    		else
    			-- print("error tick",self.status)
    		end
    
    	end,0,false)
    end
    
    -- return sucess, content, err
    -- 这里用自己的字节流实现包的解压缩,需要注意半包,多包
    function LuaTcpSocket:unpackage( content )
    end
    
    -- 发生数据,也是用字节流
    function LuaTcpSocket:send( data )
    	if not self:isConnected() then
    		self:disconnect(NET_STATUS.lostConnection)
    		return
    	end
    
    	local i, err = self.socket:send(data)
    	-- print("send,i err",i,err)
    
    	if err then
    		if err == "closed" then
    			self:disconnect(NET_STATUS.lostConnection)
    		end
    	end
    end
    
    
    function LuaTcpSocket:disconnect( status )
    	self.status = status
    	if self.socket then
    		self.socket:close()
    	end
    	self.socket = nil
    	self.lastContent = ""
    	self:stopTickScheduler()
    	self:stopConnectScheduler()
    end
    
    function LuaTcpSocket:stopTickScheduler()
    	if self.tickScheduler ~= nil then
    		scheduler:unscheduleScriptEntry(self.tickScheduler)
    	end
    	self.tickScheduler = nil
    end
    
    function LuaTcpSocket:reconnect()
    	return self:connect(self.lastIp,self.lastPort)
    end