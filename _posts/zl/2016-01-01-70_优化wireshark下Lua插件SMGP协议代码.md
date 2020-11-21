---
layout: post
title: 优化wireshark下Lua插件SMGP协议代码 
tags: [lua文章]
categories: [topic]
---
@(Linux)[学习]

[toc]

## 写在前面

由于wireshark对SMGP协议不支持，之前使用wireshark查看SMGP协议字段，一般都是拷贝下来HEX，一个字节一个字节对。眼睛都要花了！  
昨天检查问题的时候同事和我说wireshark可以支持Lua插件自己编写协议识别代码，然后给了我一个SMGP.Lua文件。我安装了一下，发现是可以识别，但是如果一个包里有多个协议包的时候只识别一个，于是决定优化一下  

### 2016-12-29修改

有时候包经过跳转，则端口不是标准的8890了，则可以在后面添加一行代码：  

    
    
    1
    
    2

|

    
    
    tcp_table:add(8890,p_SMGP)
    
    tcp_table:add(38080,p_SMGP)  
  
---|---  
  
这样就可以识别8890和38080了。想要识别什么就添加什么。很方便

## 前期准备

### Lua方面

首先，我对Lua语言之前没有接触过，不过语言大都差不多。而且我的修改比较简单，应该就是加一个控制循环语句和判断。先去看了一下Lua一些基本语法.  
`lua`的循环和`shell`差不多  

    
    
    1
    
    2
    
    3

|

    
    
    while a do
    
    	pass
    
    end  
  
---|---  
  
### sireshark方面

根据官方给的实例，搞懂基本流程  
<https://www.wireshark.org/docs/wsdg_html_chunked/lua_module_Pinfo.html#lua_class_Column>

<https://www.wireshark.org/docs/wsdg_html_chunked/PartDevelopment.html>  
要改哪里就去搜索开发文档

## 主要修改地方

### 找到主函数

在主函数下添加根据字长是否还有剩余，进行判断，如果还有数据，则接着执行  

    
    
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

    
    
    function (buf,pkt,root)
    
     	
    
     	-- else
    
     	-- 	data_dis:call(buf,pkt,root)
    
     	--将下面那段替换上面的那段
    
     	--从这里开始
    
      local buf_len = buf:len()
    
      local p_length = SMGP_dissector(buf,pkt,root)	--p_length表示匹配到的有效协议包字段长度
    
      local shengyu_length = buf_len-p_length	--表示剩余需要匹配的长度
    
      ---------如果剩余长度大于8字节，则接着匹配
    
      while shengyu_length > 8  do 
    
      	p_length = SMGP_dissector(buf(p_length,shengyu_length),pkt,root)
    
      	shengyu_length = shengyu_length-p_length
    
      end
    
      if shengyu_length == 0 then
    
     --
    
      else
    
       data_dis:call(buf(p_length,shengyu_length),pkt,root)
    
       --到这里结束
    
      end
    
     end  
  
---|---  
  
修改上面后还要修改返回值，之前函数是返回布尔型的，让他返回有效长度，修改SMGP_dissector函数  

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8

|

    
    
     local function SMGP_dissector(buf,pkt,root)
    
      local buf_len = buf:len();
    
    --if buf_len < 8 then return false end
    
      if buf_len < 8 then return 0 end	--修改返回值为字段长度，0表示有效协议字段长度为0
    
      -----省略-----
    
      --return true
    
      return v_length  --修改返回值
    
      end  
  
---|---  
  
### 初步完工

修改上面之后，基本上可以识别了多个协议包了  
但是我发现submit包里面本来是有一个8字节的保留字段的，这个字段主要是用于长短信设置udhi字段的。而这个代码里面没有，所以可以添加。  

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7

|

    
    
    local f_RecvTime = ProtoField.string("SMGP.RecvTime","RecvTime")
    
    --添加定义保留字段
    
     local f_Reserve = ProtoField.uint32("SMGP.Reserve","Reserve",base.HEX,{
    
     	[0x0001]="TP_pid",[0x0002]="TP_udhi"--,
    
     --	[0x0003]=""
    
     	})
    
     local f_ReserveValue = ProtoField.uint32("SMGP.ReserveValue","ReserveValue",base.HEX)  
  
---|---  
  
先在前面一大段定义后面添加一个定义然后在`p_SMGP.fields`字段后面添加添加的`f_RecvTime`

> 如果要完善，`f_ReserveValue`字段应该是定义函数的这里偷了个懒，以后可能添加

然后在`local function SMGP_Submit(buf,pkt,t)`函数下面添加  

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6

|

    
    
    local function SMGP_Submit(buf,pkt,t)
    
    ---省略---
    
    --添加调用保留字段
    
      t:add(f_Reserve,buf(139+v_msgLen,2))
    
      t:add(f_ReserveValue,buf(141+v_msgLen,6))
    
     end  
  
---|---  
  
### 最后一点

在高亮部分的info里面显示里面的包的协议类型  
这个我找了一上午，主要是要找怎样添加字段，而之前的用`set()`函数是重置  
一般是用`append()`函数，但是，这个是添加到字段尾部，我想添加到头部google了很久，英文的又看不懂。。。最后还是在官方api里找到了，用`preend()`函数  
 ~~我也可以说我是看英文官方文档的人了~~  
再各个协议包字段函数里面添加对应的显示字段  

    
    
    1

|

    
    
    pkt.cols.info:prepend("SMGP_Submit" .. " ")  
  
---|---  
  
顺便添加了一下`Active_Test`和`Active_TestResp`的显示  
到这里，基本上已经改好了，可以适用于大多数的SMGP协议包了。之后有可能进行修改吧。

### 安装

既然这么好，那么，哪里可以用上呢？  
先升级wireshark，我也忘了哪个版本支持Lua了，看帮助吧，里面有。  
然后在安装目录里面的`init.lua`文件最后添加一行  

    
    
    1

|

    
    
    dofile(DATA_DIR.."SMGP.lua")  
  
---|---  
  
然后  
将后面的代码复制，粘贴到一个新文件`SMGP.lua`,放到`init.lua`同级目录就可以用了  
![展示成果](https://summingyu.github.io/./1480672127792.png)

## 修改后代码

    
    
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
    
    76
    
    77
    
    78
    
    79
    
    80
    
    81
    
    82
    
    83
    
    84
    
    85
    
    86
    
    87
    
    88
    
    89
    
    90
    
    91
    
    92
    
    93
    
    94
    
    95
    
    96
    
    97
    
    98
    
    99
    
    100
    
    101
    
    102
    
    103
    
    104
    
    105
    
    106
    
    107
    
    108
    
    109
    
    110
    
    111
    
    112
    
    113
    
    114
    
    115
    
    116
    
    117
    
    118
    
    119
    
    120
    
    121
    
    122
    
    123
    
    124
    
    125
    
    126
    
    127
    
    128
    
    129
    
    130
    
    131
    
    132
    
    133
    
    134
    
    135
    
    136
    
    137
    
    138
    
    139
    
    140
    
    141
    
    142
    
    143
    
    144
    
    145
    
    146
    
    147
    
    148
    
    149
    
    150
    
    151
    
    152
    
    153
    
    154
    
    155
    
    156
    
    157
    
    158
    
    159
    
    160
    
    161
    
    162
    
    163
    
    164
    
    165
    
    166
    
    167
    
    168
    
    169
    
    170
    
    171
    
    172
    
    173
    
    174
    
    175
    
    176
    
    177
    
    178
    
    179
    
    180
    
    181
    
    182
    
    183
    
    184
    
    185
    
    186
    
    187
    
    188
    
    189
    
    190
    
    191
    
    192
    
    193
    
    194
    
    195
    
    196
    
    197
    
    198
    
    199
    
    200
    
    201
    
    202
    
    203
    
    204
    
    205
    
    206
    
    207
    
    208
    
    209

|

    
    
    -- SMGP.lua
    
    -- SMGP3.0 protocol
    
    -- author: summingyu
    
    --基于h_Davy代码
    
    ----https://www.wireshark.org/docs/wsdg_html_chunked/lua_module_Pinfo.html#lua_class_Column
    
    --https://www.wireshark.org/docs/wsdg_html_chunked/PartDevelopment.html
    
    do
    
     local p_SMGP = Proto("SMGP","SMGP","SMGP Protocol")
    
     local f_Length = ProtoField.uint32("SMGP.length","Packet Length",base.DEC)
    
     local f_CommandId = ProtoField.uint32("SMGP.RequestId","Request ID",base.HEX,{
    
      [1]="Login",      [0x80000001]="LoginResp",
    
      [2]="Submit",     [0x80000002]="SubmitResp",
    
      [3]="Deliver",    [0x80000003]="DeliverResp",
    
      [4]="ActiveTest", [0x80000004]="ActiveTestResp",
    
      [5]="Forward",    [0x80000005]="ForwardResp",
    
      [6]="Exit",       [0x80000006]="ExitResp",
    
      [7]="Query",      [0x80000007]="QueryResp"})
    
     local f_SequenceId = ProtoField.uint32("SMGP.sequenceId","Sequence ID",base.DEC);
    
     local f_ClientID = ProtoField.string("SMGP.ClientID","ClientID")
    
     local f_Authenticator = ProtoField.bytes("SMGP.Authenticator","Authenticator")
    
     local f_LoginMode = ProtoField.uint8("SMGP.LoginMode","Login Mode",base.DEC)
    
     local f_TimeStamp = ProtoField.uint32("SMGP.TimeStamp","TimeStamp",base.DEC)
    
     local f_Version = ProtoField.uint8("SMGP.Version","Version",base.HEX)
    
     local f_Status = ProtoField.uint32("SMGP.Status","Status",base.DEC,{[0]="OK"})
    
     local f_MsgType = ProtoField.uint32("SMGP.MsgType","MsgType",base.DEC,{[0]="MO",[6]="MT",[7]="P2P"})
    
     local f_NeedReport = ProtoField.uint32("SMGP.NeedReport","NeedReport",base.DEC,{[0]="N",[1]="Y"})
    
     local f_Priority = ProtoField.uint32("SMGP.Priority","Priority",base.DEC)
    
     local f_ServiceID = ProtoField.string("SMGP.ServiceID","ServiceID")
    
     local f_FeeType = ProtoField.string("SMGP.FeeType","FeeType")
    
     local f_FeeCode = ProtoField.string("SMGP.FeeCode","FeeCode")
    
     local f_FixedFee = ProtoField.string("SMGP.FixedFee","FixedFee")
    
     local f_MsgFormat = ProtoField.uint32("SMGP.MsgFormat","MsgFormat",base.DEC,{
    
      [0]="ASCII",[3]="Card",[4]="Binary",[8]="UCS2",[15]="GB18030",[246]="SIM"})
    
     local f_ValidTime = ProtoField.string("SMGP.ValidTime","ValidTime")
    
     local f_AtTime = ProtoField.string("SMGP.AtTime","AtTime")
    
     local f_SrcTermID = ProtoField.string("SMGP.SrcTermID","SrcTermID")
    
     local f_ChargeTermID = ProtoField.string("SMGP.ChargeTermID","ChargeTermID")
    
     local f_DestTermIDCount = ProtoField.uint32("SMGP.DestTermIDCount","DestTermIDCount",base.DEC)
    
     local f_DestTermID = ProtoField.string("SMGP.DestTermID","DestTermID")
    
     local f_MsgLength = ProtoField.uint8("SMGP.MsgLength","MsgLength",base.DEC)
    
     local f_MsgContent = ProtoField.string("SMGP.MsgContent","MsgContent")
    
     local f_MsgID = ProtoField.bytes("SMGP.MsgID","MsgID")
    
     local f_IsReport = ProtoField.uint8("SMGP.IsReport","IsReport",base.DEC)
    
     local f_RecvTime = ProtoField.string("SMGP.RecvTime","RecvTime")
    
     --添加定义保留字段
    
     local f_Reserve = ProtoField.uint32("SMGP.Reserve","Reserve",base.HEX,{
    
      [0x0001]="TP_pid",[0x0002]="TP_udhi"--,
    
     -- [0x0003]=""
    
      })
    
     local f_ReserveValue = ProtoField.uint32("SMGP.ReserveValue","ReserveValue",base.HEX)
    
     --
    
     p_SMGP.fields = {f_Length,f_CommandId,f_SequenceId,f_ClientID,f_Authenticator,
    
     f_LoginMode,f_TimeStamp,f_Version,f_Status,f_MsgType,f_NeedReport,f_Priority,
    
     f_ServiceID,f_FeeType,f_FeeCode,f_FixedFee,f_MsgFormat,f_ValidTime,f_AtTime,
    
     f_SrcTermID,f_ChargeTermID,f_DestTermIDCount,f_DestTermID,f_MsgLength,f_MsgContent,
    
     f_MsgID,f_IsReport,f_RecvTime,f_Reserve,f_ReserveValue}
    
     local data_dis = Dissector.get("data")
    
     --
    
     local function SMGP_Login(buf,pkt,t)
    
      t:add(f_ClientID,buf(12,8))
    
      t:add(f_Authenticator,buf(20,16))
    
      t:add(f_LoginMode,buf(36,1))
    
      t:add(f_TimeStamp,buf(37,4))
    
      t:add(f_Version,buf(41,1))
    
      pkt.cols.info:prepend("SMGP_Login;" .. " ") --添加在info栏显示包类型
    
     end
    
     --
    
     local function SMGP_LoginResp(buf,pkt,t)
    
      t:add(f_Status,buf(12,4))
    
      t:add(f_Authenticator,buf(16,16))
    
      t:add(f_Version,buf(32,1))
    
      pkt.cols.info:prepend("SMGP_LoginResp;" .. " ")
    
     end
    
     --
    
     local function SMGP_Submit(buf,pkt,t)
    
      t:add(f_MsgType,buf(12,1))
    
      t:add(f_NeedReport,buf(13,1))
    
      t:add(f_Priority,buf(14,1))
    
      t:add(f_ServiceID,buf(15,10))
    
      pkt.cols.info:prepend("SMGP_Submit" .. " ")
    
      t:add(f_FeeType,buf(25,2))
    
      t:add(f_FeeCode,buf(27,6))
    
      t:add(f_FixedFee,buf(33,6))
    
      t:add(f_MsgFormat,buf(39,1))
    
      t:add(f_ValidTime,buf(40,17))
    
      t:add(f_AtTime,buf(57,17))
    
      t:add(f_SrcTermID,buf(74,21))
    
      t:add(f_ChargeTermID,buf(95,21))
    
      t:add(f_DestTermIDCount,buf(116,1))
    
      t:add(f_DestTermID,buf(117,21))
    
      local v_msgLen = buf(138,1)
    
      t:add(f_MsgLength,v_msgLen)
    
      v_msgLen = v_msgLen:uint()
    
      t:add(f_MsgContent,buf(139,v_msgLen))
    
      --添加调用保留字段
    
      t:add(f_Reserve,buf(139+v_msgLen,2))
    
      t:add(f_ReserveValue,buf(141+v_msgLen,6))
    
     end
    
     --
    
    local function SMGP_SubmitResp(buf,pkt,t)
    
      t:add(f_MsgID,buf(12,10))
    
      t:add(f_Status,buf(22,4))
    
      pkt.cols.info:prepend("SMGP_SubmitResp;" .. " ")
    
    end
    
     local function SMGP_DelvResp(buf,pkt,t)
    
      t:add(f_MsgID,buf(12,10))
    
      t:add(f_Status,buf(22,4))
    
      pkt.cols.info:prepend("SMGP_DelvResp;" .. " ")
    
     end
    
     --
    
     local function SMGP_Deliver(buf,pkt,t)
    
      t:add(f_MsgID,buf(12,10))
    
    <di