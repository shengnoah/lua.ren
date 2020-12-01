---
layout: post
title: 每周一论文：An Empirical Evaluation of In 
tags: [lua文章]
categories: [topic]
---
## 论文概要

多版本并发控制(Multi-Version Concurrency Control，以下简称MVCC) 是当今数据库领域最流行的并发控制实现，MVCC
在最大化并发度的情况下尽可能保证事务的正确性，其好处有：

  * 写不会阻塞读
  * 只读事务无需数据库锁就能支持可重复读
  * 可以很好地支持历史数据查询

MVCC
的关键在于首先假设数据库读写冲突不会很大，其次通过维护同一份数据的多个版本，是的事务之间的冲突尽可能小；当一个事务修改数据的时候，创建一个新的版本，当一个事务读数据的时候，返回最新版本数据；所有对于数据的修改都发生在事务的私有空间内，在提交的时候进行验证。

当今主流的数据库基本都支持MVCC：  
![MVCC Implementation](https://blog-image-1258275666.cos.ap-
chengdu.myqcloud.com/MVCC-Databases.png)

本篇[论文](https://15721.courses.cs.cmu.edu/spring2018/papers/05-mvcc1/wu-
vldb2017.pdf)系统的总结了 MVCC 的技术要点，包括：

  1. 并发控制协议
  2. 多版本存储
  3. 垃圾回收
  4. 索引管理

## 并发控制协议

### MVTO

通过预先计算顺序的方式来控制并发；事务的读操作返回最新的没有被写锁锁定数据的版本；事务的写操作过程如下：

  * 当前没有活跃的事务锁定数据
  * 当前事务的事务编号大于最新数据中的读事务的事务编号
  * 如果这上述条件成立，那么创建一个新的数据版本

### MVOCC

在 MVOCC 中，事务被分成三个阶段，分别是：

  1. 读数据阶段，着这个阶段新的版本被创建出来。
  2. 验证阶段，在这个阶段一个提交编号被分配给该事务，然后基于这个编号进行验证；
  3. 提交阶段，完成提交。

### MV2PL

顾名思义，MV2PL 是传统的两阶段锁在多版本并发控制中的应用；事务读写或者创建数据版本都需要获得对应的锁。

### SSI

可串行化快照隔离(serializable snapshot isolation或SSI)是在快照隔离级别之上，支持串行化。PosgtreSQL
中实现了这种隔离级别，数据库通过维护一个串行的图移除事务并发造成的危险结构。

![MVCC-CS](https://blog-image-1258275666.cos.ap-chengdu.myqcloud.com/MVCC-
CS.png)

## 多版本存储

数据库通过无锁指针链表维护多个版本，使得事务可以方便的读取特定版本的数据。  
![MVCC Storage](https://blog-image-1258275666.cos.ap-
chengdu.myqcloud.com/MVCC-Storage.png)

### 仅限追加存储(Append-Only)

  * 所有的版本存储在同一个表空间
  * 更新的时候追加在版本链表上追加新节点
  * 链表可以以最旧到最新的方式组织，
  * 链表也可以以最新到最旧的方式组织，表头为最新版本

### 时序存储(Time-Travel Storage)

  * 每次更新的时候将之前的版本放到旧表空间
  * 更新主表空间中的版本

### 仅差异存储(Delta Storage)

  * 每次更新近存储修改的部分，将其加入链表，主表空间存储当前版本
  * 通过旧的修改部分，可以创建旧版本

## 垃圾回收

![MVCC GC](https://blog-image-1258275666.cos.ap-chengdu.myqcloud.com/MVCC-
GC.png)

MVCC 在事务过程中不可避免的会产生很多的旧版本，这些旧版本会在下列情况下被回收

  1. 对应的数据上没有活跃的事务
  2. 某版本数据的创建事务被终止

### 数据行级别垃圾回收(Tuple Level)

通过检查数据来判断是否需要回收旧版本，有两种做法：

  1. 启动一个后台线程进行数据行级的垃圾回收
  2. 当事务操作数据行时，顺便做一些垃圾回收的事情

### 事务级别垃圾回收(Transaction Level)

事务自己追踪旧版本，数据库管理系统不需要通过扫描数据行的方式来判断数据是否需要回收。

## 索引管理

数据有多个版本，而索引只有一份，更新和维护多个版本的时候如何同步索引？  
![MVCC GC](https://blog-image-1258275666.cos.ap-chengdu.myqcloud.com/MVCC-
GC.png)

### 主键(Primary Key)

主键一般指向多版本链表头

### 副索引(Secondary Indexes)

有两种做法，逻辑指针和物理地址；前者通过增加一个中间层的方式实现，缩影指向该中间层，中间层指向数据的物理地址，避免应为多版本的物理地址改变引起的索引树的更新；后者索引直接指向数据物理地址。

## 作者的其他数据库文章链接

  1. [SQL：数据世界的通用语](https://zhewuzhou.github.io/2018/08/07/SQL_as_universe_language_in_data_world/)
  2. [数据库性能之翼：SQL 语句运行时编译](https://zhewuzhou.github.io/2018/09/13/SQL_Compilation_Technology_For_Performance/)
  3. [每周一论文：A Survey of B-Tree Locking Techniques](https://zhewuzhou.github.io/2018/09/25/Weekly-Paper-A-Survey-of-B-Tree-Locking-Techniques/)
  4. [每周一论文：An Empirical Evaluation of In-Memory Multi-Version Concurrency Control](https://zhewuzhou.github.io/2018/09/29/Weekly-Paper-An-Empirical-Evalution-of-In-Memory-MVCC/)
  5. [数据库索引数据结构总结](https://zhewuzhou.github.io/2018/10/18/Database-Indexes/)