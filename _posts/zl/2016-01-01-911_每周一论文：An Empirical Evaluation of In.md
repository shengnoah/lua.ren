---
layout: post
title: 每周一论文：An Empirical Evaluation of In 
tags: [lua文章]
categories: [topic]
---
<h2 id="论文概要"><a href="#论文概要" class="headerlink" title="论文概要"></a>论文概要</h2><p>多版本并发控制(Multi-Version Concurrency Control，以下简称MVCC) 是当今数据库领域最流行的并发控制实现，MVCC 在最大化并发度的情况下尽可能保证事务的正确性，其好处有：</p>
<ul>
<li>写不会阻塞读</li>
<li>只读事务无需数据库锁就能支持可重复读</li>
<li>可以很好地支持历史数据查询</li>
</ul>
<p>MVCC 的关键在于首先假设数据库读写冲突不会很大，其次通过维护同一份数据的多个版本，是的事务之间的冲突尽可能小；当一个事务修改数据的时候，创建一个新的版本，当一个事务读数据的时候，返回最新版本数据；所有对于数据的修改都发生在事务的私有空间内，在提交的时候进行验证。</p>
<p>当今主流的数据库基本都支持MVCC：<br/><img src="https://blog-image-1258275666.cos.ap-chengdu.myqcloud.com/MVCC-Databases.png" alt="MVCC Implementation"/></p>
<p>本篇<a href="https://15721.courses.cs.cmu.edu/spring2018/papers/05-mvcc1/wu-vldb2017.pdf" target="_blank" rel="noopener noreferrer">论文</a>系统的总结了 MVCC 的技术要点，包括：</p>
<ol>
<li>并发控制协议</li>
<li>多版本存储</li>
<li>垃圾回收</li>
<li>索引管理</li>
</ol>
<h2 id="并发控制协议"><a href="#并发控制协议" class="headerlink" title="并发控制协议"></a>并发控制协议</h2><h3 id="MVTO"><a href="#MVTO" class="headerlink" title="MVTO"></a>MVTO</h3><p>通过预先计算顺序的方式来控制并发；事务的读操作返回最新的没有被写锁锁定数据的版本；事务的写操作过程如下：</p>
<ul>
<li>当前没有活跃的事务锁定数据</li>
<li>当前事务的事务编号大于最新数据中的读事务的事务编号</li>
<li>如果这上述条件成立，那么创建一个新的数据版本</li>
</ul>
<h3 id="MVOCC"><a href="#MVOCC" class="headerlink" title="MVOCC"></a>MVOCC</h3><p>在 MVOCC 中，事务被分成三个阶段，分别是：</p>
<ol>
<li>读数据阶段，着这个阶段新的版本被创建出来。</li>
<li>验证阶段，在这个阶段一个提交编号被分配给该事务，然后基于这个编号进行验证；</li>
<li>提交阶段，完成提交。</li>
</ol>
<h3 id="MV2PL"><a href="#MV2PL" class="headerlink" title="MV2PL"></a>MV2PL</h3><p>顾名思义，MV2PL 是传统的两阶段锁在多版本并发控制中的应用；事务读写或者创建数据版本都需要获得对应的锁。</p>
<h3 id="SSI"><a href="#SSI" class="headerlink" title="SSI"></a>SSI</h3><p>可串行化快照隔离(serializable snapshot isolation或SSI)是在快照隔离级别之上，支持串行化。PosgtreSQL 中实现了这种隔离级别，数据库通过维护一个串行的图移除事务并发造成的危险结构。</p>
<p><img src="https://blog-image-1258275666.cos.ap-chengdu.myqcloud.com/MVCC-CS.png" alt="MVCC-CS"/></p>
<h2 id="多版本存储"><a href="#多版本存储" class="headerlink" title="多版本存储"></a>多版本存储</h2><p>数据库通过无锁指针链表维护多个版本，使得事务可以方便的读取特定版本的数据。<br/><img src="https://blog-image-1258275666.cos.ap-chengdu.myqcloud.com/MVCC-Storage.png" alt="MVCC Storage"/></p>
<h3 id="仅限追加存储-Append-Only"><a href="#仅限追加存储-Append-Only" class="headerlink" title="仅限追加存储(Append-Only)"></a>仅限追加存储(Append-Only)</h3><ul>
<li>所有的版本存储在同一个表空间</li>
<li>更新的时候追加在版本链表上追加新节点</li>
<li>链表可以以最旧到最新的方式组织，</li>
<li>链表也可以以最新到最旧的方式组织，表头为最新版本</li>
</ul>
<h3 id="时序存储-Time-Travel-Storage"><a href="#时序存储-Time-Travel-Storage" class="headerlink" title="时序存储(Time-Travel Storage)"></a>时序存储(Time-Travel Storage)</h3><ul>
<li>每次更新的时候将之前的版本放到旧表空间</li>
<li>更新主表空间中的版本</li>
</ul>
<h3 id="仅差异存储-Delta-Storage"><a href="#仅差异存储-Delta-Storage" class="headerlink" title="仅差异存储(Delta Storage)"></a>仅差异存储(Delta Storage)</h3><ul>
<li>每次更新近存储修改的部分，将其加入链表，主表空间存储当前版本</li>
<li>通过旧的修改部分，可以创建旧版本</li>
</ul>
<h2 id="垃圾回收"><a href="#垃圾回收" class="headerlink" title="垃圾回收"></a>垃圾回收</h2><p><img src="https://blog-image-1258275666.cos.ap-chengdu.myqcloud.com/MVCC-GC.png" alt="MVCC GC"/></p>
<p>MVCC 在事务过程中不可避免的会产生很多的旧版本，这些旧版本会在下列情况下被回收</p>
<ol>
<li>对应的数据上没有活跃的事务</li>
<li>某版本数据的创建事务被终止</li>
</ol>
<h3 id="数据行级别垃圾回收-Tuple-Level"><a href="#数据行级别垃圾回收-Tuple-Level" class="headerlink" title="数据行级别垃圾回收(Tuple Level)"></a>数据行级别垃圾回收(Tuple Level)</h3><p>通过检查数据来判断是否需要回收旧版本，有两种做法：</p>
<ol>
<li>启动一个后台线程进行数据行级的垃圾回收</li>
<li>当事务操作数据行时，顺便做一些垃圾回收的事情</li>
</ol>
<h3 id="事务级别垃圾回收-Transaction-Level"><a href="#事务级别垃圾回收-Transaction-Level" class="headerlink" title="事务级别垃圾回收(Transaction Level)"></a>事务级别垃圾回收(Transaction Level)</h3><p>事务自己追踪旧版本，数据库管理系统不需要通过扫描数据行的方式来判断数据是否需要回收。</p>
<h2 id="索引管理"><a href="#索引管理" class="headerlink" title="索引管理"></a>索引管理</h2><p>数据有多个版本，而索引只有一份，更新和维护多个版本的时候如何同步索引？<br/><img src="https://blog-image-1258275666.cos.ap-chengdu.myqcloud.com/MVCC-GC.png" alt="MVCC GC"/></p>
<h3 id="主键-Primary-Key"><a href="#主键-Primary-Key" class="headerlink" title="主键(Primary Key)"></a>主键(Primary Key)</h3><p>主键一般指向多版本链表头</p>
<h3 id="副索引-Secondary-Indexes"><a href="#副索引-Secondary-Indexes" class="headerlink" title="副索引(Secondary Indexes)"></a>副索引(Secondary Indexes)</h3><p>有两种做法，逻辑指针和物理地址；前者通过增加一个中间层的方式实现，缩影指向该中间层，中间层指向数据的物理地址，避免应为多版本的物理地址改变引起的索引树的更新；后者索引直接指向数据物理地址。</p>
<h2 id="作者的其他数据库文章链接"><a href="#作者的其他数据库文章链接" class="headerlink" title="作者的其他数据库文章链接"></a>作者的其他数据库文章链接</h2><ol>
<li><a href="https://zhewuzhou.github.io/2018/08/07/SQL_as_universe_language_in_data_world/">SQL：数据世界的通用语</a></li>
<li><a href="https://zhewuzhou.github.io/2018/09/13/SQL_Compilation_Technology_For_Performance/">数据库性能之翼：SQL 语句运行时编译</a></li>
<li><a href="https://zhewuzhou.github.io/2018/09/25/Weekly-Paper-A-Survey-of-B-Tree-Locking-Techniques/">每周一论文：A Survey of B-Tree Locking Techniques</a></li>
<li><a href="https://zhewuzhou.github.io/2018/09/29/Weekly-Paper-An-Empirical-Evalution-of-In-Memory-MVCC/">每周一论文：An Empirical Evaluation of In-Memory Multi-Version Concurrency Control</a></li>
<li><a href="https://zhewuzhou.github.io/2018/10/18/Database-Indexes/">数据库索引数据结构总结</a></li>
</ol>