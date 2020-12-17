# Redis 哨兵

## Sentinel

#### Sentinel 主从切换方案

Sentinel 是 Redis 的高可用（HA）解决方案。



Redis 的 Sentinel 系统用于管理多个 Redis 服务器。该系统执行以下三个任务：

- **监控（Monitoring）**：Sentinel 会不断的检查主服务器和从服务器是否运作正常。
- **提醒（Notification）**：当被监控的某个 Redis 服务器出现问题时，Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
- **自动故障迁移（Automatic failover）**：当一个主服务器不能正常工作时，Sentinel 会开始一次自动故障迁移。它会将失效主服务器的其中一个从服务器升级为新的主服务器，并让失效主服务器的其他从服务器改为复制新的主服务器；当客户端视图连接失效的主服务器时，集群也会向客户端返回新主服务器的地址，使得集群可以使用新主服务器代替失效服务器。



Redis Sentinel 是一个分布式系统，可以在一个架构中运行多个 Sentinel 进程（progress），这些进程使用 `流言协议` 来接收关于主服务器是否下线的信息，并使用投票协议来决定是否执行自动故障迁移，以及选择哪个从服务器作为新的主服务器。



#### Sentinel 作用

- master 状态监控。
- 如果 master 异常，进行 master-slave 切换，将其中一个 salve 切换成 master，将之前的 master 作为 slave。
- master-slave 切换后，master_redis.conf、slave_conf 和 sentinel.conf 的内容都会发生改变，即 master_redis.conf 中会多出一个 slaveof 配置。sentinel.conf 的监控目标随之切换。



#### Sentinel 工作方式

- 每秒钟一次的频率向他所知的 master，slave 以及其他的 sentinel 实例发送一个 `PING` 命令。
- 如果一个实例距离最后一次有效回复 `PING` 命令的时间超过 `down-after-milliseconds` 选项所指的值，这个实例会被 sentinel 标记为**主观下线**。
- 如果一个 master 被标记为主管下线，则正在监视这个 master 的**所有 sentinel** 要以**每秒一次的频率**确认 master 的确进入了主观下线状态。
- 当有足够数量的 sentinel（大于等于配置文件指定的值）在指定的时间范围内确认 master 的确进入了主管下线状态，则 master 会被标记为**客观下线**。
- 一般情况下，每个 sentinel 会以**每 10 秒一次的频率**向它已知的所有 master、slave 发送 `INFO` 命令。
- 当 master 被 sentinel 标记为客观下线时，sentinel 向下线的 master 的所有 slave 发送 `INFO` 命令的**频率从 10 秒一次改为每秒一次**。
- 若没有足够数量的 sentinel 同意 master 已经下线，master 的客观下线就会被移除。若 master 重新向 sentinel 的 `PING` 命令返回有效回复，master 的主观下线状态就会被移除。



#### Sentinel 三个定时任务

- 每 10 秒每个 sentinel 会对 master 和 slave 执行 `INFO` 命令。这个任务有两个目的：
  - 发现 slave 节点
  - 确认主从关系
- 每两秒每个 sentinel 通过 master 节点的 channel 交换信息（pub/sub）
  - master 节点上有一个发布订阅的频道（_sentinel_：hello）。sentinel 节点通过 hello 频道进行信息交换，达成共识。
- 每 1 秒每个 sentinel 对其他 sentinel 和redis 节点执行 `PING` 操作（互相监控）
  - 其实是心跳检测，是失败判定的依据。



#### 主观下线（SDOWN）

- 指单个 sentinel 实例对服务器做出的下线判断，即单个 sentinel 认为某个服务下线。（收不到订阅，网络不通都有可能）
- sentinel 会以每秒一次的频率向所有与其建立了命令连接的实例发 `PING` 命令，通过判断 `PING` 回复是有效回复还是无效回复判断实例是否在线。
- 如果实例在 `down-after-milliseconds` 毫秒内，返回的都是无效的回复，那么 sentinel 会认为该实例已主观下线。修改其 flags 状态为 SRI_S_DOWN。



#### 客观下线（ODOWN）

- 指多个 sentinel 实例在对同一个服务器做出 `SDOWN` 判断，并且通过 SENTINEL `is-master-down-by-addr` 命令相互交流之后，得出服务器下线判断，然后开启 failover。

> 只有当 master 被认定为客观下线时，才会发生故障迁移。



#### redis-sentinel.conf 相关配置

```properties
sentinel monitor <masterName> <ip> <port> <quorum>
```

- master 为对某个 master + slave 组合的区分表示。
- quorum 为客观下线的一个依据；至少有 quorum 个 sentinel 主观认为这个 master 有故障，才会对这个 master 进行下线以及故障转移。

```properties
sentinel down-after-milliseconds <masterName> <timeout>
```

- 主观下线的依据，如果超过 `timeout` 这个时间都无法联通 master 包括 slave，就会主观认为该 master 已下线。



#### 主观下线、客观下线更多细节

> 从 sentinel 的角度来看，在一定时间没有收到合法的回复，就达到了 SDOWN 的条件。

当 sentinel 发送 `PING` 后，以下回复之一都被认为是合法的

- PING replied with +PONG

- PING replied with +LOADING error

- PING replied with +MASTERDOWN error

  其他任何回复或者没有回复都是不合法的。



> 从 SDOWN 到 ODOWN 不需要任何一致性算法，只需要一个 gossip 协议，如果一个 sentinel 收到了足够多的 sentinel 发来消息告诉他某个 master 已经 down 掉了，SDOWN 状态就会变成 ODOWN 状态。



> ODOWN 状态只适用于 master，对于不是 master 的 redis 节点 sentinel 之间不需要任何协商。



#### 配置版本号

​		当一个 sentinel 被授权后，它将会获得宕掉的 master 的一份最新配置版本号，当 failover 执行接收以后，这个版本号将会被用于最新的配置。因为大多数 sentinel 都已经知道该版本号已经被要执行 failover 的 sentinel 拿走了，所以其他的 sentinel 都不能再去使用这个版本号。**每次 failover 都会附带有一个独一无二的版本号**。

​		如果 sentinel A 推荐 sentinel B 去执行 failover，B 会等待一段时间后，自行再去对同一个 master 执行 failover，这个等待时间是通过 failover-timeout 配置项去配置的。

​		sentinel 集群中**不会在同一时间内进行重新 failover**，一次类推。

- redis sentinel 保证了活跃性：如果大多数 sentinel 能够互相通信，最终将会有一个被授权去进行 failover。
- redis sentinel 也保证了安全性：每个试图去 failover 同一个 master 的 sentinel 都会得到一个独一无二的版本号。



#### 配置传播

- 一但一个 sentinel 成功的对一个 master 进行了 failover，他将会把关于 master 的最新配置**通过广播形式通知其他 sentinel**，其他的 sentinel 则更新对应 master 的配置。
- 一个 failover 要想被成功执行，sentinel 必须能够向选为 master 的 slave 发送 `slaveof no one` 命令，然后能通过 `INFO` 命令看到新 master 的信息。
- 当一个 slave 选举为 master 并发送 `slaveof no one` 后，即使其他的 slave 还没针对新 master 重新配置自己，failover 也被认为是成功了的，然后所有 sentinels 将会发布新的配置信息。
- 新配置在集群中相互传播的方法，就是为什么我们需要当一个 sentinel 进行 failover 时必须被授权一个版本好的原因。
- 每一个配置都有一个版本号，所以以版本号最大的为标准。



#### Sentinel 的“仲裁会”

- 当 master 为不可用，并且进行 failover 时所需要的 sentinel 数量，可以称为这个参数为票数。
- 不过当 failover 主备切换真正被触发后，failover 并不会马上进行，还需要 sentinel 中的大多数 sentinel 授权后才可以进行 failover。
- 当 ODOWN 时，failover 被触发。failover 一旦被触发，尝试去进行 failover 的 sentinel 会去获得“大多数” sentinel 的授权。



#### 选举领头 sentinel（即领导者选举）

一个 redis 服务被判断为客观下线时，多个监视该服务的 sentinel 协商，选举一个领头 sentinel，对该 redis 服务进行故障转移操作。

1. 所有的 sentinel 都有被选举成领头的资格；
2. 所有的 sentinel 都有且只有一次将某个 sentinel 选举成领头的机会（在一轮选举中），一旦选举某个 sentinel 为领头，不能更改。
3. sentinel 设置领头 sentinel 时先到先得，一旦当前 sentinel 设置了领头 sentinel，以后要求设置 sentinel 为领头的请求都会被拒绝。
4. 每个发现服务客观下线的 sentinel，都会要求其他 sentinel 将自己设置成领头。
5. 当一个 sentinel 向另一个 sentinel 发送 `is-master-down-by-addr ip port current_epoch runid` 命令的时候，runid 参数不是 *，而是 sentinel 运行id，就表示源 sentinel 要求目标 sentinel 选举其为领头
6. 源 sentinel 会检查目标 sentinel 对其要求设置成领头的回复，如果回复的 leader_runid 和 leader_epoch 为源 sentinel，表示目标 sentinel 同一将源 sentinel 设置成领头。
7. 如果某个 sentinel 被半数以上的 sentinel 设置成领头，那么该 sentinel 即为领头。
8. 如果在限定时间内没有选举出领头 sentinel，暂停一段时间，再选举。



#### Sentinel 选举过程

1. 每个做主观下线的 sentinel 节点向其他 sentinel 节点发送上面那条命令，要求将它设置为领导者。
2. 收到命令的 sentinel 节点如果还没有统一过其他的 sentinel 发送的命令（还未投过票），那么就会同意，否则拒绝。
3. 如果该 sentinel 节点发现自己的票数已经过半且达到了 quorum 的值，就会称为领导者
4. 如果这个过程出现多个 sentinel 称为领导者，则会等待一段时间重新选举。



#### Sentinel API

> 默认情况下，sentinel 使用 tcp 端口 26379.

1. **通过直接发送命令**来查询被监视 redis 服务的当前状态，以及 sentinel 所知道的关于其他 sentinel 的信息。
2. 使用**发布与订阅**功能，通过接收 sentinel 发送的通知。当执行故障转移操作，或者某个被监视的服务器被判断为主观下线或者客观下线时，sentinel 就会发送响应的信息。



#### Sentinel 增加、删除

> sentinel 有自动发现机制，添加一个 sentinel 非常容易，只需要做到监控在某个 master 上，然后新添加的 sentinel 就能获得其他的所有信息。



> sentinel 永远不会删除一个已经存在过的 sentinel，即使已经失去联系很久，如果要删除需要遵守以下步骤。

1. 停止所要删除的sentinel
2. 发送一个 `sentinel reset *` 命令给所有其他的 sentinel 实例，如果你想要重置指定 master 上的 sentinel，只需要把 * 改为特定的名字。
3. 检查以下所有的 sentinels 是否都有一致的当前 sentinel 数，使用 `sentinel master masterName` 来查询。

```properties
sentinel reset *

sentinel master masterName
```



#### Slave 选举与优先级

slave 的选举主要会评估 slave 的以下几个方面：

1. 与 master 断开连接的次数
2. slave 的优先级
3. 数据复制的下标（用来评估 slave 当前拥有多少 master 的数据）
4. 进程 ID



**slave 被列入 master 候选人列表，根据以下顺序来进行排序：**

1. sentinel 首先会根据 slaves 的优先级来进行排序，并根据以下顺序来进行排序。
2. 如果优先级相同，则查看复制的下表，哪个从 master 接收的复制数据多，哪个就考前。
3. 如果优先级和小标都相同，就选择进程 ID 较小的哪个。



#### 故障转移

一次故障转移操作大致分为以下流程：

1. 发现主服务器已经进入客观下线状态。
2. 对我们的当前集群进行自增，并尝试在这个集群中当选。
3. 如果当选失败，那么在设定的故障迁移超时时间的两倍之后，重新尝试当选。如果当选成功，那么执行以下步骤。
4. 选出一个从服务器，并将他升级为主服务器。
5. 向被选中的从服务器发送 `slaveof no one` 命令，让他转变为主服务器。
6. 通过发布与订阅功能，将更新后的配置传播给所有其他 sentinel，其他 sentinel 对自己的配置进行更新。
7. 向已下线主服务器的从服务器发送 `slaveof` 命令，让他们去复制新的主服务器。
8. 当所有从服务器都已经开始复制新的主服务器时，领头 sentinel 终止这次故障迁移操作。
9. 每当一个 redis 实例被重新配置，无论是谁，sentinel 都会向被重新配置的实例发送一个 `config rewrite` 命令，从而确保这些配置会持久化在硬盘里；



#### 主从复制的大致流程

1. 当一个 slave 启动时，会向 master 发送 sync 命令。
2. master 接收到 sync 命令后会开始在后台保存快照（执行 RDB 操作），并将保存期间接收到的命令缓存起来。
3. 当快照完成后，redis会将快照文件和所有缓存的命令发送给 slave。
4. salve 收到后，会载入快照文件并执行收到的缓存命令。



#### Sentinel 启动

对于 redis-sentinel 程序，可以用以下命令来启动 Sentinel 系统：

对于 redis-server 程序，可以用以下命令来启动一个运行在 Sentinel 模式下的 Redis 服务器：

```properties
redis-server /path/to/sentinel.conf --sentinel
```



> 启动 Sentinel 实力必须指定相应的配置文件，系统会使用配置文件来保存 Sentinal 的当前状态，并在 Sentinel 重启时通过载入配置文件来进行状态还原。

