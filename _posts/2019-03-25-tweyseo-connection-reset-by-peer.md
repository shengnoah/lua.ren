---
layout: post
title: 服务器大负载情况下客户端出现Connection reset by peer 
description: 服务器大负载情况下客户端出现Connection reset by peer
date:   2019-03-24 12:50:18 +0800
tags: [openresty,lua]
categories: [topic]
image:
    feature: feature.jpg
    creditlink: http://orchina.org
---

作者：tweyseo

## **背景**

在一次交易服务扩容的之后，监控发现APIServer上在忙时会出现服务被熔断的日志 （upstream server temporarily disabled），进一步发现被熔断的服务是扩容进来的服务。进一步通过运维人员得知，由于这个机器的配置较好，所以upstream的权重也配的比较大。

## **定位**

于是开始在新机器上定位问题。发现对应进程的CPU较高，然后再通过SS发现，对应应用的端口除了ESTABLISTH以外，还有大量的SYN_RECV状态。再观察发现对应应用的端口在LISTEN状态的Send-Q列和Recv-Q列的值都是128，这说明当前应用的待accept队列的最大值只有128，并且已经满了。于是这里推论出：**由于应用的上层过于繁忙，造成应用的网络层无法及时去对已经完成三次握手的连接进程accept操作，造成待accept队列拥堵，从而造成待完成连接队列也发生拥堵（上述SS出现的大量SYN_RECV状态），最后对于客户端就出现Connection reset by peer了**。

## **原理分析**

这里进一步分析下这两个队列：![图片1](https://lua.ren/images/blog/tweyseo/connection-reset-by-peer/图片1.png)

由于TCP建立连接使用3次握手，因此，一个新连接在到达ESTABLISHED状态可以被accept系统调用返回给应用程序前，必须经过一个中间状态SYN RECEIVED（见上图中的SYN_RCVD）。这意味着，TCP/IP协议栈在实现待accept队列时，还需要额外的一个待完成连接队列：当协议栈收到一个SYN包时，响应SYN/ACK包，然后将连接的状态变成SYN RECEIVED并且加入待完成连接队列，后续当状态变更为ESTABLISHED时移入待accept队列（即收到3次握手中最后一个ACK包）。

其中待accept队列的长度由listen系统调用backlog参数的大小和系统的/proc/sys/net/core/somaxconn的大小（默认是128）的最小值确定，可以粗略的认为是min(backlog, somaxconn)；而待完成连接队列的长度则由系统的/proc/sys/net/ipv4/tcp_max_syn_backlog的大小指定（当然，在syncookies启用的情况下，待完成连接队列长度逻辑上没有最大值限制）。

当accept队列已满，而一个已完成新连接需要从待完成队列移动到accept队列的时候（这里参考kernel-3.10.0-514.el7/linux-3.10.0-514.el7.x86_64版本中的实现），在net/ipv4/tcp_ipv4.c中的tcp_v4_syn_recv_sock函数中可以看到如下代码：![图片2](https://lua.ren/images/blog/tweyseo/connection-reset-by-peer/图片2.png)

可以看到，这里会检查accept队列的长度。如果队列已满，跳到exit_overflow标签执行一些清理工作、更新/proc/net/netstat中的统计项ListenOverflows和ListenDrops，最后返回NULL：![图片3](https://lua.ren/images/blog/tweyseo/connection-reset-by-peer/图片3.png)

然后返回到tcp_check_req函数里，跳到listen_overflow标签执行代码：![图片4](https://lua.ren/images/blog/tweyseo/connection-reset-by-peer/图片4.png)

这里可以看到，除非系统的/proc/sys/net/ipv4/tcp_abort_on_overflow被设置为1（回客户端一个RST包），不然是没有做其他事情的。最后要注意的是：在SYN RECEIVED状态，如果ACK包没有收到（比如待accept队列已满造成的忽略），协议栈会重发SYN/ACK包，重试次数由系统的/proc/sys/net/ipv4/tcp_synack_retries决定。

## **总结**

当待完成连接队列长度已满，客户端在多次重发SYN包而得不到响应的时候会返回**connection time out**的错误；而当待accept队列长度已满，即使client继续向server发送ACK的包，也会不被响应（当然，具体情况由server端的配置/proc/sys/net/ipv4/tcp_abort_on_overflow来决定如何返回，0表示直接丢弃该ACK，1表示发送RST通知client）相应的，client则会分别返回**connection time out**或者**connection reset by peer**。

最后，可以通过`netstat -s | grep LISTEN`查看待完成连接队列的溢出丢弃情况；通过`netstat -s | grep TCPBacklogDrop`查看待accept队列的溢出丢弃情况。
