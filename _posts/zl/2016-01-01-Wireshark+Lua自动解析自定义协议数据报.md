---
layout: post
title: Wireshark+Lua自动解析自定义协议数据报 
tags: [lua文章]
categories: [lua文章]
---
  * 自定义协议格式
  * 原始抓包结果
  * 编写Lua脚本
  * 在Wireshark中加载Lua脚本
  * 在Wireshark中查看解析结果
  * 参考文献

在平时的工作中，经常需要根据接口文档进行开发，在调试时一般都会借助WireShark抓包进行分析，但是当协议较为复杂时，需要根据字节数手动计算进行解析，费时费力。曾经打算自己写个简单的自定义协议解析工具，后来发现WireShark提供了Lua接口，可以通过Lua脚本根据协议格式自动对获取的数据报进行解析，本文将对此进行简要介绍。

## 自定义协议格式

    
    
    #pragma pack(1)
    
    struct MsgHead {
        int msgNo_;
        unsigned char msgType_;
        unsigned char subMsgType_;
        short dataLen_;
    };
    
    struct MsgStruct {
        MsgHead msgHead_;
        unsigned char data_[MaxDataSize];
    };
    
    #pragma pack()
    

## 原始抓包结果

![https://AnonymousRookie.github.io/images/2018/10/201801004_01.png](https://AnonymousRookie.github.io/images/2018/10/201801004_01.png)

## 编写Lua脚本

    
    
    -- creates a Proto object, but doesn't register it yet
    local test_proto = Proto("test", "Test Protocol")
    
    -- a table of all of our Protocol's fields
    local test_fields =
    {
        msgNo = ProtoField.uint32("test.msgNo", "msgNo", base.DEC),
        msgType = ProtoField.uint8("test.msgType", "msgType", base.HEX),
        subMsgType = ProtoField.uint8("test.subMsgType", "subMsgType", base.HEX),
        dataLen = ProtoField.uint16("test.dataLen", "dataLen", base.DEC),
        data = ProtoField.bytes("test.data", "data"),
    }
    
    -- register the ProtoFields
    test_proto.fields = test_fields
    
    -- a table of our default settings - these can be changed by changing
    -- the preferences through the GUI or command-line; the Lua-side of that
    -- preference handling is at the end of this script file
    local default_settings =
    {
        enabled      = true, -- whether this dissector is enabled or not
        port         = 5001, -- default TCP port number for Test
    }
    
    --------------------------------------------------------------------------------
    -- The following creates the callback function for the dissector.
    -- It's the same as doing "test_proto.dissector = function (tvbuf,pkt,root)"
    -- The 'tvbuf' is a Tvb object, 'pktinfo' is a Pinfo object, and 'root' is a TreeItem object.
    -- Whenever Wireshark dissects a packet that our Proto is hooked into, it will call
    -- this function and pass it these arguments for the packet it's dissecting.
    function test_proto.dissector(tvbuf, pktinfo, root)
    
        -- set the protocol column to show our protocol name
        pktinfo.cols.protocol:set("Test")
    
        -- get the length of the packet buffer (Tvb).
        local pktlen = tvbuf:len()
    
        local offset = 0
    
        -- We start by adding our protocol to the dissection display tree.
        local tree = root:add(test_proto, tvbuf:range(offset, pktlen))
    
        tree:add(test_fields.msgNo, tvbuf(offset, 4))
        offset = offset + 4
        tree:add(test_fields.msgType, tvbuf(offset, 1))
        offset = offset + 1
        tree:add(test_fields.subMsgType, tvbuf(offset, 1))
        offset = offset + 1
        tree:add(test_fields.dataLen, tvbuf(offset, 2))
        offset = offset + 2
        tree:add(test_fields.data, tvbuf(offset, pktlen-offset))
    end
    
    --------------------------------------------------------------------------------
    -- We want to have our protocol dissection invoked for a specific TCP port,
    -- so get the TCP dissector table and add our protocol to it.
    local function enableDissector()
        -- using DissectorTable:set() removes existing dissector(s), whereas the
        -- DissectorTable:add() one adds ours before any existing ones, but
        -- leaves the other ones alone, which is better
        DissectorTable.get("tcp.port"):add(default_settings.port, test_proto)
    end
    
    local function disableDissector()
        DissectorTable.get("tcp.port"):remove(default_settings.port, test_proto)
    end
    
    -- call it now
    enableDissector()
    

完整程序详见：[https://github.com/AnonymousRookie/useful-
tools/tree/master/wireshark_protocol_dissector](https://github.com/AnonymousRookie/useful-
tools/tree/master/wireshark_protocol_dissector/)

## 在Wireshark中加载Lua脚本

将protocol_dissector_example.lua拷贝到Wireshark的安装目录，找到init.lua，在该脚本结尾加上：

    
    
    dofile(DATA_DIR.."protocol_dissector_example.lua")
    

重启Wireshark，点击按钮”表达式…/(Expression…)”，在搜索框中输入”TEST”，发现”Test
Protocol”，说明脚本已经加载成功。

![https://AnonymousRookie.github.io/images/2018/10/201801004_02.png](https://AnonymousRookie.github.io/images/2018/10/201801004_02.png)

## 在Wireshark中查看解析结果

现在就可以根据协议格式进行筛选了，并且获取到的数据报已经根据定义的格式自动进行了解析。

![https://AnonymousRookie.github.io/images/2018/10/201801004_03.png](https://AnonymousRookie.github.io/images/2018/10/201801004_03.png)

## 参考文献

  * [1] https://wiki.wireshark.org/Lua/Examples