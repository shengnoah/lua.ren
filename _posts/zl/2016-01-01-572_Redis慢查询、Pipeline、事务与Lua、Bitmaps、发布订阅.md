---
layout: post
title: Redis慢查询、Pipeline、事务与Lua、Bitmaps、发布订阅 
tags: [lua文章]
categories: [topic]
---
* * *

`Redis`提供的5种数据结构已经足够强大，但除此之外，`Redis`还提供了诸如慢查询分析、功能强大的`Redis
Shell`、`Pipeline`、事务与Lua脚本、`Bitmaps`、`HyperLogLog`、发布订阅、`GEO`等附加功能，这些功能可以在某些场景发挥重要的作用。

### 慢查询分析

许多存储系统（例如`MySQL`）提供慢查询日志帮助开发和运维人员定位系统存在的慢操作。所谓慢查询日志就是系统在命令执行前后计算每条命令的执行时间，当超过预设阀值，就将这条命令的相关信息（例如：发生时间，耗时，命令的详细信息）记录下来，`Redis`也提供了类似的功能。

`Redis`客户端执行一条命令经历4个过程：发送命令、命令排队、命令执行、返回结果

#### 慢查询的两个配置参数

`slowlog-log-slower-than`：
它的单位是微秒，默认值是10000，假如执行了一条“很慢”的命令（例如keys*），如果它的执行时间超过了10000微秒，那么它将被记录在慢查询日志中。`slowlog-
log-slower-than=0`会记录所有的命令，`slowlog-log-slower-than<0`对于任何命令都不会进行记录。

`slowlog-max-len`：`Redis`使用了一个列表来存储慢查询日志，`slowlog-max-
len`就是列表的最大长度。一个新的命令满足慢查询条件时插入到这个列表中，当慢查询日志列表已处于其最大长度时，最早插入的一个命令将从列表中移出，例如`slowlog-
max-len`设置为5，当有第6条慢查询插入的话，那么队头的第一条数据就出列，第6条慢查询就会入列。

在`Redis`中有两种修改配置的方法:

  * 修改配置文件
  * 使用`config set`命令动态修改

下面使用`config set`命令将`slowlog-log-slower-than`设置为20000微秒，`slowlog-max-
len`设置为1000，`config rewrite`将配置持久化到本地配置文件：

    
    
    config set slowlog-log-slower-than 20000
    config set slowlog-max-len 1000
    config rewrite
    

##### 获取慢查询日志

slowlog get [n]

参数n可以指定条数：

    
    
    127.0.0.1:6379> slowlog get
    1) 1) (integer) 666
    2) (integer) 1456786500
    3) (integer) 11615
    4) 1) "BGREWRITEAOF"
    2) 1) (integer) 665
    2) (integer) 1456718400
    3) (integer) 12006
    4) 1) "SETEX"
    2) "video_info_200"
    3) "300"
    4) "2"
    

可以看到每个慢查询日志有4个属性组成，分别是慢查询日志的标识id、发生时间戳、命令耗时、执行命令和参数。

##### 获取慢查询日志列表当前的长度

    
    
    127.0.0.1:6379> slowlog len
    (integer) 45
    

##### 慢查询日志重置

    
    
    127.0.0.1:6379> slowlog reset
    OK
    127.0.0.1:6379> slowlog len
    (integer) 0
    

#### Pipeline

`Redis`提供了批量操作命令（例如`mget`、`mset`等），有效地节约`RTT`。但大部分命令是不支持批量操作的，例如要执行n次`hgetall`命令，并没有`mhgetall`命令存在，需要消耗n次`RTT`。

`Pipeline`（流水线）机制能改善上面这类问题，它能将一组`Redis`命令进行组装，通过一次`RTT`传输给`Redis`，再将这组`Redis`命令的执行结果按顺序返回给客户端。

可以使用`Pipeline`模拟出批量操作的效果，但是在使用时要注意它与原生批量命令的区别，具体包含以下几点：

  * 原生批量命令是原子的，`Pipeline`是非原子的。
  * 原生批量命令是一个命令对应多个key，`Pipeline`支持多个命令。
  * 原生批量命令是`Redis`服务端支持实现的，而`Pipeline`需要服务端和客户端的共同实现。

每次`Pipeline`组装的命令个数不能没有节制，否则一次组装`Pipeline`数据量过大，一方面会增加客户端的等待时间，另一方面会造成一定的网络阻塞，可以将一次包含大量命令的`Pipeline`拆分成多次较小的`Pipeline`来完成。

#### 事务与Lua

为了保证多条命令组合的原子性，`Redis`提供了简单的事务功能以及集成`Lua`脚本来解决这个问题。

##### 事务

事务表示一组动作，要么全部执行，要么全部不执行。例如在社交网站上用户A关注了用户B，那么需要在用户A的关注表中加入用户B，并且在用户B的粉丝表中添加用户A，这两个行为要么全部执行，要么全部不执行，否则会出现数据不一致的情况。

`Redis`提供了简单的事务功能，将一组需要一起执行的命令放到`multi`和`exec`两个命令之间。`multi`命令代表事务开始，`exec`命令代表事务结束，它们之间的命令是原子顺序执行的。

    
    
    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> sadd user:a:follow user:b
    QUEUED
    127.0.0.1:6379> sadd user:b:fans user:a
    QUEUED
    

`sadd`命令此时的返回结果是`QUEUED`，代表命令并没有真正执行，而是暂时保存在`Redis`中。如果此时另一个客户端执行`sismember
user:a:follow user:b`返回结果应该为0。

    
    
    127.0.0.1:6379> sismember user:a:follow user:b
    (integer) 0
    

只有当`exec`执行后，用户A关注用户B的行为才算完成：

    
    
    127.0.0.1:6379> exec
    1) (integer) 1
    2) (integer) 1
    

另一个客户端：

    
    
    127.0.0.1:6379> sismember user:a:follow user:b
    (integer) 1
    

如果要停止事务的执行，可以使用`discard`命令代替`exec`命令即可。

如果事务中的命令出现错误，`Redis`的处理机制也不尽相同:

###### 命令错误

例如下面操作错将set写成了sett，属于语法错误，会造成整个事务无法执行，key和counter的值未发生变化:

    
    
    127.0.0.1:6379> mget key counter
    1) "hello"
    2) "100"
    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> sett key world
    (error) ERR unknown command 'sett'
    127.0.0.1:6379> incr counter
    QUEUED
    127.0.0.1:6379> exec
    (error) EXECABORT Transaction discarded because of previous errors.
    

###### 运行时错误

例如用户B在添加粉丝列表时，误把`sadd`命令写成了`zadd`命令，这种就是运行时命令，因为语法是正确的：

    
    
    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> sadd user:a:follow user:b
    QUEUED
    127.0.0.1:6379> zadd user:b:fans 1 user:a
    QUEUED
    127.0.0.1:6379> exec
    1) (integer) 0
    2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
    127.0.0.1:6379> sismember user:a:follow user:b
    (integer) 1
    

可以看到`Redis`并不支持回滚功能，`sadd user:a:follow user:b`命令已经执行成功，开发人员需要自己修复这类问题。

有些应用场景需要在事务之前，确保事务中的key没有被其他客户端修改过，才执行事务，否则不执行（类似乐观锁）。`Redis`提供了`watch`命令来解决这类问题。

“客户端-1”在执行`multi`之前执行了`watch`命令，“客户端-2”在“客户端-1”执行`exec`之前修改了key值，造成事务没有执行（`exec`结果为`nil`）

客户端1：

    
    
    127.0.0.1:6379> set key "java"
    OK
    127.0.0.1:6379> watch key
    OK
    127.0.0.1:6379> multi
    OK
    

客户端2：

    
    
    127.0.0.1:6379> append key python
    (integer) 10
    

客户端1：

    
    
    127.0.0.1:6379> append key jedis
    QUEUED
    127.0.0.1:6379> exec
    (nil)
    127.0.0.1:6379> get key
    "javapython"
    

`Redis`提供了简单的事务，之所以说它简单，主要是因为它不支持事务中的回滚特性，同时无法实现命令之间的逻辑关系计算。

##### Redis与Lua

###### 在Redis中使用Lua

在`Redis`中执行`Lua`脚本有两种方法：`eval`和`evalsha`。

**eval**

eval 脚本内容 key个数 key列表 参数列表

    
    
    127.0.0.1:6379> eval 'return "hello " .. KEYS[1] .. ARGV[1]' 1 redis world
    "hello redisworld"
    

此时KEYS[1]=”redis”，ARGV[1]=”world”，所以最终的返回结果是”hello redisworld”。

如果`Lua`脚本较长，还可以使用`redis-cli--eval`直接执行文件。

`eval`命令和`--eval`参数本质是一样的，客户端如果想执行`Lua`脚本，首先在客户端编写好`Lua`脚本代码，然后把脚本作为字符串发送给服务端，服务端会将执行结果返回给客户端。

**evalsha**

除了使用`eval`，`Redis`还提供了`evalsha`命令来执行`Lua`脚本。首先要将`Lua`脚本加载到`Redis`服务端，得到该脚本的`SHA1`校验和，`evalsha`命令使用`SHA1`作为参数可以直接执行对应`Lua`脚本，避免每次发送`Lua`脚本的开销。这样客户端就不需要每次执行脚本内容，而脚本也会常驻在服务端，脚本功能得到了复用。

**加载脚本：** `script
load`命令可以将脚本内容加载到`Redis`内存中，例如下面将`lua_get.lua`加载到`Redis`中，得到`SHA1`：

    
    
    [heql@ubuntu ~]$ redis-cli script load "$(cat lua_get.lua)"
    "7413dc2440db1fea7c0a0bde841fa68eefaf149c"
    

**执行脚本：** `evalsha`的使用方法如下，参数使用`SHA1`值，执行逻辑和`eval`一致。

evalsha 脚本SHA1值 key个数 key列表 参数列表

    
    
    127.0.0.1:6379> evalsha 7413dc2440db1fea7c0a0bde841fa68eefaf149c 1 redis world
    "hello redisworld"
    

###### Lua的Redis API

`Lua`可以使用`redis.call`函数实现对`Redis`的访问：

    
    
    127.0.0.1:6379> eval 'return redis.call("set", KEYS[1], ARGV[1])' 1 hello world
    OK
    127.0.0.1:6379> eval 'return redis.call("get", KEYS[1])' 1 hello
    "world"
    

除此之外`Lua`还可以使用`redis.pcall`函数实现对Redis的调用，`redis.call`和`redis.pcall`的不同在于，如果`redis.call`执行失败，那么脚本执行结束会直接返回错误，而`redis.pcall`会忽略错误继续执行脚本。

`Lua`脚本功能为`Redis`开发和运维人员带来如下三个好处：

  * `Lua`脚本在`Redis`中是原子执行的，执行过程中间不会插入其他命令。
  * `Lua`脚本可以帮助开发和运维人员创造出自己定制的命令，并可以将这些命令常驻在`Redis`内存中，实现复用的效果。
  * `Lua`脚本可以将多条命令一次性打包，有效地减少网络开销。

##### Redis如何管理Lua脚本

`Redis`提供了4个命令实现对`Lua`脚本的管理：

###### script load

此命令用于将`Lua`脚本加载到`Redis`内存中。

###### script exists

scripts exists sha1 [sha1 …]

此命令用于判断`sha1`是否已经加载到`Redis`内存中：

    
    
    127.0.0.1:6379> script exists a5260dd66ce02462c5b5231c727b3f7772c0bcc5
    1) (integer) 1
    

返回结果代表`sha1[sha1…]`被加载到`Redis`内存的个数。

###### script flush

此命令用于清除`Redis`内存已经加载的所有`Lua`脚本。

###### script kill

此命令用于杀掉正在执行的`Lua`脚本。如果`Lua`脚本比较耗时，甚至`Lua`脚本存在问题，那么此时`Lua`脚本的执行会阻塞Redis，直到脚本执行完毕或者外部进行干预将其结束。

执行`Lua`脚本，进入死循环，当前客户端会阻塞：

    
    
    127.0.0.1:6379> eval 'while 1==1 do end' 0
    

`Redis`提供了一个`lua-time-
limit`参数，默认是5秒，它是`Lua`脚本的“超时时间”，但这个超时时间仅仅是当`Lua`脚本时间超过`lua-time-
limit`后，向其他命令调用发送`BUSY`的信号，但是并不会停止掉服务端和客户端的脚本执行，所以当达到`lua-time-
limit`值之后，其他客户端在执行正常的命令时，将会收到`“Busy Redis is busy running a
script”`错误，并且提示使用`script kill`或者`shutdown nosave`命令来杀掉这个busy的脚本：

    
    
    127.0.0.1:6379> get hello
    (error) BUSY Redis is busy running a script. You can only call SCRIPT KILL or SHUTDOWN NOSAVE.
    

但是有一点需要注意，如果当前Lua脚本正在执行写操作，那么`script kill`将不会生效：

    
    
    127.0.0.1:6379> eval 'while 1 == 1 do redis.call("set", "k", "v") end' 0
    

此时如果执行`script kill`，会收到如下异常信息：

    
    
    127.0.0.1:6379> script kill
    (error) UNKILLABLE Sorry the script already executed write commands against the dataset. You can either wait the script termination or kill the server in a hard way using the SHUTDOWN NOSAVE command.
    

上面提示`Lua`脚本正在向`Redis`执行写命令，要么等待脚本执行结束要么使用`shutdown save`停掉`Redis`服务。

#### Bitmaps

`Redis`提供了`Bitmaps`这个可以实现对位的操作：

  * `Bitmaps`本身不是一种数据结构，实际上它就是字符串，但是它可以对字符串的位进行操作。
  * 在`Redis`中使用`Bitmaps`和使用字符串的方法不太相同。可以把`Bitmaps`想象成一个以位为单位的数组，数组的每个单元只能存储0和1，数组的下标在`Bitmaps`中叫做偏移量。

##### 命令

###### 设置值

setbit key offset value

    
    
    127.0.0.1:6379> setbit a 0 1
    (integer) 0
    127.0.0.1:6379> setbit a 5 1
    (integer) 0
    127.0.0.1:6379> setbit a 11 1
    (integer) 0
    127.0.0.1:6379> setbit a 15 1
    (integer) 0
    127.0.0.1:6379> setbit a 19 1
    (integer) 0
    

在第一次初始化`Bitmaps`时，假如偏移量非常大，那么整个初始化过程执行会比较慢，可能会造成`Redis`的阻塞。

###### 获取值

gitbit key offset

返回0说明没有设置过或`offset`根本不存在：

    
    
    127.0.0.1:6379> getbit a 19
    (integer) 1
    127.0.0.1:6379> getbit a 200
    (integer) 0
    

###### 获取Bitmaps指定范围值为1的个数

bitcount [start][end]

`[start]`和`[end]`代表起始和结束字节数：

    
    
    127.0.0.1:6379> bitcount a 
    (integer) 5
    127.0.0.1:6379> bitcount a 1 3
    (integer) 3
    

###### Bitmaps间的运算

bitop op destkey key[key….]

`bitop`是一个复合操作，它可以做多个`Bitmaps`的and（交集）、or（并集）、not（非）、xor（异或）操作并将结果保存在`destkey`中：

    
    
    127.0.0.1:6379> bitop and c a b 
    (integer) 2
    

###### 计算Bitmaps中第一个值为targetBit的偏移量

bitpos key targetBit [start] [end]

    
    
    127.0.0.1:6379> bitpos a 1
    (integer) 1
    

`[start]`和`[end]`代表起始和结束字节数。例如计算第0个字节到第1个字节之间，第一个值为0的偏移量：

    
    
    127.0.0.1:6379> bitpos a 0 0 1
    (integer) 1
    

#### HyperLogLog

`HyperLogLog`并不是一种新的数据结构（实际类型为字符串类型），而是一种基数算法，通过`HyperLogLog`可以利用极小的内存空间完成独立总数的统计。

##### 添加

`pfadd`用于向`HyperLogLog`添加元素，如果添加成功返回1：

    
    
    127.0.0.1:6379> pfadd ids "uuid-1" "uuid-2" "uuid-3" "uuid-4"
    (integer) 1
    

##### 计算独立用户数

`pfcount`用于计算一个或多个`HyperLogLog`的独立总数：

    
    
    127.0.0.1:6379> pfcount ids
    (integer) 4
    

##### 合并

pfmerge destkey sourcekey [sourcekey …]

`pfmerge`可以求出多个`HyperLogLog`的并集并赋值给`destkey`:

    
    
    127.0.0.1:6379> pfadd ids_1 "uuid-1" "uuid-2" "uuid-3" "uuid-4"
    (integer) 1
    127.0.0.1:6379> pfadd ids_2 "uuid-4" "uuid-5" "uuid-6" "uuid-7"
    (integer) 1
    127.0.0.1:6379> pfmerge ids_3 ids_1 ids_2
    OK
    127.0.0.1:6379> pfcount ids_3
    (integer) 7
    

`HyperLogLog`内存占用量非常小，但是存在错误率，选取使用`HyperLogLog`应当确认：

  * 只为了计算独立总数，不需要获取单条数据。
  * 可以容忍一定误差率，毕竟`HyperLogLog`在内存的占用量上有很大的优势。

#### 发布订阅

`Redis`提供了基于“发布/订阅”模式的消息机制，此种模式下，消息发布者和订阅者不进行直接通信，发布者客户端向指定的频道（channel）发布消息，订阅该频道的每个客户端都可以收到该消息。

##### 命令

`Redis`主要提供了发布消息、订阅频道、取消订阅以及按照模式订阅和取消订阅等命令。

###### 发布消息

publish channel message

返回结果为订阅者个数:

    
    
    127.0.0.1:6379> publish channel:sports "Tim won the championship"
    (integer) 0
    

###### 订阅消息

subscribe channel [channel …]

订阅者可以订阅一个或多个频道:

    
    
    127.0.0.1:6379> subscribe channel:sports
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"
    2) "channel:sports"
    3) (integer) 1
    

此时另一个客户端发布一条消息：

    
    
    27.0.0.1:6379> publish channel:sports "James lost the championship"
    (integer) 1
    

当前订阅者客户端会收到如下消息：

    
    
    127.0.0.1:6379> subscribe channel:sports
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"
    2) "channel:sports"
    3) (integer) 1
    1) "message"
    2) "channel:sports"
    3) "James lost the championship"
    

有关订阅命令有两点需要注意：

  * 客户端在执行订阅命令之后进入了订阅状态，只能接收`subscribe`、`psubscribe`、`unsubscribe`、`punsubscribe`的四个命令。
  * 新开启的订阅客户端，无法收到该频道之前的消息，因为`Redis`不会对发布的消息进行持久化。

###### 取消订阅

客户端可以通过`unsubscribe`命令取消对指定频道的订阅，取消成功后，不会再收到该频道的发布消息：

    
    
    127.0.0.1:6379> unsubscribe channel:sports
    1) "unsubscribe"
    2) "channel:sports"
    3) (integer) 0
    

###### 按照模式订阅和取消订阅

psubscribe pattern [pattern…]  
punsubscribe [pattern [pattern …]]

除了`subcribe`和`unsubscribe`命令，`Redis`命令还支持`glob`风格的订阅命令`psubscribe`和取消订阅命令`punsubscribe`，例如下面操作订阅以it开头的所有频道：

    
    
    127.0.0.1:6379> psubscribe it*
    Reading messages... (press Ctrl-C to quit)
    1) "psubscribe"
    2) "it*"
    3) (integer) 1
    

###### 查询订阅

 **查看活跃的频道：**

pubsub channels [pattern]

所谓活跃的频道是指当前频道至少有一个订阅者，其中`[pattern]`是可以指定具体的模式：

    
    
    127.0.0.1:6379> pubsub channels
    1) "channel:sports"
    127.0.0.1:6379> 
    127.0.0.1:6379> pubsub channels channel:*s
    1) "channel:sports"
    

**查看频道订阅数：**

pubsub numsub [channel …]

    
    
    127.0.0.1:6379> pubsub numsub channel:sports
    1) "channel:sports"
    2) (integer) 1