# 索引

## 索引类型

#### 数据结构角度

- B+ 树索引
- hash 索引
- FULLTEXT 索引
- R- Tree 索引

#### 物理存储角度

- 聚簇索引
- 非聚簇索引

#### 逻辑角度

- **主键索引**：是一种特殊的唯一索引，不允许有空值。
- **普通索引** 或者 **单列索引**
- **多列索引（符合索引）**：复合索引指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用符合索引时遵循最左匹配原则
- **唯一索引或非唯一索引**
- **空间索引**：空间索引是对空间数据类型的字段建立的索引，MYSQL中的空间数据类型有4种，分别是GEOMETRY、POINT、LINESTRING、POLYGON。



## 底层索引数据结构

> 引用：
>
> ​		https://juejin.cn/post/6844903920511221774
>
> ​		https://zhuanlan.zhihu.com/p/113917726



## 索引失效的几种原因

1. 隐式类型转换导致索引失效。

   ```sql
   # 失效
   select * from test where tu_mdn=13333333333;
   
   select * from test where tu_mdn='13333333333';
   ```

2. 对索引列进行运算导致索引失效

   ```sql
   # 失效
   select * from test where id-1=9;
   
   select * from test where id=10;
   ```

3. 使用一下语句会失效

   - 使用  <> 、not in 、not exist、!=
   - like "%_" 百分号在前（可采用在建立索引时用reverse(columnName)这种方法处理）
   - 单独引用复合索引里非第一位置的索引列，应总是使用索引的第一个列，如果索引是建立在多个列上, 只有在它的第一个 列被where子句引用时，优化器才会选择使用该索引。
   - 当变量采用的是times变量，而表的字段采用的是date变量时.或相反情况。

4. 将空的变量值直接与比较运算符（符号）比较

   ​		如果变量可能为空，应使用 IS NULL 或 IS NOT NULL 进行比较，或者使用 ISNULL 函数。

5. 不要在 sql 代码中使用双引号

6. 如果条件中有 or，即使其中有条件带索引也不会使用。

   

## innodb和myisam的区别

> 引用：https://cloud.tencent.com/developer/article/1181374



