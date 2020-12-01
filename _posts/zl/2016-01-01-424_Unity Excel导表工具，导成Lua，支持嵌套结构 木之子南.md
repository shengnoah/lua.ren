---
layout: post
title: Unity Excel导表工具，导成Lua，支持嵌套结构 木之子南 
tags: [lua文章]
categories: [topic]
---
#### 为什么要使用导表工具？

几乎所有游戏公司数据由策划来配置，程序负责逻辑，策划看不懂代码，excel是可以相对具象的让策划了解一个模块的数据配置，起到了策划和程序之间的桥梁作用，也可以方便策划对数据的把控。

根据各个公司各个项目的不同，导表工具的输出形式不同，输出形式有：json，sqlite，txt，lua等，根据不同的项目需求，可以选不同的导表工具。本文章主要是介绍Excel导出Lua文件。

#### 此工具的功能

  1. 支持导出Lua文件，自动换行对齐
  2. 支持自定义字段不导入Lua
  3. 支持无限嵌套的树状结构（table套table）
  4. 支持的Excel格式 .xlsx, .xlsm, .xltx, .xltm
  5. 导出路径如果已存在同名的Lua文件，则会覆盖

#### Excel和导出文件的效果

##### excel的格式

![在这里插入图片描述](https://yiyuan1130.github.io//styles/images/excel2lua/excel2lua.png)

##### 导出的lua格式

    
    
    return {
        [1] = {
            id = 1,
            name = {
                CN = "安娜",
                EN = "Anna",
            },
            age = 12,
            isGirl = true,
            hp = 100,
            mp = 100,
            skill = {
                skill1 = {
                    name = "神罗天征",
                    attact = 100.3,
                },
                skill2 = {
                    name = "绝对防御",
                    attact = 10.5,
                    aaa = {
                        test1 = 111,
                        test2 = "111.0",
                    },
                },
            },
        },
        [2] = {
            id = 2,
            name = {
                CN = "雷欧",
                EN = "Leo",
            },
            age = 13,
            isGirl = false,
            hp = 100,
            mp = 100,
            skill = {
                skill1 = {
                    name = "螺旋丸",
                    attact = 45.9,
                },
                skill2 = {
                    name = "23.0",
                    attact = 134.3,
                    aaa = {
                        test1 = 222,
                        test2 = "222.0",
                    },
                },
            },
        },
    }
    

#### 项目地址

[点击此处查看 github 工程](https://github.com/yiyuan1130/tools-excel2lua)