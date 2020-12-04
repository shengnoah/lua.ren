---
layout: post
title: 基于Lua的State Pattern 
tags: [lua文章]
categories: [topic]
---
<p>代码来自于最近写的Pacman，更多请查看 – <a href="https://github.com/bennychen/Moai-based-Pacman" target="_blank" rel="noopener noreferrer">https://github.com/bennychen/Moai-based-Pacman</a></p>

<p>class.lua实现了在Lua中创建类的模拟，非常方便。class.lua参考自<a href="http://lua-users.org/wiki/SimpleLuaClasses" target="_blank" rel="noopener noreferrer">http://lua-users.org/wiki/SimpleLuaClasses</a></p>

<pre class="brush: lua; collapse: true; light: false; title: ; toolbar: true; notranslate" title="">-- class.lua
-- Compatible with Lua 5.1 (not 5.0).

function class(base, init)
   local c = {}    -- a new class instance
   if not init and type(base) == &#39;function&#39; then
      init = base
      base = nil
   elseif type(base) == &#39;table&#39; then
    -- our new class is a shallow copy of the base class!
      for i,v in pairs(base) do
         c[i] = v
      end
      c._base = base
   end
   -- the class will be the metatable for all its objects,
   -- and they will look up their methods in it.
   c.__index = c

   -- expose a constructor which can be called by &lt;classname&gt;(&lt;args&gt;)
   local mt = {}
   mt.__call = function(class_tbl, ...)
   local obj = {}
   setmetatable(obj,c)

-- below 2 lines are updated based on the Comments from &#39;http://lua-users.org/wiki/SimpleLuaClasses&#39;
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
</pre>

<p>State基类，包含三个stub函数，enter()和exit()分别在进入和退出state时被执行，onUpdate()函数将会在state被激活时的每帧被执行。</p>

<pre class="brush: lua; title: ; notranslate" title="">require &#34;class&#34;

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
</pre>

<p>StateMachine类，该类集成了<a href="http://getmoai.com" target="_blank" rel="noopener noreferrer">Moai</a>的MOAIThread类。MOAIThread类似于Lua中的coroutine，但是在Moai中被yield的MOAIThread，会在game loop的每帧中被自动resume，见StateMachine:updateState函数，利用此特点，来实现每帧执行State:onUpdate函数。</p>

<pre class="brush: lua; title: ; notranslate" title="">require &#34;State&#34;

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
			print( &#34;WARNING @ StateMachine::setCurrentState - &#34; ..
				   &#34;var state [&#34; .. state.name .. &#34;] is the same as current state&#34; )
			return
		end
		self.lastState = self.currentState
		self.currentState = state
		if ( self.lastState )
		then
			print( &#34;exiting state [&#34; .. self.lastState.name .. &#34;]&#34; )
			self.lastState:exit()
		end
		print( &#34;entering state [&#34; .. self.currentState.name .. &#34;]&#34; )
		self.currentState:enter()
	else
		print( &#34;ERROR @ StateMachine::setCurrentState - &#34; ..
			   &#34;var [state] is not a class type of State&#34; )
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
</pre>

<p>如何利用State和StateMachine类的示例，首先定义两个state。</p>

<p>SampleState.lua</p>

<pre class="brush: lua; title: ; notranslate" title="">require &#34;State&#34;

State1 = class( State ) 

function State1:init()
	State.init( self, &#34;State1&#34; )
end

function State1:enter()
	self.i = 0
end

function State1:exit()
	self.i = 0
end

function State1:onUpdate()
	print( self.name .. &#34; is updated&#34; )
	self.i = self.i + 1
	print( &#34;self.i=&#34; .. self.i )
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
	State.init( self, &#34;State2&#34; )
end

function State2:onUpdate()
	print( &#34;State2 is updated&#34; )
end
</pre>

<p>test.lua</p>

<pre class="brush: lua; title: ; notranslate" title="">require &#34;StateMachine&#34;
require &#34;SampleState&#34;

SM = StateMachine()
SM:run()
state1 = State1()
state2 = State2()
SM:setCurrentState( state1 )
</pre>