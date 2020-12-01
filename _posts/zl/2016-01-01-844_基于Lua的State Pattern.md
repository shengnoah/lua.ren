---
layout: post
title: 基于Lua的State Pattern 
tags: [lua文章]
categories: [topic]
---
代码来自于最近写的Pacman，更多请查看 – <https://github.com/bennychen/Moai-based-Pacman>

class.lua实现了在Lua中创建类的模拟，非常方便。class.lua参考自<http://lua-
users.org/wiki/SimpleLuaClasses>

    
    
    -- class.lua
    -- Compatible with Lua 5.1 (not 5.0).
    
    function class(base, init)
       local c = {}    -- a new class instance
       if not init and type(base) == 'function' then
          init = base
          base = nil
       elseif type(base) == 'table' then
        -- our new class is a shallow copy of the base class!
          for i,v in pairs(base) do
             c[i] = v
          end
          c._base = base
       end
       -- the class will be the metatable for all its objects,
       -- and they will look up their methods in it.
       c.__index = c
    
       -- expose a constructor which can be called by <classname>(<args>)
       local mt = {}
       mt.__call = function(class_tbl, ...)
       local obj = {}
       setmetatable(obj,c)
    
    -- below 2 lines are updated based on the Comments from 'http://lua-users.org/wiki/SimpleLuaClasses'
    --   if init then
    --      init(obj,...)
       if class_tbl.init then
          class_tbl.init(obj,...)
       else 
          -- make sure that any stuff from the base class is initialized!
          if base and base.init then
          base.init(obj, ...)
          end
       end
       return obj
       end
       c.init = init
       c.is_a = function(self, klass)
          local m = getmetatable(self)
          while m do 
             if m == klass then return true end
             m = m._base
          end
          return false
       end
       setmetatable(c, mt)
       return c
    end
    

State基类，包含三个stub函数，enter()和exit()分别在进入和退出state时被执行，onUpdate()函数将会在state被激活时的每帧被执行。

    
    
    require "class"
    
    State = class()
    
    function State:init( name )
    	self.name = name
    end
    
    function State:enter()
    end
    
    function State:onUpdate()
    end
    
    function State:exit()
    end
    

StateMachine类，该类集成了[Moai](http://getmoai.com)的MOAIThread类。MOAIThread类似于Lua中的coroutine，但是在Moai中被yield的MOAIThread，会在game
loop的每帧中被自动resume，见StateMachine:updateState函数，利用此特点，来实现每帧执行State:onUpdate函数。

    
    
    require "State"
    
    StateMachine = class()
    
    function StateMachine:init()
    	self.currentState = nil
    	self.lastState = nil
    end
    
    function StateMachine:run()
    	if ( self.mainThread == nil )
    	then
    		self.mainThread = MOAIThread.new()
    		self.mainThread:run( self.updateState, self )
    	end
    end
    
    function StateMachine:stop()
    	if ( self.mainThread )
    	then
    		self.mainThread:stop()
    	end
    end
    
    function StateMachine:setCurrentState( state )
    	if ( state and state:is_a( State ) )
    	then
    		if ( state == self.currentState )
    		then
    			print( "WARNING @ StateMachine::setCurrentState - " ..
    				   "var state [" .. state.name .. "] is the same as current state" )
    			return
    		end
    		self.lastState = self.currentState
    		self.currentState = state
    		if ( self.lastState )
    		then
    			print( "exiting state [" .. self.lastState.name .. "]" )
    			self.lastState:exit()
    		end
    		print( "entering state [" .. self.currentState.name .. "]" )
    		self.currentState:enter()
    	else
    		print( "ERROR @ StateMachine::setCurrentState - " ..
    			   "var [state] is not a class type of State" )
    	end
    end
    
    function StateMachine:updateState()
    	while ( true )
    	do
    		if ( self.currentState ~= nil )
    		then
    			self.currentState:onUpdate()
    		end
    		coroutine.yield()
    	end
    end
    

如何利用State和StateMachine类的示例，首先定义两个state。

SampleState.lua

    
    
    require "State"
    
    State1 = class( State ) 
    
    function State1:init()
    	State.init( self, "State1" )
    end
    
    function State1:enter()
    	self.i = 0
    end
    
    function State1:exit()
    	self.i = 0
    end
    
    function State1:onUpdate()
    	print( self.name .. " is updated" )
    	self.i = self.i + 1
    	print( "self.i=" .. self.i )
    	if ( self.i == 10 )
    	then
    		print( state2 )
    		SM:setCurrentState( state2 )
    		self.i = 0
    	end
    end
    
    -----------------------
    
    State2 = class( State ) 
    
    function State2:init()
    	State.init( self, "State2" )
    end
    
    function State2:onUpdate()
    	print( "State2 is updated" )
    end
    

test.lua

    
    
    require "StateMachine"
    require "SampleState"
    
    SM = StateMachine()
    SM:run()
    state1 = State1()
    state2 = State2()
    SM:setCurrentState( state1 )