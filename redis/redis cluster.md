# Redis 集群

## cluster

#### 集群设计

主要设计目标是达到线性可扩展性、可用性、数据一致性。

- **线性拓展**：官方推荐最大节点数量为 1000，由于 cluster 架构无 Proxy 层，master 与 slave 之间使用异步 replication。

- **数据一致性**：客户端容忍一定程度的数据丢失，集群尽可能保存 Client write 操作的数据，保证数据一致性。

- **可用性**：Redis集群通过 `partition（分区）` 来提供一定程度的可用性，当集群中的一部分节点失效或者无法进行通讯时，集群仍可以继续提供服务。

  - 只要集群中大多数 master 可达，且失效的 master 至少有一个 slave 可达，即集群非 Fail 状态，集群都是可用的。

    <img src="/redis cluster.assets\image-20201217142256509.png" alt="image-20201217142256509" style="zoom: 67%;" />

  - Redis集群的replicas migration机制可以将拥有多个Slave的Master的某个Slave，迁移到没有Slave的Master下，即Slave分布相对平衡，确保Master都有一定数量的Slave备份。



#### 集群节点属性

集群中每个 master node 负责**存储数据**、**集群状态**，包括 **slots 与 nodes 对应关系**。

master nodes 能够自动发现其他 nodes，检测 failure 节点，当某个 master 节点失效时，集群能将核实的 slave 提升为 master。

节点关联信息：

```properties
# 从左至右
# 节点ID、ip地址和端口、节点角色标志、最后发送PING时间、最后接到PONG时间、连接状态、节点负责处理的hash slot
1fc2412b7429e4ab5d8704fcd39520815ea2727b 10.9.42.37:6103 master - 0 1494082584680 9 connected 10923-13652  
08e70bb3edd7d3cabda7a2ab220f2f3610db38cd 10.9.33.204:6202 slave ad1334bd09ee73fdeb7b8f16194550fc2bf3a038 0 1494082586686 8 connected  
edaafc250f616e9e12c5182f0322445ea9a89085 10.9.33.204:6203 slave 1fc2412b7429e4ab5d8704fcd39520815ea2727b 0 1494082586184 9 connected  
06cd6f24caf98a1c1df0862eadac2b05254f909d 10.9.33.204:6201 slave d458c22ccced2f29358b6e6814a206d08285374e 0 1494082584179 7 connected  
3892b7fb410a4d6339364dbdda2ebc666ffee843 10.9.42.37:6203 slave 73f7d44c03ada58bf5adaeb340359e2c043ecfa0 0 1494082582679 12 connected  
73f7d44c03ada58bf5adaeb340359e2c043ecfa0 10.9.33.204:6103 master - 0 1494082585181 3 connected 13653-16383  
4004a64211bea5050a8f46b8436564d40380cd60 10.9.33.204:6101 master - 0 1494082583678 1 connected 2731-5460  
d458c22ccced2f29358b6e6814a206d08285374e 10.9.42.37:6101 master - 0 1494082588189 7 connected 0-2730  
f8868d59c0f3d935d3dbe35601506039520f7107 10.9.42.37:6201 slave 4004a64211bea5050a8f46b8436564d40380cd60 0 1494082587187 10 connected  
45ba0d6fc3d48a43ff72e10bcc17d2d8b2592cdf 10.9.33.204:6102 master - 0 1494082583179 2 connected 8192-10922  
007d7e17bfd26a3c1e21992bb5b656a92eb65686 10.9.42.37:6202 slave 45ba0d6fc3d48a43ff72e10bcc17d2d8b2592cdf 0 1494082588189 11 connected  
ad1334bd09ee73fdeb7b8f16194550fc2bf3a038 10.9.42.37:6102 myself,master - 0 0 8 connected 5461-8191  
```

> 集群可以自动识别出ip/port的变化，并通过Gossip（最终一致性，分布式服务数据同步算法）协议广播给其他节点知道。
>
> Gossip也称“病毒感染算法”、“谣言传播算法”。



#### Cluster BUS

​		每个 node 都有一个特定的 TCP 端口，用来接收其他 nodes 的链接；此端口号为面向 client 的端口号 +10000，比如客户端端口号为 `6379`，那么此 node 的 BUS 端口号为 `16379`，**客户端端口号可以在配置文件中声明**。由此可见，nodes 之间的交互通讯是通过 Bus端口进行，使用了特定的二进制协议，此端口通常应该只对 nodes 可用，可以借助防火墙技术来屏蔽其他非法访问。



#### 集群拓补

​		redis cluster 中每个 node 都与其他 nodes 的 Bus 端口建立 TCP 链接（full mesh，全网）。比如在由 N 个节点组成的集群中，每个 node 有 N-1 个向外发出的 TCP 链接，以及 N-1 个其他 nodes 发过来的 TCP 链接；这些 TCP 链接总是 **keepalive**，不是按需创建的。如果 ping 发出之后，node 在足够长的时间内仍然没有 `PONG` 响应，那么此 node 将会被标记为 “**不可达**”，那么与此 node 的链接将会被**刷新或者重建**。nodes 之间通过 gossip协议 和配置更新的机制，来避免每次都交互大量的消息，最终确保在 nodes 之间的信息传送量是可控的。



#### 数据分片-哈希槽

Redis 有 16384 个哈希槽，每个 key 通过 CRC16 校验后对 16384 取模来决定放置哪个槽，集群的每个节点负责一部分哈希槽。

这种结构很容易添加或者删除节点。由于从一个节点将哈希槽移动到另一个节点并不会停止服务，所以无论添加删除或者改变某个节点的哈希槽的数量都不会造成集群不可用的状态。

hash槽算法：

```properties
HASH_SLOT = CRC16(key) mod 16384
```



#### Redis 集群主从复制模型

为了使在部分节点失败或者大部分节点无法通信的情况下集群仍然可用，所以集群使用了主从复制模型，每个节点都会有N-1个 slave。

在我们例子中具有A，B，C三个节点的集群，在没有复制模型的情况下，如果节点B失败了，那么整个集群就会以为缺少5501-11000这个范围的槽而不可用.

然而如果在集群创建的时候（或者过一段时间）我们为每个节点添加一个从节点A1，B1，C1，那么整个集群便有三个master节点和三个slave节点组成，这样在节点B失败后，集群便会选举B1为新的主节点继续服务，整个集群便不会因为槽找不到而不可用了

不过当B和B1 都失败后，集群是不可用的。



#### Redis 集群一致性保证

Redis 并不能保证数据的**强一致性**。这意味这在实际中集群在特定的条件下可能会丢失写操作。有两个原因：

- **异步replication机制**：master 以及对应的 slaves 之间使用异步的机制，在节点 failover 后，新的 master 将会最终替代其他的 replicas。

  > write 命令提交到 master，master 执行完毕后向 client 返回 “OK”，但由于一部分 replication，此时数据还没传播给 slave；如果此时 master 不可达的时间超过阀值，此时集群将触发对应的 slave 选举为新的 master，此时没有 replication 同步到 slave 的数据将丢失。 

- **network partition**：在 network partition 时，总有一个窗口期（node timeout）可能会导致数据丢失。

  > 由于网络分区，此时 master 不可达，且 client 与 master 处于一个分区，且此时集群处于“OK”。此时 failover 机制，将其中一个 slave 提升为新的 master，等待网络分区消除后，老的 master 再次可达，此时节点被切换为 slave，而在这段期间，处于网络分区期间，client 仍然将 write 提交到老的 master，因为该 master 被认为是仍然有效的。当老的 master 再次加入集群，被切换成 slave 后，这些数据将永远丢失。



#### replicas migration（复制品迁移）机制

当 master 节点发生 failover 后，集群会动态重新分配、平衡 slaves 的分布，有效地提高了集群的可用性。

##### 从节点选举逻辑

- 节点时已下线 master 对应的 slave。
- `FAIL` 状态的 master 负责的 `hash slot` 为空。
- 主从节点之间的 replication link 断线的时长不能超过 `NODE_TIMEOUT * REDIS_CLUSTER_SLAVE_VALIDITY_MULT` 。



#### Nodes handshake

nodes 通过端口发送 `PING`、`PONG`，除了 `PING` 之外，节点会拒绝其他所有非本集群节点的 packets，一个节点注册成为集群的新成员有2中方法：

- 通过 cluster meet 指令引入，即将指定的 node 加入集群，集群将认为指定的 node 为 “可信任”。
- 当其他 nodes 通过 gossip 引入了新的 nodes，这些 nodes 也是被认为是 “可信任的”。即：如果 A 信任 B，B 信任 C，且 B 向 A 传播关于 C 的信息，那么 A 也信任 C，并尝试连接 C。



#### MOVED 重定向

client 可以将请求发给任意一个 node，包含 slaves.

node 解析命令，检查语法，multiple keys 是否在同一个 slot。如果当前 node 持有该 slot，那么命令直接执行并返回，否则当前 node 向 client 返回 “MOVED” 错误。

```properties
10.9.33.204:6101> keys *  
1) test9  
10.9.33.204:6101> get test9  
value9  
10.9.33.204:6101> get test8  
(error) MOVED 905 10.9.42.37:6101
```



#### 节点失效检测

Redis Cluster 节点间通过持续的心跳来保持信息同步，不过 Redis Cluster 节点信息同步是内部实现的。

集群中的 nodes 持续交换 `PING`、`PONG` 数据，消息协议使用 Gossip，这两种 packet 数据结构一样，它们之间通过 type 字段区分。



节点定时向其他节点发送 `PING` 命令，它会随机选择存储的其他集群节点的其中三个进行信息 “广播”，例如广播的信息包含一项是节点是否被标记为 `PFAIL` / `FAIL` 。

- `PFAIL` 表示**“可能已失效”**，是尚未完全确认的失效状态（即可能是某个节点或少数 master 认为其不可达）。

  > ​		所谓不可达，就是当 “active ping”（发送 ping 且能收到 pong ）尚未成功的时间超过 `NODE_TIMEOUT`，因此我们设定的 **NODE_TIMEOUT** 的值应该**比网络交互往返的时间延迟要大一些（通常要大的多**，以至于交互往返时间可以忽略）。为了避免误判，当一个 node 在半个NODE_TIMEOUT 时间内仍未能 pong，那么当前 node 将会尽力尝试重新建立连接进行重试，以排除 PONG 未能接收是因为当前链接故障的问题。

- `FAIL` 表示 node 被集群大多数的 masters 认定为**失效**。

  > ​		（即大多数 master 已认定为不可达，且不可达的时间已经超过配置的NODE_TIMEOUT）。PFAIL 只是当前 node 有关于其他 nodes 的本地视图，可能每个 node 对其他 nodes 的本地视图都不一样，所以 PFAIL 还不足以触发 failover。处于 PFAIL 状态下的 node 可以被提升到 FAIL 状态。如上所述，每个 node 在向其他 nodes 发送 gossip 消息时，都会包含本地视图中几个随机 nodes 的状态信息；每个 node 最终都会从其他 nodes 发送的消息中获得一组 nodes 的 flags。因此，每个 node 都可以通过这种机制来通知其他 nodes，它检测到的故障情况。

  

##### PFAIL 被上升为 FAIL 的集中情况

1. 比如 A 节点，认为 B 为 PFAIL。
2. 那么 A 通过 gossip 信息，收集集群中大多数 masters 关于 B 的状态视图。
3. 多数 master 都认为 B 为 PFAIL，或者 PFAIL 情况持续时间为 NODE_TIMEOUT * FAIL_REPORT_VALIDITY_MULT（此值当前为2）

如果上述条件成立，那么 A 将会：

1. 将 B 节点设定为 FAIL。
2. 将 FAIL 信息发送给其所有能到达的所有节点。

> 每个接收到 FAIL 消息的节点都会强制将此 node 标记为 FAIL 状态，不管此节点在本地视图中是否为 PFAIL。FAIL 状态时单项的，即 PFAIL 可以转换为 FAIL，但是 FAIL 状态只能清楚，不能回转为 PFAIL。



#### 集群失效检测

- 当某个 master 或者 slave 不能被大多数 nodes 可达时，用于故障迁移并将合适 slave 提升为 master。

- 当 slave 提升未能成功，集群不能正常工作。即集群不能处理 client 的命令的请求，当 client 发出命令请求时，集群节点都将返回错误内容的 respone。



> 集群正常工作时，负责处理 16384 个 slot 的节点中，全部节点均正常。反之，若集群中有一部分 hash slot 不能正常使用，集群亦停止工作，即集群进入了 `FAIL` 状态。

对于集群进入 `FAIL` 状态，会有以下两种情况：

- 至少有一个 hash slot 不可用。
- 集群中大部分 master 都进入了 `PFAIL` 状态。



#### Redis 集群基本配置

```properties
port 7000

# 用于开实例的集群模式
cluster-enabled yes

# 设置保存节点配置文件的路径
cluster-config-file nodes.conf

# 集群通信超时时间
cluster-node-timeout 5000

appendonly yes
```



#### 问题

1. **Redis 集群为了解决什么问题而存在的？**

   ​		解决线性可扩展性。

   

2. **Redis集群诞生以前怎么解决这个问题？**

   ​		客户端分片、代理协助分片(Twemproxy)、查询路由、预分片、一致性哈希、客户端代理/转发等。

   

3. **Redis集群采用什么方式保证线性可扩展性、可用性、数据一致性？**

   ​		Hash槽、查询路由、节点互联的混合模式。

   

4. **Redis集群化面临的问题是什么？**

   ​		Redis集群本身要解决的是可伸缩问题，同时数据一致、集群可用等一系列问题。

   ​		前者涉及到了节点的哈希槽的分配(含重分配)，节点的增删，主从关系指定与变更(含自动迁移)这些具体的交互过程；

   ​		后者则是故障发现，故障转移，选举过程等详细的过程。

   

5. **Redis集群实现的核心思想和思路是什么？**

   ​		通过消息的交互（Gossip）实现去中心化(指的是集群自身的实现，不是指数据)，通过Hash槽分配，实现集群线性可拓展。

   

> 引用：
>
> ​		https://jishusuishouji.github.io/2017/04/03/redis/Redis_Cluster%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/
>
> ​		https://juejin.cn/post/6844903480960745480



