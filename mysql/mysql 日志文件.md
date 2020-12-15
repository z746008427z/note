## binlog

#### 导读

- binlog 让 mysql 有集群、数据备份、恢复数据的能力。
- 是 mysql 的二进制文件。
- binlog 是由 mysql 的 server 层产生的。



#### 存放位置

存放位置由 **datadir** 参数控制。

```sql
 show variables like '%datadir%';
```



#### 相关配置项

一般关于 binlog 的配置都是写在 mysql 的配置文件 my.cnf ，以方便启动 mysql 时直接让这些配置生效。

```properties
[mysqld]
# binlog相关配置

# 指定binlog日志存储的位置
datadir = /home/mysql/mysql/var

# 规范binlog的命名为 mysql-bin.0000XX
# 如果加这行配置，binlog文件名为主机名
log-bin = mysql-bin

# 索引当前所有的binlog
log-bin-index = mysql-bin.index

# 最大的大小
max_binlog_size = 1G

# binlog的sync时机
sync-binlog = 1

# binlog的格式
binlog-format = ROW

# 保留七天的binlog
expire_logs_days = 7
```



#### binlog 作用

1. binlog 可以恢复数据。
2. 搭建一套一主两从的 MySQL 集群，binlog 帮助完成数据主从同步。
3. 审计，通过分析 binlog 可以排查是否存在 SQL 注入攻击。



**默认binlog是不开启的，因为开启后会稍微降低一点 MySQL 性能。**



#### binlog 高速缓存

所有未 commit 的事务产生的 binlog，都会被先记录到 binlog 的告诉缓存中。等该事务被 commit 时，再将缓存中的数据写入 binlog 日志文件中。



高速缓存的大小由参数 binlog_cache_size 默认大小为：32768，并且每个 session 都有自己的独立的缓存。多个会话间彼此不影响。



#### binlog 刷盘机制

binlog 写入磁盘的机制由参数 sync_binlog 控制。

- sync_binlog = 0；表示 innodb 不会主动控制将 binlog 落盘，innodb 仅仅会将 binlog 写入到 os cache 中，至于什么时间将 binlog 刷入磁盘中完全依赖于操作系统。这种策略一旦操作系统宕机，os cache中得 binlog会丢失。
- sync_binlog = 1；表示事务 commit 时将 binlog 落盘。即使宕机了也能确保写入磁盘中。
- sync_binlog = N；当N > 1时，表示开启组提交，也就是 group commit，比如N = 5，那 MySQL 就会等收集5个binlog 后再将这5个 binlog 一次写到磁盘上。同样，服务器宕机会丢失部分 binlog。



#### 官方推荐配置

```sql
mysql> show variables like '%innodb_flush_log_at_trx_commit%';
+--------------------------------+-------+
| Variable_name                  | Value |
+--------------------------------+-------+
| innodb_flush_log_at_trx_commit | 1     |
+--------------------------------+-------+
1 row in set (0.00 sec)

mysql> show variables like '%sync_binlog%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| sync_binlog   | 1     |
+---------------+-------+
1 row in set (0.00 sec)
```





## redo log

#### 作用

当发生事务（增、删、改）时会导致缓存也变成脏页，于此同时 MySQL 会将事务涉及到得：对 xxx表空间 xxx数据页 xxx偏移量的地方做 xxx更新。

所以当 MySQL 意外宕机也没关系，只需要重启时解析 redo log 中的事务然后重放一遍。将 Buffer pool 中的缓存页重做成脏页。后续刷盘即可。



> redo log 侧重于重做，redo log 中记录的是物理层面的**数据页**、**偏移量**。
>
> 应对的问题是：MySQL 异常宕机后，如何将没来的及提交的事务数据重做出来。



#### redo log back

redo 并不是一条一条写入磁盘的。在 MySQL 的设定中，redo log是按快，一块一块的写入到磁盘中去的。

事务会产生好多 redo log。这些 redo 会先被持续不断的写入到 log block 中。

同一事务产生的 redo log 会被标记为一个 **redo log group**。



#### redo log buffer

redo log buffer 中会划分出多个 redo log block。redo log buffer 占用一块连续的内存空间，默认16MB.

MySQL允许我们通过参数 innodb_log_buffer_size 动态的调整他。

```sql
mysql> show variables like '%innodb_log_buffer_size%';
+------------------------+---------+
| Variable_name          | Value   |
+------------------------+---------+
| innodb_log_buffer_size | 7340032 |
+------------------------+---------+
1 row in set (0.00 sec)
```



#### redo log 刷盘机制

事务提交时，率先将 redo log 持久化进磁盘。

mysql提供了参数 innodb_flush_log_at_trx_commit

- 1：当你 commit 时，MySQL 必须将 redo log buffer 中的数据刷新进磁盘中。确保只要是 commit 是成功的，磁盘上就得有对应的 redo log 日志。是最安全的情况
- 0：每秒写一次日志并将其刷新到磁盘。
- 2：表示当 commit 时，将 redo log buffer 中的数据刷新进 os cache 中，然后依托操作系统每秒刷新一次的机制将数据同步到磁盘中，也存在丢失的风险。





## undo log



#### 概述

- MVCC 事务特性的重要组成部分。当我们对记录做了**变更操作时就会产生undo记录**，默认被记录到系统表空间（ibdata）中，5.6开始可以使用独立的undo表空间。
- undo记录中存储的时老版本数据，一个旧的事务需要读取数据时，为了能读取到老版本的数据，需要顺着undo链找到满足其可见性的记录。
- 大多数对数据的变更操作包括 insert、delete、update
  - 其中 insert 操作再事务提交前只对当前事务可见，因此产生undo日志可以再事务提交后直接删除。
  - 对于update、delete则需要维护多版本信息，再innodb中，update和delete操作产生的undo 日志被归成一类，即update_undo。



#### undo log 表空间

表空间由很多 segment 段组成，众多得段中有一种就是 undo segment。默认情况下 undo segment 会存放于系统表空间下中，或者说 undo log 默认会记录在共享表空间中。



#### roll back segment

Innodb存储引擎会先初始化好 rollback segment（回滚段），每个回滚段中会记录 n 个 undo log segment。而我们说的 undo log 就是在 undo log segment中申请出来得。



#### undo log truncate

截断

当不需要某个表中得数据时，可以执行 truncate sql 将表中得数据清空掉。同样的 undo log 的 truncate 本质就是为 undo log 表空间文件瘦身，将不需要的 undo log 从 undo log 表空间文件中清理掉。



#### undo log 类型

- **insert undo log**：记录的是 insert 语句对应的 undo log。
- **update undo log**：记录的是update、delete语句对应的 undo log。



#### 基本文件结构

为了保证事务并发操作时，写各自的undo log并不冲突，innodb采用回滚段的方式来维护 undo log的并发写入和持久化。回滚段实际上是一种undo 文件组织方式，每个回滚段又有多个undo log slot。



> 引用：http://mysql.taobao.org/monthly/2015/04/01/