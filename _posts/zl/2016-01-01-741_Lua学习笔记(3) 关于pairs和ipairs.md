---
layout: post
title: Lua学习笔记(3) 关于pairs和ipairs 
tags: [lua文章]
categories: [topic]
---
[TOC]

## pairs

遍历table

    
    
    local tbTestPairs ={
    	[1] = 1,
    	nTest_1 = 2,
    	szTest = "test",
    	tbTest = {},
    	nTest_2,
    }
    
    for k, v in pairs(tbTestPairs) do
    	print (k, v)
    end
    

结果

    
    
    szTest	test
    tbTest	table: 000000000033a630
    nTest_1	2
    

## ipairs

按顺序便利table

    
    
    local tbTestIpairs = {1, 3, 5, nil, 7}
    for k, v in ipairs(tbTestIpairs) do
    	print(k ,v)
    end
    

结果

    
    
    1	1
    2	3
    3	5
    

## 总结

  * pairs是无序的，ipairs是有序的。这点很重要，如果要按存储顺序处理表中内容的时候，pairs就可能得不到预期的结果；
  * ipairs遇到nil就停止打印，所以只打印出来前面三个，7没有打印出来
  * pairs中为值nil的元素，即使你定义了key是有效的，也是遍历不出来的。所以要小心对key有要求的应用！
  * 从上面的结果可以看到tbTestPairs中nTest_2没有打印， _这不是因为nTest_2这个key对应的值为nil_ ，而是没有定义key的元素在table里面从前往后依次从[1][2]…开始作为key的，所以nTest_2的key为[1],也覆盖了[1]=1，因此[1]=1也没有打印出来
  * 存储结构上pairs对应的是hash表，ipairs对应的是数组。这个很多帖子都有说明，看代码也更明白

# 理解层次

## 学习和理解

说到底，pairs和ipairs都是lua中的迭代器。迭代器是一种可以遍历一种结合中所有元素的机制[1]。可以看泛型for的语义：

    
    
    for <var-list> in <exp-list> do
    	<body>
    end
    

for在循环过程中保存了迭代器函数。for做的第一件事就是对in后面的表达式求值，这些表达式应该返回3个值供for保存（这里也就知道如果自己实现迭代器需要根据语法要求来写）：

  * 迭代器函数
  * 恒定状态
  * 控制变量

在初始化完成之后，for会以恒定状态和控制变量来调用迭代器函数。可以看下面的代码，更加清晰：

    
    
    for var_1, ..., var_n in <explist> do 
    	<block>
    end
    
    -- 等价于以下代码
    
    do
    	local _f, _s, _var = <explist>	-- 返回迭代器函数、恒定状态和控制变量的初值
    	while true do
    		local var_1, ..., var_n = _f(_s, _var)
    		_var = var_1
    		if _var == nil then
    			break
    		end
    		<block>
    	end
    end
    

参考[1]中专门对这个泛型for问题说的非常详细！感谢作者^_^

## 学以致用

  * pairs

    
    
    local function iter(a, k)
    	k, v = next(a, k);
    	if v then
    		return k, v;
    	end
    end
    
    function pairs(t)
    	return iter, t, nil;
    end
    

  * iparis

    
    
    local function iter(a, i)
    	i = i + 1;
    	if a[i] then
    		return i, a[i];
    	end
    end
    
    function ipairs(t)
    	return iter, t, 0;
    end
    

# 参考

  * [1][lua中的迭代器与泛型for](http://www.jellythink.com/archives/506)