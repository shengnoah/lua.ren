---
layout: post
title: [SQL]升级SQL Server Evaluation 版本成为正式版本 
tags: [lua文章]
categories: [topic]
---

                        <p>如果不小心安装错 SQL Server 为 Evaluation 的版本，要小心当超过 180 天之后，系统就会无法正常使用了</p>
                    <br />
                    <p>这几天遇到一个蛮特别的案例，原本收到的问题是 “维护计划” 忽然无法使用，即便是里面没有任何的 Task，都无法顺利地执行。但从对方所提供的错误消息却看不出来问题在哪里，因此花了一点时间研究一下。</p>

<p>原来这台机器正常有设定维护计划去备份数据库，但不知道是否没有准备儿童节礼物 – 乖乖，居然过完儿童节就忽然无法正常执行，因此就先写了一段 SQL 语法给朋友，看是否把语法放到排程内是否可以顺利执行。</p>

<pre>
<code>DECLARE @cursor AS CURSOR
DECLARE @dbname AS VARCHAR(20),  @query AS VARCHAR(100)
DECLARE @today  AS VARCHAR(12) = CONVERT( varchar(12), GETDATE() , 112 ) 

SET @cursor = CURSOR SCROLL FOR
	SELECT name FROM master..sysdatabases 
		WHERE name NOT IN('master','model','msdb','tempdb','distribution')
OPEN @cursor
FETCH NEXT FROM @cursor INTO @dbname
WHILE @@FETCH_STATUS = 0
BEGIN
	SET @query = 'BACKUP DATABASE ['+ @dbname +'] TO DISK = ''C:Temp'+@dbname+'_'+@today+'.bak '' WITH INIT,COMPRESSION'
	EXEC(@query)
	FETCH NEXT FROM @cursor INTO @dbname
END
CLOSE @cursor
DEALLOCATE @cursor</code></pre>

<p>

<p>透过朋友响应，如果让排程去执行上述的 Backup 的语法，就可以正常地去执行备份，因此整个推论应该不是 SQL Agent 或者是权限的问题，那看起来就真的很难隔空把脉了，似乎要连线看一下状况。而查看之后发现当 SQL Agent 去执行维护计划的 Job 的时候，会当出 0xC0000033 的错误消息，有了这个蛛丝马迹，就可以从官方文档的“Integration Services 错误和消息参考”的说明中找到，原来那个错误消息的意思是 SSIS 超过试用期了 ?! 正常来说 SSIS 和 SQL Engine 不可能一个是正式版，而另外一个是试用版，因此决定查了一下 SQL Server 安装的纪录，以及 SQL Server 的启动纪录中来查看，原来一开始的时候安装是试用版，只是刚好这段时间都没有把 SQL Server 给重新启动，因此虽然 SQL Engine 也过了试用期，但因为还没有重新启动，因此都还可以正常使用，但 SSIS 则是刚好执行的时候去检查，因此就会当出那个过期的资讯了。</p>

<p>既然知道问题的原因，那就好解决了。</p>

<p>首先我们先检查一下目前有问题的 SQL Server ，透过 Error Log 中的启动纪录，我们可以看到目前是安装 SQL Server Evaluation 的版本<img height="798" src="https://az787680.vo.msecnd.net/user/jamesfu/6a04b2c4-8316-4898-97c2-48c21dfb324f/1554894739_28274.png" width="1120"></p>

<p>接着我们拿出安装片出来，执行安装程序之后，选择“维护”→“版本升级”<img height="732" src="https://az787680.vo.msecnd.net/user/jamesfu/6a04b2c4-8316-4898-97c2-48c21dfb324f/1554894676_20912.png" width="982"></p>

<p>记得不要在选错了，这次确定要安装正式的版本<img height="751" src="https://az787680.vo.msecnd.net/user/jamesfu/6a04b2c4-8316-4898-97c2-48c21dfb324f/1554894997_72331.png" width="1007"><img height="751" src="https://az787680.vo.msecnd.net/user/jamesfu/6a04b2c4-8316-4898-97c2-48c21dfb324f/1554895071_45682.png" width="1007"></p>

<p>确认一下我们要升级的 Instance<img height="751" src="https://az787680.vo.msecnd.net/user/jamesfu/6a04b2c4-8316-4898-97c2-48c21dfb324f/1554895115_64591.png" width="1007"></p>

<p>确认没有人使用之后，就可以按下“升级”的按钮了<img height="751" src="https://az787680.vo.msecnd.net/user/jamesfu/6a04b2c4-8316-4898-97c2-48c21dfb324f/1554895187_0138.png" width="1007"></p>

<p>经过了几秒钟之后，就可以看到升级完成的画面了<img height="751" src="https://az787680.vo.msecnd.net/user/jamesfu/6a04b2c4-8316-4898-97c2-48c21dfb324f/1554895308_46227.png" width="1007"></p>

<hr>
<p>一般来说我们只要这样做就可以完成升级了，但实在不免好奇，到底在这个升级过程中，SQL Server 是怎么做的，因此查看一下 Error Log，从消息中我可以看出一些端倪，SQL Server 在升级过程中，其实是把 SQL Server Engine 的服务给关闭，然后加入一些启动参数来做启动。<img height="774" src="https://az787680.vo.msecnd.net/user/jamesfu/6a04b2c4-8316-4898-97c2-48c21dfb324f/1554895506_7087.png" width="1204"></p>

<p>而做完上述整个进程之后，在重新把 SQL Server 给启动起来，就完成相关的升级动作了。<img height="897" src="https://az787680.vo.msecnd.net/user/jamesfu/6a04b2c4-8316-4898-97c2-48c21dfb324f/1554895761_37029.png" width="1040"></p>

<p>看起来下次要是手边没有安装光碟片，或许也可以来测试看看，就不用那么麻烦还要去找安装片了。</p>

<p>PS. 又安装一台起来测试，不能直接利用启动参数来改变，那样是没有效果的，看起来还要在测试看看有没有更快的方式了。</p>