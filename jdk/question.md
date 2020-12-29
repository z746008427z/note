## 并发下不安全的集合类如何解决

### List 不安全

#### synchronized

#### Collections.synchronizedList()

```java
List<String> list = Collections.synchronizedList(new ArrayList());
```

#### Vector

#### CopyOnWriteList

copyonwrite 写时复制，cow 计算机领域一种优化策略。

多个线程调用的时候，list，读取的时候，固定的。

写入的时候避免覆盖，造成数据问题。

读写分离

```java
List<String> list - new CopyOnWriteArrayList<>();
```



### Set 不安全

#### Collections.synchronizedSet()

```java
Set<String> set = Collections.synchronizedSet(new HashSet<>());
```

#### CopyOnWriteArraySet

```java
Set<String> set = new CopyOnWriteArraySet<>();
```

set 的本质就是 map 的 key 是无法重复的。



### HashMap 不安全

扩容操作，rehash。

#### ConcurrentHashMap

```java
Map<String, String> map = new ConcurrentHashMap<>();
```

#### HashTable

线程安全，效率低。



































