# 数据类型

考察点：主要是看你有没有全面的了解过redis，有哪些数据类型，适用于什么场景，实际项目里的具体使用等？

## 常见数据类型

### **String**

> 常⽤命令: set,get,decr,incr,mget 等。

String数据结构是简单的key-value类型，value其实不仅可以是String，也可以是数字，一个字符串允许存储的最大容量是512MB。 

底层实现：

常规keyvalue缓存应⽤； 常规计数：微博数，粉丝数等。

### **Hash**

> 常⽤命令： hget,hset,hgetall 等。

类似java中的Map，一般可以将**结构化**的数据，比如一个对象（前提是**这个对象没有嵌套其他的对象**）给缓存在redis，每次读写缓存，可以操作相应的**某个字段**。

### **List**

> 常⽤命令: lpush,rpush,lpop,rpop,lrange等

list 就是链表，Redis list 的应⽤场景⾮常多，也是Redis最重要的数据结构之⼀，⽐如微博的关注列表，粉丝列表，消息列表等功能都可以⽤Redis的 list 结构来实现。 Redis list 的实现为⼀个**双向链表**，即可以⽀持反向查找和遍历，更⽅便操作，不过带来了部分额外 的内存开销。 另外可以通过 lrange 命令，就是从某个元素开始读取多少个元素，可以基于list实现**分⻚查询**，这 个很棒的⼀个功能，基于 redis 实现简单的⾼性能分⻚，可以做类似微博那种下拉不断分⻚的东⻄ （⼀⻚⼀⻚的往下⾛），性能⾼。

### **Set**

> 常⽤命令： sadd,spop,smembers,sunion 等

无序集合，自动去重

可以进行一些交集、并集、差集的操作，比如共同好友等

```
sinterstore key1 key2 key3 将交集存在key1内
```

### **ZSet**

> 常⽤命令： zadd,zrange,zrem,zcard等

有序集合，自动去重，写进去的时候给一个分数，自动根据分数排序

有序集合使用散列表、跳表实现，读取时间复杂度为O(log(n))，比列表更耗费内存

举例： 在直播系统中，实时排⾏信息包含直播间在线⽤户列表，各种礼物排⾏榜，弹幕消息（可以理 解为按消息维度的消息排⾏榜）等信息，适合使⽤ Redis 中的 Sorted Set 结构进⾏存储。

## **其他数据类型**

### **Bitmaps**

### HyperLogLogs

### Streams 

# 线程模型

# 持久化

# 过期策略



# 事务

# 主从复制

# 哨兵机制

**哨兵的介绍**

sentinel，中文名是哨兵。哨兵是 Redis 集群架构中非常重要的一个组件，主要有以下功能：

- 集群监控：负责监控 Redis master 和 slave 进程是否正常工作。
- 消息通知：如果某个 Redis 实例有故障，那么哨兵负责发送消息作为报警通知给管理员。
- 故障转移：如果 master node 挂掉了，会自动转移到 slave node 上。
- 配置中心：如果故障转移发生了，通知 client 客户端新的 master 地址。

哨兵用于实现 Redis 集群的高可用，本身也是分布式的，作为一个哨兵集群去运行，互相协同工作。

- 故障转移时，判断一个 master node 是否宕机了，需要大部分的哨兵都同意才行，涉及到了分布式选举的问题。
- 即使部分哨兵节点挂掉了，哨兵集群还是能正常工作的，因为如果一个作为高可用机制重要组成部分的故障转移系统本身是单点的，那就很坑爹了。

**哨兵的核心知识**

- 哨兵至少需要 3 个实例，来保证自己的健壮性。
- 哨兵 + Redis 主从的部署架构，是**不保证数据零丢失**的，只能保证 Redis 集群的高可用性。
- 对于哨兵 + Redis 主从这种复杂的部署架构，尽量在测试环境和生产环境，都进行充足的测试和演练。

哨兵集群必须部署 2 个以上节点，如果哨兵集群仅仅部署了 2 个哨兵实例，quorum = 1。

```
+----+         +----+
| M1 |---------| R1 |
| S1 |         | S2 |
+----+         +----+Copy to clipboardErrorCopied
```

配置 `quorum=1` ，如果 master 宕机， s1 和 s2 中只要有 1 个哨兵认为 master 宕机了，就可以进行切换，同时 s1 和 s2 会选举出一个哨兵来执行故障转移。但是同时这个时候，需要 majority，也就是大多数哨兵都是运行的。

```
2 个哨兵，majority=2
3 个哨兵，majority=2
4 个哨兵，majority=2
5 个哨兵，majority=3
...Copy to clipboardErrorCopied
```

如果此时仅仅是 M1 进程宕机了，哨兵 s1 正常运行，那么故障转移是 OK 的。但是如果是整个 M1 和 S1 运行的机器宕机了，那么哨兵只有 1 个，此时就没有 majority 来允许执行故障转移，虽然另外一台机器上还有一个 R1，但是故障转移不会执行。

经典的 3 节点哨兵集群是这样的：

```
       +----+
       | M1 |
       | S1 |
       +----+
          |
+----+    |    +----+
| R2 |----+----| R3 |
| S2 |         | S3 |
+----+         +----+Copy to clipboardErrorCopied
```

配置 `quorum=2` ，如果 M1 所在机器宕机了，那么三个哨兵还剩下 2 个，S2 和 S3 可以一致认为 master 宕机了，然后选举出一个来执行故障转移，同时 3 个哨兵的 majority 是 2，所以还剩下的 2 个哨兵运行着，就可以允许执行故障转移。

**Redis 哨兵主备切换的数据丢失问题**

**导致数据丢失的两种情况：**

主备切换的过程，可能会导致数据丢失：

- 异步复制导致的数据丢失

因为 master->slave 的复制是异步的，所以可能有部分数据还没复制到 slave，master 就宕机了，此时这部分数据就丢失了。

- 脑裂导致的数据丢失

脑裂，也就是说，某个 master 所在机器突然**脱离了正常的网络**，跟其他 slave 机器不能连接，但是实际上 master 还运行着。此时哨兵可能就会**认为**master 宕机了，然后开启选举，将其他 slave 切换成了 master。这个时候，集群里就会有两个 master ，也就是所谓的**脑裂**。

此时虽然某个 slave 被切换成了 master，但是可能 client 还没来得及切换到新的 master，还继续向旧 master 写数据。因此旧 master 再次恢复的时候，会被作为一个 slave 挂到新的 master 上去，自己的数据会清空，重新从新的 master 复制数据。而新的 master 并没有后来 client 写入的数据，因此，这部分数据也就丢失了。

**数据丢失问题的解决方案**

进行如下配置：

```bash
min-slaves-to-write 1
min-slaves-max-lag 10Copy to clipboardErrorCopied
```

表示，要求至少有 1 个 slave，数据复制和同步的延迟不能超过 10 秒。

如果说一旦所有的 slave，数据复制和同步的延迟都超过了 10 秒钟，那么这个时候，master 就不会再接收任何请求了。

- 减少异步复制数据的丢失

有了 `min-slaves-max-lag` 这个配置，就可以确保说，一旦 slave 复制数据和 ack 延时太长，就认为可能 master 宕机后损失的数据太多了，那么就拒绝写请求，这样可以把 master 宕机时由于部分数据未同步到 slave 导致的数据丢失降低的可控范围内。

- 减少脑裂的数据丢失

如果一个 master 出现了脑裂，跟其他 slave 丢了连接，那么上面两个配置可以确保说，如果不能继续给指定数量的 slave 发送数据，而且 slave 超过 10 秒没有给自己 ack 消息，那么就直接拒绝客户端的写请求。因此在脑裂场景下，最多就丢失 10 秒的数据。

**sdown 和 odown 转换机制**

- sdown 是主观宕机，就一个哨兵如果自己觉得一个 master 宕机了，那么就是主观宕机
- odown 是客观宕机，如果 quorum 数量的哨兵都觉得一个 master 宕机了，那么就是客观宕机

sdown 达成的条件很简单，如果一个哨兵 ping 一个 master，超过了 `is-master-down-after-milliseconds` 指定的毫秒数之后，就主观认为 master 宕机了；如果一个哨兵在指定时间内，收到了 quorum 数量的其它哨兵也认为那个 master 是 sdown 的，那么就认为是 odown 了。

**哨兵集群的自动发现机制**

哨兵互相之间的发现，是通过 Redis 的 `pub/sub` 系统实现的，每个哨兵都会往 `__sentinel__:hello` 这个 channel 里发送一个消息，这时候所有其他哨兵都可以消费到这个消息，并感知到其他的哨兵的存在。

每隔两秒钟，每个哨兵都会往自己监控的某个 master+slaves 对应的 `__sentinel__:hello` channel 里**发送一个消息**，内容是自己的 host、ip 和 runid 还有对这个 master 的监控配置。

每个哨兵也会去**监听**自己监控的每个 master+slaves 对应的 `__sentinel__:hello` channel，然后去感知到同样在监听这个 master+slaves 的其他哨兵的存在。

每个哨兵还会跟其他哨兵交换对 `master` 的监控配置，互相进行监控配置的同步。

**slave 配置的自动纠正]**

哨兵会负责自动纠正 slave 的一些配置，比如 slave 如果要成为潜在的 master 候选人，哨兵会确保 slave 复制现有 master 的数据；如果 slave 连接到了一个错误的 master 上，比如故障转移之后，那么哨兵会确保它们连接到正确的 master 上。

**slave->master 选举算法**

如果一个 master 被认为 odown 了，而且 majority 数量的哨兵都允许主备切换，那么某个哨兵就会执行主备切换操作，此时首先要选举一个 slave 来，会考虑 slave 的一些信息：

- 跟 master 断开连接的时长
- slave 优先级
- 复制 offset
- run id

如果一个 slave 跟 master 断开连接的时间已经超过了 `down-after-milliseconds` 的 10 倍，外加 master 宕机的时长，那么 slave 就被认为不适合选举为 master。

```
(down-after-milliseconds * 10) + milliseconds_since_master_is_in_SDOWN_stateCopy to clipboardErrorCopied
```

接下来会对 slave 进行排序：

- 按照 slave 优先级进行排序，slave priority 越低，优先级就越高。
- 如果 slave priority 相同，那么看 replica offset，哪个 slave 复制了越多的数据，offset 越靠后，优先级就越高。
- 如果上面两个条件都相同，那么选择一个 run id 比较小的那个 slave。

**quorum 和 majority**

每次一个哨兵要做主备切换，首先需要 quorum 数量的哨兵认为 odown，然后选举出一个哨兵来做切换，这个哨兵还需要得到 majority 哨兵的授权，才能正式执行切换。

如果 quorum < majority，比如 5 个哨兵，majority 就是 3，quorum 设置为 2，那么就 3 个哨兵授权就可以执行切换。

但是如果 quorum >= majority，那么必须 quorum 数量的哨兵都授权，比如 5 个哨兵，quorum 是 5，那么必须 5 个哨兵都同意授权，才能执行切换。

**configuration epoch**

哨兵会对一套 Redis master+slaves 进行监控，有相应的监控的配置。

执行切换的那个哨兵，会从要切换到的新 master（salve->master）那里得到一个 configuration epoch，这就是一个 version 号，每次切换的 version 号都必须是唯一的。

如果第一个选举出的哨兵切换失败了，那么其他哨兵，会等待 failover-timeout 时间，然后接替继续执行切换，此时会重新获取一个新的 configuration epoch，作为新的 version 号。

**configuration 传播**

哨兵完成切换之后，会在自己本地更新生成最新的 master 配置，然后同步给其他的哨兵，就是通过之前说的 `pub/sub` 消息机制。

这里之前的 version 号就很重要了，因为各种消息都是通过一个 channel 去发布和监听的，所以一个哨兵完成一次新的切换之后，新的 master 配置是跟着新的 version 号的。其他的哨兵都是根据版本号的大小来更新自己的 master 配置的。

# 集群

# 分区

# 分布式









# 补充

待解决问题：分片问题中的hash_tag、虚拟槽

**什么是Redis？Redis有哪些优缺点  **

缓存（CPU高速缓存、内存）、特性（丰富的数据结构、支持事务、持久化、LRU驱动事件、多种集群方案  等）

每秒可以处理超过 10万次读写操作  

优点：读写性能优异 、支持数据持久化、支持事务、数据结构丰富、支持主从

缺点：容量受到物理主机的限制、Redis不具备自动容错和恢复功能、宕机有可能导致数据不一致、Redis较难支持在线扩容

**Redis为什么这么快？**



**Redis持久化数据如何扩容？**

一致性哈希（缓存）、集群（需持久化）



# Q&A

## **在项目中缓存是如何使用的？**

考察点：你的项目中是如何使用缓存的？ 为什么要使用缓存？ 不用缓存行不行？缓存使用不当会造成什么问题？

如何使用缓存根据自身的项目以及Redis的特性来回答

主要是基于**高性能**、**高并发**的考量，对于一些频繁操作或者mysql耗时且长时间不会变的数据都可以考虑用redis替代，内存的读写速度更快，且单机支撑并发量轻松10w+；

缓存分为本地缓存和分布式缓存。以 Java 为例，使⽤⾃带的 map 或者 guava 实现的是本地缓存，最 主要的特点是轻量以及快速，⽣命周期随着 jvm 的销毁⽽结束，并且在多实例的情况下，每个实例都 需要各⾃保存⼀份缓存，缓存不具有⼀致性。 使⽤ redis 或 memcached 之类的称为分布式缓存，在多实例的情况下，各实例共⽤⼀份缓存数据，缓 存具有⼀致性。缺点是需要保持 redis 或 memcached服务的⾼可⽤，整个程序架构上᫾为复杂。

缓存常见的问题有**缓存与数据库双写不一致**、**缓存雪崩**、**缓存穿透**、**缓存击穿**、**缓存并发竞争**等问题

## **Redis的雪崩、穿透和击穿，如何应对？**

考察点：什么是Redis的雪崩、穿透和击穿？发生这些情况之后会怎么样？系统如何应对这种情况？

**缓存雪崩**

缓存雪崩：缓存中数据大批量达到过期时间或者缓存机器以外宕机，此时查询数据库压力过大，导致数据库宕机，重启之后，又会被新的流量给打死。

应对方法：

- 事前：Redis 高可用，主从+哨兵，Redis cluster，避免全盘崩溃。
- 事中：本地 ehcache 缓存 + hystrix 限流&降级，避免 MySQL 被打死。
- 事后：Redis 持久化，一旦重启，自动从磁盘上加载数据，快速恢复缓存数据

![缓存雪崩应对方案](../picture/redis-caching-avalanche-solution.png)

**缓存穿透**

是指缓存和数据库中并**不存在**的数据，却被不断发起请求进行查询，导致数据库压力过大。

**应对方法**：最基本的就是⾸先做好参数校验，⼀些不合法的参数请求直接抛出异常信息返回给客户端。⽐如查询的 数据库 id 不能⼩于 0、传⼊的邮箱格式不对的时候直接返回错误消息给客户端等等。

1）**缓存无效 key** : 如果缓存和数据库都查不到某个 key 的数据就写⼀个到 redis 中去并设置过期时间，具体命令如下： SET key value EX 10086 。

这种⽅式可以解决请求的key变化不频繁的情况，如果⿊客恶意攻击，每次构建不同的请求key，会导致 redis 中缓存⼤量⽆效的 key 。

很明显， 这种⽅案并不能从根本上解决此问题。如果⾮要⽤这种⽅式来解决穿透问题的话，尽量将⽆效的key的过期时间设置短⼀点⽐如1分钟。 另外，⼀般情况下我们是这样设计 key 的：表名:列名:主键名:主键值 。

2）**布隆过滤器**：通过它我们可以⾮常⽅便地判断⼀个给定数据是否存在于海量数据中。具体是这样做的：把所有可能存在的请求的值都存放在布隆过滤器中，当⽤户请求过来，我会先判断⽤户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信 息给客户端，存在的话才会⾛下⾯的流程。

- 请求数据的 key 不存在于布隆过滤器中，可以确定数据就一定不会存在于数据库中，系统可以立即返回不存在。
- 请求数据的 key 存在于布隆过滤器中，则继续再向缓存中查询。

**缓存击穿**

是指缓存中没有但数据库中有的数据，某些key**并发访问非常大**，当在缓存失效瞬间，大量的请求就击穿了缓存，同时集中式高并发的访问数据库，这时数据库的压力就会很大。

应对方法：

- 若缓存的数据是基本不会发生更新的，则可尝试将该热点数据设置为永不过期。
- 若缓存的数据更新不频繁，且缓存刷新的整个流程耗时较少的情况下，则可以采用基于 Redis、zookeeper 等分布式中间件的分布式互斥锁，或者本地互斥锁以保证仅少量的请求能请求数据库并重新构建缓存，其余线程则在锁释放后能访问到新缓存。
- 若缓存的数据更新频繁或者在缓存刷新的流程耗时较长的情况下，可以利用定时线程在缓存过期前主动地重新构建缓存或者延后缓存的过期时间，以保证所有的请求能一直访问到对应的缓存。

## **如何保证缓存与数据库双写一致性？**

考察点：使用了缓存就可能会出现缓存与数据库存储双写，双写就会有数据一致性问题，若严格要求数据一致性，则读请求和写请求串行化，串到一个内存队列中去，这样可以保证数据一致性，但系统吞吐量会大幅度降低，需要部署更多的机器去支撑线上的一个请求

最经典的缓存+数据库读写的模式是Cache Aside Pattern。即：

- 读的时候，先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回响应。
- 更新的时候，**先更新数据库，然后再删除缓存**。------**懒加载**

假如更新缓存的代价很高，如对于比较复杂的缓存数据计算，一个缓存可能涉及多张表，缓存也频繁更新，但这个缓存可能并不会被频繁访问，所以此时采取了用到缓存才去算缓存，即懒加载。

Q1：先更新数据库，再删除缓存，此时删除缓存失败，即数据库中是新数据，缓存中是旧数据，导致数据不一致

S1：先删除缓存，再更新数据库。若数据库更新失败，再次查询缓存为空，去读数据库中的旧数据，然后更新到缓存，不存在不一致。

S2：延时双删。先更新数据库，再删除缓存，不同的是，这个删除的动作，在不久之后再执行一次，如5s之后，延时删除可以选择DelayQueue、MQ，需综合各种因素来设计，从而给出一个合理方案。

Q2：数据发生变更，此时先删除了缓存，在更新数据库之前，一个请求过来，去读缓存，而此时缓存为空，则去查询数据库，查到旧数据并将其放入缓存，随后数据变更更新了数据库的值，此时缓存跟数据库的值不一致

S1：更新数据时，根据数据的唯一标识，将操作路由之后，发送到一个jvm队列。当读取数据时，若数据不在缓存中，那么将重新执行“读取数据+更新缓存”的操作，根据唯一标识路由之后，也发送到同一个jvm内部队列中。

优化点：一个队列中，其实**多个更新缓存请求串在一起是没意义的**，因此可以做过滤，如果发现队列中已经有一个更新缓存的请求了，那么就不用再放个更新请求操作进去了，直接等待前面的更新操作请求完成即可.

待那个队列对应的工作线程完成了上一个操作的数据库的修改之后，才会去执行下一个操作，也就是缓存更新的操作，此时会从数据库中读取最新的值，然后写入缓存中。如果请求还在等待时间范围内，不断轮询发现可以取到值了，那么就直接返回；如果请求等待的时间超过一定时长，那么这一次直接从数据库中读取当前的旧值。

高并发的场景下可能会导致的问题：待补充详细

- 读请求长时阻塞
- 读请求并发量过高
- 多服务实例部署的请求路由
- 热点商品的路由问题，导致请求的倾斜

## **如何解决Redis的并发竞争问题？（参考其他解决方法）**

考察点：Redis的并发竞争问题是什么？如何解决这个问题？了解Redis事务的CAS方案吗？

并发竞争问题：多客户端同时并发写一个key，可能本来应该先到的数据后到了，导致数据版本错了；或者多客户端同时获取一个key，修改值之后再写回去，顺序一错，数据也就错了

S1:可以基于**zookeeper实现分布式锁**，每个系统通过zk获取分布式锁，确保同一时间只能有一个系统实例在操作某个key，别人都不允许读和写。

基于zookeeper临时有序节点可以实现的分布式锁。⼤致思想为：每个客户端对某个⽅法加锁时，在zookeeper上的与该⽅法对应的指定节点的⽬录下，⽣成⼀个唯⼀的瞬时有序节点。 判断是否获取锁的 ⽅式很简单，只需要判断有序节点中序号最⼩的⼀个。 当释放锁的时候，只需将这个瞬时节点删除即 可。同时，其可以避免服务宕机导致的锁⽆法释放，⽽产⽣的死锁问题。完成业务流程后，删除对应的⼦节点释放锁。

S2:你要写入缓存的数据，都是从 mysql 里查出来的，都得写入 mysql 中，写入 mysql 中的时候必须保存一个时间戳，从 mysql 查出来的时候，时间戳也查出来。

S3:每次要写之前，先判断一下当前这个 value 的时间戳是否比缓存里的 value 的时间戳要新。如果是的话，那么可以写，否则，就不能用旧的数据覆盖新的数据

## **Redis和Memcached有什么区别？**

考察点：Redis和Memcached有什么区别？Redis的线程模型是什么？为什么Redis单线程却能支持高并发？

**Redis和Memcached的区别**

- redis⽀持更丰富的数据类型（⽀持更复杂的应⽤场景）：Redis不仅仅⽀持简单的k/v类型的数据，同时还提供list，set，zset，hash等数据结构的存储。memcache⽀持简单的数据类型，String。
- Redis⽀持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进⾏使 ⽤,⽽Memecache把数据全部存在内存之中。
- 原⽣支持集群模式，memcached没有原⽣的集群模式，需要依靠客户端来实现往集群中分⽚写⼊数据。
- Memcached是多线程，⾮阻塞IO复⽤的⽹络模型；Redis使⽤单线程的多路 IO 复⽤模型。

**性能对比**

Redis使用单核，而Memcached使用多核，所以在存储小数据时Redis性能更高，而在100k以上的数据中，Memcached性能更高。

## **Redis的线程模型**

Redis 内部使用文件事件处理器 `file event handler` ，这个文件事件处理器是单线程的，所以 Redis 才叫做单线程的模型。它采用 IO 多路复用机制同时监听多个 socket，将产生事件的 socket 压入内存队列中，事件分派器根据 socket 上的事件类型来选择对应的事件处理器进行处理。

文件事件处理器的结构包含 4 个部分：

- 多个 socket
- IO 多路复用程序
- 文件事件分派器
- 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

多个 socket 可能会并发产生不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 socket，会将产生事件的 socket 放入队列中排队，事件分派器每次从队列中取出一个 socket，根据 socket 的事件类型交给对应的事件处理器进行处理。

来看客户端与 Redis 的一次通信过程：

![Redis-single-thread-model](https://doocs.github.io/advanced-java/docs/high-concurrency/images/redis-single-thread-model.png)

首先，Redis 服务端进程初始化的时候，会将 server socket 的 `AE_READABLE` 事件与连接应答处理器关联。

客户端 socket01 向 Redis 进程的 server socket 请求建立连接，此时 server socket 会产生一个 `AE_READABLE` 事件，IO 多路复用程序监听到 server socket 产生的事件后，将该 socket 压入队列中。文件事件分派器从队列中获取 socket，交给**连接应答处理器**。连接应答处理器会创建一个能与客户端通信的 socket01，并将该 socket01 的 `AE_READABLE` 事件与命令请求处理器关联。

假设此时客户端发送了一个 `set key value` 请求，此时 Redis 中的 socket01 会产生 `AE_READABLE` 事件，IO 多路复用程序将 socket01 压入队列，此时事件分派器从队列中获取到 socket01 产生的 `AE_READABLE` 事件，由于前面 socket01 的 `AE_READABLE` 事件已经与命令请求处理器关联，因此事件分派器将事件交给命令请求处理器来处理。命令请求处理器读取 socket01 的 `key value` 并在自己内存中完成 `key value` 的设置。操作完成后，它会将 socket01 的 `AE_WRITABLE` 事件与命令回复处理器关联。

如果此时客户端准备好接收返回结果了，那么 Redis 中的 socket01 会产生一个 `AE_WRITABLE` 事件，同样压入队列中，事件分派器找到相关联的命令回复处理器，由命令回复处理器对 socket01 输入本次操作的一个结果，比如 `ok` ，之后解除 socket01 的 `AE_WRITABLE` 事件与命令回复处理器的关联。这样便完成了一次通信

## **为什么Redis单线程性能也能这么高？**

- **纯内存操作**。
- 高效的数据结构及合理的数据编码
- C 语言实现，一般来说，C 语言实现的程序“距离”操作系统更近，执行速度相对会更快。
- 线程模型，**单线程**反而避免了多线程的频繁上下文切换问题，预防了多线程可能产生的竞争问题；同时采用了**多路I/O复用模型**来处理事件。

## Redis的过期策略都有哪些？

Redis 的过期策略都有哪些？内存淘汰机制都有哪些？手写一下 LRU 代码实现？

内存是有限的，并且Redis的删除策略决定哪些数据应该被删除。

**Redis过期策略**：**定期删除+惰性删除**

Redis默认每隔100ms就**随机抽取**一些设置了过期时间的key，**检查其是否过期**，若过期则删除。

**内存淘汰机制**

- noeviction: 当内存不足以容纳新写入数据时，新写入操作会报错，极少使用。
- **allkeys-lru**：当内存不足以容纳新写入数据时，在**键空间**中，移除最近最少使用的 key（这个是**最常用**的）。
- allkeys-random：当内存不足以容纳新写入数据时，在**键空间**中，随机移除某个 key。
- volatile-lru：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，移除最近最少使用的 key（这个一般不太合适）。
- volatile-random：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，**随机移除**某个 key。
- volatile-ttl：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，有**更早过期时间**的 key 优先移除。

4.0版本后增加以下两种：

- volatile-lfu：从已设置过期时间的数据集(server.db[i].expires)中挑选最不经常使⽤的数据淘汰
- allkeys-lfu：当内存不⾜以容纳新写⼊数据时，在键空间中，移除最不经常使⽤的key

**手写LRU算法**

```

```

## **Redis主从架构**

单机的 Redis，能够承载的 QPS 大概就在上万到几万不等。对于缓存来说，一般都是用来支撑**读高并发**的。因此架构做成主从(master-slave)架构，一主多从，主负责写，并且将数据复制到其它的 slave 节点，从节点负责读。所有的**读请求全部走从节点**。这样也可以很轻松实现水平扩容，**支撑读高并发**。

Redis replication -> 主从架构 -> 读写分离 -> 水平扩容支撑读高并发

**Redis replication 的核心机制**

- Redis 采用**异步方式**复制数据到 slave 节点，不过 Redis2.8 开始，slave node 会周期性地确认自己每次复制的数据量；
- 一个 master node 是可以配置多个 slave node 的；
- slave node 也可以连接其他的 slave node；
- slave node 做复制的时候，不会 block master node 的正常工作；
- slave node 在做复制的时候，也不会 block 对自己的查询操作，它会用旧的数据集来提供服务；但是复制完成的时候，需要删除旧数据集，加载新数据集，这个时候就会暂停对外服务了；
- slave node 主要用来进行横向扩容，做读写分离，扩容的 slave node 可以提高读的吞吐量。

注意，如果采用了主从架构，那么建议必须**开启** master node 的[持久化](https://doocs.gitee.io/advanced-java/#/docs/high-concurrency/redis-persistence)，不建议用 slave node 作为 master node 的数据热备，因为那样的话，如果你关掉 master 的持久化，可能在 master 宕机重启的时候数据是空的，然后可能一经过复制， slave node 的数据也丢了。

另外，master 的各种备份方案，也需要做。万一本地的所有文件丢失了，从备份中挑选一份 rdb 去恢复 master，这样才能**确保启动的时候，是有数据的**，即使采用了后续讲解的**高可用机制**，slave node 可以自动接管 master node，但也可能 sentinel 还没检测到 master failure，master node 就自动重启了，还是可能导致上面所有的 slave node 数据被清空。

**Redis 主从复制的核心原理**

当启动一个 slave node 的时候，它会发送一个 `PSYNC` 命令给 master node。

如果这是 slave node 初次连接到 master node，那么会触发一次 `full resynchronization` 全量复制。此时 master 会启动一个后台线程，开始生成一份 `RDB` 快照文件，同时还会将从客户端 client 新收到的所有写命令缓存在内存中。 `RDB` 文件生成完毕后， master 会将这个 `RDB` 发送给 slave，slave 会先**写入本地磁盘，然后再从本地磁盘加载到内存**中，接着 master 会将内存中缓存的写命令发送到 slave，slave 也会同步这些数据。slave node 如果跟 master node 有网络故障，断开了连接，会自动重连，连接之后 master node 仅会复制给 slave 部分缺少的数据。

![Redis-master-slave-replication](https://doocs.gitee.io/advanced-java/docs/high-concurrency/images/redis-master-slave-replication.png)

**主从复制的断点续传**

从 Redis2.8 开始，就支持主从复制的断点续传，如果主从复制过程中，网络连接断掉了，那么可以接着上次复制的地方，继续复制下去，而不是从头开始复制一份。

master node 会在内存中维护一个 backlog，master 和 slave 都会保存一个 replica offset 还有一个 master run id，offset 就是保存在 backlog 中的。如果 master 和 slave 网络连接断掉了，slave 会让 master 从上次 replica offset 开始继续复制，如果没有找到对应的 offset，那么就会执行一次 `resynchronization` 。

> 如果根据 host+ip 定位 master node，是不靠谱的，如果 master node 重启或者数据出现了变化，那么 slave node 应该根据不同的 run id 区分。

**无磁盘化复制**

master 在内存中直接创建 `RDB` ，然后发送给 slave，不会在自己本地落地磁盘了。只需要在配置文件中开启 `repl-diskless-sync yes` 即可。

```bash
repl-diskless-sync yes

# 等待 5s 后再开始复制，因为要等更多 slave 重新连接过来
repl-diskless-sync-delay 5Copy to clipboardErrorCopied
```

**过期 key 处理**

slave 不会过期 key，只会等待 master 过期 key。如果 master 过期了一个 key，或者通过 LRU 淘汰了一个 key，那么会模拟一条 del 命令发送给 slave。

**复制的完整流程**

slave node 启动时，会在自己本地保存 master node 的信息，包括 master node 的 `host` 和 `ip` ，但是复制流程没开始。

slave node 内部有个定时任务，每秒检查是否有新的 master node 要连接和复制，如果发现，就跟 master node 建立 socket 网络连接。然后 slave node 发送 `ping` 命令给 master node。如果 master 设置了 requirepass，那么 slave node 必须发送 masterauth 的口令过去进行认证。master node **第一次执行全量复制**，将所有数据发给 slave node。而在后续，master node 持续将写命令，异步复制给 slave node。

![Redis-master-slave-replication-detail](https://doocs.gitee.io/advanced-java/docs/high-concurrency/images/redis-master-slave-replication-detail.png)

**全量复制**

- master 执行 bgsave ，在本地生成一份 rdb 快照文件。
- master node 将 rdb 快照文件发送给 slave node，如果 rdb 复制时间超过 60 秒（repl-timeout），那么 slave node 就会认为复制失败，可以适当调大这个参数(对于千兆网卡的机器，一般每秒传输 100MB，6G 文件，很可能超过 60s)
- master node 在生成 rdb 时，会将所有新的写命令缓存在内存中，在 slave node 保存了 rdb 之后，再将新的写命令复制给 slave node。
- 如果在复制期间，内存缓冲区持续消耗超过 64MB，或者一次性超过 256MB，那么停止复制，复制失败。

```bash
client-output-buffer-limit slave 256MB 64MB 60Copy to clipboardErrorCopied
```

- slave node 接收到 rdb 之后，清空自己的旧数据，然后重新加载 rdb 到自己的内存中，同时**基于旧的数据版本**对外提供服务。
- 如果 slave node 开启了 AOF，那么会立即执行 BGREWRITEAOF，重写 AOF。

**增量复制**

- 如果全量复制过程中，master-slave 网络连接断掉，那么 slave 重新连接 master 时，会触发增量复制。
- master 直接从自己的 backlog 中获取部分丢失的数据，发送给 slave node，默认 backlog 就是 1MB。
- master 就是根据 slave 发送的 psync 中的 offset 来从 backlog 中获取数据的。

**heartbeat**

主从节点互相都会发送 heartbeat 信息。

master 默认每隔 10 秒发送一次 heartbeat，slave node 每隔 1 秒发送一个 heartbeat。

**异步复制**

master 每次接收到写命令之后，先在内部写入数据，然后异步发送给 slave node。

## **Redis集群模式的工作原理**

Redis 集群模式的工作原理能说一下么？在集群模式下，Redis 的 key 是如何寻址的？分布式寻址都有哪些算法？了解一致性 hash 算法吗？

在前几年，Redis 如果要搞几个节点，每个节点存储一部分的数据，得**借助一些中间件**来实现，比如说有 `codis` ，或者 `twemproxy` ，都有。有一些 Redis 中间件，你读写 Redis 中间件，Redis 中间件负责将你的数据分布式存储在多台机器上的 Redis 实例中。

这两年，Redis 不断在发展，Redis 也不断有新的版本，现在的 Redis 集群模式，可以做到在多台机器上，部署多个 Redis 实例，每个实例存储一部分的数据，同时每个 Redis 主实例可以挂 Redis 从实例，自动确保说，如果 Redis 主实例挂了，会自动切换到 Redis 从实例上来。

现在 Redis 的新版本，大家都是用 Redis cluster 的，也就是 Redis 原生支持的 Redis 集群模式，那么面试官肯定会就 Redis cluster 对你来个几连炮。要是你没用过 Redis cluster，正常，以前很多人用 codis 之类的客户端来支持集群，但是起码你得研究一下 Redis cluster 吧。

如果你的数据量很少，主要是承载高并发高性能的场景，比如你的缓存一般就几个 G，单机就足够了，可以使用 replication，一个 master 多个 slaves，要几个 slave 跟你要求的读吞吐量有关，然后自己搭建一个 sentinel 集群去保证 Redis 主从架构的高可用性。

Redis cluster，主要是针对**海量数据+高并发+高可用**的场景。Redis cluster 支撑 N 个 Redis master node，每个 master node 都可以挂载多个 slave node。这样整个 Redis 就可以横向扩容了。如果你要支撑更大数据量的缓存，那就横向扩容更多的 master 节点，每个 master 节点就能存放更多的数据了。

**Redis cluster 介绍**

- 自动将数据进行分片，每个 master 上放一部分数据
- 提供内置的高可用支持，部分 master 不可用时，还是可以继续工作的

在 Redis cluster 架构下，每个 Redis 要放开两个端口号，比如一个是 6379，另外一个就是 加 1w 的端口号，比如 16379。

16379 端口号是用来进行节点间通信的，也就是 cluster bus 的东西，cluster bus 的通信，用来进行故障检测、配置更新、故障转移授权。cluster bus 用了另外一种二进制的协议， `gossip` 协议，用于节点间进行高效的数据交换，占用更少的网络带宽和处理时间。

**节点间的内部通信机制**

**基本通信原理**

集群元数据的维护有两种方式：集中式、Gossip 协议。Redis cluster 节点间采用 gossip 协议进行通信。

**集中式**是将集群元数据（节点信息、故障等等）几种存储在某个节点上。集中式元数据集中存储的一个典型代表，就是大数据领域的 `storm` 。它是分布式的大数据实时计算引擎，是集中式的元数据存储的结构，底层基于 zookeeper（分布式协调的中间件）对所有元数据进行存储维护。

![zookeeper-centralized-storage](https://doocs.gitee.io/advanced-java/docs/high-concurrency/images/zookeeper-centralized-storage.png)

Redis 维护集群元数据采用另一个方式， `gossip` 协议，所有节点都持有一份元数据，不同的节点如果出现了元数据的变更，就不断将元数据发送给其它的节点，让其它节点也进行元数据的变更。

![Redis-gossip](https://doocs.gitee.io/advanced-java/docs/high-concurrency/images/redis-gossip.png)

**集中式**的**好处**在于，元数据的读取和更新，时效性非常好，一旦元数据出现了变更，就立即更新到集中式的存储中，其它节点读取的时候就可以感知到；**不好**在于，所有的元数据的更新压力全部集中在一个地方，可能会导致元数据的存储有压力。

gossip 好处在于，元数据的更新比较分散，不是集中在一个地方，更新请求会陆陆续续打到所有节点上去更新，降低了压力；不好在于，元数据的更新有延时，可能导致集群中的一些操作会有一些滞后。

- 10000 端口：每个节点都有一个专门用于节点间通信的端口，就是自己提供服务的端口号+10000，比如 7001，那么用于节点间通信的就是 17001 端口。每个节点每隔一段时间都会往另外几个节点发送 `ping` 消息，同时其它几个节点接收到 `ping` 之后返回 `pong` 。
- 交换的信息：信息包括故障信息，节点的增加和删除，hash slot 信息等等。

**gossip 协议**

gossip 协议包含多种消息，包含 `ping` , `pong` , `meet` , `fail` 等等。

- meet：某个节点发送 meet 给新加入的节点，让新节点加入集群中，然后新节点就会开始与其它节点进行通信。

```bash
Redis-trib.rb add-nodeCopy to clipboardErrorCopied
```

其实内部就是发送了一个 gossip meet 消息给新加入的节点，通知那个节点去加入我们的集群。

- ping：每个节点都会频繁给其它节点发送 ping，其中包含自己的状态还有自己维护的集群元数据，互相通过 ping 交换元数据。
- pong：返回 ping 和 meeet，包含自己的状态和其它信息，也用于信息广播和更新。
- fail：某个节点判断另一个节点 fail 之后，就发送 fail 给其它节点，通知其它节点说，某个节点宕机啦。

**ping 消息深入**

ping 时要携带一些元数据，如果很频繁，可能会加重网络负担。

每个节点每秒会执行 10 次 ping，每次会选择 5 个最久没有通信的其它节点。当然如果发现某个节点通信延时达到了 `cluster_node_timeout / 2` ，那么立即发送 ping，避免数据交换延时过长，落后的时间太长了。比如说，两个节点之间都 10 分钟没有交换数据了，那么整个集群处于严重的元数据不一致的情况，就会有问题。所以 `cluster_node_timeout` 可以调节，如果调得比较大，那么会降低 ping 的频率。

每次 ping，会带上自己节点的信息，还有就是带上 1/10 其它节点的信息，发送出去，进行交换。至少包含 `3` 个其它节点的信息，最多包含 `总节点数减 2` 个其它节点的信息。

**分布式寻址算法**

- hash 算法（大量缓存重建）
- 一致性 hash 算法（自动缓存迁移）+ 虚拟节点（自动负载均衡）
- Redis cluster 的 hash slot 算法

**hash 算法**

来了一个 key，首先计算 hash 值，然后对节点数取模。然后打在不同的 master 节点上。一旦某一个 master 节点宕机，所有请求过来，都会基于最新的剩余 master 节点数去取模，尝试去取数据。这会导致**大部分的请求过来，全部无法拿到有效的缓存**，导致大量的流量涌入数据库。

![hash](https://doocs.gitee.io/advanced-java/docs/high-concurrency/images/hash.png)

**一致性 hash 算法**

一致性 hash 算法将整个 hash 值空间组织成一个虚拟的圆环，整个空间按顺时针方向组织，下一步将各个 master 节点（使用服务器的 ip 或主机名）进行 hash。这样就能确定每个节点在其哈希环上的位置。

来了一个 key，首先计算 hash 值，并确定此数据在环上的位置，从此位置沿环**顺时针“行走”**，遇到的第一个 master 节点就是 key 所在位置。

在一致性哈希算法中，如果一个节点挂了，受影响的数据仅仅是此节点到环空间前一个节点（沿着逆时针方向行走遇到的第一个节点）之间的数据，其它不受影响。增加一个节点也同理。

然而，一致性哈希算法在节点太少时，容易因为节点分布不均匀而造成**缓存热点**的问题。为了解决这种热点问题，一致性 hash 算法引入了虚拟节点机制，即对每一个节点计算多个 hash，每个计算结果位置都放置一个虚拟节点。这样就实现了数据的均匀分布，负载均衡。

![consistent-hashing-algorithm](https://doocs.gitee.io/advanced-java/docs/high-concurrency/images/consistent-hashing-algorithm.png)

**Redis cluster 的 hash slot 算法**

Redis cluster 有固定的 `16384` 个 hash slot，对每个 `key` 计算 `CRC16` 值，然后对 `16384` 取模，可以获取 key 对应的 hash slot。

Redis cluster 中每个 master 都会持有部分 slot，比如有 3 个 master，那么可能每个 master 持有 5000 多个 hash slot。hash slot 让 node 的增加和移除很简单，增加一个 master，就将其他 master 的 hash slot 移动部分过去，减少一个 master，就将它的 hash slot 移动到其他 master 上去。移动 hash slot 的成本是非常低的。客户端的 api，可以对指定的数据，让他们走同一个 hash slot，通过 `hash tag` 来实现。

任何一台机器宕机，另外两个节点，不影响的。因为 key 找的是 hash slot，不是机器。

![hash-slot](https://doocs.gitee.io/advanced-java/docs/high-concurrency/images/hash-slot.png)

## **Redis cluster 的高可用与主备切换原理**

Redis cluster 的高可用的原理，几乎跟哨兵是类似的。

**判断节点宕机**

如果一个节点认为另外一个节点宕机，那么就是 `pfail` ，**主观宕机**。如果多个节点都认为另外一个节点宕机了，那么就是 `fail` ，**客观宕机**，跟哨兵的原理几乎一样，sdown，odown。

在 `cluster-node-timeout` 内，某个节点一直没有返回 `pong` ，那么就被认为 `pfail` 。

如果一个节点认为某个节点 `pfail` 了，那么会在 `gossip ping` 消息中， `ping` 给其他节点，如果**超过半数**的节点都认为 `pfail` 了，那么就会变成 `fail` 。

**从节点过滤**

对宕机的 master node，从其所有的 slave node 中，选择一个切换成 master node。

检查每个 slave node 与 master node 断开连接的时间，如果超过了 `cluster-node-timeout * cluster-slave-validity-factor` ，那么就**没有资格**切换成 `master` 。

**从节点选举**

每个从节点，都根据自己对 master 复制数据的 offset，来设置一个选举时间，offset 越大（复制数据越多）的从节点，选举时间越靠前，优先进行选举。

所有的 master node 开始 slave 选举投票，给要进行选举的 slave 进行投票，如果大部分 master node `（N/2 + 1）` 都投票给了某个从节点，那么选举通过，那个从节点可以切换成 master。

从节点执行主备切换，从节点切换为主节点。

**与哨兵比较**

整个流程跟哨兵相比，非常类似，所以说，Redis cluster 功能强大，直接集成了 replication 和 sentinel 的功能。

## **redis 持久化机制**

​	很多时候我们需要持久化数据也就是将内存中的数据写⼊到硬盘⾥⾯，⼤部分原因是为了之后重⽤数据 （⽐如重启机器、机器故障之后恢复数据），或者是为了防⽌系统故障⽽将数据备份到⼀个远程位置。

 Redis不同于Memcached的很重⼀点就是，Redis⽀持持久化，⽽且⽀持两种不同的持久化操作。Redis的 ⼀种持久化⽅式叫快照（snapshotting，RDB），另⼀种⽅式是只追加⽂件（append-only file,AOF）。这两种⽅法各有千秋，下⾯我会详细这两种持久化⽅法是什么，怎么⽤，如何选择适合⾃ ⼰的持久化⽅法。

**快照（snapshotting）持久化（RDB）**

Redis可以通过创建快照来获得存储在内存⾥⾯的数据在某个时间点上的副本。Redis创建快照之后，可 以对快照进⾏备份，可以将快照复制到其他服务器从⽽创建具有相同数据的服务器副本（Redis主从结 构，主要⽤来提⾼Redis性能），还可以将快照留在原地以便重启服务器的时候使⽤。 快照持久化是Redis默认采⽤的持久化⽅式，在redis.conf配置⽂件中默认有此下配置：

```
save 900 1 #在900秒(15分钟)之后，如果⾄少有1个key发⽣变化，
Redis就会⾃动触发BGSAVE命令创建快照。
save 300 10 #在300秒(5分钟)之后，如果⾄少有10个key发⽣变化，
Redis就会⾃动触发BGSAVE命令创建快照。
save 60 10000 #在60秒(1分钟)之后，如果⾄少有10000个key发⽣变化，
Redis就会⾃动触发BGSAVE命令创建快照。
```

**AOF（append-only file）持久化**

与快照持久化相⽐，AOF持久化 的实时性更好，因此已成为主流的持久化⽅案。默认情况下Redis没有 开启AOF（append only file）⽅式的持久化，可以通过appendonly参数开启：

```
appendonly yes
```

开启AOF持久化后每执⾏⼀条会更改Redis中的数据的命令，Redis就会将该命令写⼊硬盘中的AOF⽂件。 AOF⽂件的保存位置和RDB⽂件的位置相同，都是通过dir参数设置的，默认的⽂件名是 appendonly.aof。

在Redis的配置⽂件中存在三种不同的 AOF 持久化⽅式，它们分别是：

```
appendfsync always #每次有数据修改发⽣时都会写⼊AOF⽂件,这样会严重降
低Redis的速度
appendfsync everysec #每秒钟同步⼀次，显示地将多个写命令同步到硬盘
appendfsync no #让操作系统决定何时进⾏同步
```

为了兼顾数据和写⼊性能，⽤户可以考虑 appendfsync everysec选项 ，让Redis每秒同步⼀次AOF⽂ 件，Redis性能⼏乎没受到任何影响。⽽且这样即使出现系统崩溃，⽤户最多只会丢失⼀秒之内产⽣的 数据。当硬盘忙于执⾏写⼊操作的时候，Redis还会优雅的放慢⾃⼰的速度以便适应硬盘的最⼤写⼊速 度。

**Redis 4.0 对于持久化机制的优化**

Redis 4.0 开始⽀持 RDB 和 AOF 的混合持久化（默认关闭，可以通过配置项 aof-use-rdbpreamble 开启）。

如果把混合持久化打开，AOF 重写的时候就直接把 RDB 的内容写到 AOF ⽂件开头。这样做的好处是可 以结合 RDB 和 AOF 的优点, 快速加载同时避免丢失过多的数据。当然缺点也是有的， AOF ⾥⾯的 RDB 部分是压缩格式不再是 AOF 格式，可读性᫾差。

**补充内容：AOF 重写**

AOF重写可以产⽣⼀个新的AOF⽂件，这个新的AOF⽂件和原有的AOF⽂件所保存的数据库状态⼀样，但体 积更⼩。 AOF重写是⼀个有歧义的名字，该功能是通过读取数据库中的键值对来实现的，程序⽆须对现有AOF⽂件 进⾏任何读⼊、分析或者写⼊操作。 在执⾏ BGREWRITEAOF 命令时，Redis 服务器会维护⼀个 AOF 重写缓冲区，该缓冲区会在⼦进程创建 新AOF⽂件期间，记录服务器执⾏的所有写命令。当⼦进程完成创建新AOF⽂件的⼯作之后，服务器会将 重写缓冲区中的所有内容追加到新AOF⽂件的末尾，使得新旧两个AOF⽂件所保存的数据库状态⼀致。最 后，服务器⽤新的AOF⽂件替换旧的AOF⽂件，以此来完成AOF⽂件重写操作

**Redis事务？**

Redis 通过 MULTI、EXEC、WATCH 等命令来实现事务(transaction)功能。事务提供了⼀种将多个命令 请求打包，然后⼀次性、按顺序地执⾏多个命令的机制，并且在事务执⾏期间，服务器不会中断事务⽽ 改去执⾏其他客户端的命令请求，它会将事务中的所有命令都执⾏完毕，然后才去处理其他客户端的命 令请求。 在传统的关系式数据库中，常常⽤ ACID 性质来检验事务功能的可靠性和安全性。在 Redis 中，事务 总是具有原⼦性（Atomicity）、⼀致性（Consistency）和隔离性（Isolation），并且当 Redis 运⾏ 在某种特定的持久化模式下时，事务也具有持久性（Durability）。

> 补充内容： redis同⼀个事务中如果有⼀条命令执⾏失败，其后的命令仍然会被执⾏，没有回滚。

## **Redis的高可用如何部署？**

**面试官心里剖析**

看看你了解不了解你们公司的 Redis 生产集群的部署架构，如果你不了解，那么确实你就很失职了，你的 Redis 是主从架构？集群架构？用了哪种集群方案？有没有做高可用保证？有没有开启持久化机制确保可以进行数据恢复？线上 Redis 给几个 G 的内存？设置了哪些参数？压测后你们 Redis 集群承载多少 QPS？

兄弟，这些你必须是门儿清的，否则你确实是没好好思考过。

**面试题剖析**

Redis cluster，10 台机器，5 台机器部署了 Redis 主实例，另外 5 台机器部署了 Redis 的从实例，每个主实例挂了一个从实例，5 个节点对外提供读写服务，每个节点的读写高峰 QPS 可能可以达到每秒 5 万，5 台机器最多是 25 万读写请求每秒。

机器是什么配置？32G 内存+ 8 核 CPU + 1T 磁盘，但是分配给 Redis 进程的是 10g 内存，一般线上生产环境，Redis 的内存尽量不要超过 10g，超过 10g 可能会有问题。

5 台机器对外提供读写，一共有 50g 内存。

因为每个主实例都挂了一个从实例，所以是高可用的，任何一个主实例宕机，都会自动故障迁移，Redis 从实例会自动变成主实例继续提供读写服务。

你往内存里写的是什么数据？每条数据的大小是多少？商品数据，每条数据是 10kb。100 条数据是 1mb，10 万条数据是 1g。常驻内存的是 200 万条商品数据，占用内存是 20g，仅仅不到总内存的 50%。目前高峰期每秒就是 3500 左右的请求量。

其实大型的公司，会有基础架构的 team 负责缓存集群的运维。

## **Redis6.0的多线程特性？**

[新特性](https://blog.csdn.net/qq_42979842/article/details/106973086)

