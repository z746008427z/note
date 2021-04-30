# Mybatis

SqlSessionFactory 和 SqlSession 是 MyBatis 的核心组件。



## SqlSessionFactory

#### 构建过程

- 通过 XMLConfigBuilder 解析配置的 xml 文件，读出配置参数，包括基础配置 xml 文件和映射器 xml 文件。
- 使用 Configuration 对象创建 SqlSessionFactory。SqlSessionFactory 是一个接口，提供了一个默认的实现类 DefaultSqlSessionFactory。

> 即使将我们的所有配置解析为 Configuration 对象，在整个生命周期内，可以通过该对象获取需要的配置。



#### MappedStatement

​		它保存映射器的一个节点（select|insert|delete|update），包括配置的 SQL，SQL 的 id、缓存信息、resultMap、parameterType、resultType 等重要配置内容。



#### SqlSource

​		对于参数和 sql，主要反映在 BoundSql 类对象上，通过它获取到当前运行的 sql 和参数以及参数规则，作出适当修改，满足特殊要求。

##### BoundSql 3个主要属性

- **parameterObject**：参数本身，可以传递简单对象、POJO、Map或@Param注解的参数。

  - 传递简单对象（int、float、String），会把参数转换成对应的类。int 会转换成 Integer。

  - 如果传递的是 POJO 或 Map ，paramterObject 就是传入的 POJO 或 Map 不变。

  - 如果传递多个参数，没有 @Param 注解，parameterObject 就是一个 Map<String,Object> 对象，类似这样的形式 {"1":p1 , "2":p2 , "3":p3 ... "param1":p1 , "param2":p2 , "param3",p3 ...}，所以在编写的时候可以使用 #{param1} 或 #{1} 去引用第一个参数。

  - 如果传递多个参数，有 @Param 注解，与没有注解的类似，只是将序号的 key 替换为 @Param 指定的 name。

    

- **parameterMappings**：它是一个List，元素是 ParameterMapping 对象，这个对象会描绘 sql 中的参数引用，包括名称、表达式、javaType、jdbcType、typeHandler 等信息。

- **sql**：写在映射器里面的一条 SQL。

```java
// 构建 SqlSessionFactory
sqlSessionFactory = new SqlSessionFactoryBuilder().bulid(inputStream);
```





## SqlSession

#### 映射器的动态代理

mapper 映射通过动态代理实现，使用 jdk 动态代理返回代理对象，供调用者访问。



**调用过程**：

返回的 Mapper 对象是代理对象，当调用它的某个方法时，其实是调用 MapperProxy#invoke 方法，映射器的 xml 文件的命名空间（namespace）对应的就是这个接口的全路径，会根据全路径和方法名，能够绑定起来，定位到 sql。最后使用 SqlSession 接口的方法使他能够执行查询。



#### SqlSession 四大对象

​		映射器就是一个动态代理对象，到了 MapperMethod 的 execute() 方法，经过判断进入了 SqlSession 的删除、更新、插入、选择等方法。



##### Mapper 执行过程

​		通过 Executor、StatementHandler、ParameterHandler 和 ResultHandler 来完成数据库操作和返回。编写插件的关键。

- **Executor**：执行器，由它同一调度其他三个对象来执行对应的 SQL。
- **StatementHandler**：使用数据库的 Statement 执行操作。
- **ParameterHandler**：用于 Sql 对参数的处理。
- **ResultHandler**：进行最后数据集的封装返回处理。



##### 三种执行器

- **SIMPLE**：简易执行器，默认的执行器。
- **REUSE**：执行重用预处理语句。
- **BATCH**：执行重用语句和批量更新，针对批量专用的执行器。

























