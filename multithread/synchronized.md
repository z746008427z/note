# synchronized



## 锁类型

- **可重入锁（synchronized 和 ReentrantLock）**：在执行对象中所有同步方法不用再次获得锁。
- **可中断锁（synchronized 不是可中断锁，Lock 是可中断锁）**：在等待获取锁过程中可终端。
- **公平锁（ReentrantLock 和 ReentrantReadWriteLock）**：按等待获取锁的线程的等待时间进行获取，等待时间长的具有优先获取锁权利。
- **读写锁（ReadWriteLock 和 ReentrantReadWriteLock）**：对资源读取和写入的时候拆分为 2 部分处理，读的时候可以多线程一起读，写的时候必须同步的写。



## 锁的四种状态/锁升级

**无锁、偏向锁，轻量级锁，重量级锁**，锁状态只能升级，不能降级。

![image-20201224164302903](synchronized.assets/image-20201224164302903.png)

| 锁状态   | 存储内容                                              | 标志位 |
| -------- | ----------------------------------------------------- | ------ |
| 无锁     | 对象的hashCode、对象分代年龄、是否是偏向锁(0)         | 01     |
| 偏向锁   | 偏向线程ID、偏向时间戳、对象分代年龄、是否是偏向锁(1) | 01     |
| 轻量级锁 | 指向栈中锁记录的指针                                  | 00     |
| 重量级锁 | 指向互斥量的指针                                      | 11     |



## synchronized 的使用

可以使用 `synchronized` 及 `Object` 中的配套方法 `wait()/notify() 、 wait()/notifyAll` 来实现线程的通信。

synchronized 保证了线程对变量访问的 **可见性** 和 **排他性**。



## synchronized 等待/通知机制

如果使用 synchronized 来实现线程间的通信，我们需要结合 Object 中的配套方法 `wait()/notify() 、 wait()/notifyAll`。

| 方法名称        |                             描述                             |
| :-------------- | :----------------------------------------------------------: |
| wait()          | 调用该方法的线程进入 `WAITING` 状态，只有等待另外线程的通知或被中断才会返回，需要注意，线程调用 wait() 方法前，需要获得对象的监视器。当调用 wait() 方法后，会释放对象的监视器 |
| wait(long)      | 调用该方法的线程进入 `TIMED_WAITING` 状态，这里的参数时间是毫秒，等待对应毫秒事件，如果没有收到其他线程通知，则超时返回 |
| wait(long, int) | 调用该方法的线程进入 `TIMED_WAITING` 状态，基本作用同 wait(long)，第二个参数代表为纳秒，也就是等待时间为毫秒+纳秒。 |
| notify()        | 通知一个在对象监视器上等待的线程，使其从 wait() 方法返回，而返回的前提是该线程获取到了对象的监视器。 |
| notifyAll()     |   通知所有在监视器上等待的线程，具体唤醒那个线程由CPU决定    |

​		使用 Object 的 wait() / notify()、wait() / notifyall() ，其实是我们经常使用的 `等待/通知机制`，所谓的 等待/通知机制 是指一个线程 A 调用了对象 O 的 wait() 方法进入等待状态，而另一个线程 B 调用了对象 O 的 notify 或者 notifyAll 方法。线程 A 收到通知后从对象 O 的 wait() 方法返回，进而执行后续的操作。

#### 结论

- 使用 wait()、notify() 和 notifyAll 时需要先获取对象的监视器（执行 monitorenter 指令成功）
- 调用 wait() 方法后，线程状态由 **RUNNING** 变为 **WAITING**，并将该线程加入等待队列。
- notify() 或 notifyAll() 方法调用后，等待线程依旧不会从 wait() 返回，需要调用 notify() 或 notfifyAll() 的线程释放对象的监视器（也就是执行 monitorexit 指令）后，等待线程才会有机会从wait()返回。
- notify() 方法将等待队列中的一个等待线程从等待队列移到同步队列中，而 notifyAll() 方法则是将等待队列中所有的线程全部移动到同步队列，被移动的线程状态由 **WAITING** 变为 **BLOCKED** 。
- 从 wait() 方法返回的前提是获得了调用对象的监视器 (执行 monitorenter 指令成功）。



## Lock 等待/通知机制

```java
class LockDemo {

    static boolean flag = true;
    static Lock lock = new ReentrantLock();
    static Condition codition = lock.newCondition();


    public static void main(String[] args) throws InterruptedException {
        new Thread(new WaitRunnable(), "WaitThread").start();
        TimeUnit.SECONDS.sleep(1);//这里睡眠，是保证Wait线程先执行
        new Thread(new NotifyRunnable(), "NotifyThread").start();
    }

    static class WaitRunnable implements Runnable {
        @Override
        public void run() {
            lock.lock();
            try {
                while (flag) {
                    String name = Thread.currentThread().getName();
                    System.out.println(name + "--->wait in " + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                    codition.await();
                    System.out.println(name + "--->wake up in " + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }

        }
    }

    static class NotifyRunnable implements Runnable {
        @Override
        public void run() {
            lock.lock();
            try {
                String name = Thread.currentThread().getName();
                System.out.println(name + "--->notify all in " + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                flag = false;
                codition.signalAll();
            } finally {
                lock.unlock();
            }
        }
    }
}
```



## 等待/通知经典范式

#### 等待方

等待方应遵循如下原则：

1. 获取对象的锁。
2. 如果条件不满足，调用对象的 wait() 方法， 被通知后仍然要检查条件。
3. 条件满足则执行响应逻辑。

#### 通知方

通知方应遵循如下原则：

1. 获得对象的锁。
2. 改变条件。
3. 通知所有等待在对象上的线程。



## synchronized 与 Lock 区别

| 类别     | synchronized                                                 | Lock                                                         |
| :------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存在层次 | Java 的关键字，在 jvm 层面上                                 | 是一个接口                                                   |
| 锁的释放 | 1.已获取锁的线程执行完同步代码，释放锁。<br />2.线程执行发生异常，jvm 回让线程释放锁。 | 在 finally 种必须释放锁，不然容易造成线程死锁。              |
| 锁的获取 | 假设 A 线程获得锁，B 线程等待。<br />如果 A 线程阻塞，B 线程会一致等待。 | 分情况而定，Lock 有多个锁获取的方式，大致就是可以尝试获得锁，线程可以不用一直等待（可以通过 tryLock 判断有没有锁） |
| 锁状态   | 无法判断                                                     | 可以判断                                                     |
| 锁类型   | 可重入、不可中断、非公平                                     | 可重入 可判断 可公平（两者皆可）                             |
| 性能     | 少量同步                                                     | 大量同步<br />1.Lock 可以提高多个线程进行读操作的效率。（可以通过 readwritelock实现读写分离）<br />2.在资源竞争不是很激烈的情况下，Synchronized的性能要优于ReetrantLock，但是在资源竞争很激烈的情况下，Synchronized的性能会下降几十倍，但是ReetrantLock的性能能维持常态<br />3.ReentrantLock提供了多样化的同步，比如有时间限制的同步，可以被Interrupt的同步（synchronized的同步是不能Interrupt的）等。在资源竞争不激烈的情形下，性能稍微比synchronized差点点。但是当同步非常激烈的时候，synchronized的性能一下子能下降好几十倍。而ReentrantLock确还能维持常态。 |



## synchronized 与 static synchronized

访问 **static synchronized 方法占用的是类锁**，访问**非 static synchronized 方法占用的是对象锁**。

static synchronized 控制类的所有实例（对象）的访问（相应代码块）。

synchronized 相当于 this.synchronized。static synchronized 相当于 something.synchronized。



## synchronized 原理

> https://xiaomi-info.github.io/2020/03/24/synchronized/



## synchronized 与 lock 用法区别

- **synchronized**：在需要同步的对象中加入此控制，synchronized可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。
- **lock**：需要显示指定起始位置和终止位置。一般使用 ReentrantLock 类做为锁，多个线程中必须要使用一个 ReentrantLock 类做为对象才能保证锁的生效。且在加锁和解锁处需要通过 lock() 和 unlock() 显示指出。所以一般会在 finally 块中写 unlock() 以防死锁。

 

## synchronized 与 lock 性能区别

​		synchronized 是托管给 jvm 执行的，而 lock 是 java 写的控制锁的代码。在Java1.5中，synchronized 是性能低效的。因为这是一个重量级操作，需要调用操作接口，导致有可能加锁消耗的系统时间比加锁以外的操作还多。相比之下使用 Java 提供的 Lock 对象，性能更高一些。但是到了Java1.6，发生了变化。synchronized 在语义上很清晰，可以进行很多优化，有适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。导致在Java1.6上 synchronized 的性能并不比 Lock 差。官方也表示，他们也更支持 synchronized，在未来的版本中还有优化余地。

​		synchronized 原始采用的是 CPU **悲观锁** 机制，即线程获得的是独占锁。独占锁意味着其他线程只能依靠阻塞来等待线程释放锁。而在 CPU 转换线程阻塞时会引起线程上下文切换，当有很多线程竞争锁的时候，会引起 CPU 频繁的上下文切换导致效率很低。

​		而 Lock 用的是 **乐观锁** 方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁实现的机制就是 CAS 操作（Compare and Swap）。我们可以进一步研究 ReentrantLock 的源代码，会发现其中比较重要的获得锁的一个方法是 compareAndSetState。这里其实就是调用的 CPU 提供的特殊指令。

​		现代的CPU提供了指令，可以自动更新共享数据，而且能够检测到其他线程的干扰，而 compareAndSet() 就用这些代替了锁定。这个算法称作非阻塞算法，意思是一个线程的失败或者挂起不应该影响其他线程的失败或挂起的算法。



## Lock类型

#### 公平锁/非公平锁

- 公平锁是指多个线程按照申请锁的顺序来获取锁。
- 非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。
- 对于 ReentrantLock 而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。
- 对于 Synchronized 而言，也是一种非公平锁。由于其并不像 ReentrantLock 是通过 AQS 的来实现线程调度，所以并没有任何办法使其变成公平锁。



#### 可重入锁

- 可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。
- 对于 Java ReentrantLock 而言，他的名字就可以看出是一个可重入锁，其名字是 Reentrant Lock 重新进入锁。
- 对于 Synchronized 而言，也是一个可重入锁。可重入锁的一个好处是可一定程度避免死锁。



#### 独享锁/共享锁

- 独享锁是指该锁一次只能被一个线程所持有。
- 共享锁是指该锁可被多个线程所持有。
- 对于 Java ReentrantLock 而言，其是独享锁。但是对于 Lock 的另一个实现类 ReadWriteLock，其读锁是共享锁，其写锁是独享锁。
- 读锁的共享锁可保证并发读是非常高效的，读写，写读 ，写写的过程是互斥的。
- 独享锁与共享锁也是通过 AQS 来实现的，通过实现不同的方法，来实现独享或者共享。
- 对于Synchronized而言，当然是独享锁。



#### 互斥锁/读写锁

- 上面讲的独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。
- 互斥锁在Java中的具体实现就是 ReentrantLock
- 读写锁在Java中的具体实现就是 ReadWriteLock



#### 乐观锁/悲观锁

- 乐观锁与悲观锁不是指具体的什么类型的锁，而是指看待并发同步的角度。
- 悲观锁认为对于同一个数据的并发操作，一定是会发生修改的，哪怕没有修改，也会认为修改。因此对于同一个数据的并发操作，悲观锁采取加锁的形式。悲观的认为，不加锁的并发操作一定会出问题。
- 乐观锁则认为对于同一个数据的并发操作，是不会发生修改的。在更新数据的时候，会采用尝试更新，不断重新的方式更新数据。乐观的认为，不加锁的并发操作是没有事情的。
- 从上面的描述我们可以看出，悲观锁适合写操作非常多的场景，乐观锁适合读操作非常多的场景，不加锁会带来大量的性能提升。
- 悲观锁在Java中的使用，就是利用各种锁。
- 乐观锁在Java中的使用，是无锁编程，常常采用的是CAS算法，典型的例子就是原子类，通过CAS自旋实现原子操作的更新。



#### 分段锁

- 分段锁其实是一种锁的设计，并不是具体的一种锁，对于 ConcurrentHashMap 而言，其并发的实现就是通过分段锁的形式来实现高效的并发操作。
- 我们以 ConcurrentHashMap 来说一下分段锁的含义以及设计思想，ConcurrentHashMap 中的分段锁称为 Segment，它即类似于HashMap(JDK7与JDK8中HashMap的实现) 的结构，即内部拥有一个 Entry 数组，数组中的每个元素又是一个链表；同时又是一个ReentrantLock(Segment继承了ReentrantLock)。
- 当需要 put 元素的时候，并不是对整个 hashmap 进行加锁，而是先通过 hashcode 来知道他要放在那一个分段中，然后对这个分段进行加锁，所以当多线程 put 的时候，只要不是放在一个分段中，就实现了真正的并行的插入。
- 但是，在统计 size 的时候，可就是获取 hashmap 全局信息的时候，就需要获取所有的分段锁才能统计。
- 分段锁的设计目的是细化锁的粒度，当操作不需要更新整个数组的时候，就仅仅针对数组中的一项进行加锁操作。



#### 偏向锁/轻量级锁/重量级锁

- 这三种锁是指锁的状态，并且是针对 Synchronized。在 java 5 通过引入锁升级的机制来实现高效 synchronized。这三种锁的状态是通过对象监视器在对象头中的字段来表明的。
- 偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁。降低获取锁的代价。
- 轻量级锁是指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。
- 重量级锁是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让其他申请的线程进入阻塞，性能降低。



#### 自旋锁

- 在 java 中，自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗 CPU。
- 我们知道，java 线程其实是映射在内核之上的，线程的挂起和恢复会极大的影响开销. 并且 jdk 官方人员发现，很多线程在等待锁的时候，在很短的一段时间就获得了锁，所以它们在线程等待的时候，并不需要把线程挂起，而是让他无目的的循环，一般设置10次。 这样就避免了线程切换的开销，极大的提升了性能。
- 而适应性自旋，是赋予了自旋一种学习能力，它并不固定自旋10次一下。他可以根据它前面线程的自旋情况，从而调整它的自旋，甚至是不经过自旋而直接挂起。



## synchronized 与 ReentrantLock 区别

都是**加锁方式同步**，都是**阻塞式的同步**。

#### API 层面

synchronized 是 java 语言的关键子，是原生语法层面的互斥。需要 jvm 实现。

ReentrantLock 是 jdk1.5 之后提供的 API 层面的互斥锁，需要 lock 和 unlock 配合 try/finally 来完成。

#### 等待可中断

- synchronized。如果 Thread1 不释放，Thread2 将一直等待，不能被中断。synchronized 也可以说是 Java 提供的原子性内置锁机制。内部锁扮演了互斥锁（mutual exclusion lock ，mutex）的角色，一个线程引用锁的时候，别的线程阻塞等待。

- ReentrantLock。如果 Thread1 不释放，Thread2 等待了很长时间以后，可以中断等待，转而去做别的事情。

#### 公平锁

- synchronized 的锁是非公平锁。
- ReentrantLock 默认情况下也是非公平锁，但可以通过带布尔值的构造函数要求使用公平锁。

#### 锁绑定多个条件

- synchronized 中，锁对象的 wait() 和 notify() 或 notifyAll() 方法可以实现一个隐含的条件。但如果要和多于一个的条件关联的时候，就不得不额外添加一个锁。
- ReentrantLock 可以同时绑定多个 Condition 对象，只需多次调用 newCondition 方法即可。



> 引用：https://juejin.cn/post/6844903695298068487
>





















