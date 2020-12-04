---
layout: post
title: Kafka cluaster partitions replicas group 
tags: [lua文章]
categories: [topic]
---
<h2 id="partition--replicas">Partition &amp; Replicas</h2>
<h3 id="kafka-集群默认自动分配解析">Kafka 集群默认自动分配解析</h3>
<ul>
  <li>下面以一个Kafka集群中4个Broker举例，创建1个topic包含4个Partition，2 Replication；数据Producer流动如图所示：</li>
</ul>

<table>
  <thead>
    <tr>
      <th style="text-align: center">Broker1</th>
      <th style="text-align: center">Broker2</th>
      <th style="text-align: center">Broker3</th>
      <th style="text-align: center">Broker4</th>
      <th style="text-align: center">BrokerX</th>
      <th style="text-align: center">BrokerX</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center">P0</td>
      <td style="text-align: center">P1</td>
      <td style="text-align: center">P2</td>
      <td style="text-align: center">P3</td>
      <td style="text-align: center">N/A</td>
      <td style="text-align: center">N/A</td>
    </tr>
    <tr>
      <td style="text-align: center">P3</td>
      <td style="text-align: center">P0</td>
      <td style="text-align: center">P1</td>
      <td style="text-align: center">P2</td>
      <td style="text-align: center">N/A</td>
      <td style="text-align: center">N/A</td>
    </tr>
  </tbody>
</table>

<ul>
  <li>当集群中新增2节点，Partition增加到6个时分布情况如下：</li>
</ul>

<table>
  <thead>
    <tr>
      <th style="text-align: center">Broker1</th>
      <th style="text-align: center">Broker2</th>
      <th style="text-align: center">Broker3</th>
      <th style="text-align: center">Broker4</th>
      <th style="text-align: center">Broker5</th>
      <th style="text-align: center">Broker6</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center">P0</td>
      <td style="text-align: center">P1</td>
      <td style="text-align: center">P2</td>
      <td style="text-align: center">P3</td>
      <td style="text-align: center">P4</td>
      <td style="text-align: center">P5</td>
    </tr>
    <tr>
      <td style="text-align: center">P3</td>
      <td style="text-align: center">P0</td>
      <td style="text-align: center">P1</td>
      <td style="text-align: center">P2</td>
      <td style="text-align: center"> </td>
      <td style="text-align: center">P4</td>
    </tr>
    <tr>
      <td style="text-align: center">P5</td>
      <td style="text-align: center"> </td>
      <td style="text-align: center"> </td>
      <td style="text-align: center"> </td>
      <td style="text-align: center"> </td>
      <td style="text-align: center"> </td>
    </tr>
  </tbody>
</table>

<h3 id="副本分配逻辑规则如下">副本分配逻辑规则如下：</h3>

<ul>
  <li>在Kafka集群中，每个Broker都有均等分配Partition的Leader机会。</li>
  <li>上述图Broker Partition中，箭头指向为副本，以Partition-0为例:broker1中parition-0为Leader，Broker2中Partition-0为副本。</li>
  <li>上述图种每个Broker(按照BrokerId有序)依次分配主Partition,下一个Broker为副本，如此循环迭代分配，多副本都遵循此规则。</li>
</ul>

<h3 id="副本分配算法如下">副本分配算法如下：</h3>
<ul>
  <li>将所有N Broker和待分配的i个Partition排序.</li>
  <li>将第i个Partition分配到第(i mod n)个Broker上.</li>
  <li>将第i个Partition的第j个副本分配到第((i + j) mod n)个Broker上.</li>
</ul>

<h2 id="group-on-consumers">Group on Consumers</h2>
<h3 id="kafka-partition--group">Kafka Partition &amp; Group</h3>
<ul>
  <li>原理图</li>
  <li>原理描述</li>
</ul>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>一个topic 可以配置几个partition，produce发送的消息分发到不同的partition中，consumer接受数据的时候是按照group来接受，kafka确保每个partition只能同一个group中的同一个consumer消费，如果想要重复消费，那么需要其他的组来消费。Zookeerper中保存这每个topic下的每个partition在每个group中消费的offset  
新版kafka把这个offsert保存到了一个__consumer_offsert的topic下  
这个__consumer_offsert 有50个分区，通过将group的id哈希值%50的值来确定要保存到那一个分区.  这样也是为了考虑到zookeeper不擅长大量读写的原因。  
所以，如果要一个group用几个consumer来同时读取的话，需要多线程来读取，一个线程相当于一个consumer实例。当consumer的数量大于分区的数量的时候，有的consumer线程会读取不到数据。   
假设一个topic test 被groupA消费了，现在启动另外一个新的groupB来消费test，默认test-groupB的offset不是0，而是没有新建立，除非当test有数据的时候，groupB会收到该数据，该条数据也是第一条数据，groupB的offset也是刚初始化的ofsert, 除非用显式的用–from-beginnging 来获取从0开始数据   
</code></pre></div></div>

<ul>
  <li>查看topic-group的offsert</li>
</ul>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>位置：zookeeper 
路径：[zk: localhost:2181(CONNECTED) 3] ls /brokers/topics/__consumer_offsets/partitions 
在zookeeper的topic中有一个特殊的topic __consumer_offserts 
计算方法：（放入哪个partitions）

int hashCode = Math.abs(&#34;ttt&#34;.hashCode());
int partition = hashCode % 50;
先计算group的hashCode，再除以分区数(50),可以得到partition的值 
使用命令查看： kafka-simple-consumer-shell.sh --topic __consumer_offsets --partition 11 --broker-list localhost:9092,localhost:9093,localhost:9094 --formatter &#34;kafka.coordinator.GroupMetadataManager$OffsetsMessageFormatter&#34;
</code></pre></div></div>

<ul>
  <li>
    <p>参数<br/>
auto.offset.reset:默认值为largest，代表最新的消息，smallest代表从最早的消息开始读取，当consumer刚开始创建的时候没有offset这种情况，如果设置了largest，则为当收到最新的一条消息的时候开始记录offsert,若设置为smalert，那么会从头开始读partition</p>
  </li>
  <li>
    <p>Consumer Group 
使用Consumer high level API时，同一Topic的一条消息只能被同一个Consumer Group内的一个Consumer消费，但多个Consumer Group可同时消费这一消息。<br/>
这是Kafka用来实现一个Topic消息的广播（发给所有的Consumer）和单播（发给某一个Consumer）的手段。一个Topic可以对应多个Consumer Group。如果需要实现广播，只要每个Consumer有一个独立的Group就可以了。要实现单播只要所有的Consumer在同一个Group里。用Consumer Group还可以将Consumer进行自由的分组而不需要多次发送消息到不同的Topic。<br/>
实际上，Kafka的设计理念之一就是同时提供离线处理和实时处理。根据这一特性，可以使用Storm这种实时流处理系统对消息进行实时在线处理，同时使用Hadoop这种批处理系统进行离线处理，还可以同时将数据实时备份到另一个数据中心，只需要保证这三个操作所使用的Consumer属于不同的Consumer Group即可。<br/>
下面这个例子更清晰地展示了Kafka Consumer Group的特性。首先创建一个Topic (名为topic1，包含3个Partition)，然后创建一个属于group1的Consumer实例，并创建三个属于group2的Consumer实例，最后通过Producer向topic1发送key分别为1，2，3的消息。结果发现属于group1的Consumer收到了所有的这三条消息，同时group2中的3个Consumer分别收到了key为1，2，3的消息。</p>
  </li>
</ul>