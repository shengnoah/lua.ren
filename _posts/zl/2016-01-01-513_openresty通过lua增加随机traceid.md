---
layout: post
title: openresty通过lua增加随机traceid 
tags: [lua文章]
categories: [topic]
---
在没有引入zipkin（或者阿里的鹰眼，百度的华佗）这种trace系统的时候，排查问题的一般思路都是按照请求链路来寻找问题源。因此如果能在请求链路中有一个唯一的标识就最好了，而在nginx/openresty做接入层的架构中，可以通过lua脚本生成一个随机traceid。

随机数的生成原理，都是先初始化一个随机数种子，由于伪随机数的特性，种子的随机性就显得格外重要，而一般种子的生成都是通过时间的倒序来选取

#### lua 随机数生成方法

首先我们看下通常lua的随机数生成方法

    
    
    math.randomseed(tonumber(tostring(os.time()):reverse():sub(1,6)))
    math.random(m,n)
    

通过时间字符串的逆序初始化随机种子，这里注意到有个`sub`函数做了截断，是因为

> `math.randomseed` will call the underlying C function `srand` which takes an
> unsigned integer valueLua will cast the value of the seed to this format. In
> case of an overflow the seed will actually become a bad seed, without
> warning

所以需要避免出现高类型向低类型转换的溢出问题

### common 方法的问题

但上面的方法有个问题，就是`os.time()`函数返回的是秒(10位整数),
所以在做web请求的traceid时很容易就出现重复，影响问题追踪的效率，而lua如果要以毫秒为单位的时间来初始化随机种子，需要引入socket等外部模块，对于openresty来说一般都是封装好的，不方便去做这种定制

#### 利用openresty 与lua生成traceid

幸运的是，nginx-lua中有个函数`ngx.now`会返回一个浮点数（当然在lua中统一为number类型），3位小数即为毫秒位，所以问题就变得简单了

    
    
    access_by_lua_block {
        math.randomseed(tonumber(tostring(ngx.now()*1000):reverse():sub(1,9)))
        local randvar = string.format("%.0f",math.random(1000000000000000000,9223372036854775807))
        ngx.req.set_header("traceid", randvar)
    }
    

通过`ngx.now()*1000`拿到毫秒数据转换为字符串取反，这样毫秒数据的变化才能显出效果; unsigned int（64bit机器下为4byte)
最大值为10位数，我们取前9位避免数据溢出带来的转换问题;
lua在显示大于64bit的数据时会自动用科学技术法表示，所以我们需要通过string.format函数来将其转换为19位数字，然后通过ngx.req.set_header添加到请求头中去

[ ⤧ Next post samba安装及配置说明 ](/ops/OPS-
samba%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/) [ ⤧ Previous post
[译]ssh免密登录失败问题排查思路 ](/ops/OPS-
ssh-%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95%E5%A4%B1%E8%B4%A5%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E6%80%9D%E8%B7%AF/)