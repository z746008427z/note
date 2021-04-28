## zookeeper安装

```shell
# -C 当前目录下将 zookeeper-3.4.10.tar.gz 文件解压缩到其他目录
tar -zxvf zookeeper-3.4.10.tar.gz -C apps/

# 修改配置文件
cd conf/
mv zoo_sample.cfg zoo.cfg
vi zoo.cfg

# 修改 zoo.cfg 配置文件
dataDir=/home/hadoop/data/zkdata/
```

#### 基本配置

- **tickTime**：心跳基本时间单位，毫秒级，zk基本上所有得时间都是这个时间的整数倍。

- **initLimit**：tickTime的个数，表示在leader选举结束后，followers与leader同步需要的时间，如果followers比较多或者说leader的数据灰常多时，同步时间相应可能会增加，那么这个值也需要相应增加。当然，这个值也是follower和observer在开始同步leader的数据时的最大等待时间(setSoTimeout)

- **synclimit**：tickTime的个数，这时间容易和上面的时间混淆，它也表示follower和observer与leader交互时的最大等待时间，只不过是在与leader同步完毕之后，进入正常请求转发或ping等消息交互时的超时时间。

- **dataDir**：内存数据库快照存放地址，如果没有指定事务日志存放地址（dataLogDir），默认也是存放这个路径下，建议两个地址分开存放到不同的设备上。

- **clientPort**：配置zk监听客户端连接的端口

  ![image-20210428112521418](zk%20%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA.assets/image-20210428112521418.png)

  ```shell
  # server：固定写法
  # serverid：每个服务器的指定ID(必须处于1-255之间，必须每一台机器不能重复)
  # host：主机名
  # tickpot：心跳通信端口
  # electionport：选举端口
  server.serverid=host:tickpot:electionport
  ```

#### 高级配置

- **dataLogDir**：将事务日志存储在该路径下，比较重要，这个日志存储的设备效率会影响 zk 的写吞吐量。
- **globalOutstandingLimit**：默认值1000，限定了所有连接到服务器上但是还没有返回响应的请求个数(所有客户端请求的总数，不是连接总数)，这个参数是针对单台服务器而言，设定太大可能会导致内存溢出。
- **preAllocSize**：默认值64M。以KB为的单位。预先分配额定空间用于后续transactionlog 写入，每当剩余空间小于4K时，就会又分配64M，如此循环。如果SNAP做得比较频繁(snapCount比较小的时候)，那么请减少这个值。
- **snapCount**：默认值100,000，当transaction每达到snapCount/2+rand.nextInt(snapCount/2)时，就做一次SNAPSHOT，默认情况下是50,000~100,000条transactionlog就会做一次，之所以用随机数是为了避免所有服务器可能在同一时间做snapshot.
- **maxClientCnxns**：**默认值是10，一个客户端能够连接到同一个服务器上的最大连接数，根据IP来区分。如果设置为0，表示没有任何限制。设置该值一方面是为了防止DoS攻击。**
- **clientPortAddress**：与clientPort匹配，表示某个IP地址，如果服务器有多个网络接口(多个IP地址)，如果没有设置这个属性，则clientPort会绑定到所有IP地址上，否则只绑定到该设置的IP地址上。
- **minSessionTimeout**：最小的session time时间，默认值是2个tick time,客户端设置的session time 如果小于这个值，则会被强制协调为这个最小值。
- **maxSessionTimeout**：最大的session time 时间，默认值是20个tick time. ,客户端设置的session time 如果大于这个值，则会被强制协调为这个最大值。

#### 将配置文件分发到集群其他机器

```shell
scp -r zookeeper-3.4.10/hadoop2:$PWD
scp -r zookeeper-3.4.10/hadoop3:$PWD
```

然后是最重要的步骤，一定不能忘了。 

​		去你的各个 ZooKeeper 服务器节点，新建目录 dataDir=/home/hadoop/data/zkdata，这个目录就是你在 zoo.cfg 中配置的 dataDir 的目录，建好之后，在里面新建一个文件，文件名叫 myid，里面存放的内容就是服务器的 id，就是 server.1=hadoop01:2888:3888 当中的 id, 就是 1，那么对应的每个服务器节点都应该做类似的操作 拿服务器 hadoop1 举例：

```shell
mkdir /home/hadoop/data/zkdata
cd data/zkdata
echo 1 > myid
```

当所有步骤完成时，以为我们zk的配置文件相关修改都完成。



#### 配置环境变量

```shell
vi .bashrc

#Zookeeper
export ZOOKEEPER_HOME=/home/hadoop/apps/zookeeper-3.4.10
export PATH=$PATH:$ZOOKEEPER_HOME/bin

source .bash
```

#### 验证安装成功

```shell
# 启动
zkServer.sh start
# 停止
zkServer.sh stop
# 查看状态
zkServer.sh status
```

![image-20210428172223895](zk%20%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA.assets/image-20210428172223895.png)

#### 查看进程

3台机器上都有QuorumPeerMain进程

```shell
jps
```

![image-20210428174436300](zk%20%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA.assets/image-20210428174436300.png)





















