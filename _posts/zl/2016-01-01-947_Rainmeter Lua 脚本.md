---
layout: post
title: Rainmeter Lua 脚本 
tags: [lua文章]
categories: [topic]
---
用Rainmeter也快10年了，从乐此不疲得写RM项目到现在的桌面怎么简洁怎么来，期间也发布过不少下载量超六位数Skin，也没有为Rainmeter写过啥，今天写一下Lua和Rainmeter的使用，也是让大家能运用这个高级脚本写出更好的Skin。

Script（脚本）这个Measure已不是什么新的Measure了，但是几乎没有多少中使用到它。可能是它使用的脚本语言门槛比较高的缘故。Rainmeter官网中的介绍（English）：

> [Rainmeter 官方文档](http://rainmeter.net/RainCMS/?q=LuaForRainmeter)

Measure=Script Measure就类似于“Plugin”
Measure，可以拓展RM的功能。但脚本的编写却比Plugin（使用C++或C#编写的dll）的要简单得多。

在皮肤配置中，lua脚本语法如下：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    [MeasureLuaScript]  
    Measure=Script            
    ScriptFile=MyScript.lua  
    TableName=MyScriptTable   
    MySetting="SomeSetting"   
    UpdateDivider=1  
      
  
---|---  
  
我解释一下各项参数：

> Measure=Script  
> 标识 Measure 类型是 Script 扩展类型

> ScriptFile=MyScript.lua  
> 这个参数指定使用的脚本的路径，是必需的

> TableName=MyScriptTable  
> 这个参数可以是任意值，它是用来与其它的脚本Measure区别开来的，所以值是唯一的，也是必需的

> MySetting=”SomeSetting”  
> 这个参数不是必须的，它是用来向当前使用的Lua脚本的传递参数的，参数的名称与数量都要与Lua脚本中的表PROPERTIES相对应。

脚本中的内容：  
一个表：  
用来存放在皮肤中的变量，如上面的 MySetting  
PROPERTIES=  
{  
MySetting=””;  
}

> UpdateDivider=1  
> 更新时间

#### Lua必要函数

> function Initialize()  
> 初始化函数，皮肤刷新时，会调用这个函数

> function Update()  
> 皮肤每更新更新一次，都会调用这个函数，

> function GetStringValue()  
> function GetValue()  
> 这两个函数有且只能有一个，它的功能是返回字符串（GetStringValue）或数值（GetValue）给皮肤中调用该脚本的Measure

#### 实例

    
    
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

    
    
    [Rainmeter]   
    DynamicWindowSize=1   
    Update=1000   
      
    [MeasureLuaScript]   
    Measure=Script   
    ScriptFile=#CURRENTPATH#getinistring.lua  
    TableName=GetString  
    FilePath="#CURRENTPATH#timesetting.cfg"  
    secName="time1"  
    KeyName="h"  
    Defstr=" "  
      
    [MeterLua]   
    Meter=String   
    MeasureName=MeasureLuaScript   
    FontSize=12   
    FontColor=255,255,255,255  
    Solidcolor=0,0,0,100  
      
  
---|---  
      
    
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
    

|

    
    
    PROPERTIES =  
    {  
    filepath="";  
    secName="";  
    keyname="";  
    defstr="";  
    }  
      
    function Initialize()  
    FilePath =PROPERTIES.filepath;  
    Secstr=PROPERTIES.secName;  
    Keystr=PROPERTIES.keyname;  
    Defstr=PROPERTIES.Defstr;  
      
    end -- function Initialize  
      
    function Update()  
      
    StrVal=ReadIniFile(FilePath,Secstr,"H","0");  
    end -- function Update  
      
    function GetStringValue()  
    if not StrVal then StrVal="can not get anystring!" end ;  
    return StrVal;  
      
    end -- function GetStringValue  
      
    function ReadIniFile(filename,section,Key,default)  
            local gotsec=false;  
            local i,j=nil,nil;  
            local Keyvalue=nil;  
      
            if not filename or filename=="" then  
                    return "missing "filename"";  
      
            elseif not section or section=="" then  
                    return "missing "secName"";  
            elseif not Key or Key=="" then  
                    return "missing "keyName"";  
            end ;  
      
            section=string.lower(section);  
            Key=string.lower(Key);  
                    for tmp in io.lines(filename,r) do  
                    tmp=string.lower(tmp);  
                            if not gotsec then  
                                    i,j=string.find(tmp,section.."]");  
                                    if i then gotsec=true end;  
                            else  
                                    i,j,Keyvalue=string.find(tmp,Key.."%s*=%s*(.*)%s*");  
                                    --print(Keyvalue);  
                                    if i then break end;  
                            end;  
                    end;  
                    ----io.close(FileN);  
                    if not Keyvalue then Keyvalue=default end;  
            return Keyvalue;  
    end  
      
  
---|---  
  
实例中，通过获取 Rainmeter 配置文件，然后打印各项设置参数，