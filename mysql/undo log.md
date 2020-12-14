## undo log



#### 概述

- MVCC 事务特性的重要组成部分。当我们对记录做了**变更操作时就会产生undo记录**，默认被记录到系统表空间（ibdata）中，5.6开始可以使用独立的undo表空间。
- undo记录中存储的时老版本数据，一个旧的事务需要读取数据时，为了能读取到老版本的数据，需要顺着undo链找到满足其可见性的记录。
- 大多数对数据的变更操作包括 insert、delete、update
  - 其中 insert 操作再事务提交前只对当前事务可见，因此产生undo日志可以再事务提交后直接删除。
  - 对于update、delete则需要维护多版本信息，再innodb中，update和delete操作产生的undo 日志被归成一类，即update_undo。



#### 基本文件结构

为了保证事务并发操作时，写各自的undo log并不冲突，innodb采用回滚段的方式来维护 undo log的并发写入和持久化。回滚段实际上是一种undo 文件组织方式，每个回滚段又有多个undo log slot。

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201215010445493.png)



> 引用：http://mysql.taobao.org/monthly/2015/04/01/