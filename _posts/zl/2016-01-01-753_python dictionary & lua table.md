---
layout: post
title: python dictionary & lua table 
tags: [lua文章]
categories: [topic]
---
python dictionary 与 lua table

  

公司前端的技术栈是 C++ & lua & python, C++、lua做业务开发，python做工具。  
lua特别好用的东西就是load, 将某些数据（一般是excel）转成lua的table存储，用的时候直接load进来。  
但是，有些数据，并不是excel转过来的，通过多个excel来组成。

例如：  
描述: 一个卡牌游戏，肯定有一张卡牌表，在表中我可以知道每张卡牌属于哪个职业。

Question: 在游戏中，我想知道某职业所有卡牌，怎么办呢?  
我需要遍历这个卡牌表，然后把相应职业的卡牌筛选出来。  
但是，这个方法随着卡牌库的增加，所花费的时间也不断增加，早晚会出事呀。  
当然，可以通过各种人为规定，算法设计来加快速度。  
可是，都没有根本去解决问题。  
回过头来，可以发现，当我们有了这张卡牌表，我就可以知道某职业都有哪些牌。  
这就是像卡牌表一样，不需要在游戏中动态去生成，直接可以本地存储静态的一个职业对应卡牌的表。  
这样，即使生成这张表再慢，也不会消耗游戏运行中的时间和性能。

  

# 解决问题

## 分析

现在目标很明确了，我要把n个表，根据需求组合拼装成1个表。  
麻烦的是，表的生成这些东西，我们都已经做成脚本，交给策划生成了。  
这个东西，肯定也需要交给策划进行。  
不可能让策划打完表以后，程序再去执行一遍lua脚本。  
（PS：这种东西一定要规避掉，一定要学会自动化，工具化。不要把我们的时间浪费在这些可以自动化、工具化的东西上。）

所以，目前状况是：  
语言：python  
输入：n张lua表  
输出：1张lua表  
逻辑：一系列插入、查找、遍历等

中间的逻辑就不用多说了，python的dict 和 lua的table用法也挺像的。  
主要处理两个问题：

  1. 将lua table 转换成 python dict
  2. 将python dict 转换成 lua table

## 大概思路

在动手前，肯定要看有没有好的现成的轮子。重复造轮子还是不要了吧，不一定造的比人家好呢。  
经过一番查找，发现了 slpp[ <https://github.com/SirAnthony/slpp> ]  
它可以实现lua table 和 python dict之间的转换。  
但是，它是python2.x版本。  
然后，在推送请求上看到，有人已经做了python3.x版本。 slpp-23[
<https://github.com/IlyaSkriblovsky/slpp-23> ]

## 一些细节

大概的方向差不多了，剩下的就是符合我所需要的格式。  
因为，我的文件格式是这样的:

    
    
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

    
    
    local xx_xx = {  
    	[1] = {  
    		...  
    	},  
    	[2] = {  
    		...  
    	},  
    	...  
    }  
    return xx_xx  
      
  
---|---  
  
对于输入要求，还简单一些：

    
    
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

    
    
    """  
        描述:  
            将lua的table转成python的dict  
          
        参数:  
            table_file_name - string  
                文件名（需要加后缀）  
      
        返回:  
            dict格式的table  
    """  
    def (table_file_name):  
        content = ""  
        with open(table_file_name, 'r', encoding='utf-8') as f:  
            content = f.read()  
      
        content = content[content.find('{') : content.rfind('}')+1]  
        content = lua.decode(content)  
      
        return content  
      
  
---|---  
  
但是输出，就不能用 lua.encode 了，因为它输出东西是这样子的：

    
    
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
    54  
    55  
    56  
    57  
    58  
    59  
    60  
    61  
    62  
    63  
    64  
    65  
    66  
    67  
    68  
    69  
    70  
    71  
    72  
    73  
    

|

    
    
    {  
    {  
    	{  
    			z = 335,  
    			x = 1322,  
    			cityId = 2,  
    			id = 4  
    		},  
    	{  
    			z = 140,  
    			x = 3396,  
    			cityId = 4,  
    			id = 3  
    		}  
    	},  
    {  
    	{  
    			z = 267.9,  
    			x = 2639.4,  
    			cityId = 3,  
    			id = 3  
    		},  
    	{  
    			z = -1.2,  
    			x = 2006,  
    			cityId = 1,  
    			id = 8  
    		}  
    	},  
    {  
    	{  
    			cityId = 5,  
    			id = 6  
    		},  
    	{  
    			cityId = 2,  
    			id = 5  
    		}  
    	},  
    {  
    	{  
    			z = 122,  
    			x = 968,  
    			cityId = 5,  
    			id = 6  
    		},  
    	{  
    			cityId = 1,  
    			id = 5  
    		}  
    	},  
    {  
    	{  
    			cityId = 4,  
    			id = 5  
    		}  
    	},  
    {  
    		info = 	{  
    			123 = 123,  
    			age = 98,  
    			sex = "man",  
    			hobby = "basketball",  
    			other = 		{  
    				addr = "China",  
    				id_card = "1234567890abc"  
    			},  
    			name = "ltree98"  
    		},  
    		2 = "hello ltree98",  
    		id = "12345"  
    	}  
    }  
      
  
---|---  
  
没错，它的格式就是这样的，包括对齐。  
只能说能用。  
我想了一下，发现我需要的输出格式其实很简单，递归的输出就行。  
我也懒得改它的源码了，直接手撸了一个。

    
    
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
    

|

    
    
      
    def outPutTab(depth, outputHandler):  
        for i in range(0, depth):  
            outputHandler.write('t')  
      
    # 将字典递归转化输出  
    def RecursiveSerachDict(depth, dic, outputHandler):  
        for key, val in dic.items():  
            # output key  
            outPutTab(depth, outputHandler)  
      
            if isinstance(key, Number):     
                outputHandler.write('[%s] = ' % str(key))  
            elif isinstance(key, str):  
                outputHandler.write('%s = ' % key)  
      
            # output val  
            if isinstance(val, dict):  
                outputHandler.write('{n')  
                RecursiveSerachDict(depth+1, val, outputHandler)  
                outPutTab(depth, outputHandler)  
                outputHandler.write('},')  
            elif isinstance(val, Number):  
                outputHandler.write('%s,' % str(val))  
            elif isinstance(val, str):  
                outputHandler.write(''%s',' % val)  
      
            outputHandler.write('n')  
      
    # 将字典写入lua  
    def write_dict_lua(dict_tmp, table_name, file_path):  
        if(type(dict_tmp) != dict):  
            print("ERROR: only parse dict type!")  
            return  
      
        table_file_path = os.path.join(file_path, '%s.lua' % table_name)  
        outputHandler = codecs.open(table_file_path, 'w', encoding='utf-8')  
      
        outputHandler.write('nlocal %s = {n' % table_name)  
      
        RecursiveSerachDict(1, dict_tmp, outputHandler)  
      
        outputHandler.write('}n')  
        outputHandler.write('nreturn %snn' % table_name)  
        outputHandler.close()  
      
  
---|---  
  
我的输出：

    
    
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
    54  
    55  
    56  
    57  
    58  
    59  
    60  
    61  
    62  
    63  
    64  
    65  
    66  
    67  
    68  
    69  
    70  
    71  
    72  
    73  
    74  
    75  
    

|

    
    
    local dict_portal_group = {  
    	[1] = {  
    		[1] = {  
    			z = 335,  
    			x = 1322,  
    			cityId = 2,  
    			id = 4,  
    		},  
    		[2] = {  
    			z = 140,  
    			x = 3396,  
    			cityId = 4,  
    			id = 3,  
    		},  
    	},  
    	[2] = {  
    		[1] = {  
    			z = 267.9,  
    			x = 2639.4,  
    			cityId = 3,  
    			id = 3,  
    		},  
    		[2] = {  
    			z = -1.2,  
    			x = 2006,  
    			cityId = 1,  
    			id = 8,  
    		},  
    	},  
    	[3] = {  
    		[1] = {  
    			cityId = 5,  
    			id = 6,  
    		},  
    		[2] = {  
    			cityId = 2,  
    			id = 5,  
    		},  
    	},  
    	[4] = {  
    		[1] = {  
    			z = 122,  
    			x = 968,  
    			cityId = 5,  
    			id = 6,  
    		},  
    		[2] = {  
    			cityId = 1,  
    			id = 5,  
    		},  
    	},  
    	[5] = {  
    		[1] = {  
    			cityId = 4,  
    			id = 5,  
    		},  
    	},  
    	[6] = {  
    		info = {  
    			[123] = 123,  
    			age = 98,  
    			sex = 'man',  
    			hobby = 'basketball',  
    			other = {  
    				addr = 'China',  
    				id_card = '1234567890abc',  
    			},  
    			name = 'ltree98',  
    		},  
    		[2] = 'hello ltree98',  
    		id = '12345',  
    	},  
    }  
      
    return dict_portal_group  
      
  
---|---  
  
嗯，大概就是这样了。

  

# 总结

首先，就要遵循一个思路， **静态的数据结构尽量不要再游戏过程中动态生成** 。  
然后，前端要负责的东西太多了， **尽量要把这些东西工具化、脚本化** 。  
最后， **代码不止能用，还要能看** 。