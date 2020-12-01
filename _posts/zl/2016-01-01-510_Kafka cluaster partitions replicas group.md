---
layout: post
title: Kafka cluaster partitions replicas group 
tags: [lua文章]
categories: [topic]
---
## Partition & Replicas

### Kafka 集群默认自动分配解析

  * 下面以一个Kafka集群中4个Broker举例，创建1个topic包含4个Partition，2 Replication；数据Producer流动如图所示：

Broker1 | Broker2 | Broker3 | Broker4 | BrokerX | BrokerX  
---|---|---|---|---|---  
P0 | P1 | P2 | P3 | N/A | N/A  
P3 | P0 | P1 | P2 | N/A | N/A  
  
  * 当集群中新增2节点，Partition增加到6个时分布情况如下：

Broker1 | Broker2 | Broker3 | Broker4 | Broker5 | Broker6  
---|---|---|---|---|---  
P0 | P1 | P2 | P3 | P4 | P5  
P3 | P0 | P1 | P2 |  | P4  
P5 |  |  |  |  |  
  
### 副本分配逻辑规则如下：

  * 在Kafka集群中，每个Broker都有均等分配Partition的Leader机会。
  * 上述图Broker Partition中，箭头指向为副本，以Partition-0为例:broker1中parition-0为Leader，Broker2中Partition-0为副本。
  * 上述图种每个Broker(按照BrokerId有序)依次分配主Partition,下一个Broker为副本，如此循环迭代分配，多副本都遵循此规则。

### 副本分配算法如下：

  * 将所有N Broker和待分配的i个Partition排序.
  * 将第i个Partition分配到第(i mod n)个Broker上.
  * 将第i个Partition的第j个副本分配到第((i + j) mod n)个Broker上.

## Group on Consumers

### Kafka Partition & Group

  * 原理图
  * 原理描述

    
    
    一个topic 可以配置几个partition，produce发送的消息分发到不同的partition中，consumer接受数据的时候是按照group来接受，kafka确保每个partition只能同一个group中的同一个consumer消费，如果想要重复消费，那么需要其他的组来消费。Zookeerper中保存这每个topic下的每个partition在每个group中消费的offset  
    新版kafka把这个offsert保存到了一个__consumer_offsert的topic下  
    这个__consumer_offsert 有50个分区，通过将group的id哈希值%50的值来确定要保存到那一个分区.  这样也是为了考虑到zookeeper不擅长大量读写的原因。  
    所以，如果要一个group用几个consumer来同时读取的话，需要多线程来读取，一个线程相当于一个consumer实例。当consumer的数量大于分区的数量的时候，有的consumer线程会读取不到数据。   
    假设一个topic test 被groupA消费了，现在启动另外一个新的groupB来消费test，默认test-groupB的offset不是0，而是没有新建立，除非当test有数据的时候，groupB会收到该数据，该条数据也是第一条数据，groupB的offset也是刚初始化的ofsert, 除非用显式的用–from-beginnging 来获取从0开始数据   
    

  * 查看topic-group的offsert

    
    
    位置：zookeeper 
    路径：[zk: localhost:2181(CONNECTED) 3] ls /brokers/topics/__consumer_offsets/partitions 
    在zookeeper的topic中有一个特殊的topic __consumer_offserts 
    计算方法：（放入哪个partitions）
    
    int hashCode = Math.abs("ttt".hashCode());
    int partition = hashCode % 50;
    先计算group的hashCode，再除以分区数(50),可以得到partition的值 
    使用命令查看： kafka-simple-consumer-shell.sh --topic __consumer_offsets --partition 11 --broker-list localhost:9092,localhost:9093,localhost:9094 --formatter "kafka.coordinator.GroupMetadataManager$OffsetsMessageFormatter"
    

  * 参数  
auto.offset.reset:默认值为largest，代表最新的消息，smallest代表从最早的消息开始读取，当consumer刚开始创建的时候没有offset这种情况，如果设置了largest，则为当收到最新的一条消息的时候开始记录offsert,若设置为smalert，那么会从头开始读partition

  * Consumer Group 使用Consumer high level API时，同一Topic的一条消息只能被同一个Consumer Group内的一个Consumer消费，但多个Consumer Group可同时消费这一消息。  
这是Kafka用来实现一个Topic消息的广播（发给所有的Consumer）和单播（发给某一个Consumer）的手段。一个Topic可以对应多个Consumer
Group。如果需要实现广播，只要每个Consumer有一个独立的Group就可以了。要实现单播只要所有的Consumer在同一个Group里。用Consumer
Group还可以将Consumer进行自由的分组而不需要多次发送消息到不同的Topic。  
实际上，Kafka的设计理念之一就是同时提供离线处理和实时处理。根据这一特性，可以使用Storm这种实时流处理系统对消息进行实时在线处理，同时使用Hadoop这种批处理系统进行离线处理，还可以同时将数据实时备份到另一个数据中心，只需要保证这三个操作所使用的Consumer属于不同的Consumer
Group即可。  
下面这个例子更清晰地展示了Kafka Consumer Group的特性。首先创建一个Topic
(名为topic1，包含3个Partition)，然后创建一个属于group1的Consumer实例，并创建三个属于group2的Consumer实例，最后通过Producer向topic1发送key分别为1，2，3的消息。结果发现属于group1的Consumer收到了所有的这三条消息，同时group2中的3个Consumer分别收到了key为1，2，3的消息。