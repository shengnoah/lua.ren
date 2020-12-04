---
layout: post
title: 翻译 《An Empirical Evaluation of In 
tags: [lua文章]
categories: [topic]
---
<p>工作中对于MVCC接触的已经比较多了，但是依然对MVCC的实现细节以及设计思路没有比较清晰的认知，因此在这里翻译一篇关于内存介质的MVCC的整体介绍，也作为笔记。
<br/></p>

<hr/>
<p><br/>
<br/></p>
<h2 id="摘要">摘要</h2>
<p>MVCC（Multi-version concurrency control多版本并发控制）是现代数据库管理系统最流行的事务管理方式。尽管早在19世纪70年代就已经发明了MVCC技术，但它仍然被近十年主流的数据库系统广泛使用。MVCC通过维护多个版本的数据，一定程度上，在不影响事务的可串行性的同时提升了数据库的并发度。然而在多核的内存操作环境下，拓展MVCC并不是一件容易的事情：当有大量线程在并行运行时，线程间同步的开销可能足以抵消MVCC带来的并发优化。</p>

<p>为了了解现代硬件环境下MVCC机制如何处理事务，我们从四个主要维度进行了广泛的调研：并发控制协议、版本存储、垃圾回收以及索引管理。我们用了以上技术的最先进方法实现了内存数据库，并且用OLTP的负载来进行评测。我们分析出了各种设计上的取舍导致的关键瓶颈。</p>

<h2 id="1-简介">1. 简介</h2>
<p>多核的出现带来了计算机架构的进化，内存数据库系统在不牺牲可串行性的前提下，实现了高效的事务管理机制。近十年，数据库系统中发展起来的最流行的技术就是MVCC。为了并行地处理同一个对象，数据库同时维护这一个逻辑对象的多个物理版本，这就是MVCC的基本思想。这个对象可以是任意粒度的，但是几乎所有的MVCC数据库都使用了元组（tuple）粒度来进行控制，因为这在多版本追踪和多并发之间取得了比较平衡的效果。多版本允许只读事务获取较为旧的数据，而不必阻止同时执行的读写事务产生同一数据的新版本。和单版本系统比较起来，事务总是在更新的时候用较新的数据覆盖原有的元组。</p>

<p>有趣的是，数据库使用MVCC并不是一个最近发展起来的趋势。MVCC在1979年第一次出现在学术论文当中，1981年的InterBase DBMS第一个实现了MVCC（现已成为开源项目FireBird）。此外在许多面向磁盘的数据库当中也广泛使用了MVCC，比如Oracle（1984年），Postgres（1985年）以及MySQL的InnoDB引擎（2001年）。但是同时代的也有大量数据库使用单版本管理（如IBM DB2、Sybase），几乎所有新的支持事务的数据库都支持MVCC，而避免了这种单版本管理。不论是商业系统（例如 Microsoft Hekaton、SAP HANA、MemSQL、NuoDB）还是学术系统（例如 HYRISE、HyPer）都是如此。</p>

<p>尽管新生的这些系统都采用了MVCC，但并没有一个“标准”实现。在不同的取舍以及性能表现下，有许多设计上的选择。目今为止，对现代数据库系统环境下的MVCC，还没有详尽的评测。最新的比较详尽的研究，是在1980年代，然而这项研究使用了面向磁盘存储的数据库系统，并且在单核CPU上运行模拟负载。在古老的磁盘数据库上的设计取舍，对于一个运行在多核CPU的内存数据库来说是不合适的。因此，最近的无锁的、可串行化的并发控制的趋势并不能通过以前的这些研究反映出来，内存存储以及混合的负载也是一样。</p>

<p>本文中，我们研究了MVCC数据库系统中几个关键的事务管理设计：(1) 并发控制协议，(2)版本存储，(3)垃圾回收，(4)索引管理。对于每个主题，我们都描述了在内存数据库上最先进的实现以及他们分别的设计权衡。也同样突出了如何避免为更大线程数量以及更复杂负载而扩展的问题。作为研究的一部分，我们在Peloton内存数据库当中实现了所有这些方法。这使得我们可以用统一的平台来比较这些实现，而不必考虑其他架构因素带来的问题。我们在一台有40核的机器上部署了Peloton，并且使用两个OLTP场景测试来进行评测。我们分析了对不同实现的加压方案并且讨论了减轻他们的方案。</p>

<h2 id="2-背景">2. 背景</h2>
<p>我们首先对MVCC进行高度概括式的概览，并讨论DBMS用来进行事务追踪以及版本信息维护的元信息。</p>

<h3 id="21-mvcc概览">2.1 MVCC概览</h3>
<p>事务管理系统允许用户在保持隔离性的同时，以并发的方式访问数据库。这保证了数据库系统的原子性和隔离性。</p>

<p>多版本系统有很多优势是对现代数据库应用非常重要的。最重要的是，多版本系统提供了比单版本系统更高的并发度。比如，一个采用了MVCC的数据库系统，会允许事务读取一个值的比较旧的版本，与此同时，另外一个事务还可以更新这个值。这对于在执行读写事务(read-write transaction)的同时做只读查询的场景非常重要。如果DBMS永远不删除旧版本数据，这个系统就可以支持“时间序列”操作。如果某些数据在历史时间点上存在过，则这一操作允许应用程序访问这一数据的连续的快照（consistent snapshot）</p>

<p>以上的好处使得MVCC成为了近年来新的DBMS的主要实现手段。表1给出了近三十年的MVCC实现的总结。但是在DBMS当中实现多版本系统的方式非常多样，每种各有额外的计算以及存储开销。这些设计上的取舍是相互依存的。因此，要讨论哪一种实现更好以及为什么，这不是一件容易的事情。尤其是对于内存型数据库，摆脱了磁盘这一主要瓶颈的数据库技术。</p>

<p>接下来的章节内，我们会讨论这些设计方案在实现细节以及性能权衡上（trade off）的比较。我们将在第七节给出一个全面的评测。本文中仅考虑可串行化事务执行。尽管日志和恢复也是DBMS架构当中非常重要的部分，但是我们在这篇文章中不涉及这些部分，因为这些部分和单版本系统并没有什么不同，并且内存型数据库的日志系统已经在别的地方被覆盖。</p>

<table>
  <thead>
    <tr>
      <th style="text-align: center"> </th>
      <th style="text-align: center">年份</th>
      <th style="text-align: center">协议</th>
      <th style="text-align: center">存储介质类型</th>
      <th style="text-align: center">垃圾回收</th>
      <th style="text-align: center">索引类型</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center">Oracle</td>
      <td style="text-align: center">1984</td>
      <td style="text-align: center">MV2PL</td>
      <td style="text-align: center">Delta</td>
      <td style="text-align: center">Tuple-level (VAC)</td>
      <td style="text-align: center">Logical Pointers (TupleId)</td>
    </tr>
    <tr>
      <td style="text-align: center">Postgres</td>
      <td style="text-align: center">1985</td>
      <td style="text-align: center">MV2PL/SSI</td>
      <td style="text-align: center">Append-only (O2N)</td>
      <td style="text-align: center">Tuple-level (VAC)</td>
      <td style="text-align: center">Physical Pointers</td>
    </tr>
    <tr>
      <td style="text-align: center">MySQL-InnoDB</td>
      <td style="text-align: center">2001</td>
      <td style="text-align: center">MV2PL</td>
      <td style="text-align: center">Delta</td>
      <td style="text-align: center">Tuple-level (VAC)</td>
      <td style="text-align: center">Logical Pointers (PKey)</td>
    </tr>
    <tr>
      <td style="text-align: center">HYRISE</td>
      <td style="text-align: center">2010</td>
      <td style="text-align: center">MVOCC</td>
      <td style="text-align: center">Append-only (N2O)</td>
      <td style="text-align: center">-</td>
      <td style="text-align: center">Physical Pointers</td>
    </tr>
    <tr>
      <td style="text-align: center">Hekaton</td>
      <td style="text-align: center">2011</td>
      <td style="text-align: center">MVOCC</td>
      <td style="text-align: center">Append-only (O2N)</td>
      <td style="text-align: center">Tuple-level (COOP)</td>
      <td style="text-align: center">Physical Pointers</td>
    </tr>
    <tr>
      <td style="text-align: center">MemSQL</td>
      <td style="text-align: center">2012</td>
      <td style="text-align: center">MVOCC</td>
      <td style="text-align: center">Append-only (N2O)</td>
      <td style="text-align: center">Tuple-level (VAC)</td>
      <td style="text-align: center">Physical Pointers</td>
    </tr>
    <tr>
      <td style="text-align: center">SAP HANA</td>
      <td style="text-align: center">2012</td>
      <td style="text-align: center">MV2PL</td>
      <td style="text-align: center">Time-travel</td>
      <td style="text-align: center">Hybrid</td>
      <td style="text-align: center">Logical Pointers (TupleId)</td>
    </tr>
    <tr>
      <td style="text-align: center">NuoDB</td>
      <td style="text-align: center">2013</td>
      <td style="text-align: center">MV2PL</td>
      <td style="text-align: center">Append-only (N2O)</td>
      <td style="text-align: center">Tuple-level (VAC)</td>
      <td style="text-align: center">Logical Pointers (PKey)</td>
    </tr>
    <tr>
      <td style="text-align: center">HyPer</td>
      <td style="text-align: center">2015</td>
      <td style="text-align: center">MVOCC</td>
      <td style="text-align: center">Delta</td>
      <td style="text-align: center">Transaction-level</td>
      <td style="text-align: center">Logical Pointers (TupleId)</td>
    </tr>
  </tbody>
</table>

<h5 id="表-1-mvcc实现">表-1 MVCC实现</h5>
<h5 id="商用以及研究用mvcc数据库的设计决策总结表中年份指对应系统除oracle以外第一次发布的时间表中oracle的年份属性是指它第一次采用mvcc方案的年份除了oraclemysql以及postgres以外所有系统的主存都是内存">商用以及研究用MVCC数据库的设计决策总结。表中年份指对应系统（除Oracle以外）第一次发布的时间。表中Oracle的年份属性，是指它第一次采用MVCC方案的年份。除了Oracle、MySQL以及Postgres以外，所有系统的主存都是内存。</h5>

<h3 id="22-dbms元数据">2.2 DBMS元数据</h3>
<p>无论数据库采用了何种实现，都会用到几种常见的元数据，这些元数据被DBMS用来处理事务和数据库元组（database tuple）。</p>

<p>事务： DBMS为事务T分配了一个唯一的、单调递增的时间戳来作为这些事务提交到系统内的标识（T<sub>id</sub>）。并发控制协议使用这个标志来标记这一事务所使用到的元组。某些协议也使用它来作为事务串行化的顺序。</p>

<p><img src="https://img.dazhuanlan.com/2019/11/28/5ddf8a5b11e99.png" alt="image01"/></p>
<h5 id="图-1-元组格式---元组的物理版本的基本布局">图-1 元组格式 - 元组的物理版本的基本布局</h5>

<p>元组：如图1所示，每一个物理版本在头部有四个元数据字段，被DBMS用来协调事务的并发执行（接下来的章节中所要讨论的某些协议当中会包含其他字段）。txn-id字段作为事务的写锁使用。任意一个未被写锁保护的元组都需要将这一字段设置为0。大多数DBMS使用了一个64位的txn-id，以便于可以使用单次的“比较并交换”（CaS）指令来原子地更新这一个值。如果标识为T<sub>id</sub>的事务T想要更新元组A，那么DBMS会检查A的txn-id字段是否为0。如果检查成功，那么DBMS会使用CaS指令来将txn-id设置为T<sub>id</sub>。</p>

<p>任何想要更新A的事务，都会在txn-id不为0且不和事务自身的T<sub>id</sub>相等的情况下被终止。接下来的两个元数据字段是begin-ts和end-ts，他们是两个时间戳，用来表示这个元组版本的生命周期。这些字段的初始值都是0。当元组被事务删除的时候，DBMS会把begin-ts设置为无限大（INF）。最后一个字段是pointer，存储了相邻的（前一个或者后一个）版本（如果存在的话）。</p>

<h2 id="3-并发控制协议">3 并发控制协议</h2>

<p>每个DBMS都包含一个并发控制协议（concurrency control protocol），以此来协调事务的并发执行。这一协议决定了一个事务能否在数据库运行过程中，获取或者修改一个指定版本的元组；也决定了一个事务是否能提交它的修改。尽管这些协议的基础在19世纪80年代就已经确定下来，但由于磁盘操作的减少，它们的性能特性在多核和内存作为主存的环境下有了非常大的改变。例如，出现了新的高性能变体，它们无锁，使用集中式数据结构（centralized data structure），并且为字节寻址设备进行了优化。</p>

<p>在这一章节当中，我们描述了四种核心的并发控制协议。只考虑使用了元组级别锁的协议，因为这足以保证串行化执行。我们忽略掉范围查询，因为多版本技术对于避免幻影读（phantom）没有任何好处。现有的提供可序列化事务的方法通常使用 1）索引上面额外的锁 2)事务提交时候的额外验证。</p>

<p><img src="https://img.dazhuanlan.com/2019/11/28/5ddf8a5b9c9f2.png" alt="image02"/></p>
<h5 id="图-2-并发控制协议---示意这些协议如何执行事务内的写后读">图-2 并发控制协议 - 示意这些协议如何执行事务内的写后读</h5>

<h3 id="31-时间戳序列mvto">3.1 时间戳序列（MVTO）</h3>

<p>1979年提出的MVTO算法被认为是最原始的MVCC控制协议。这一方法的关键点在于使用事务的标识（T<sub>id</sub>）来预计算事务的序列化顺序。除了在章节2.2当中描述的字段之外，版本头部还包含了最后一个读取这一元组的事务标识（read-ts）。当某个事务尝试读写某一个正在被其他事务写锁定的版本时，这一事务会被DBMS中止。</p>

<p>当事务T对逻辑元组A发起读操作时，DBMS会搜索是否存在某个物理版本，这个版本使得T<sub>id</sub>在begin-ts到end-ts的区间范围之内。如图 2a所示，如果版本A<sub>x</sub>没有被其他事务锁定写锁，那么事务T才可以读取这个版本，因为MVTO不允许事务读到未提交的版本。一旦读取了A<sub>x</sub>，如果当前的read-ts值小于T<sub>id</sub>，DBMS会把A<sub>x</sub>的read-ts设置为T<sub>id</sub>。否则，事务会读取一个更旧的版本，并且不更新read-ts字段。</p>

<p>在MVTO协议中，一个事务总是更新元组的最新版本。当以下条件成立时，事务T会创建一个新的版本B<sub>x+1</sub>: (1) 没有活跃的事务持有B<sub>x</sub>的写锁，并且 (2)T<sub>id</sub>大于B<sub>x</sub>的read-ts字段。如果以上条件满足，DBMS会创建一个新版本B<sub>x+1</sub>，并且设置txn-id为T<sub>id</sub>。当T提交之后，DBMS会把B<sub>x+1</sub>的begin-ts和end-ts字段分别设置为T<sub>id</sub>和INF，并把B<sub>x</sub>的end-ts字段设置为T<sub>id</sub></p>

<h3 id="32-乐观并发控制mvocc">3.2 乐观并发控制（MVOCC）</h3>

<p>接下来的协议是基于1981年提出的乐观并发控制（OCC）的。在DBMS当中OCC的设计动机是假设事务倾向于没有冲突，因此事务并不总需要在读写元组的时候持有锁。为了适应多版本系统，对于原始的OCC协议做了一些改变。最重要的是，DBMS不再为事务维护一个私有的工作空间（workspace），因为元组的版本信息已经可以阻止某些事务读或写他们本来不可见的版本。</p>

<p>MVOCC协议将事务分割成三阶段。当事务开始的时候，它处于读阶段（read phase）。这一阶段事务进行读和更新操作。就像MVTO协议一样，如要对元组A执行一个读操作，DBMS会首先基于begin-ts和end-ts找到一个可见的版本A<sub>x</sub>。如果A<sub>x</sub>的写锁没有被锁定的话，T就可以更新它。在多版本控制中，如果事务更新了版本B<sub>x</sub>DBMS会创建版本B<sub>x+1</sub>，新版本的txn-id设置为T<sub>id</sub>。</p>

<p>当一个事务通知DBMS进行提交，就会进入验证阶段（validation phase）。首先，DBMS将给事务分配新的时间戳（T<sub>commit</sub>）来决定事务的序列化顺序。接下来DBMS将会判断这个事务的读集合中的元组是否被已经提交的事务更新过。如果事务通过了这一项检查，那么它就会进入写入阶段（write phase），这一阶段中DBMS会将所有新的版本安装进来，并且将他们的begin-ts设置为T<sub>commit</sub>并且将end-ts设置为INF。</p>

<p>事务只可以更新元组的最新版本。但是直到创建这个元组的事务提交之前都不能读到这个新的元组。一个读到了过期版本的事务会在验证阶段被发现并中断。</p>

<h3 id="33-两阶段锁定mv2pl">3.3 两阶段锁定（MV2PL）</h3>

<p>这一协议使用了两阶段锁定（2PL）方法来保证事务可序列化。每个事务都需要在可以读写对应版本的逻辑元组之前获取适当的锁。在基于磁盘的DBMS中，锁是和元组分离的，永远不会被刷到磁盘上。在内存DBMS当中，这一分离已经没有必要了，因此在MV2PL协议当中，锁是被嵌入到元组头部的。元组的写锁是txn-id字段。DBMS使用了read-cnt字段来记录读取这一元组的活跃事务数量，以此来作为读锁。尽管并非必要，DBMS可以将txn-id和read-cnt打包到一个64位的字段，这样，DBMS就可以使用单次CaS指令来同时更新他们。</p>

<p>为了对元组A进行读操作，DBMS通过将事务的T<sub>id</sub>和元组的begin-ts字段进行比较来找到可见版本。如果找到了一个可用版本，DBMS会把read-cnt字段自增，此时txn-id字段应该等于0（这意味着没有其他事务持有这一元组的写锁）。类似的，当且仅当read-cnt和txn-id都被置为0的时候，事务才可以更新版本B<sub>x</sub>。当一个事务提交，DBMS为它分配一个唯一的时间戳（T<sub>commit</sub>），将这一事务创建的所有版本的begin-ts字段更新为T<sub>commit</sub>，然后释放掉这一事务持有的所有锁。</p>

<p>2PL协议之间的关键不同是如何处理死锁。以往的研究显示不等待策略（no-wait policy），是拓展性最强的避免死锁的方法。在这种策略下，如果某个事务不能获取某个元组上的锁，它会被DBMS立即中止（相反的，其他策略会等待到某个锁被释放）。由于事务从来不等待，DBMS就没有必要启动后台线程来检测和解决死锁。</p>

<h3 id="34-串行化验证">3.4 串行化验证</h3>

<p>在最后一个协议中，DBMS维护了一个序列化图（serialization graph）来探测和移除由并发事务构成的“危险的结构”。可以将这种基于验证的方法应用在低隔离级别上，来获得更好的性能，同时允许特定的异常情况。</p>

<p>首先被提出的验证方法是可序列化快照隔离（SSI）；这种方法通过在快照隔离情况下避免写偏序（write-skew）异常来保证可序列化。SSI使用了事务的标识来查找元组的可见版本。仅当某个元组的txn-id被设置为0，事务才可以修改它。为了保证可序列化，DBMS在内部的图当中记录了反向依赖边（anti-denpendency）。这一动作发生在当某个版本被其他事务读取，当前事务创建新的版本时。DBMS维护了每个事务的标记，以此跟踪反向依赖边的出度和入度。当DBMS检测到事务之间有两个连续的反向依赖边时，会中断其中一个。</p>

<p>序列化安全网（SSN）是一个更新的基于验证的协议。不同于SSI，它不仅可以用于快照隔离级别，也可以适用于任何不弱于提交读（READ COMMITTED）的隔离级别。它使用了一种更为精确的异常检测机制，这样来减少不必要的中断。SSN将事务的依赖信息编码到元信息字段当中，并且，通过计算一个警戒线（watermark）来验证T的一致性，这一警戒线描述了那些在T之前进行了提交但必须在T之后序列化的事务。SSN更适合于事务只读或者读写比较大的场景，可以减少大量的不必要中断。</p>

<h3 id="35-讨论">3.5 讨论</h3>

<p>这些协议处理冲突的方式不同，因此他们各有更适合的业务场景。MV2PL使用自己的读锁来访问各个版本。因此，事务对某个版本进行读/写操作的时候会导致其他想对同一版本做相同操作的事务中断。MVTO则使用了read-ts来记录各个版本的访问信息。MVOCC在读/写过程中并不修改元组头部的任何字段。这避免了线程间不必要的协调，某个事务对某版本的访问不会引起另一个要更新相同版本的事务的中断。但是MVOCC需要DBMS校验事务的读集合来验证事务读操作的正确性。这会导致只读的大事务陷入饥饿状态。Certifier协议可以减少中断，因为协议不会对读进行验证，但是它的反向依赖验证会带来额外的开销。</p>

<p>有一些优化这些协议的提案来提高MVCC DBMS的效率。其中一个方法是允许事务推测性地访问其他事务创建的未提交的版本。代价是协议必须记录事务的读依赖，来保证事务的可序列化。每个工作线程维护一个其他事务读取它未提交数据的引用计数。仅当事务引用计数为0的时候，它才允许提交，然后DBMS反转它的引用列表，自减每个等待它结束的事务的引用计数。类似的，另一种优化机制允许事务即时更新正在被未提交事务访问的版本。这一优化同样需要DBMS维护一个中心化数据结构来追踪事务之间的依赖关系。仅当某个事务依赖的所有事务都提交之后，它才可以提交。</p>

<p>以上描述的各种优化都可以减少某种场景（workload）下的不必要的中断。进一步的，我们发现维护一份中心化的数据结构可能成为一个主要的性能瓶颈点，阻碍DBMS扩展到大数量的核心。</p>

<h2 id="4-版本存储">4 版本存储</h2>

<p>在MVCC机制下，DBMS总是在事务尝试更新某个元组的时候构建一个新的物理版本。DBMS的版本存储方案（version storage scheme）指定了系统如何存储这些版本，以及各个版本包含什么信息。DBMS使用了元组的pointer域来创建了一个无锁链表，被称为版本链（version chain）。版本链使得DBMS可以找到所需的一个对某个事务可见的版本。在我们接下来的讨论中，版本链的头（head）是最新或最旧版本。</p>

<p>我们现在更深入的探讨这些方案。我们聚焦于这些方案对于更新操作的权衡，因为这是DBMS处理多版本的地方。DBMS可以插入一个新元组，而无需更新其他版本。类似的，DBMS如果要删除一个元组，只需要</p>