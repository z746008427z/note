# redis 分布式锁

## 分布式锁的特点

- **互斥性**：同一时刻只能由一个线程持有锁。
- **可重入性**：同一节点上的同一个线程如果获取了锁之后能够再次获取锁。
- **锁超时**：和 J.U.C 中的锁一样支持锁超时，防止死锁。
- **高性能和高可用**：加锁和解锁需要高效，同时也需要保证高可用，防止分布式锁失效。
- **具备阻塞和非阻塞性**：能够及时从阻塞状态中被唤醒。



## Redis 分布式锁实现

#### 利用 setnx + expire 命令（不合适）

Redis 的 `SETNX` 命令，`setnx key value`，将 key 设置为 value。

当键不存在时，才能成功，若键存在，什么也不做，成功返回 1 ，失败返回 0 。

`SETNX` 实际上就是 `SET IF NOT Exists` 的缩写

```java
public boolean tryLock(String key, String requset, int timeout) {
    Long result = jedis.setnx(key, requset);
    // result = 1时，设置成功，否则设置失败
    if (result == 1L) {
        return jedis.expire(key, timeout) == 1L;
    } else {
        return false;
    }
}
```

> 上面的步骤是有问题的，setnx 和 expire 是分开的两步操作，不具有原子性，如果执行完第一条指令应用异常或者重启了，锁将无法过期。



#### 使用 Lua 脚本

```java
public boolean tryLock_with_lua(String key, String UniqueId, int seconds) {
    String lua_scripts = "if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then" +
            "redis.call('expire',KEYS[1],ARGV[2]) return 1 else return 0 end";
    List<String> keys = new ArrayList<>();
    List<String> values = new ArrayList<>();
    keys.add(key);
    values.add(UniqueId);
    values.add(String.valueOf(seconds));
    Object result = jedis.eval(lua_scripts, keys, values);
    //判断是否成功
    return result.equals(1L);
}
```



#### 使用 set key value [EX seconds][PX milliseconds][NX|XX] 命令 (正确做法)

Redis 在 2.6.12 版本开始，为 set 命令增加一系列选项。

```properties
SET key value[EX seconds][PX milliseconds][NX|XX]
```

- EX seconds: 设定过期时间，单位为秒
- PX milliseconds: 设定过期时间，单位为毫秒
- NX: 仅当key不存在时设置值
- XX: 仅当key存在时设置值

**set 命令的 nx 选项，就等同于 setnx 命令。**

```java
public boolean tryLock_with_set(String key, String UniqueId, int seconds) {
    return "OK".equals(jedis.set(key, UniqueId, "NX", "EX", seconds));
}
```



#### 释放锁的实现

释放锁是需要验证 value，解锁时，需要判断锁是否是自己的，基于 value 值来判断。

```java
public boolean releaseLock_with_lua(String key,String value) {
    String luaScript = "if redis.call('get',KEYS[1]) == ARGV[1] then " +
            "return redis.call('del',KEYS[1]) else return 0 end";
    return jedis.eval(luaScript, Collections.singletonList(key), 					Collections.singletonList(value)).equals(1L);
}
```



#### RedLock

引用的就是redisson。

> https://mp.weixin.qq.com/s?__biz=MzU5ODUwNzY1Nw==&mid=2247484155&idx=1&sn=0c73f45f2f641ba0bf4399f57170ac9b&scene=21#wechat_redirect



> 引用：https://juejin.cn/post/6844903830442737671#heading-4