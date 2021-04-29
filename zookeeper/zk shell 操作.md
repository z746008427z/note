## shell 命令工具

```shell
# 客户端连接
zkCli.sh -server hadoops2:2181

[hadoop@hadoop1 ~]$ zkCli.sh -server hadoop2:2181
Connecting to hadoop2:2181
2018-03-21 19:55:53,744 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
2018-03-21 19:55:53,748 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=hadoop1
2018-03-21 19:55:53,749 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_73
2018-03-21 19:55:53,751 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2018-03-21 19:55:53,751 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/local/jdk1.8.0_73/jre
2018-03-21 19:55:53,751 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/home/hadoop/apps/zookeeper-3.4.10/bin/../build/classes:/home/hadoop/apps/zookeeper-3.4.10/bin/../build/lib/*.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../lib/slf4j-log4j12-1.6.1.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../lib/slf4j-api-1.6.1.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../lib/netty-3.10.5.Final.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../lib/log4j-1.2.16.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../lib/jline-0.9.94.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../zookeeper-3.4.10.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../src/java/lib/*.jar:/home/hadoop/apps/zookeeper-3.4.10/bin/../conf::/usr/local/jdk1.8.0_73/lib:/usr/local/jdk1.8.0_73/jre/lib
2018-03-21 19:55:53,751 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2018-03-21 19:55:53,751 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2018-03-21 19:55:53,751 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2018-03-21 19:55:53,752 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2018-03-21 19:55:53,752 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2018-03-21 19:55:53,752 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=2.6.32-573.el6.x86_64
2018-03-21 19:55:53,752 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=hadoop
2018-03-21 19:55:53,752 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/home/hadoop
2018-03-21 19:55:53,752 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/home/hadoop
2018-03-21 19:55:53,755 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=hadoop2:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@5c29bfd
Welcome to ZooKeeper!
2018-03-21 19:55:53,789 [myid:] - INFO  [main-SendThread(hadoop2:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server hadoop2/192.168.123.103:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
2018-03-21 19:55:53,931 [myid:] - INFO  [main-SendThread(hadoop2:2181):ClientCnxn$SendThread@876] - Socket connection established to hadoop2/192.168.123.103:2181, initiating session
2018-03-21 19:55:53,977 [myid:] - INFO  [main-SendThread(hadoop2:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server hadoop2/192.168.123.103:2181, sessionid = 0x262486284b70000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: hadoop2:2181(CONNECTED) 0]
```

#### help 命令

![image-20210429141043300](zk%20shell%20%E6%93%8D%E4%BD%9C.assets/image-20210429141043300.png)

## zk 命令

查看当前zk中所包含的内容：**ls /**

```shel
[zk: hadoop2:2181(CONNECTED) 1] ls /
[zookeeper]
[zk: hadoop2:2181(CONNECTED) 2] 
```

创建一个Znode节点aa，以及和它相关字符

```shel
[zk: hadoop2:2181(CONNECTED) 2] create /aa "my first zk"
Created /aa
[zk: hadoop2:2181(CONNECTED) 3] 
```

创建带编号的持久性节点"bb"

```shell
[zk: localhost:2181(CONNECTED) 1] create -s /bb "bb"
Created /bb0000000001
[zk: localhost:2181(CONNECTED) 2] 
```

创建不带编号的临时节点"cc"

```shell
[zk: localhost:2181(CONNECTED) 2] create -e /cc "cc"
Created /cc
[zk: localhost:2181(CONNECTED) 3] 
```

创建带编号的临时节点"dd"

```shell
[zk: localhost:2181(CONNECTED) 3] create -s -e /dd "dd"
Created /dd0000000003
[zk: localhost:2181(CONNECTED) 4] 
```

使用ls命名来查看现在zk包含的内容

```shell
[zk: localhost:2181(CONNECTED) 4] ls /
[cc, dd0000000003, zookeeper, bb0000000001]
[zk: localhost:2181(CONNECTED) 5]
```

关闭本次会话session，重新打开一个链接

```shell
[zk: localhost:2181(CONNECTED) 5] close
2018-03-22 13:03:29,137 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x1624c10e8d90000 closed
[zk: localhost:2181(CLOSED) 6] 2018-03-22 13:03:29,139 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@519] - EventThread shut down for session: 0x1624c10e8d90000

[zk: localhost:2181(CLOSED) 6] ls /
Not connected
[zk: localhost:2181(CLOSED) 7] connect hadoop1:2181
```

重新查看节点，临时节点已经随上次会话关闭删除了

```shel
[zk: hadoop1:2181(CONNECTED) 8] ls /
[zookeeper, bb0000000001]
[zk: hadoop1:2181(CONNECTED) 9] 
```

使用get命令来确认第二步中所创建的Znode是否包含我们创建的字符串：get /aa

```shell
[zk: hadoop2:2181(CONNECTED) 4] get /aa
my first zk
cZxid = 0x100000002
ctime = Wed Mar 21 20:01:02 CST 2018
mZxid = 0x100000002
mtime = Wed Mar 21 20:01:02 CST 2018
pZxid = 0x100000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 11
numChildren = 0
[zk: hadoop2:2181(CONNECTED) 5] 
```

通过 set 命令来对zk所关联的字符串进行设置：set /aa haha123

```shel
[zk: hadoop2:2181(CONNECTED) 6] set /aa haha123 
cZxid = 0x100000002
ctime = Wed Mar 21 20:01:02 CST 2018
mZxid = 0x100000004
mtime = Wed Mar 21 20:04:10 CST 2018
pZxid = 0x100000002
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 0
[zk: hadoop2:2181(CONNECTED) 7] 
```

再次使用 get 命令来查看，上次修改的内容，执行命令：get /aa

```shel
[zk: hadoop2:2181(CONNECTED) 7] get /aa
haha123
cZxid = 0x100000002
ctime = Wed Mar 21 20:01:02 CST 2018
mZxid = 0x100000004
mtime = Wed Mar 21 20:04:10 CST 2018
pZxid = 0x100000002
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 0
[zk: hadoop2:2181(CONNECTED) 8]
```

讲刚才创建的Znode删除，执行命令：delete /aa

```shel
[zk: hadoop2:2181(CONNECTED) 8] delete /aa
[zk: hadoop2:2181(CONNECTED) 9] 
```

最后再次使用ls命令查看 zk 的内容

```shell
[zk: hadoop2:2181(CONNECTED) 9] ls /
[zookeeper]
[zk: hadoop2:2181(CONNECTED) 10] 
```

退出，执行命令：quit

```shel
[zk: hadoop2:2181(CONNECTED) 10] quit
Quitting...
2018-03-21 20:07:11,133 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x262486284b70000 closed
2018-03-21 20:07:11,139 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@519] - EventThread shut down for session: 0x262486284b70000
[hadoop@hadoop1 ~]$ 
```



## 状态信息

查看一个文件的状态信息

```shel
[zk: localhost:2181(CONNECTED) 1] stat /a
cZxid = 0x200000009
ctime = Thu Mar 22 13:07:19 CST 2018
mZxid = 0x200000009
mtime = Thu Mar 22 13:07:19 CST 2018
pZxid = 0x200000009
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 1
numChildren = 0
[zk: localhost:2181(CONNECTED) 2] 
```









































