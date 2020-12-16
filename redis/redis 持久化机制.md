# 持久化机制

- RDB 持久化能够在指定时间间隔对你的数据进行快照存储。
- AOF 持久化记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF 命令以 redis 协议追加保存每次写的操作到文件末尾。
- Redis 还**能对 AOF 进行重写**，使得 AOF 体积不至于过大。**可以同时开启两种持久化方式**，当 Redis 重启的时候会优先载入AOF文件来恢复原始数据。因为通常状况下 AOF 文件保存的要比 RDB 完整。



## RDB

RDB 持久化会生成 RDB 文件，该文件是一个压缩过的二进制文件，可以通过该文件还原快照时的数据库状态，即生成该 RDB 文件时的服务器数据。RDB 文件默认为当前工作目录下的 dump.rdb，可以根据配置文件中的 dbfilename 和 dir 设置 RDB 的文件名和文件位置。

```properties
# 设置 dump 的文件名
dbfilename dump.rdb

# 工作目录
# 上面的 dbfilename 只制定了文件名，但是他会写入到这个目录下。
dir ./
```



#### 触发快照的时机

- 执行 `save` 和 `bgsave` 命令
- 配置文件设置 `save <seconds> <changes>` 规则，自动间隔性 `bgsave` 命令。 
- 主从复制时，从库全量复制同步主库数据，主库会执行 `bgsave`。
- 执行 `flushall` 命令清空服务器数据
- 执行 `shutdown` 命令关闭 Redis 时，会执行 `save` 命令。



#### save和bgsave命令

执行 `save` 和 `bgsave` 命令，可以手动触发快照，生成 RDB 文件，两者的区别如下。

- 使用 `save` 命令会阻塞 Redis 服务器进行，服务器进程在 RDB 文件创建完成之前是不能处理任何的命令请求。

  ```properties
  127.0.0.1:6379> save
  OK
  ```

- 使用 `bgsave` 命令会 `fork` 一个子进程，然后该紫禁城会负责创建 RDB 文件，而服务器进程会继续处理命令请求。

  ```properties
  127.0.0.1:6379> bgsave
  Background saving started
  ```

  > **`fork()` 是由操作系统提供的函数，作用是创建当前进程的一个副本作为子进程。**

  

  > **`fork()` 一个子进程，子进程会把数据集先写入临时文件，写入成功之后，再替换之前的 RDB 文件，用二进制压缩存储，这样可以保证 RDB 文件始终存储的是完整的持久化内容。**

  

#### 自动间隔触发

在配置文件中设置 `save <seconds> <changes>` 规则，可以自动间隔性执行 `bgsave` 命令。

```properties
save 900 1
save 300 10
save 60 10000
```



## AOF

AOF 持久化会把被执行的写命令写道 AOF 文件的末尾，记录数据的变化。默认情况下，Redis 是没有开启 AOF 持久化的，开启后，每执行一条更改 Redis 数据的命令，都会把该命令追加到 AOF 文件中，这事会降低 Redis 的性能，但大部分情况下这个影响是能够接受的。



### AOF 参数

可以通过配置 `redis.conf` 文件开启 AOF 持久化。

```properties
# appendonly 参数开启文件持久化
appendonly on

# AOF持久化的文件名，默认是appendonly.aof
appendfilename "appendonly.aof"

# AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的
dir ./

# 同步策略
# appendfsync always
appendfsync everysec
# appendfsync no

# aof重写期间是否同步
no-appendfsync-on-rewrite no

# 重写触发配置
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 加载aof出错如何处理
aof-load-truncated yes

# 文件重写策略
aof-rewrite-incremental-fsync yes
```



### AOF 实现

AOF 需要记录 Redis 的每个写命令，步骤为：命令追加（append）、文件写入（write）和文件同步（sync）。



#### 命令追加(append)

​		开启 AOF 功能后，服务器没执行一个写命令，都会把该命令以协议格式先追加到 `aof_buf` 缓存区的末尾，而不是直接写入文件，避免每次由命令都直接写入硬盘，减少硬盘 IO 次数。



#### 文件写入(write)和文件同步(sync)

对于何时把 `aof_buf` 缓冲区的内容写入保存在AOF文件中，Redis提供了多种策略.

- **appendfsync always**：将 `aof_buf` 缓冲区的所有内容写入并同步到 AOF 中，每个写命令同步写入磁盘。
- **appendfsync everysec**：将 `aof_buf` 缓冲区的内容写入 AOF 文件，每秒同步一次，该操作由一个线程专门负责。
- **appendfsync no**：将 `aof_buf` 缓存区的内容写入 AOF 文件，什么时候同步由操作系统来决定。



关于 AOF 的同步策略是设计到操作系统的 `write` 函数和 `fsync` 函数的。

> 为了提高文件写入效率，在现代操作系统中，当用户调用`write`函数，将一些数据写入文件时，操作系统通常会将数据暂存到一个内存缓冲区里，当缓冲区的空间被填满或超过了指定时限后，才真正将缓冲区的数据写入到磁盘里。
>
> 这样的操作虽然提高了效率，但也为数据写入带来了安全问题：如果计算机停机，内存缓冲区中的数据会丢失。为此，系统提供了`fsync`、`fdatasync`同步函数，可以强制操作系统立刻将缓冲区中的数据写入到硬盘里，从而确保写入数据的安全性。



#### AOF 重写(rewrite)

​		Redis 执行的写命令会越来越多，AOF 文件也会越来越大，过大的 AOF 文件可能会对 Redis 服务器造成影响，如果使用 AOF 文件来进行数据还原所需事件也会越长。

​		时间长了，AOF 文件中通常会有一些冗余命令，比如：`过期数据的命令`、`无效的命令`、`多个命令可合并为一个命令`。所以 AOF 文件是由精简压缩的空间的。



**AOF 重写的目的就是减小 AOF 文件的体积。**

> AOF 文件重写并部需要对现有的 AOF 文件进行任何读取、分享和写入操作，而是通过读取服务器当前的数据状态来实现的。



#### 重写命令

​		可分为手动触发和自动触发，手动触发执行 `bgrewriteaof` 命令，该命令的执行跟 `bgsave` 触发快照时类似的，都是先 `fork` 一个子进程做具体的工作。

```properties
127.0.0.1:6379> bgrewriteaof
Background append only file rewriting started
```

​		自动触发会根据 `auto-aof-rewrite-percentage` 和 `auto-aof-rewrite-min-size 64mb` 配置来自动执行 `bgrewriteaof` 命令

```properties
# 表示当AOF文件的体积大于64MB，且AOF文件的体积比上一次重写后的体积大了一倍（100%）时，会执行`bgrewriteaof`命令
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```



#### bgrewriteaof 命令重写流程

- 重写会有大量的写入操作，所以服务器进程会 fork 一个子进程来创建一个新的 AOF 文件。
- 在重写期间，服务器进程继续处理命令请求，如果由写入的命令，追加到 `aof_buf` 的同事，还会追加到 `aof_rewrite_buf` AOF重写缓冲区。
- 当子进程完成重写之后，会给父进程一个信号，然后进程会把 AOF 重写缓冲区的内容写进新的 AOF 临时文件中，再对新的 AOF 文件改名完成替换，这样可以保证新的 AOF 文件与当前数据库数据的一致性。



## 数据恢复

Redis 4.0 开始支持 RDB 和 AOF 的混合持久化（可以通过配置项 `aof-use-rdb-preamble` 开启）

- 如果是 redis 进程挂掉，那么重启 redis 进程即可，直接基于 AOF 日志文件恢复数据
- 如果是 redis 进程挂掉，那么重启机器后，尝试重启 redis 进程，尝试直接基于 AOF 日志文件进行数据恢复，如果 AOF 文件破损，那么用 `redis-check-aof fix` 命令修复。
- 如果没有 AOF 文件，回去加载 RDB 文件。
- 如果 redis 当前最新的 AOF 和 RDB 文件出现了丢失/损坏，那么可以尝试基于该集器上当前的某个最新的 RDB 数据副本进行数据恢复。



## RDB vs AOF

### RDB

#### 优点

- RDB 快照是一个压缩过的非常紧凑的文件，保存着某个时间点的数据集，实和做数据的备份，灾难恢复。
- 可以最大化 Redis 的性能，在保存 RDB 文件，服务器进程秩序 fork 一个子进程来完成 RDB 文件的创建，父进程不需要做 IO 操作。
- 与 AOF 相比，恢复大数据集的时候会更快。

#### 缺点

- RDB 的数据安全性是不如 AOF 的，保存整个数据的过程是比较繁重的，根据配置可能要几分钟才快照一次，如果服务器宕机，那么就可能丢失几分钟的数据。
- Redis 数据集较大时，fork 的子进程要完成快照会比耗 CPU、耗时。



### AOF

#### 优点

- 数据更完整，安全性更高，秒级数据丢失（取决 fsync 策略，如果时 everysec，最多丢失 1 秒的数据）
- AOF 文件时一个只进行追加的日志文件，且写入操作是以 Redis 协议的格式保存的，内容是可读的，适合误删紧急恢复。

#### 缺点

- 对于相同的数据集，AOF 文件的体积要大于 RDB 文件，数据恢复也会比较慢。
- 根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB。不过在一般的情况下，每秒 fsync 的性能依然很高。



## 如何选择 RDB 和 AOF

- 如果是数据不那么敏感，且可以从其他地方重新生成补回的，那么可以关闭持久化

- 如果是数据比较重要，不想再从其他地方获取，且可以承受数分钟的数据丢失，比如缓存等，那么可以只使用RDB
- 如果是用做内存数据库，要使用Redis的持久化，建议是RDB和AOF都开启，或者定期执行`bgsave`做快照备份，RDB方式更适合做数据的备份，AOF可以保证数据的不丢失。
