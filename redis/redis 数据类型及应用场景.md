## string

string 类型是 redis 中最基本的数据类型，最常用的数据类型。

string 类型在 redis 中是二进制安全的，意味着 string 值关心二进制的字符串，不关心具体格式。

string 类型的值最大能存储 512 MB。



redis 使用 SDS（简单动态字符串）这个结构体来存储字符串。

- len：字符串的长度（实际使用的长度）
- alloc：分配内存的大小
- flags：标志位，低三位表示类型，其余五位未使用
- buf：字符数组



#### 常用命令

get、set、incr、decr、mget 等。



#### 应用场景

- 常规 key-value 缓存应用。

- 常规计数：微博数，粉丝数。
  - 实现方式：string 在 redis 内部存储默认就是一个字符串，被 redisObject 所引用，当遇到 incr，decr 等操作时会转成数值型进行计算，此时 redisObject 的 encoding 字段为 int。

```properties
redis 127.0.0.1:6379> SET name "runoob"
"OK" 
redis 127.0.0.1:6379> GET name
"runoob"
```



## hash

hash 是一个键值（key -> value）对集合。redis hash 是一个 string 类型的 field 和 value 的映射表，hash 适合用于存储对象。



#### 常用命令

hget、hset、hgetall 。。。



#### 应用场景

存储用户信息对象数据。

```properties
redis> HSET myhash field1 "Hello" field2 "World"
"OK" 
redis> HGET myhash field1
"Hello" 
redis> HGET myhash field2
"World"
```

> 每个 hash 可以存储 2的32次方-1 键值对（40多亿）



#### 内部编码

- **ziplist（压缩列表）**

  ​		当 **哈希类型** 元素个数 **小于** `hash-max-ziplist-entries` 配置（默认 `512` 个）、同时 **所有值** 都 **小于** `hash-max-ziplist-value` 配置（默认 `64` 字节）时，`Redis` 会使用 `ziplist` 作为 **哈希** 的 **内部实现**，`ziplist` 使用更加 **紧凑的结构** 实现多个元素的 **连续存储**，所以在 **节省内存** 方面比 `hashtable` 更加优秀。

  

- **hashtable（哈希表）**

  ​		当 **哈希类型** 无法满足 `ziplist` 的条件时，`Redis` 会使用 `hashtable` 作为 **哈希** 的 **内部实现**，因为此时 `ziplist` 的 **读写效率** 会下降，而 `hashtable` 的读写 **时间复杂度** 为 `O（1）`。



## list

简单的字符串列表奥，按照插入顺序排序。可以添加一个元素到列表的头部（左边）或者（右边）。



#### 常用命令

lpush（左边插入）、rpush、lpop（左边移除第一个）、rpop、lrange（获取列表片段 LRANGE key start stop）。。。



```properties
redis 127.0.0.1:6379> lpush runoob redis
(integer) 1 redis 127.0.0.1:6379> lpush runoob mongodb
(integer) 2 redis 127.0.0.1:6379> lpush runoob rabitmq
(integer) 3 redis 127.0.0.1:6379> lrange runoob 0 10
1) "rabitmq"
2) "mongodb"
3) "redis" 
redis 127.0.0.1:6379>
```

> 列表最多可存储 2的32次方 - 1 元素 (4294967295, 每个列表可存储40多亿)。



#### 应用场景

关注列表，粉丝列表等。

消息低劣系统



## set

set 是 string 类型的无序集合。集合是通过 hashtable 实现的。可以交集，并集，差集等，set 中的元素没有顺序，所以添加、删除、查找的复杂度都是 O(1)。

> sadd 命令：添加一个 string 元素到 key 对应的 set 集合中，**成功返回1**，如果元素**已经在集合中返回 0**，如果 key 对应的 set **不存在则返回错误**。



#### 常用命令

sadd、spop、smembers、sunion。。。



#### 应用场景

> 对外提供的功能于 list 类似是一个列表的功能，特殊之处在于 set 是可以自动排重的。

```properties
sadd key member
```

微博，将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。



1. redis 还为集合提供了求交集、并集、差集等操作，可以非常方便的实现如共同关注、共同喜好、二度好友等。

```properties
# book表存储book名称
set book:1:name    ”The Ruby Programming Language”
set book:2:name    ”Ruby on rail”
set book:3:name    ”Programming Erlang” 

# tag表使用集合来存储数据，因为集合擅长求交集、并集
sadd tag:ruby 1 sadd tag:ruby 2 sadd tag:web 2 sadd tag:erlang 3

# 即属于ruby又属于web的书？
inter_list = redis.sinter("tag.web", "tag:ruby") 
# 即属于ruby，但不属于web的书？
inter_list = redis.sdiff("tag.ruby", "tag:web") 
# 属于ruby和属于web的书的合集？
inter_list = redis.sunion("tag.ruby", "tag:web")
```

2. 获取某段时间所有数据去重值。

```properties
redis 127.0.0.1:6379> sadd runoob redis
(integer) 1 redis 127.0.0.1:6379> sadd runoob mongodb
(integer) 1 redis 127.0.0.1:6379> sadd runoob rabitmq
(integer) 1 redis 127.0.0.1:6379> sadd runoob rabitmq
(integer) 0 redis 127.0.0.1:6379> smembers runoob
1) "redis"
2) "rabitmq"
3) "mongodb"
```

> 集合中最大的成员数为 2的32次方 - 1(4294967295, 每个集合可存储40多亿个成员)。



## zset(sorted set)

和 set 一样也是 string 类型元素的集合，且不允许重复的成员。



#### 常用命令

zadd、zrange、zrem、zcard。。。



#### 使用场景

sorted set 可以通过用户额外提供一个优先级（score）的参数来为成员排序，并且是插入排序的，即自动排序。



#### 实现方式

sorted set 的内部使用 hashmap 和 跳表（skiplist）来暴增数据的存储和有序，hashmap 里放的是成员到 score 的映射，而跳表里存放的是所有的成员。排序依据是 hashmap 里存的 score，使用跳表的结构可以获得比较高的查询效率，并且在实现上比较简单。

```properties
zadd key score member
```

```properties
redis 127.0.0.1:6379> zadd runoob 0 redis
(integer) 1 redis 127.0.0.1:6379> zadd runoob 0 mongodb
(integer) 1 redis 127.0.0.1:6379> zadd runoob 0 rabitmq
(integer) 1 redis 127.0.0.1:6379> zadd runoob 0 rabitmq
(integer) 0 redis 127.0.0.1:6379> > ZRANGEBYSCORE runoob 0 1000
1) "mongodb"
2) "rabitmq"
3) "redis"
```



## skiplist

一种随机化的数据，以有序的方式在层次化的链表中保存元素，效率和平衡树媲美。

![image-20201218145945305](D:\note\redis\redis 数据类型及应用场景.assets\image-20201218145945305.png)

跳跃表主要由以下部分构成：

- 表头：负责维护跳表的节点指针。

- 跳表节点：保存元素值，以及多个层。

- 层：保存着指向其他元素的指针。高层的指针越过的元素数量大于等于低层的指针，为了提高查找的效率，程序总是从高层开始访问，然后随着元素值范围的缩小，慢慢降低层次。

- 表尾：全部由 NULL 组成，表示跳跃表的末尾。

  

#### 使用场景

跳跃表在 Redis 的唯一作用， 就是实现有序集数据类型。

跳跃表将指向有序集的 `score` 值和 `member` 域的指针作为元素， 并以 `score` 值为索引， 对有序集元素进行排序。



#### 实现方式

1. 允许重复的 `score` 值：多个不同的 `member` 的 `score` 值可以相同。
2. 进行对比操作时，不仅要检查 `score` 值，还要检查 `member` ：当 `score` 值可以重复时，单靠 `score` 值无法判断一个元素的身份，所以需要连 `member` 域都一并检查才行。
3. 每个节点都带有一个高度为 1 层的后退指针，用于从表尾方向向表头方向迭代：当执行 [ZREVRANGE](http://redis.readthedocs.org/en/latest/sorted_set/zrevrange.html#zrevrange) 或 [ZREVRANGEBYSCORE](http://redis.readthedocs.org/en/latest/sorted_set/zrevrangebyscore.html#zrevrangebyscore) 这类以逆序处理有序集的命令时，就会用到这个属性。



## bitmap

通过一个bit位来表示某个元素对应的值或者状态,其中的key就是对应元素本身。

可以把 Bitmaps 想象成一个以 `位` 为单位的数组，数组的每个单元只能存储 0 和 1，数组的下标在Bitmaps中叫做偏移量。



#### 常用命令

设置值

```properties
SETBIT key offset value
```

获取值

```properties
GETBIT key offset
```

获取Bitmaps 指定范围值为 1 的位个数

```properties
BITCOUNT key start
```



> 引用：https://juejin.cn/post/6844903561839525902#heading-6



## HyperLogLog

用来做基数统计的算法。

**优点**

在输入元素数量或者体积非常大的时候，计算基数所需的空间总是固定的，并且是很小的。



#### HyperLogLog 结构

在 Redis 中每个键占用的内容都是 12K，理论存储近似**接近 2^64 个值**，不管存储的内容是什么。这是一个**基于基数估计的算法**，只能**比较准确的估算出基数**，可以使用**少量固定的内存去存储**并识别集合中的唯一元素。但是这个**估算的基数并不一定准确**，是一个带有 **0.81% 标准错误**（standard error）的近似值。



#### 常用命令

PFADD、PFCOUNT、PFMERGE。。。

- **PFADD**：

  将任意数量的元素添加到指定的 HyperLogLog 里面。

  时间复杂度： 每添加一个元素的复杂度为 O(1) 。

  如果 HyperLogLog 估计的近似基数（approximated cardinality）在命令执行之后出现了变化， 那么命令返回 1 ， 否则返回 0 。 

  如果命令执行时给定的键不存在， 那么程序将先创建一个空的 HyperLogLog 结构， 然后再执行命令。

  ```properties
  # 命令格式：PFADD key element [element …]
  # 如果给定的键不存在，那么命令会创建一个空的 HyperLogLog，并向客户端返回 1
  127.0.0.1:6379> PFADD ip_20190301 "192.168.0.1" "192.168.0.2" "192.168.0.3"
  (integer) 1
  # 元素估计数量没有变化，返回 0（因为 192.168.0.1 已经存在）
  127.0.0.1:6379> PFADD ip_20190301 "192.168.0.1"
  (integer) 0
  # 添加一个不存在的元素，返回 1。注意，此时 HyperLogLog 内部存储会被更新，因为要记录新元素
  127.0.0.1:6379> PFADD ip_20190301 "192.168.0.4"
  (integer) 1
  ```

- **PFCOUNT**：

  当 PFCOUNT key [key …] 命令作用于单个键时，返回储存在给定键的 HyperLogLog 的近似基数，如果键不存在，那么返回 0，复杂度为 O(1)，并且具有非常低的平均常数时间；

  当 PFCOUNT key [key …] 命令作用于多个键时，返回所有给定 HyperLogLog 的并集的近似基数，这个近似基数是通过将所有给定 HyperLogLog 合并至一个临时 HyperLogLog 来计算得出的，复杂度为 O(N)，常数时间也比处理单个 HyperLogLog 时要大得多。

  ```properties
  # 返回 ip_20190301 包含的唯一元素的近似数量
  127.0.0.1:6379> PFCOUNT ip_20190301
  (integer) 4
  127.0.0.1:6379> PFADD ip_20190301 "192.168.0.5"
  (integer) 1
  127.0.0.1:6379> PFCOUNT ip_20190301
  (integer) 5
  127.0.0.1:6379> PFADD ip_20190302 "192.168.0.1" "192.168.0.6" "192.168.0.7"
  (integer) 1
  # 返回 ip_20190301 和 ip_20190302 包含的唯一元素的近似数量
  127.0.0.1:6379> PFCOUNT ip_20190301 ip_20190302
  (integer) 7
  ```

- **PFMERGE**：

  将多个 HyperLogLog 合并（merge）为一个 HyperLogLog，合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合（observed set）的并集。

  时间复杂度是 O(N)，其中 N 为被合并的 HyperLogLog 数量，不过这个命令的常数复杂度比较高。

  命令格式：PFMERGE destkey sourcekey [sourcekey …]，合并得出的 HyperLogLog 会被储存在 destkey 键里面，如果该键并不存在，那么命令在执行之前，会先为该键创建一个空的 HyperLogLog。

  ```properties
  # ip_2019030102 是 ip_20190301 与 ip_20190302 并集
  127.0.0.1:6379> PFMERGE ip_2019030102 ip_20190301 ip_20190302
  OK
  127.0.0.1:6379> PFCOUNT ip_2019030102
  (integer) 7
  ```

  

#### 应用场景

计算日活，7日活，月活数据。



## geospatial（地理图）

redis 3.2 版本中加入了地理空间 (geospatial) 以及索引半径查询。主要用在需要地理位置的应用上。



// TODO



## 各数据类型应用场景

|         类型         |                             简介                             |                             特性                             |                             场景                             |
| :------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|    string(字符串)    |                          二进制安全                          | 可以包含任何数据，比如 jpg 图片或者序列化的对象，一个键最大能存储 512 M |                             ---                              |
|     hash（字典）     |             键值对结合，即编程语言中的 map 类型              | 适合存储对象，并且可以像数据库中 update 一个属性一样只修改某一项属性值（Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去） |                   存储、读取、修改用户属性                   |
|     list（列表）     |                       链表（双向链表）                       |              增删快，提供了操作某一段元素的 API              |   1、最新朋友圈排行等功能（比如朋友圈的时间线）2、消息队列   |
|     set（集合）      |                    哈希表实现，元素不重复                    | 1、添加、删除、查找的复杂度都是O(1)。2、为集合提供了求交集、并集、差集等操作 | 1、共同好友、2、利用唯一性，统计访问网站的所有独立ip。3、好友推荐时，根据 tag 求交集，大于某个阈值就可以推荐。 |
| sort set（有序集合） | 将 set 中的元素增加一个权重参数 score，元素按 score 有序排列。 |              数据插入集合时，已经进行天然排序。              |               1、排行版。2、带权重的消息队列。               |



## 实际应用场景

1. 显示最新的项目列表

   ​		每条评论都有一个唯一的递增的 ID 字段。我们可以使用分页来制作主页和评论页，使用 redis 的模板，每次新评论发表时，我们会将它的 ID 添加到一个 redis 列表。

   ```properties
   LPUSH latest.comments <ID>
   ```

   ​		将列表裁剪为指定长度，因此Redis只需要保存最新的5000条评论

   ```properties
   LTRIM latest.comments 0 5000
   ```

   ​		获取最新评论的项目范围时，我们调用一个函数来完成（使用伪代码）

   ```c
   FUNCTION get_latest_comments(start, num_items):  
       id_list = redis.lrange("latest.comments",start,start+num_items - 1)  
       IF id_list.length < num_items  
           id_list = SQL_DB("SELECT ... ORDER BY time LIMIT ...")  
       END  
       RETURN id_list  
   END	
   ```

2. 删除与过滤

   ​		可以使用 LREM 来删除评论。如果删除操作非常少，另一个选择是直接跳过评论条目的入口，报告说该评论已经不存在。

   ```properties
   redis 127.0.0.1:6379> LREM KEY_NAME COUNT VALUE
   ```

3. 排行榜相关

   - 列出前 100 名高分选手
   - 列出某用户当前的全球排名

   ```properties
   ZADD leaderboard <score> <username>
   
   # 得到前100名高分选手
   ZREVRANGE leaderboard 0 99
   # 用户的全球排名
   ZRANK leaderboard
   ```

4. 按照用户投票和时间排序

5. 处理过期项目

6. 计数

   ​		可以加上各种计数，用GETSET重置，或者是让它们过期。

   ```properties
   INCR user:<id> EXPIRE 
   user:<id> 60
   ```

7. 特定时间内的特定项目

   ​		统计在某段特点时间里有多少特定用户访问了某个特定资源。比如：获得一次新的页面浏览时的数据：

   ```properties
   SADD page:day1:<page_id> <user_id>
   ```

   ​		测试某个特定用户是否访问了这个页面？

   ```properties
   SISMEMBER page:day1:<page_id>
   ```

8. 实时分析正在发生的情况，用于数据统计与防止垃圾邮件等

9. Pub / Sub

10. 队列

11. 缓存

    

    

    

    

    









