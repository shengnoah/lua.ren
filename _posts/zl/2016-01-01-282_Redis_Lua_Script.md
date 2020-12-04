---
layout: post
title: Redis_Lua_Script 
tags: [lua文章]
categories: [topic]
---
<ul>
<li>NoSql现在被大量应用在大数据操作，其中原因以IO速度较以往关联式数据库快且耗费资源较低吸引我，而Redis性能更是数一数二。</li>
<li>Redis简单来说就是一个KeyValue架构的NoSql，而Value的类型可以是String、List、Hash…….，不过不是这篇要说的重点。</li>
<li>试想一个最常被用来讨论的主题，提款机存款与提款，在账户只有一人操作存提款不会有什么问题，基本上就是存多少钱就可以提多少钱；但现在若是有两人，A在提款时先远端连线要回账号数据，此时B已经提领钱出来，如此A在领钱时就会发生A端电脑存款与远端Server不同步情形。解决方法有很多，这边用数据原质性解决此问题。</li>
<li>一般我们会将数据取出后对数据做完操作在写回数据库，但在这期间可能有其他人捷足先登，我们可以将取出跟写入在数据库里一并执行，Redis可以借由脚本LuaScript来达成此目的。</li>
<li>这里提供<a href="https://github.com/zongu/RedisLuaScript" target="_blank" rel="noopener noreferrer">RedisLuaScript范例</a></li>
</ul>