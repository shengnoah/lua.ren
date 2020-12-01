---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 16 Object-oriented Programming

  * 对象和方法
    
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

    
        Account = {  
    	balance = 0,  
    	withdraw = function (self, v)  
    		self.balance = self.balance - v  
    	end  
    }  
    function (v)  
    	self.balance = self.balance + v  
    end  
    Account.deposit(Account, 200.00)  
    Account:withdraw(100.00)  
      
  
---|---  

在用冒号`:`而不是`.`创建或使用一个table的函数时，会自动生成或调用一个`self`指代table自身

  * 类，用`__index`来类的实例化
    
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
    

|

    
        function Account:new(o)  
    	o = o or {}   
    	setmetatable(o, self)  
    	self.__index = self  
    	return o  
    end  
    a = Account:new{balance = 0}  
    print(a.balance) --> 0  
    a:deposit(100.00)  
    print(a.balance) --> 100  
      
  
---|---  

初次调用`a.balance = a.balance +
v`，右侧的`balance`是类的默认值，右侧是对象的新域，之后再调用，右侧的`balance`为对象的域

  * 继承，子类可以在父类的基础上修改方法，修改后的方法是在子类自身的域
    
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
    

|

    
        SpecialAccount = Account:new()  
    s = SpecialAccount:new{limit = 1000.00}  
    function SpecialAccount:withdraw(v)  
    	if v - self.balance >= self:getLimit() then  
    		error"insufficient funds"  
    	end  
    	self.balance = self.balance - v  
    end  
    function SpecialAccount:getLimit()  
    	return self.limit or 0  
    end  
    print(s.balance) --> 0  
    s:deposit(100.00)  
    print(s.balance) --> 100  
    s:withdraw(200.00)  
    print(s.balance) --> 200  
      
  
---|---  
  * 多重继承
    
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
    

|

    
        -- look up for 'k' in list of tables 'plist'  
    local function search(k, plist)  
    	for i = 1, #plist do  
    		local v = plist[i][k] -- try 'i'-th superclass  
    		if v then return v end  
    	end  
    end  
    function createClass(...)  
    	local c = {} -- new class  
    	local parents = {...}  
    	-- class will search for each method in the list of its parents  
    	setmetatable(c, {__index = function (t,k)  
    		return search(k, parents)  
    	end})  
    	-- prepare 'c' to be the metatable of its instances  
    	c.__index = c  
    	-- define a new constructor for this new class  
    	function c:new(o)  
    		o = o or {}  
    		setmetatable(o, c)  
    		return o  
    	end  
    	return c -- return new class  
    end  
    Named = {}  
    function Named:getname()  
    	return self.name  
    end  
    function Named:setname(n)  
    	self.name = n  
    end  
    NamedAccount = createClass(Account, Named)  
    account = NamedAccount:new{name = "Paul"}  
    print(account:getname()) --> Paul  
    print(account)  
      
  
---|---  

用`createClass()`创造一个继承自两个父类的子类。为了减少`search()`在父类中搜索的复杂度，可以采用记忆化搜索，将搜索的结果保存在原table中  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    setmetatable(c, {__index = function (t, k)  
    	local v = search(k, parents)  
    	t[k] = v -- save for next access  
    	return v  
    end})  
      
  
---|---  
  
这样做的缺点：It is difficult to change method definitions after the system is
running, because these changes do not propagate down the hierarhy
chain.（说实话，我没看懂这句话……）

  * privacy，用局部变量和闭包构造私有成员，使得私有成员从外部无法直接获得，只能在内部获得
    
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
    

|

    
        function newAccount(initialBalance)  
    	local self = {  
    		balance = initialBalance,  
    		LIM = 10000.00,  
    	}  
    	local extra = function ()  
    		if self.balance > self.LIM then  
    			return self.balance * 0.10  
    		else  
    			return 0  
    		end  
    	end  
    	local withdraw = function (v)  
    		self.balance = self.balance - v  
    	end  
    	local deposit = function (v)  
    		self.balance = self.balance + v  
    	end  
    	local getBalance = function ()  
    		return self.balance + extra()  
    	end  
    	return {  
    		withdraw = withdraw,  
    		deposit = deposit,  
    		getBalance = getBalance  
    	}  
    end  
    acc1 = newAccount(100.00)  
    acc1.withdraw(110.00)  
    print(acc1.getBalance()) --> -10  
    acc1.deposit(50.00)  
    print(acc1.getBalance()) --> 40  
      
  
---|---  
  * single-method object，用闭包创建对象，无需创建一个接口table，根据参数调用不同的方法，没有继承，但是有私有性
    
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
    

|

    
        function newObject(value)  
    	return function (action, v)  
    		if action == "get" then return value  
    		elseif action == "set" then value = v  
    		else error("invalid action")  
    		end  
    	end  
    end  
      
    d = newObject(0)  
    print(d("get")) --> 0  
    d("set", 10)  
    print(d("get")) --> 10  
      
  
---|---