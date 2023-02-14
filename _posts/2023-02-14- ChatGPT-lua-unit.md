---
layout: post
title:  用ChatGPT生成Lua单体测试代码 
description:  用ChatGPT生成Lua单体测试代码 
ags: [openresty,lua]
categories: [topic]
image:
    feature: feature.jpg
    creditlink: https://lua.ren
---



> ## Excerpt
> 0x01 前言OpenAI的ChatGPT智能AI引擎，在全世界范围流行，各种ChatGPT应用场景也遍地开花。 问题回答、文章续写、代码审计、自动生成单体测试、木马生成等等，都可以实现。 用户方面也有很多相关活动，有媒体号拉…

---
OpenAI的ChatGPT智能AI引擎，在全世界范围流行，各种ChatGPT应用场景也遍地开花。 问题回答、文章续写、代码审计、自动生成单体测试、木马生成等等，都可以实现。

用户方面也有很多相关活动，有媒体号拉群推广技术、有中间商提供相关服务、内容代理转发机器人、相关组织加紧封号，技术人员先行进入战场，然后是自媒体流量段子飞起。

简单测试一下，ChatGPT在软件开发方面的功能是否好用，就用一个自动生成单元测试代码作为例子。文章最后，提供一些好用的ChatGPT插件，覆盖了日常最常用的ChatGPT的使用场景。

**0x02 ChatGPT自动生成代码单元测试用例**

源代码，先写一个简单的Lua函数代码，太复杂的理解的也慢，如下：

  

```text
function ChatGPT(num1, num2)
local ret = num1 + num2
return ret
end
ChatGPT(5,7)
```

  

然后，查看ChatGPT返回的单元测试代码，如下：

  

![](https://pic4.zhimg.com/v2-b7e9fcc27d56c5dc5f89855b43cfe6c3_b.jpg)

![](https://pic4.zhimg.com/80/v2-b7e9fcc27d56c5dc5f89855b43cfe6c3_720w.webp)

  

**图-1 ChatGPT生成的Lua单元测试的代码（两个用例）**

  

看到了ChatGPT返回的两个测试用例。

**测试用例1**

  

```text
function test_ChatGPT()
assert(ChatGPT(1, 2) == 3)
end
```

  

**测试用例2**

```text
function test_ChatGPT()
assert(ChatGPT(-2, 3) == 1)
end
```

  

把单体测试代码复制源代码中，等待执行。

  

![](https://pic4.zhimg.com/v2-ccb28590f35aa435e4ed187804daf56f_b.jpg)

![](https://pic4.zhimg.com/80/v2-ccb28590f35aa435e4ed187804daf56f_720w.webp)

  

**图-2 测试执行Lua单元测试代码**

正常执行代码，显示空内容。

  

![](https://pic1.zhimg.com/v2-c4554c5614bfabcf0e17e8888f4b9aac_b.jpg)

![](https://pic1.zhimg.com/80/v2-c4554c5614bfabcf0e17e8888f4b9aac_720w.webp)

  

**图-3 返回结果为空**

**0x03 改造AI生成单元测试代码**

ChatGPT返回的Lua单体测度代码，都是断言（Assert）正确的结果，手段把生成的代码改了，生成的是不出断言出错的，改成会造成代码断言出错的。

改之前，代码：

```text
function test_ChatGPT()
assert(ChatGPT(1, 2) == 3)
end
```

  

改之后，代码：

```text
function test_ChatGPT()
assert(ChatGPT(1, 2) == 5)
end
```

  

故意把断言（assert）的条件改错，出现下面的执行结果。

  

![](https://pic4.zhimg.com/v2-8d468b6e235d6aeb77269ad0b3aead7f_b.jpg)

![](https://pic4.zhimg.com/80/v2-8d468b6e235d6aeb77269ad0b3aead7f_720w.webp)

  

**图-4 单体断言出错**

正常用ChatGPT自动生成的函数单元测试代码是没有问题的， 用例都是断言（Assert）对的用例，“异常系”的用例，需要自己改造测试代码，从这个角度看，ChatGPT生成单元测试代码，还是可以提高工作效率的。

  

**0x04 好用有的ChatGPT插件汇总**

最后，推荐几个常用插件， 可以让用户在多种情况下，使用ChatGPT的功能，这三个插件都有一定的用户基数， 用起来还是可以的，Obsidian笔记社区本身支持也很强大。ChatGPT最开始是从日文站看到的。

  

**Chrome浏览器插件：ChatGPT Everywhere**

  

![](https://pic2.zhimg.com/v2-dd7c27cac4de8560454f1d2607befe25_b.jpg)

![](https://pic2.zhimg.com/80/v2-dd7c27cac4de8560454f1d2607befe25_720w.webp)

  

**图-5 浏览器插件**

这是一个浏览器插件，可以在插件市场中下载，对不同的浏览器找对应的版本。

  

**VSCode插件：ChatGPT**

  

![](https://pic3.zhimg.com/v2-b1c8e3378c8dba5177d4ef998854c0f2_b.jpg)

![](https://pic3.zhimg.com/80/v2-b1c8e3378c8dba5177d4ef998854c0f2_720w.webp)

  

**图-6 VSCode插件**

VSCode的插件，本文生成单元测试代码就靠这个插件。

  

**Obisidian笔记插件：Text Generator**

  

![](https://pic4.zhimg.com/v2-917ffbfeb362093318b28aab822dad83_b.jpg)

![](https://pic4.zhimg.com/80/v2-917ffbfeb362093318b28aab822dad83_720w.webp)

  

**图-7 Obsidian笔记插件**

用Obsidian笔记做知识管理和工作流的可以安装这个插件使用ChatGPT。以上这三个插件，是比较常用的，覆盖了基本使用场景，不用切换到浏览器下使用。

  

**0x05 总结**

以上插件使用，遇到什么相关问题，请留言，提供相关支持。
