## zk server 状态

server 在工作过程中有三种状态：

- **LOOKING：**当前 server 不知道 leader 是谁，正在搜寻
- **LEADING：**当前 server 即为选举出来的 leader
- **FOLLOWING：**leader 已经选举出来，当前 server 与之同步



## 节点数据操作流程

<img src="zk%20%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90.assets/image-20210430110745110.png" alt="image-20210430110745110" style="zoom: 67%;" />

#### 写入流程

1. 在 client 向 follower 发出一个写的请求。
2. follower 把请求发送给 leader。
3. leader 接受到以后开始发起投票并通知 follower 进行投票。
4. follower 把投票结果发送给 leader。
5. leader 将结果汇总后如果需要写入，则开始写入同时把写入操作通知给 leader，然后 commit。
6. follower 把请求结果返回给 client。



#### follower 主要的四种功能

1. 向 leader 发送请求（PING消息、REQUEST消息、ACK消息、REVALIDATE消息）
2. 接受 leader 消息并进行处理
3. 接受 client 的请求，如果为写请求，发送给 leader 进行投票
4. 返回 client 结果



#### Follower 消息循环处理几种来自 leader 的消息

1. PING 消息：心跳消息；
2. PROPOSAL 消息：leader 发起的提案，要求 follower 投票；
3. COMMIT 消息：服务器端最新依次提案的信息；
4. UPDATE 消息：表明同步完成；
5. RECALIDATE 消息：根据 leader 的 REVALIDATE 结果，关闭待 revalidate 的 session 还是允许其接受消息；
6. SYNC 消息：返回 SYNC 结果到客户端，这个消息最初由客户端发起，用来强制得到最新的更新；





























