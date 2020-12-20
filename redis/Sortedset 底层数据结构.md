# SortedSet 底层存储结构

sortedset 同时会由两种数据结构支持，`ziplist` 和 `skiplist`。

只有同时满足如下条件是使用 `ziplist`，其他时候则是使用 `skiplist`。

- 有序集合保存的元素数量小于 128 个。
- 有序集合保存的所有元素的长度小于 64 字节。



> ​		当 `ziplist` 作为存储结构时候，每个集合元素使用两个紧挨在一起的压缩列表结点来保存，第一个节点保存元素的成员，第二个元素保存元素的分值。

> ​		当使用 `skiplist` 作为存储结构时，使用 `skiplist` 按序保存元素分值，使用 `dict` 来保存元素和分值的对应关系。



## 有序集 SortedSet

#### 命令简介

```properties
# 添加元素
# table1为有序集的名字	100为用于排序字段（redis把它叫做score）	a为我们要存储的元素
127.0.0.1:6379> zadd table1 100 a
(integer) 1
127.0.0.1:6379> zadd table1 200 b
(integer) 1
127.0.0.1:6379> zadd table1 300 c
(integer) 1

# 按照元素索引返回有序集中的元素，索引从0开始
127.0.0.1:6379> zrange table1 0 1
1) "a"
2) "b"

# 按照元素排序范围返回有序集中的元素，这里用于排序的字段在redis中叫做score
127.0.0.1:6379> zrangebyscore table1 150 400
1) "b"
2) "c"

# 删除元素
127.0.0.1:6379> zrem table1 b
(integer) 1
```

**在有序集中，用于排序的值叫做 score，实际存储的值叫做 member。**



## 跳跃表（skiplist）

跳跃表是一种基于有序链表的扩展，简称跳表。跳表会维护多个索引链表和原链表。

> 不断得提升新的关键节点形成新的有序链表，通过空间换时间。

对有序链表进行`关键点`的提升，作为插入对比的索引。

<img src="/Sortedset 底层数据结构.assets\image-20201217223812246.png" alt="image-20201217223812246" style="zoom: 50%;" />

#### 插入逻辑

链表插入新的元素，只需对比新元素的值和关键点链表，这样就能将对比次数缩减。



#### 关键点提取逻辑

因为在跳表中，使用空间换时间，会有多层的索引链表，每一层索引链表都是上层的一半，当数据量较大的时候，就能够很轻松的降低链表查询消耗的性能。

- 提取极限：同一层只有两个节点。

这样的多层链表结构称之为**跳表**。



#### 提取新的索引节点

跳表采用抛硬币的做法，在插入之后会随机判断新插入的元素是否成为新的关键点。



#### 跳表插入节点的流程

1. 新节点和各层索引节点逐一比较，确定原链表的插入位置 O(logN)。
2. 把索引插入到原链表，O(1)。
3. 利用抛硬币的随机方式，决定新节点是否提升为上一级索引 O(logN)。

**总体上跳表插入的时间复杂度是O(logN)，空间复杂度是O(N)。**



#### 节点删除的流程

1. 在索引层找到相应节点进行删除，删除每一层的相同节点 O(logN)。
2. 如果某一层在删除节点后只剩下一个节点，那么这个链表就可以删除了。O(logN)。

**跳表删除的时间复杂度是 O(logN)。**



#### 使用场景

###### 排行榜

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Set;
import java.util.UUID;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Tuple;
public class GameRankSample {
    static int TOTAL_SIZE = 20;
    public static void main(String[] args) 
    {
        //连接信息，从控制台可以获得
        String host = "xxxxxxxxxx.m.cnhz1.kvstore.aliyuncs.com";
        int port = 6379;
        Jedis jedis = new Jedis(host, port);
        try {
            //实例密码
            String authString = jedis.auth("password");//password
            if (!authString.equals("OK"))
            {
                System.err.println("AUTH Failed: " + authString);
                return;
            }
            //Key(键)
            String key = "游戏名：奔跑吧，阿里！";
            //清除可能的已有数据
            jedis.del(key);
            //模拟生成若干个游戏玩家
            List<String> playerList = new ArrayList<String>();
            for (int i = 0; i < TOTAL_SIZE; ++i)
            {
                //随机生成每个玩家的ID
                playerList.add(UUID.randomUUID().toString());
            }
            System.out.println("输入所有玩家 ");
            //记录每个玩家的得分
            for (int i = 0; i < playerList.size(); i++)
            {
                //随机生成数字，模拟玩家的游戏得分
                int score = (int)(Math.random()*5000);
                String member = playerList.get(i);
                System.out.println("玩家ID：" + member + "， 玩家得分: " + score);
                //将玩家的ID和得分，都加到对应key的SortedSet中去
                jedis.zadd(key, score, member);
            }
            //输出打印全部玩家排行榜
            System.out.println();
            System.out.println("       "+key);
            System.out.println("       全部玩家排行榜                    ");
            //从对应key的SortedSet中获取已经排好序的玩家列表
            Set<Tuple> scoreList = jedis.zrevrangeWithScores(key, 0, -1);
            for (Tuple item : scoreList) {  
                System.out.println("玩家ID："+item.getElement()+"， 玩家得分:"+Double.valueOf(item.getScore()).intValue());
            }  
            //输出打印Top5玩家排行榜
            System.out.println();
            System.out.println("       "+key);
            System.out.println("       Top 玩家");
            scoreList = jedis.zrevrangeWithScores(key, 0, 4);
            for (Tuple item : scoreList) {  
                System.out.println("玩家ID："+item.getElement()+"， 玩家得分:"+Double.valueOf(item.getScore()).intValue());
            }
            //输出打印特定玩家列表
            System.out.println();
            System.out.println("         "+key);
            System.out.println("          积分在1000至2000的玩家");
            //从对应key的SortedSet中获取已经积分在1000至2000的玩家列表
            scoreList = jedis.zrangeByScoreWithScores(key, 1000, 2000);
            for (Tuple item : scoreList) {  
                System.out.println("玩家ID："+item.getElement()+"， 玩家得分:"+Double.valueOf(item.getScore()).intValue());
            } 
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            jedis.quit();
            jedis.close();
        }
    }
}
```

###### 运行结果

```properties
输入所有玩家 
玩家ID：9193e26f-6a71-4c76-8666-eaf8ee97ac86， 玩家得分: 3860
玩家ID：db03520b-75a3-48e5-850a-071722ff7afb， 玩家得分: 4853
玩家ID：d302d24d-d380-4e15-a4d6-84f71313f27a， 玩家得分: 2931
玩家ID：bee46f9d-4b05-425e-8451-8aa6d48858e6， 玩家得分: 1796
玩家ID：ec24fb9e-366e-4b89-a0d5-0be151a8cad0， 玩家得分: 2263
玩家ID：e11ecc2c-cd51-4339-8412-c711142ca7aa， 玩家得分: 1848
玩家ID：4c396f67-da7c-4b99-a783-25919d52d756， 玩家得分: 958
玩家ID：a6299dd2-4f38-4528-bb5a-aa2d48a9f94a， 玩家得分: 2428
玩家ID：2e4ec631-1e4e-4ef0-914f-7bf1745f7d65， 玩家得分: 4478
玩家ID：24235a85-85b9-476e-8b96-39f294f57aa7， 玩家得分: 1655
玩家ID：e3e8e1fa-6aac-4a0c-af80-4c4a1e126cd1， 玩家得分: 4064
玩家ID：99bc5b4f-e32a-4295-bc3a-0324887bb77e， 玩家得分: 4852
玩家ID：19e2aa6b-a2d8-4e56-bdf7-8b59f64bd8e0， 玩家得分: 3394
玩家ID：cb62bb24-1318-4af2-9d9b-fbff7280dbec， 玩家得分: 3405
玩家ID：ec0f06da-91ee-447b-b935-7ca935dc7968， 玩家得分: 4391
玩家ID：2c814a6f-3706-4280-9085-5fe5fd56b71c， 玩家得分: 2510
玩家ID：9ee2ed6d-08b8-4e7f-b52c-9adfe1e32dda， 玩家得分: 63
玩家ID：0293b43a-1554-4157-a95b-b78de9edf6dd， 玩家得分: 1008
玩家ID：674bbdd1-2023-46ae-bbe6-dfcd8e372430， 玩家得分: 2265
玩家ID：34574e3e-9cc5-43ed-ba15-9f5405312692， 玩家得分: 3734
              游戏名：奔跑吧，阿里！                    
              全部玩家排行榜                    
玩家ID：db03520b-75a3-48e5-850a-071722ff7afb， 玩家得分:4853
玩家ID：99bc5b4f-e32a-4295-bc3a-0324887bb77e， 玩家得分:4852
玩家ID：2e4ec631-1e4e-4ef0-914f-7bf1745f7d65， 玩家得分:4478
玩家ID：ec0f06da-91ee-447b-b935-7ca935dc7968， 玩家得分:4391
玩家ID：e3e8e1fa-6aac-4a0c-af80-4c4a1e126cd1， 玩家得分:4064
玩家ID：9193e26f-6a71-4c76-8666-eaf8ee97ac86， 玩家得分:3860
玩家ID：34574e3e-9cc5-43ed-ba15-9f5405312692， 玩家得分:3734
玩家ID：cb62bb24-1318-4af2-9d9b-fbff7280dbec， 玩家得分:3405
玩家ID：19e2aa6b-a2d8-4e56-bdf7-8b59f64bd8e0， 玩家得分:3394
玩家ID：d302d24d-d380-4e15-a4d6-84f71313f27a， 玩家得分:2931
玩家ID：2c814a6f-3706-4280-9085-5fe5fd56b71c， 玩家得分:2510
玩家ID：a6299dd2-4f38-4528-bb5a-aa2d48a9f94a， 玩家得分:2428
玩家ID：674bbdd1-2023-46ae-bbe6-dfcd8e372430， 玩家得分:2265
玩家ID：ec24fb9e-366e-4b89-a0d5-0be151a8cad0， 玩家得分:2263
玩家ID：e11ecc2c-cd51-4339-8412-c711142ca7aa， 玩家得分:1848
玩家ID：bee46f9d-4b05-425e-8451-8aa6d48858e6， 玩家得分:1796
玩家ID：24235a85-85b9-476e-8b96-39f294f57aa7， 玩家得分:1655
玩家ID：0293b43a-1554-4157-a95b-b78de9edf6dd， 玩家得分:1008
玩家ID：4c396f67-da7c-4b99-a783-25919d52d756， 玩家得分:958
玩家ID：9ee2ed6d-08b8-4e7f-b52c-9adfe1e32dda， 玩家得分:63
      游戏名：奔跑吧，阿里！                    
         Top 玩家                    
玩家ID：db03520b-75a3-48e5-850a-071722ff7afb， 玩家得分:4853
玩家ID：99bc5b4f-e32a-4295-bc3a-0324887bb77e， 玩家得分:4852
玩家ID：2e4ec631-1e4e-4ef0-914f-7bf1745f7d65， 玩家得分:4478
玩家ID：ec0f06da-91ee-447b-b935-7ca935dc7968， 玩家得分:4391
玩家ID：e3e8e1fa-6aac-4a0c-af80-4c4a1e126cd1， 玩家得分:4064
          游戏名：奔跑吧，阿里！                    
          积分在1000至2000的玩家              
玩家ID：0293b43a-1554-4157-a95b-b78de9edf6dd， 玩家得分:1008
玩家ID：24235a85-85b9-476e-8b96-39f294f57aa7， 玩家得分:1655
玩家ID：bee46f9d-4b05-425e-8451-8aa6d48858e6， 玩家得分:1796
玩家ID：e11ecc2c-cd51-4339-8412-c711142ca7aa， 玩家得分:1848
```



> 引用：
>
> ​		https://juejin.cn/post/6844903512413831181#heading-2
>
> ​		https://help.aliyun.com/document_detail/26366.html?spm=a2c4g.11186623.6.795.634b5bbaVlFol9
>
> ​		https://www.jianshu.com/p/14dde3031e0b

