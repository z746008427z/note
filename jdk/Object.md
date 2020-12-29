# Object



## getClass

```java
public final native Class<?> getClass();
```

final，获取对象的运行时 class 对象，class 对象就是描述对象所属的对象。

通常和 Java 反射机制搭配使用的。



## hashCode

```java
public native int hashCode();
```

该方法主要用于获取对象的散列值。Object 中该方法默认返回的是对象的堆内存地址。



## equals

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

方法用于比较两个对象，如果两个对象引用指向是同一个对象，那么返回 true。否则返回 false。

一般 equals 和 == 不一样，但是在 object 中两者是一样的，子类一般重写这个方法。



## clone

```java
protected native Object clone() throws CloneNotSupportedException;
```

该方法是保护方法，实现对象的浅复制，只有来实现了 Cloneable 接口才可以调用该方法。否则抛异常。

默认的 clone 是浅拷贝，

- **浅拷贝**：指对象内属性引用的对象只会拷贝引用地址，而不会将引用的对象重新分配内存。
- **深拷贝**：会连引用的对象也重新创建。



## toString

```java
public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

返回一个 String 对象，一般子类都有覆盖。

默认返回格式：对象的 class 名称 + @ + hashCode  的十六进制字符串。



## notify

```java
public final native void notify();
```

final 方法，只要用于唤醒在该对象上等待的某个线程。



## notifyAll

```java
public final native void notifyAll();
```

final 方法，主要用于唤醒在该对象上等待的所有线程。



## wait(long timeout)

```java
public final native void wait(long timeout) throws InterruptedException;
```

wait 方法就是使当前线程等待该对象的锁，当前线程必须是具有该对象的锁。

wait() 方法一直等待，直到获得锁或者被中断。

wait(long timeout) 设定一个超时间隔，如果在规定时间内没有获得锁就返回。



## wait(long timeout, int nanos)

```java
public final void wait(long timeout, int nanos) throws InterruptedException {
  if (timeout < 0) {
    throw new IllegalArgumentException("timeout value is negative");
  }

  if (nanos < 0 || nanos > 999999) {
    throw new IllegalArgumentException(
              "nanosecond timeout value out of range");
  }

  if (nanos >= 500000 || (nanos != 0 && timeout == 0)) {
    timeout++;
  }

  wait(timeout);
}
```

参数说明：

timeout：最大等待时间（毫秒）

nanos：附加时间在毫秒范围（0-999999）

该方法导致当前线程等待，知道其他线程调用此对象的 notify() 或者 notifyAll()，或在指定已经过去的时间。该方法类似于 wait 方法的一个参数，允许更好的控制的时间等待一个通知放弃之前的量。

> 实时量：以毫微秒计算：公式：1000000 * timeout + nanos

wait(0,0) 表示和 wait(0) 相同。



## wait()

```java
public final void wait() throws InterruptedException {
    wait(0);
}
```

可以看到 wait() 方法实际上调用的是 wait(long timeout) 方法，只不过 timeout 为 0，即不等待。



## finalize

```java
protected void finalize() throws Throwable { }
```

保护方法，用于在 GC 时再次被调用。

如果实现了这个方法，对象可能在这个方法中再次复活，从而避免被 GC 回收。