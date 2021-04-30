# Java集合-Map

Map是一种(key/value)的映射结构,Map中的元素是一个key只能对应一个value，不能存在重复的key。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/20201215091803.png)

java提供的map实现主要有HashMap,TreeMap,LinkedHashMap,WeakHashMap,ConcurrentHashMap,ConcurrentSkipHashMap,还有就是线程安全的 HashTable及它的实现Properties

关于Map的问题：

1.什么是散列表？

2.怎么实现散列表？

3.java中的HashMap实现方式的演进？

4.HashMapd的容量有什么特点？

5.HashMap怎么进行扩容？

6.HashMap中的元素是否有序？

7.HashMap何时进行树化？何时进行反树化？

8.HashMap 是怎么进行缩容的？

9.HashMap 插入、删除、查询元素的时间复杂度各是多少？

10.HashMap 中红黑树实现部分可以用其他数据结构代替么？

11.LinkedHashMap是怎么实现的？

12.LinkedHashMap是有序的么？ 如果有，怎么排序的？

13.LinkedHashMap 如何实现LRU缓存淘汰策略

​    缓存淘汰算法--LRU：https://zhuanlan.zhihu.com/p/34989978

14.WeakHashMap使用的数据结构？

15.WeakHashMap具有什么特性？

16.WeakHashMap通常用在什么场景？

17.WeakHashMap使用String作为key需要注意什么？为什么要注意？

18.什么是弱引用？

19.红黑树具有哪些特性？

20.TreeMap是有序的么？有序的特征是什么？

21.TreeMap是否需要扩容？

22.什么是左旋？什么是右旋？

23.红黑树怎么插入元素？

24.红黑树怎么删除元素？

25.红黑树为什么要进行平衡？

26.如何实现红黑树的遍历？

27.TreeMap是怎么遍历的？

28.TreeMap插入、删除、查询元素的时间复杂度各是多少？

29.HashMap在多线程的环境中什么时候会出现问题？

30.ConcurrentHashMap 的存储结构？

31.ConcurrentHashMap是什么保证并发安全的？

32.ConcurrentHashMap 怎么扩容？

33.ConcurrentHashMap 的size()方法的实现？

34.ConcurrentHashMap 是强一致性的么？

35.ConcurrentHashMap 不能解决什么问题？

36.ConcurrentHashMap中哪些地方运用到了分段锁的思想？

37.什么是伪共享？怎么避免伪共享？

38.什么是跳表？

40.ConcurrentSkipList是有序的吗？

41.ConcurrentSkipList是如何保证线程安全的？

42.ConcurrentSkipList 插入、删除、查询元素的时间复杂度各是多少？

43.ConcurrentSkipList 的索引具有什么特性？

44.为什么redis选择使用跳表而非红黑树来实现有序集合？

## 1.HashMap

HashMap采用key/value存储结构，每个key对应唯一的value，查询和修改的速度都很快，能达到O(1)的平均时间复杂度。它是非线程安全的，且不保证元素存储的顺序；

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/20201215095439.png)

HashMap 实现了Cloneable接口，可以被克隆。

HashMap 实现了Serializable接口，可以被序列化和反序列化。

HashMap继承自AbstractMap，实现了Map接口，具有Map的所有功能。

### 1.1 存储结构

![image-20201215100806949](https://gitee.com/zhouxiaoliang/img/raw/master/img/20201215164215.png)

java中HashMap采用了（数组+链表+红黑树）的复杂数据结构，数组的一个元素又被称作桶

在添加元素时，会根据hash值算出元素在数组中的位置，如果该位置没有元素，则直接把元素放置在此处，如果该位置有元素了，则把元素以链表的形式放置在链表的尾部。

当一个链表的元素个数达到一定的数量（且数组的长度达到一定的长度）后，则把链表转化为红黑树，从而提高效率。

数组的查询效率为O(1)，链表的查询效率是O(k)，红黑树的查询效率是O(log k)，k为桶中的元素个数，所以当元素数量非常多的时候，转化为红黑树能极大地提高效率。

### 1.2 属性

```
/**
 * 默认的初始容量为16
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

/**
 * 最大的容量为2的30次方
 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * 默认的装载因子
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * 当一个桶中的元素个数大于等于8时进行树化
 */
static final int TREEIFY_THRESHOLD = 8;

/**
 * 当一个桶中的元素个数小于等于6时把树转化为链表
 */
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * 当桶的个数达到64的时候才进行树化
 */
static final int MIN_TREEIFY_CAPACITY = 64;

/**
 * 数组，又叫作桶（bucket）
 */
transient Node<K,V>[] table;

/**
 * 作为entrySet()的缓存
 */
transient Set<Map.Entry<K,V>> entrySet;

/**
 * 元素的数量
 */
transient int size;

/**
 * 修改次数，用于在迭代的时候执行快速失败策略
 */
transient int modCount;

/**
 * 当桶的使用数量达到多少时进行扩容，threshold = capacity * loadFactor
 */
int threshold;

/**
 * 装载因子
 */
final float loadFactor;
```

（1）容量

容量为数组的长度，亦即桶的个数，默认为16，最大为2的30次方，当容量达到64时才可以树化。

（2）装载因子

装载因子用来计算容量达到多少时才进行扩容，默认装载因子为0.75。

（3）树化

树化，当容量达到64且链表的长度达到8时进行树化，当链表的长度小于6时反树化。

### 1.3 内部类

Node

```
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```

TreeNode

它继承自LinkedHashMap中的Entry类，关于LInkedHashMap.Entry这个类下面会讲。

TreeNode是一个典型的树型节点，其中，prev是链表中的节点，用于在删除元素的时候可以快速找到它的前置节点。

```
// 位于HashMap中
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
}
```

```
// 位于LinkedHashMap中，典型的双向链表节点
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

### 1.4 构造方法

空参构造方法，全部使用默认值。

```
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

容量参数构造方法，

```
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

调用HashMap(int initialCapacity, float loadFactor)构造方法，传入默认装载因子。

判断传入的初始容量和装载因子是否合法，并计算扩容门槛，扩容门槛为传入的初始容量往上取最近的2的n次方。

```
public HashMap(int initialCapacity, float loadFactor) {
    // 检查传入的初始容量是否合法
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // 检查装载因子是否合法
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    // 计算扩容门槛
    this.threshold = tableSizeFor(initialCapacity);
}
```

计算扩容门槛

```
static final int tableSizeFor(int cap) {
    // 扩容门槛为传入的初始容量往上取最近的2的n次方
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

以容量16为例： int 4字节   4*8 = 32位    （**无符号右移>>>(不论正负,高位均补0)  注意：无符号，所以都是当正数操作的**，**有符号右移>>（若正数,高位补0,负数,高位补1）**）

![image-20201215210036979](https://gitee.com/zhouxiaoliang/img/raw/master/img/image-20201215210036979.png)



初始化  容量扩展门槛为 16  与容量相等。

16 最近的2的n次方是 2^4  =16



如果传入的初始容量为65：

![image-20201215210133227](https://gitee.com/zhouxiaoliang/img/raw/master/img/image-20201215210133227.png)



65 初始化  容量扩展门槛为 128

65 最近的2的n次方是 2^6=64，   65 往上取最近的2的n次方是2^7=128.

**为什么这么操作？**

容量不为0 ，那么二进制数肯定存在1，先来分析有关n位操作部分：先来假设n的二进制为01xxx...xxx。接着

n|n>>>1    对n右移1位：001xx...xxx，再位或：011xx...xxx

n|n>>>2    对n右移2为：00011...xxx，再位或：01111...xxx

此时前面已经有四个1了，再右移4位且位或可得8个1，同理，有8个1，右移8位肯定会让后八位也为1。

综上可得，该算法让最高位的1后面的位全变为1。最后再让结果n+1，即得到了2的整数次幂的值了

### 1.5 put方法

添加元素的入口。

```
public V put(K key, V value) {
    // 调用hash(key)计算出key的hash值
    return putVal(hash(key), key, value, false, true);
}
```

```
static final int hash(Object key) {
    int h;
    // 如果key为null，则hash值为0，否则调用key的hashCode()方法
    // 并让高16位与整个hash异或，这样做是为了使计算出的hash更分散
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K, V>[] tab;
    Node<K, V> p;
    int n, i;
    // 如果桶的数量为0，则初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        // 调用resize()初始化
        n = (tab = resize()).length;
    // (n - 1) & hash 计算元素在哪个桶中
    // 如果这个桶中还没有元素，则把这个元素放在桶中的第一个位置
    if ((p = tab[i = (n - 1) & hash]) == null)
        // 新建一个节点放在桶中
        tab[i] = newNode(hash, key, value, null);
    else {
        // 如果桶中已经有元素存在了
        Node<K, V> e;
        K k;
        // 如果桶中第一个元素的key与待插入元素的key相同，保存到e中用于后续修改value值
        if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            // 如果第一个元素是树节点，则调用树节点的putTreeVal插入元素
            e = ((TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);
        else {
            // 遍历这个桶对应的链表，binCount用于存储链表中元素的个数
            for (int binCount = 0; ; ++binCount) {
                // 如果链表遍历完了都没有找到相同key的元素，说明该key对应的元素不存在，则在链表最后插入一个新节点
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 如果插入新节点后链表长度大于8，则判断是否需要树化，因为第一个元素没有加到binCount中，所以这里-1
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 如果待插入的key在链表中找到了，则退出循环
                if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 如果找到了对应key的元素
        if (e != null) { // existing mapping for key
            // 记录下旧值
            V oldValue = e.value;
            // 判断是否需要替换旧值
            if (!onlyIfAbsent || oldValue == null)
                // 替换旧值为新值
                e.value = value;
            // 在节点被访问后做点什么事，在LinkedHashMap中用到
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 到这里了说明没有找到元素
    // 修改次数加1
    ++modCount;
    // 元素数量加1，判断是否需要扩容
    if (++size > threshold)
        // 扩容
        resize();
    // 在节点插入后做点什么事，在LinkedHashMap中用到
    afterNodeInsertion(evict);
    // 没找到元素返回null
    return null;
}
```

（1）计算key的hash值；

（2）如果桶（数组）数量为0，则初始化桶；

（3）如果key所在的桶没有元素，则直接插入；

​           <!--“(n - 1) & hash 计算元素在哪个桶中”,这个【 (n - 1) & hash】该如何理解呢？-->

**（length-1）&hash    ==   hash%(length-1)的前提是 length是2的 n 次方；**

**HashMap 的长度为什么是2的幂次方?**

为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。Hash 值的范围值-2147483648到2147483647  <!--[-2^31~(2^31-1)]-->

前后加起来大概40亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个40亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是“ **(n - 1) & hash**”。（n代表数组长度）。这也就解释了 HashMap 的长度为什么是2的幂次方。

**取余(%)操作中如果除数是2的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是2的 n 次方；）。**” 并且 **采用二进制位操作 &，相对于%能够提高运算效率**，这就解释了 HashMap 的长度为什么是2的幂次方。

举例说明:
若 a = 10 , b = 8 , 10与8取余应得2.
8的二进制为: 1000 ; 7的二进制为: 0111.
也就是说-----**2的n次幂减一**这样的数的二进制都是如0000111111这样**前半部分是0后半部分是1**的形式.
所以, 用**2的n次幂减一**这样的数 **&** 另一个数就相当于 这个数取余 (%) **2的n次幂**

（4）如果key所在的桶中的第一个元素的key与待插入的key相同，说明找到了元素，转后续流程（9）处理；

 （5）如果第一个元素是树节点，则调用树节点的putTreeVal()寻找元素或插入树节点；

（6）如果不是以上三种情况，则遍历桶对应的链表查找key是否存在于链表中；

（7）如果找到了对应key的元素，则转后续流程（9）处理；

（8）如果没找到对应key的元素，则在链表最后插入一个新节点并判断是否需要树化；

（9）如果找到了对应key的元素，则判断是否需要替换旧值，并直接返回旧值；

（10）如果插入了元素，则数量加1并判断是否需要扩容；

### 1.6 resize方法

```
final Node<K, V>[] resize() {
    // 旧数组
    Node<K, V>[] oldTab = table;
    // 旧容量
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 旧扩容门槛
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 如果旧容量达到了最大容量，则不再进行扩容，此时oldCap * 2比Integer.MAX_VALUE大，因此无法进行重新分布，只是单纯的将阈值扩容到最大
            threshold = Integer.MAX_VALUE;
            return oldTab;
        } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 如果旧容量的两倍小于最大容量并且旧容量大于默认初始容量（16），则容量扩大为两部，扩容门槛也扩大为两倍
            newThr = oldThr << 1; // double threshold
    } else if (oldThr > 0) // initial capacity was placed in threshold
        // 使用非默认构造方法创建的map，第一次插入元素会走到这里
        // 如果旧表的容量为0, 旧表的阈值大于0, 是因为初始容量被放入阈值，则将新表的容量设置为旧表的阈值
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        // 调用默认构造方法创建的map，第一次插入元素会走到这里
        // 如果旧容量旧扩容门槛都是0，说明还未初始化过，则初始化容量为默认容量，扩容门槛为默认容量*默认装载因子
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        // 如果新扩容门槛为0，则计算为容量*装载因子，但不能超过最大容量
        float ft = (float) newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ?
                (int) ft : Integer.MAX_VALUE);
    }
    // 赋值扩容门槛为新门槛
    threshold = newThr;
    // 新建一个新容量的数组
    @SuppressWarnings({"rawtypes", "unchecked"})
    Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap];
    // 把桶赋值为新数组
    table = newTab;
    // 如果旧数组不为空，则搬移元素
    if (oldTab != null) {
        // 遍历旧数组
        for (int j = 0; j < oldCap; ++j) {
            Node<K, V> e;
             // 如果索引值为j的旧表头节点不为空，则e指向该头节点
            if ((e = oldTab[j]) != null) {
                // 清空旧桶，便于GC回收  
                oldTab[j] = null;
                // 如果这个桶中只有一个元素，则计算它在新桶中的位置并把它搬移到新桶中
                // 因为每次都扩容两倍，所以这里的第一个元素搬移到新桶的时候新桶肯定还没有元素
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    // 如果第一个元素是树节点，则把这颗树打散成两颗树插入到新桶中去
                    ((TreeNode<K, V>) e).split(this, newTab, j, oldCap);
                else { // preserve order
                    // 如果这个链表不止一个元素且不是一颗树
                    // 则分化成两个链表插入到新的桶中去
                    Node<K, V> loHead = null, loTail = null;// 存储“原索引位置”的节点
                    Node<K, V> hiHead = null, hiTail = null; // 存储“原索引位置+oldCap”的节点
                    Node<K, V> next;
                    do {
                        next = e.next;
                        // (e.hash & oldCap) == 0的元素放在低位链表中
                        // 比如，3 & 4 == 0
                        //参考 文档：HashMap 详解.md
//链接：http://note.youdao.com/noteshare?id=77cd412052655e5b709dbabb27fc84af&sub=E9820C6ABC7246018CAEC8774CBD6740
//4.3扩容机制
//oldCap 容量2的n次方，二进制32位只有一位是1，所以与e.hash,&操作只会有0 或 1两个结果。  
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)//链表尾部为空
                                loHead = e;//将节点设置为头
                            else
                                loTail.next = e;//如果链表末尾存在节点，则将新节点添加到末尾。
                            loTail = e;//并将loTail赋值为新增的节点
                        } else {
                            // (e.hash & oldCap) != 0的元素放在高位链表中
                            // 如果e的hash值与旧表的容量进行与运算为1,则扩容后的索引位置为:旧表的索引位置＋oldCap
                            if (hiTail == null)//如果hiTail为空, 代表该节点为第一个节点
                                hiHead = e;//则将hiHead赋值为第一个节点
                            else
                                hiTail.next = e;//否则将节点添加在hiTail后面
                            hiTail = e; // 并将hiTail赋值为新增的节点
                        }
                    } while ((e = next) != null);
                    // 遍历完成分化成两个链表了
                    // 如果loTail不为空（说明旧表的数据有分布到新表上“原索引位置”的节点），则将最后一个节点
              // 的next设为空，并将新表上索引位置为“原索引位置”的节点设置为对应的头节点
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                     // 如果hiTail不为空（说明旧表的数据有分布到新表上“原索引+oldCap位置”的节点），则将最后
                 // 一个节点的next设为空，并将新表上索引位置为“原索引+oldCap”的节点设置为对应的头节点
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

具体流程图请参考：https://www.processon.com/view/link/5fdab6157d9c0806f73ba01f

### 1.7 TreeNode.putTreeVal方法

插入元素到红黑树中的方法。

```
final TreeNode<K, V> putTreeVal(HashMap<K, V> map, Node<K, V>[] tab,
                                int h, K k, V v) {
    Class<?> kc = null;
    // 标记是否找到这个key的节点
    boolean searched = false;
    // 找到树的根节点
    TreeNode<K, V> root = (parent != null) ? root() : this;
    // 从树的根节点开始遍历
    for (TreeNode<K, V> p = root; ; ) {
        // dir=direction，标记是在左边还是右边
        // ph=p.hash，当前节点的hash值
        int dir, ph;
        // pk=p.key，当前节点的key值
        K pk;
        if ((ph = p.hash) > h) {
            // 当前hash比目标hash大，说明在左边
            dir = -1;
        }
        else if (ph < h)
            // 当前hash比目标hash小，说明在右边
            dir = 1;
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            // 两者hash相同且key相等，说明找到了节点，直接返回该节点
            // 回到putVal()中判断是否需要修改其value值
            return p;
        else if ((kc == null &&
                // 如果k是Comparable的子类则返回其真实的类，否则返回null
                (kc = comparableClassFor(k)) == null) ||
                // 如果k和pk不是同样的类型则返回0，否则返回两者比较的结果
                (dir = compareComparables(kc, k, pk)) == 0) {
            // 这个条件表示两者hash相同但是其中一个不是Comparable类型或者两者类型不同
            // 比如key是Object类型，这时可以传String也可以传Integer，两者hash值可能相同
            // 在红黑树中把同样hash值的元素存储在同一颗子树，这里相当于找到了这颗子树的顶点
            // 从这个顶点分别遍历其左右子树去寻找有没有跟待插入的key相同的元素
            //（如果当前节点和要添加的节点哈希值相等，但是两个节点的键不是一个类，只好去挨个对比左右孩子 ）
            if (!searched) {
                TreeNode<K, V> q, ch;
                searched = true;
                // 遍历左右子树找到了直接返回
                if (((ch = p.left) != null &&
                        (q = ch.find(h, k, kc)) != null) ||
                        ((ch = p.right) != null &&
                                (q = ch.find(h, k, kc)) != null))
                    return q;
            }
            // 如果两者类型相同，再根据它们的内存地址计算hash值进行比较
            dir = tieBreakOrder(k, pk);
        }

        TreeNode<K, V> xp = p;
          //这里有个判断，如果当前节点还没有左孩子或者右孩子时才能插入，否则就进入下一轮循环 
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            // 如果最后确实没找到对应key的元素，则新建一个节点
            Node<K, V> xpn = xp.next;
            TreeNode<K, V> x = map.newTreeNode(h, k, v, xpn);
            if (dir <= 0)
                xp.left = x;
            else
                xp.right = x;
            xp.next = x;
            x.parent = x.prev = xp;
            if (xpn != null)
                ((TreeNode<K, V>) xpn).prev = x;
            // 插入树节点后平衡
            // 把root节点移动到链表的第一个节点
            moveRootToFront(tab, balanceInsertion(root, x));
            return null;
        }
    }
}
```

通过上面的代码可以知道，HashMap 中往红黑树中添加一个新节点 n 时，有以下操作：

- 从根节点开始遍历当前红黑树中的元素 p，对比 n 和 p 的哈希值；
- 如果哈希值相等并且键也相等，就判断为已经有这个元素（这里不清楚为什么不对比值）；
- 如果哈希值就通过其他信息，比如引用地址来给个大概比较结果，这里可以看到红黑树的比较并不是很准确，注释里也说了，只是保证个相对平衡即可；
- 最后得到哈希值比较结果后，如果当前节点 p 还没有左孩子或者右孩子时才能插入，否则就进入下一轮循环;
- 插入元素后还需要进行红黑树例行的平衡调整，还有确保根节点的领先地位。

### 1.8 treeifyBin

如果插入元素后链表的长度大于等于8则判断是否需要树化。

```
final void treeifyBin(Node<K, V>[] tab, int hash) {
    int n, index;
    Node<K, V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        // 如果桶数量小于64，直接扩容而不用树化
        // 因为扩容之后，链表会分化成两个链表，达到减少元素的作用
        // 当然也不一定，比如容量为4，里面存的全是除以4余数等于3的元素
        // 这样即使扩容也无法减少链表的长度
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K, V> hd = null, tl = null;
        // 把所有节点换成树节点
        do {
            TreeNode<K, V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        // 如果进入过上面的循环，则从头节点开始树化
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```



真正树化的方法。

```
final void treeify(Node<K, V>[] tab) {
    TreeNode<K, V> root = null;
    for (TreeNode<K, V> x = this, next; x != null; x = next) {
        next = (TreeNode<K, V>) x.next;
        x.left = x.right = null;
        // 第一个元素作为根节点且为黑节点，其它元素依次插入到树中再做平衡
        if (root == null) {
            x.parent = null;
            x.red = false;
            root = x;
        } else {
            K k = x.key;
            int h = x.hash;
            Class<?> kc = null;
            // 从根节点查找元素插入的位置
            for (TreeNode<K, V> p = root; ; ) {
                int dir, ph;
                K pk = p.key;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((kc == null &&
                        (kc = comparableClassFor(k)) == null) ||
                        (dir = compareComparables(kc, k, pk)) == 0)
                    dir = tieBreakOrder(k, pk);

                // 如果最后没找到元素，则插入
                TreeNode<K, V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    x.parent = xp;
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    // 插入后平衡，默认插入的是红节点，在balanceInsertion()方法里
                    root = balanceInsertion(root, x);
                    break;
                }
            }
        }
    }
    // 把根节点移动到链表的头节点，因为经过平衡之后原来的第一个元素不一定是根节点了
    moveRootToFront(tab, root);
```

（1）从链表的第一个元素开始遍历；

（2）将第一个元素作为根节点；

（3）其它元素依次插入到红get(Object key)黑树中，再做平衡；

（4）将根节点移到链表第一元素的位置（因为平衡的时候根节点会改变）；

### 1.9 get(Object key)

```
public V get(Object key) {
    Node<K, V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K, V> getNode(int hash, Object key) {
    Node<K, V>[] tab;
    Node<K, V> first, e;
    int n;
    K k;
    // 如果桶的数量大于0并且待查找的key所在的桶的第一个元素不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
        // 检查第一个元素是不是要查的元素，如果是直接返回
        if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // 如果第一个元素是树节点，则按树的方式查找
            if (first instanceof TreeNode)
                return ((TreeNode<K, V>) first).getTreeNode(hash, key);

            // 否则就遍历整个链表查找该元素
            do {
                if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

（1）计算key的hash值；

（2）找到key所在的桶及其第一个元素；

（3）如果第一个元素的key等于待查找的key，直接返回；

（4）如果第一个元素是树节点就按树的方式来查找，否则按链表方式查找；

TreeNode.getTreeNode(int h, Object k)方法

```
final TreeNode<K, V> getTreeNode(int h, Object k) {
    // 从树的根节点开始查找
    return ((parent != null) ? root() : this).find(h, k, null);
}

final TreeNode<K, V> find(int h, Object k, Class<?> kc) {
    TreeNode<K, V> p = this;
    do {
        int ph, dir;
        K pk;
        TreeNode<K, V> pl = p.left, pr = p.right, q;
        if ((ph = p.hash) > h)
            // 左子树
            p = pl;
        else if (ph < h)
            // 右子树
            p = pr;
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            // 找到了直接返回
            return p;
        else if (pl == null)
            // hash相同但key不同，左子树为空查右子树
            p = pr;
        else if (pr == null)
            // 右子树为空查左子树
            p = pl;
        else if ((kc != null ||
                (kc = comparableClassFor(k)) != null) &&
                (dir = compareComparables(kc, k, pk)) != 0)
            // 通过compare方法比较key值的大小决定使用左子树还是右子树
            p = (dir < 0) ? pl : pr;
        else if ((q = pr.find(h, k, kc)) != null)
            // 如果以上条件都不通过，则尝试在右子树查找
            return q;
        else
            // 都没找到就在左子树查找
            p = pl;
    } while (p != null);
    return null;
}
```

经典二叉查找树的查找过程，先根据hash值比较，再根据key值比较决定是查左子树还是右子树。

### 1.10 remove(Object key)方法

```
public V remove(Object key) {
    Node<K, V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
}

final Node<K, V> removeNode(int hash, Object key, Object value,
                            boolean matchValue, boolean movable) {
    Node<K, V>[] tab;
    Node<K, V> p;
    int n, index;
    // 如果桶的数量大于0且待删除的元素所在的桶的第一个元素不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
        Node<K, V> node = null, e;
        K k;
        V v;
        if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
            // 如果第一个元素正好就是要找的元素，赋值给node变量后续删除使用
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                // 如果第一个元素是树节点，则以树的方式查找节点
                node = ((TreeNode<K, V>) p).getTreeNode(hash, key);
            else {
                // 否则遍历整个链表查找元素
                do {
                    if (e.hash == hash &&
                            ((k = e.key) == key ||
                                    (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        // 如果找到了元素，则看参数是否需要匹配value值，如果不需要匹配直接删除，如果需要匹配则看value值是否与传入的value相等
        if (node != null && (!matchValue || (v = node.value) == value ||
                (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                // 如果是树节点，调用树的删除方法（以node调用的，是删除自己）
                ((TreeNode<K, V>) node).removeTreeNode(this, tab, movable);
            else if (node == p)
                // 如果待删除的元素是第一个元素，则把第二个元素移到第一的位置
                tab[index] = node.next;
            else
                // 否则删除node节点
                p.next = node.next;
            ++modCount;
            --size;
            // 删除节点后置处理
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

（1）先查找元素所在的节点；

（2）如果找到的节点是树节点，则按树的移除节点处理；

（3）如果找到的节点是桶中的第一个节点，则把第二个节点移到第一的位置；

（4）否则按链表删除节点处理；

（5）修改size，调用移除节点后置处理等；

TreeNode.removeTreeNode(...)方法

```
final void removeTreeNode(HashMap<K, V> map, Node<K, V>[] tab,
                          boolean movable) {
    int n;
    // 如果桶的数量为0直接返回
    if (tab == null || (n = tab.length) == 0)
        return;
    // 节点在桶中的索引
    int index = (n - 1) & hash;
    // 第一个节点，根节点，根左子节点
    TreeNode<K, V> first = (TreeNode<K, V>) tab[index], root = first, rl;
    // 后继节点，前置节点
    TreeNode<K, V> succ = (TreeNode<K, V>) next, pred = prev;

    if (pred == null)
        // 如果前置节点为空，说明当前节点是根节点，则把后继节点赋值到第一个节点的位置，相当于删除了当前节点
        tab[index] = first = succ;
    else
        // 否则把前置节点的下个节点设置为当前节点的后继节点，相当于删除了当前节点
        pred.next = succ;

    // 如果后继节点不为空，则让后继节点的前置节点指向当前节点的前置节点，相当于删除了当前节点
    if (succ != null)
        succ.prev = pred;

    // 如果第一个节点为空，说明没有后继节点了，直接返回
    if (first == null)
        return;

    // 如果根节点的父节点不为空，则重新查找父节点
    if (root.parent != null)
        root = root.root();

    // 如果根节点为空，则需要反树化（将树转化为链表）
    // 如果需要移动节点且树的高度比较小，则需要反树化
    if (root == null
            || (movable
            && (root.right == null
            || (rl = root.left) == null
            || rl.left == null))) {
        tab[index] = first.untreeify(map);  // too small
        return;
    }

    // 分割线，以上都是删除链表中的节点，下面才是直接删除红黑树的节点（因为TreeNode本身即是链表节点又是树节点）

    // 删除红黑树节点的大致过程是寻找右子树中最小的节点放到删除节点的位置，然后做平衡，此处不过多注释
    TreeNode<K, V> p = this, pl = left, pr = right, replacement;
    if (pl != null && pr != null) {
        TreeNode<K, V> s = pr, sl;
        while ((sl = s.left) != null) // find successor
            s = sl;
        boolean c = s.red;
        s.red = p.red;
        p.red = c; // swap colors
        TreeNode<K, V> sr = s.right;
        TreeNode<K, V> pp = p.parent;
        if (s == pr) { // p was s's direct parent
            p.parent = s;
            s.right = p;
        } else {
            TreeNode<K, V> sp = s.parent;
            if ((p.parent = sp) != null) {
                if (s == sp.left)
                    sp.left = p;
                else
                    sp.right = p;
            }
            if ((s.right = pr) != null)
                pr.parent = s;
        }
        p.left = null;
        if ((p.right = sr) != null)
            sr.parent = p;
        if ((s.left = pl) != null)
            pl.parent = s;
        if ((s.parent = pp) == null)
            root = s;
        else if (p == pp.left)
            pp.left = s;
        else
            pp.right = s;
        if (sr != null)
            replacement = sr;
        else
            replacement = p;
    } else if (pl != null)
        replacement = pl;
    else if (pr != null)
        replacement = pr;
    else
        replacement = p;
    if (replacement != p) {
        TreeNode<K, V> pp = replacement.parent = p.parent;
        if (pp == null)
            root = replacement;
        else if (p == pp.left)
            pp.left = replacement;
        else
            pp.right = replacement;
        p.left = p.right = p.parent = null;
    }

    TreeNode<K, V> r = p.red ? root : balanceDeletion(root, replacement);

    if (replacement == p) {  // detach
        TreeNode<K, V> pp = p.parent;
        p.parent = null;
        if (pp != null) {
            if (p == pp.left)
                pp.left = null;
            else if (p == pp.right)
                pp.right = null;
        }
    }
    if (movable)
        moveRootToFront(tab, r);
}
```

（1）TreeNode本身既是链表节点也是红黑树节点；

（2）先删除链表节点；

（3）再删除红黑树节点并做平衡；

## 2. LinkedHashMap

LinkedHashMap内部维护了一个双向链表，能保证元素按插入的顺序访问，也能以访问顺序访问，可以用来实现LRU缓存策略。

LinkedHashMap可以看成是 LinkedList + HashMap。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/LinkedHashMap.png)

LinkedHashMap继承HashMap，拥有HashMap的所有特性，并且额外增加的按一定顺序访问的特性。

### 2.1 存储结构

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/LinkedHashMap-structure.png)

我们知道HashMap使用（数组 + 单链表 + 红黑树）的存储结构，那LinkedHashMap是怎么存储的呢？

通过上面的继承体系，我们知道它继承了Map，所以它的内部也有这三种结构，但是它还额外添加了一种“双向链表”的结构存储所有元素的顺序。

添加删除元素的时候需要同时维护在HashMap中的存储，也要维护在LinkedList中的存储，所以性能上来说会比HashMap稍慢。

### 2.2 属性

```
/**
 * 双向链表的头节点，旧数据存在头节点。
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * 双向链表的尾节点，新数据存在尾节点。
 */
transient LinkedHashMap.Entry<K,V> tail;

/**
 *是否需要按访问顺序排序，如果为false则按插入顺序存储元素，如果是true则按访问顺序存储元素。
 *
 * @serial
 */
final boolean accessOrder;
```

### 2.3 内部类

```
// 位于LinkedHashMap中
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}

// 位于HashMap中
static class Node<K, V> implements Map.Entry<K, V> {
    final int hash;
    final K key;
    V value;
    Node<K, V> next;
}
```

存储节点，继承自HashMap的Node类，next用于单链表存储于桶中，before和after用于双向链表存储所有元素。

### 2.4 构造方法

```
/**
 * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
 * with the specified initial capacity and load factor.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
public LinkedHashMap(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor);
    accessOrder = false;
}

/**
 * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
 * with the specified initial capacity and a default load factor (0.75).
 *
 * @param  initialCapacity the initial capacity
 * @throws IllegalArgumentException if the initial capacity is negative
 */
public LinkedHashMap(int initialCapacity) {
    super(initialCapacity);
    accessOrder = false;
}

/**
 * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
 * with the default initial capacity (16) and load factor (0.75).
 */
public LinkedHashMap() {
    super();
    accessOrder = false;
}

/**
 * Constructs an insertion-ordered <tt>LinkedHashMap</tt> instance with
 * the same mappings as the specified map.  The <tt>LinkedHashMap</tt>
 * instance is created with a default load factor (0.75) and an initial
 * capacity sufficient to hold the mappings in the specified map.
 *
 * @param  m the map whose mappings are to be placed in this map
 * @throws NullPointerException if the specified map is null
 */
public LinkedHashMap(Map<? extends K, ? extends V> m) {
    super();
    accessOrder = false;
    putMapEntries(m, false);
}

/**
 * Constructs an empty <tt>LinkedHashMap</tt> instance with the
 * specified initial capacity, load factor and ordering mode.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @param  accessOrder     the ordering mode - <tt>true</tt> for
 *         access-order, <tt>false</tt> for insertion-order
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
public LinkedHashMap(int initialCapacity,
                     float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

前四个构造方法accessOrder都等于false，说明双向链表是按插入顺序存储元素。

最后一个构造方法accessOrder从构造方法参数传入，如果传入true，则就实现了按访问顺序存储元素，这也是实现LRU缓存策略的关键。

### 2.5 afterNodeInsertion方法

```
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

在节点插入之后做些什么，在HashMap中的putVal()方法中被调用，可以看到HashMap中这个方法的实现为空。

```
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}

protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

evict，驱逐的意思。

（1）如果evict为true，且头节点不为空，且确定移除最老的元素，那么就调用HashMap.removeNode()把头节点移除（这里的头节点是双向链表的头节点，而不是某个桶中的第一个元素）；

（2）HashMap.removeNode()从HashMap中把这个节点移除之后，会调用afterNodeRemoval()方法；

（3）afterNodeRemoval()方法在LinkedHashMap中也有实现，用来在移除元素后修改双向链表，见下文；

**（4）默认removeEldestEntry()方法返回false，也就是不删除元素。**

### 2.6 afterNodeAccess(Node<K,V> e)

在节点访问之后被调用，主要在**put()已经存在的元素**或**get()时被调用**，如果accessOrder为true，调用这个方法把访问到的节点移动到双向链表的末尾。

```
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    // 如果accessOrder为true，并且访问的节点不是尾节点
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        // 把p节点从双向链表中移除
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;

        if (a != null)
            a.before = b;
        else
            last = b;

        // 把p节点放到双向链表的末尾
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        // 尾节点等于p
        tail = p;
        ++modCount;
    }
}
```

（1）如果accessOrder为true，并且访问的节点不是尾节点；

（2）从双向链表中移除访问的节点；

（3）把访问的节点加到双向链表的末尾；（末尾为最新访问的元素）

### 2.7 get(Object key)方法

```
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}
```

如果查找到了元素，且accessOrder为true，则调用afterNodeAccess()方法把访问的节点移到双向链表的末尾。

### 2.8 LinkedHashMap如何实现LRU缓存淘汰策略

LRU，Least Recently Used，最近最少使用，也就是优先淘汰最近最少使用的元素。

如果使用LinkedHashMap，我们把accessOrder设置为true是不是就差不多能实现这个策略了呢？答案是肯定的。请看下面的代码：

```
package com.coolcoding.code;

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * @author: tangtong
 * @date: 2019/3/18
 */
public class LRUTest {
    public static void main(String[] args) {
        // 创建一个只有5个元素的缓存
        LRU<Integer, Integer> lru = new LRU<>(5, 0.75f);
        lru.put(1, 1);
        lru.put(2, 2);
        lru.put(3, 3);
        lru.put(4, 4);
        lru.put(5, 5);
        lru.put(6, 6);
        lru.put(7, 7);

        System.out.println(lru.get(4));

        lru.put(6, 666);

        // 输出: {3=3, 5=5, 7=7, 4=4, 6=666}
        // 可以看到最旧的元素被删除了
        // 且最近访问的4被移到了后面
        System.out.println(lru);
    }
}

class LRU<K, V> extends LinkedHashMap<K, V> {

    // 保存缓存的容量
    private int capacity;

    public LRU(int capacity, float loadFactor) {
        super(capacity, loadFactor, true);
        this.capacity = capacity;
    }

    /**
    * 重写removeEldestEntry()方法设置何时移除旧元素
    * @param eldest
    * @return
    */
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // 当元素个数大于了缓存的容量, 就移除元素
        return size() > this.capacity;
    }
}
```

## 3. WeakHashMap

WeakHashMap是一种弱引用map，内部的key会存储为弱引用，当jvm gc的时候，如果这些key没有强引用存在的话，会被gc回收掉，下一次当我们操作map的时候会把对应的Entry整个删除掉，基于这种特性，WeakHashMap特别适用于缓存处理。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/WeakHashMap.png)

WeakHashMap没有实现Clone和Serializable接口，所以不具有克隆和序列化的特性。

### 3.1存储结构

WeakHashMap因为gc的时候会把没有强引用的key回收掉，所以注定了它里面的元素不会太多，因此也就不需要像HashMap那样元素多的时候转化为红黑树来处理了。

因此，WeakHashMap的存储结构只有（数组 + 链表）。

### 3.2 属性

```
/**
 * 默认初始容量为16
 */
private static final int DEFAULT_INITIAL_CAPACITY = 16;

/**
 * 最大容量为2的30次方
 */
private static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * 默认装载因子
 */
private static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * 桶
 */
Entry<K,V>[] table;

/**
 * 元素个数
 */
private int size;

/**
 * 扩容门槛，等于capacity * loadFactor
 */
private int threshold;

/**
 * 装载因子
 */
private final float loadFactor;

/**
 * 引用队列，当弱键失效的时候会把Entry添加到这个队列中
 */
private final ReferenceQueue<Object> queue = new ReferenceQueue<>();
```

当弱键失效的时候会把Entry添加到这个队列中，当下次访问map的时候会把失效的Entry清除掉。

### 3.3 内部类

WeakHashMap内部的存储节点, 没有key属性。

```
//WeakHashMap.java
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
    // 可以发现没有key, 因为key是作为弱引用存到Referen类中
    V value;
    final int hash;
    Entry<K,V> next;

    Entry(Object key, V value,
          ReferenceQueue<Object> queue,
          int hash, Entry<K,V> next) {
        // 调用WeakReference的构造方法初始化key和引用队列
        super(key, queue);
        this.value = value;
        this.hash  = hash;
        this.next  = next;
    }
}

//WeakReference.java
public class WeakReference<T> extends Reference<T> {
    public WeakReference(T referent, ReferenceQueue<? super T> q) {
        // 调用Reference的构造方法初始化key和引用队列
        super(referent, q);
    }
}

//Reference.java
public abstract class Reference<T> {
    // 实际存储key的地方
    private T referent;         /* Treated specially by GC */
    // 引用队列
    volatile ReferenceQueue<? super T> queue;

    Reference(T referent, ReferenceQueue<? super T> queue) {
        this.referent = referent;
        this.queue = (queue == null) ? ReferenceQueue.NULL : queue;
    }
}
```

从Entry的构造方法我们知道，key和queue最终会传到到Reference的构造方法中，这里的key就是Reference的referent属性，它会被gc特殊对待，即当没有强引用存在时，当下一次gc的时候会被清除。

### 3.4 构造方法

```
public WeakHashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Initial Capacity: "+
                initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;

    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal Load factor: "+
                loadFactor);
    int capacity = 1;
    while (capacity < initialCapacity)
        capacity <<= 1;
    table = newTable(capacity);
    this.loadFactor = loadFactor;
    threshold = (int)(capacity * loadFactor);
}

public WeakHashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public WeakHashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}

public WeakHashMap(Map<? extends K, ? extends V> m) {
    this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
            DEFAULT_INITIAL_CAPACITY),
            DEFAULT_LOAD_FACTOR);
    putAll(m);
}
```

构造方法与HashMap基本类似，初始容量为大于等于传入容量最近的2的n次方，扩容门槛threshold等于capacity * loadFactor。

### 3.5 put(K key, V value)

```
public V put(K key, V value) {
    // 如果key为空，用空对象代替
    Object k = maskNull(key);
    // 计算key的hash值
    int h = hash(k);
    // 获取桶
    Entry<K,V>[] tab = getTable();
    // 计算元素在哪个桶中，h & (length-1)
    int i = indexFor(h, tab.length);

    // 遍历桶对应的链表
    for (Entry<K,V> e = tab[i]; e != null; e = e.next) {
        if (h == e.hash && eq(k, e.get())) {
            // 如果找到了元素就使用新值替换旧值，并返回旧值
            V oldValue = e.value;
            if (value != oldValue)
                e.value = value;
            return oldValue;
        }
    }

    modCount++;
    // 如果没找到就把新值插入到链表的头部
    Entry<K,V> e = tab[i];
    tab[i] = new Entry<>(k, value, queue, h, e);
    // 如果插入元素后数量达到了扩容门槛就把桶的数量扩容为2倍大小
    if (++size >= threshold)
        resize(tab.length * 2);
    return null;
}

final int hash(Object k) {
        int h = k.hashCode();

        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
    
private static int indexFor(int h, int length) {
        return h & (length-1);
    }
```

（1）计算hash；

这里与HashMap有所不同，HashMap中如果key为空直接返回0，这里是用空对象来计算的。

另外打散方式也不同，HashMap只用了一次异或，这里用了四次，HashMap给出的解释是一次够了，而且就算冲突了也会转换成红黑树，对效率没什么影响。



（2）计算在哪个桶中；

（3）遍历桶对应的链表；

（4）如果找到元素就用新值替换旧值，并返回旧值；

（5）如果没找到就在链表头部插入新元素；

HashMap就插入到链表尾部。

（6）如果元素数量达到了扩容门槛，就把容量扩大到2倍大小；

HashMap中是大于threshold才扩容，这里等于threshold就开始扩容了。

### 3.6 resize(int newCapacity)

扩容方法。

```
void resize(int newCapacity) {
    Entry<K,V>[] oldTable = getTable();
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry<K,V>[] newTable = newTable(newCapacity);
    transfer(oldTable, newTable);
    table = newTable;

    /*
     * If ignoring null elements and processing ref queue caused massive
     * shrinkage, then restore old table.  This should be rare, but avoids
     * unbounded expansion of garbage-filled tables.
     */
      // 如果元素个数大于扩容门槛的一半，则使用新桶和新容量，并计算新的扩容门槛
    if (size >= threshold / 2) {
        threshold = (int)(newCapacity * loadFactor);
    } else {
    	// 否则把元素再转移回旧桶，还是使用旧桶
        // 因为在transfer的时候会清除失效的Entry，所以元素个数可能没有那么大了，就不需要扩容了
        expungeStaleEntries();
        transfer(newTable, oldTable);
        table = oldTable;
    }
}
```

（1）判断旧容量是否达到最大容量；

（2）新建新桶并把元素全部转移到新桶中；

（3）如果转移后元素个数不到扩容门槛的一半，则把元素再转移回旧桶，继续使用旧桶，说明不需要扩容；

（4）否则使用新桶，并计算新的扩容门槛；

（5）转移元素的过程中会把key为null的元素清除掉，所以size会变小；

```
private void transfer(Entry<K,V>[] src, Entry<K,V>[] dest) {
    // 遍历旧桶
    for (int j = 0; j < src.length; ++j) {
        Entry<K,V> e = src[j];
        src[j] = null;
        while (e != null) {
            Entry<K,V> next = e.next;
            Object key = e.get();
            // 如果key等于了null就清除，说明key被gc清理掉了，则把整个Entry清除
            if (key == null) {
                e.next = null;  // Help GC
                e.value = null; //  "   "
                size--;
            } else {
                // 否则就计算在新桶中的位置并把这个元素放在新桶对应链表的头部
                int i = indexFor(e.hash, dest.length);
                e.next = dest[i];
                dest[i] = e;
            }
            e = next;
        }
    }
}
```

### 3.7 get(Object key)

获取元素。

```
public V get(Object key) {
    Object k = maskNull(key);
    int h = hash(k);
    Entry<K,V>[] tab = getTable();
    int index = indexFor(h, tab.length);
    Entry<K,V> e = tab[index];
    while (e != null) {
        if (e.hash == h && eq(k, e.get()))
            return e.value;
        e = e.next;
    }
    return null;
}
```

（1）计算hash值；

（2）遍历所在桶对应的链表；

（3）如果找到了就返回元素的value值；

（4）如果没找到就返回空；

### 3.8 remove(Object key)

移除元素。

```
public V remove(Object key) {
    Object k = maskNull(key);
    // 计算hash
    int h = hash(k);
    Entry<K,V>[] tab = getTable();
    int i = indexFor(h, tab.length);
    // 元素所在的桶的第一个元素
    Entry<K,V> prev = tab[i];
    Entry<K,V> e = prev;

    // 遍历链表
    while (e != null) {
        Entry<K,V> next = e.next;
        if (h == e.hash && eq(k, e.get())) {
            // 如果找到了就删除元素
            modCount++;
            size--;

            if (prev == e)
                // 如果是头节点，就把头节点指向下一个节点
                tab[i] = next;
            else
                // 如果不是头节点，删除该节点
                prev.next = next;
            return e.value;
        }
        prev = e;
        e = next;
    }

    return null;
}
```

（1）计算hash；

（2）找到所在的桶；

（3）遍历桶对应的链表；

（4）如果找到了就删除该节点，并返回该节点的value值；

（5）如果没找到就返回null；

3.9 expungeStaleEntries() 

剔除失效的Entry。

```
private void expungeStaleEntries() {
    // 遍历引用队列
    for (Object x; (x = queue.poll()) != null; ) {
        synchronized (queue) {
            @SuppressWarnings("unchecked")
            Entry<K,V> e = (Entry<K,V>) x;
            int i = indexFor(e.hash, table.length);
            // 找到所在的桶
            Entry<K,V> prev = table[i];
            Entry<K,V> p = prev;
            // 遍历链表
            while (p != null) {
                Entry<K,V> next = p.next;
                // 找到该元素
                if (p == e) {
                    // 删除该元素
                    if (prev == e)
                        table[i] = next;
                    else
                        prev.next = next;
                    // Must not null out e.next;
                    // stale entries may be in use by a HashIterator
                    e.value = null; // Help GC
                    size--;
                    break;
                }
                prev = p;
                p = next;
            }
        }
    }
}
```

（1）当key失效的时候gc会自动把对应的Entry添加到这个引用队列中；

（2）所有对map的操作都会直接或间接地调用到这个方法先移除失效的Entry，比如getTable()、size()、resize()；

（3）这个方法的目的就是遍历引用队列，并把其中保存的Entry从map中移除掉，具体的过程请看类注释；

（4）从这里可以看到移除Entry的同时把value也一并置为null帮助gc清理元素，防御性编程。

### 3.9 使用场景

```
package com.coolcoding.code;

import java.util.Map;
import java.util.WeakHashMap;

public class WeakHashMapTest {

public static void main(String[] args) {
    Map<String, Integer> map = new WeakHashMap<>(3);

    // 放入3个new String()声明的字符串
    map.put(new String("1"), 1);
    map.put(new String("2"), 2);
    map.put(new String("3"), 3);

    // 放入不用new String()声明的字符串
    map.put("6", 6);

    // 使用key强引用"3"这个字符串
    String key = null;
    for (String s : map.keySet()) {
        // 这个"3"和new String("3")不是一个引用
        if (s.equals("3")) {
            key = s;
        }
    }

    // 输出{6=6, 1=1, 2=2, 3=3}，未gc所有key都可以打印出来
    System.out.println(map);

    // gc一下
    System.gc();

    // 放一个new String()声明的字符串
    map.put(new String("4"), 4);

    // 输出{4=4, 6=6, 3=3}，gc后放入的值和强引用的key可以打印出来
    System.out.println(map);

    // key与"3"的引用断裂
    key = null;

    // gc一下
    System.gc();

    // 输出{6=6}，gc后强引用的key可以打印出来
    System.out.println(map);
}
}
```

在这里通过new String()声明的变量才是弱引用，使用"6"这种声明方式会一直存在于常量池中，不会被清理，所以"6"这个元素会一直在map里面，其它的元素随着gc都会被清理掉。

（1）WeakHashMap使用（数组 + 链表）存储结构；

（2）WeakHashMap中的key是弱引用，gc的时候会被清除；

（3）每次对map的操作都会剔除失效key对应的Entry；

（4）使用String作为key时，一定要使用new String()这样的方式声明key，才会失效，其它的基本类型的包装类型是一样的；

（5）WeakHashMap常用来作为缓存使用；

### 3.10 强、软、弱、虚引用

（1）强引用

使用最普遍的引用。如果一个对象具有强引用，它绝对不会被gc回收。如果内存空间不足了，gc宁愿抛出OutOfMemoryError，也不是会回收具有强引用的对象。

（2）软引用

如果一个对象只具有软引用，则内存空间足够时不会回收它，但内存空间不够时就会回收这部分对象。只要这个具有软引用对象没有被回收，程序就可以正常使用。

（3）弱引用

如果一个对象只具有弱引用，则不管内存空间够不够，当gc扫描到它时就会回收它。

（4）虚引用

如果一个对象只具有虚引用，那么它就和没有任何引用一样，任何时候都可能被gc回收。

软（弱、虚）引用必须和一个引用队列（ReferenceQueue）一起使用，当gc回收这个软（弱、虚）引用引用的对象时，会把这个软（弱、虚）引用放到这个引用队列中。

比如，上述的Entry是一个弱引用，它引用的对象是key，当key被回收时，Entry会被放到queue中。

## 4.TreeMap

TreeMap使用红黑树存储元素，可以保证元素按key值的大小进行遍历。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/TreeMap.png)

TreeMap实现了Map、SortedMap、NavigableMap、Cloneable、Serializable等接口。

SortedMap规定了元素可以按key的大小来遍历，它定义了一些返回部分map的方法。

```
public interface SortedMap<K,V> extends Map<K,V> {
    // key的比较器
    Comparator<? super K> comparator();
    // 返回fromKey（包含）到toKey（不包含）之间的元素组成的子map
    SortedMap<K,V> subMap(K fromKey, K toKey);
    // 返回小于toKey（不包含）的子map
    SortedMap<K,V> headMap(K toKey);
    // 返回大于等于fromKey（包含）的子map
    SortedMap<K,V> tailMap(K fromKey);
    // 返回最小的key
    K firstKey();
    // 返回最大的key
    K lastKey();
    // 返回key集合
    Set<K> keySet();
    // 返回value集合
    Collection<V> values();
    // 返回节点集合
    Set<Map.Entry<K, V>> entrySet();
}
```

NavigableMap是对SortedMap的增强，定义了一些返回离目标key最近的元素的方法。

```
public interface NavigableMap<K,V> extends SortedMap<K,V> {
    // 小于给定key的最大节点
    Map.Entry<K,V> lowerEntry(K key);
    // 小于给定key的最大key
    K lowerKey(K key);
    // 小于等于给定key的最大节点
    Map.Entry<K,V> floorEntry(K key);
    // 小于等于给定key的最大key
    K floorKey(K key);
    // 大于等于给定key的最小节点
    Map.Entry<K,V> ceilingEntry(K key);
    // 大于等于给定key的最小key
    K ceilingKey(K key);
    // 大于给定key的最小节点
    Map.Entry<K,V> higherEntry(K key);
    // 大于给定key的最小key
    K higherKey(K key);
    // 最小的节点
    Map.Entry<K,V> firstEntry();
    // 最大的节点
    Map.Entry<K,V> lastEntry();
    // 弹出最小的节点
    Map.Entry<K,V> pollFirstEntry();
    // 弹出最大的节点
    Map.Entry<K,V> pollLastEntry();
    // 返回倒序的map
    NavigableMap<K,V> descendingMap();
    // 返回有序的key集合
    NavigableSet<K> navigableKeySet();
    // 返回倒序的key集合
    NavigableSet<K> descendingKeySet();
    // 返回从fromKey到toKey的子map，是否包含起止元素可以自己决定
    NavigableMap<K,V> subMap(K fromKey, boolean fromInclusive,
                             K toKey,   boolean toInclusive);
    // 返回小于toKey的子map，是否包含toKey自己决定
    NavigableMap<K,V> headMap(K toKey, boolean inclusive);
    // 返回大于fromKey的子map，是否包含fromKey自己决定
    NavigableMap<K,V> tailMap(K fromKey, boolean inclusive);
    // 等价于subMap(fromKey, true, toKey, false)
    SortedMap<K,V> subMap(K fromKey, K toKey);
    // 等价于headMap(toKey, false)
    SortedMap<K,V> headMap(K toKey);
    // 等价于tailMap(fromKey, true)
    SortedMap<K,V> tailMap(K fromKey);
}
```

### 4.1存储结构

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/TreeMap-structure.png)

TreeMap只使用到了红黑树，所以它的时间复杂度为O(log n)，红黑树的特性。

（1）每个节点或者是黑色，或者是红色。

（2）根节点是黑色。

（3）每个叶子节点（NIL）是黑色。（注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！）

（4）如果一个节点是红色的，则它的子节点必须是黑色的。

（5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。

### 4.2 属性

```
/**
 * 比较器，如果没传则key要实现Comparable接口
 */
private final Comparator<? super K> comparator;

/**
 * 根节点
 */
private transient Entry<K,V> root;

/**
 * 元素个数
 */
private transient int size = 0;

/**
 * 修改次数
 */
private transient int modCount = 0;
```

（1）comparator

按key的大小排序有两种方式，一种是key实现Comparable接口，一种方式通过构造方法传入比较器。

（2）root

根节点，TreeMap没有桶的概念，所有的元素都存储在一颗树中。

### 4.3 内部类

存储节点，典型的红黑树结构。

```
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
    boolean color = BLACK;
}
```

### 4.4 构造方法

```
/**
 * 默认构造方法，key必须实现Comparable接口
 */
public TreeMap() {
    comparator = null;
}

/**
 * 使用传入的comparator比较两个key的大小
 */
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}

/**
 * key必须实现Comparable接口，把传入map中的所有元素保存到新的TreeMap中
 */
public TreeMap(Map<? extends K, ? extends V> m) {
    comparator = null;
    putAll(m);
}

/**
 * 使用传入map的比较器，并把传入map中的所有元素保存到新的TreeMap中
 */
public TreeMap(SortedMap<K, ? extends V> m) {
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```

构造方法主要分成两类，一类是使用comparator比较器，一类是key必须实现Comparable接口。

同时实现Comparable接口，又使用了Comparator比较器的key,Comparator比较器优先级大于Comparable接口接口。

4.5  get（Object  key）

获取元素，典型的二叉查找树的查找方法。

```
public V get(Object key) {
    // 根据key查找元素
    Entry<K,V> p = getEntry(key);
    // 找到了返回value值，没找到返回null
    return (p==null ? null : p.value);
}

final Entry<K,V> getEntry(Object key) {
    // 如果comparator不为空，使用comparator的版本获取元素
    if (comparator != null)
        return getEntryUsingComparator(key);
    // 如果key为空返回空指针异常
    if (key == null)
        throw new NullPointerException();
    // 将key强转为Comparable
    @SuppressWarnings("unchecked")
    Comparable<? super K> k = (Comparable<? super K>) key;
    // 从根元素开始遍历
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            // 如果小于0从左子树查找
            p = p.left;
        else if (cmp > 0)
            // 如果大于0从右子树查找
            p = p.right;
        else
            // 如果相等说明找到了直接返回
            return p;
    }
    // 没找到返回null
    return null;
}

final Entry<K,V> getEntryUsingComparator(Object key) {
    @SuppressWarnings("unchecked")
    K k = (K) key;
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        // 从根元素开始遍历
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = cpr.compare(k, p.key);
            if (cmp < 0)
                // 如果小于0从左子树查找
                p = p.left;
            else if (cmp > 0)
                // 如果大于0从右子树查找
                p = p.right;
            else
                // 如果相等说明找到了直接返回
                return p;
        }
    }
    // 没找到返回null
    return null;
}
```

（1）从root遍历整个树；

（2）如果待查找的key比当前遍历的key小，则在其左子树中查找；

（3）如果待查找的key比当前遍历的key大，则在其右子树中查找；

（4）如果待查找的key与当前遍历的key相等，则找到了该元素，直接返回；

（5）从这里可以看出是否有comparator分化成了两个方法，但是内部逻辑一模一样，因此可见笔者`comparator = (k1, k2) -> ((Comparable<? super K>)k1).compareTo(k2);`这种改造的必要性。

### 4.5 左旋

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/left-rotation.jpg)

左旋，就是以某个节点为支点向左旋转。

整个左旋过程如下：

（1）将 y的左节点 设为 x的右节点，即将 β 设为 x的右节点；

（2）将 x 设为 y的左节点的父节点，即将 β的父节点 设为 x；

（3）将 x的父节点 设为 y的父节点；

（4）如果 x的父节点 为空节点，则将y设置为根节点；如果x是它父节点的左（右）节点，则将y设置为x父节点的左（右）节点；

（5）将 x 设为 y的左节点；

（6）将 x的父节点 设为 y；

让我们来看看TreeMap中的实现：

```
/**
 * 以p为支点进行左旋
 * 假设p为图中的x
 */
private void rotateLeft(Entry<K,V> p) {
    if (p != null) {
        // p的右节点，即y
        Entry<K,V> r = p.right;

        // （1）将 y的左节点 设为 x的右节点
        p.right = r.left;

        // （2）将 x 设为 y的左节点的父节点（如果y的左节点存在的话）
        if (r.left != null)
            r.left.parent = p;

        // （3）将 x的父节点 设为 y的父节点
        r.parent = p.parent;

        // （4）...
        if (p.parent == null)
            // 如果 x的父节点 为空，则将y设置为根节点
            root = r;
        else if (p.parent.left == p)
            // 如果x是它父节点的左节点，则将y设置为x父节点的左节点
            p.parent.left = r;
        else
            // 如果x是它父节点的右节点，则将y设置为x父节点的右节点
            p.parent.right = r;

        // （5）将 x 设为 y的左节点
        r.left = p;

        // （6）将 x的父节点 设为 y
        p.parent = r;
    }
}
```

### 4.6 右旋

右旋，就是以某个节点为支点向右旋转。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/right-rotation.jpg)

整个右旋过程如下：

（1）将 x的右节点 设为 y的左节点，即 将 β 设为 y的左节点；

（2）将 y 设为 x的右节点的父节点，即 将 β的父节点 设为 y；

（3）将 y的父节点 设为 x的父节点；

（4）如果 y的父节点 是 空节点，则将x设为根节点；如果y是它父节点的左（右）节点，则将x设为y的父节点的左（右）节点；

（5）将 y 设为 x的右节点；

（6）将 y的父节点 设为 x；

让我们来看看TreeMap中的实现：

```
/**
 * 以p为支点进行右旋
 * 假设p为图中的y
 */
private void rotateRight(Entry<K,V> p) {
    if (p != null) {
        // p的左节点，即x
        Entry<K,V> l = p.left;

        // （1）将 x的右节点 设为 y的左节点
        p.left = l.right;

        // （2）将 y 设为 x的右节点的父节点（如果x有右节点的话）
        if (l.right != null) l.right.parent = p;

        // （3）将 y的父节点 设为 x的父节点
        l.parent = p.parent;

        // （4）...
        if (p.parent == null)
            // 如果 y的父节点 是 空节点，则将x设为根节点
            root = l;
        else if (p.parent.right == p)
            // 如果y是它父节点的右节点，则将x设为y的父节点的右节点
            p.parent.right = l;
        else
            // 如果y是它父节点的左节点，则将x设为y的父节点的左节点
            p.parent.left = l;

        // （5）将 y 设为 x的右节点
        l.right = p;

        // （6）将 y的父节点 设为 x
        p.parent = l;
    }
}
```

### 4.7 插入元素

插入元素，如果元素在树中存在，则替换value；如果元素不存在，则插入到对应的位置，再平衡树。

```
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {
        // 如果没有根节点，直接插入到根节点
        compare(key, key); // type (and possibly null) check
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    // key比较的结果
    int cmp;
    // 用来寻找待插入节点的父节点
    Entry<K,V> parent;
    // 根据是否有comparator使用不同的分支
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        // 如果使用的是comparator方式，key值可以为null，只要在comparator.compare()中允许即可
        // 从根节点开始遍历寻找
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                // 如果小于0从左子树寻找
                t = t.left;
            else if (cmp > 0)
                // 如果大于0从右子树寻找
                t = t.right;
            else
                // 如果等于0，说明插入的节点已经存在了，直接更换其value值并返回旧值
                return t.setValue(value);
        } while (t != null);
    }
    else {
        // 如果使用的是Comparable方式，key不能为null
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
        // 从根节点开始遍历寻找
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                // 如果小于0从左子树寻找
                t = t.left;
            else if (cmp > 0)
                // 如果大于0从右子树寻找
                t = t.right;
            else
                // 如果等于0，说明插入的节点已经存在了，直接更换其value值并返回旧值
                return t.setValue(value);
        } while (t != null);
    }
    // 如果没找到，那么新建一个节点，并插入到树中
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        // 如果小于0插入到左子节点
        parent.left = e;
    else
        // 如果大于0插入到右子节点
        parent.right = e;

    // 插入之后的平衡
    fixAfterInsertion(e);
    // 元素个数加1（不需要扩容）
    size++;
    // 修改次数加1
    modCount++;
    // 如果插入了新节点返回空
    return null;
}
```

插入再平衡

插入的元素默认都是红色，因为插入红色元素只违背了第4条特性，那么我们只要根据这个特性来平衡就容易多了。

根据不同的情况有以下几种处理方式：

1.插入的元素如果是根节点，则直接涂成黑色即可，不用平衡；

2.插入的元素的父节点如果为黑色，不需要平衡；

3.插入的元素的父节点如果为红色，则违背了特性4，需要平衡，平衡时又分成下面三种情况：

**（如果父节点是祖父节点的左节点）**

| 情况                                                         | 策略                                                         |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| 1）父节点为红色，叔叔节点也为红色                            | （1）将父节点设为黑色；<br/> （2）将叔叔节点设为黑色；<br/> （3）将祖父节点设为红色；<br/> （4）将祖父节点设为新的当前节点，进入下一次循环判断； |
| 2）父节点为红色，叔叔节点为黑色，且当前节点是其父节点的右节点 | （1）将父节点作为新的当前节点； <br/>（2）以新当节点为支点进行左旋，进入情况3）； |
| 3）父节点为红色，叔叔节点为黑色，且当前节点是其父节点的左节点 | （1）将父节点设为黑色； <br/>（2）将祖父节点设为红色； <br/>（3）以祖父节点为支点进行右旋，进入下一次循环判断； |

**（如果父节点是祖父节点的右节点，则正好与上面反过来）**

| 情况                                                         | 策略                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1）父节点为红色，叔叔节点也为红色                            | （1）将父节点设为黑色； <br/>（2）将叔叔节点设为黑色； <br/>（3）将祖父节点设为红色； <br/>（4）将祖父节点设为新的当前节点，进入下一次循环判断； |
| 2）父节点为红色，叔叔节点为黑色，且当前节点是其父节点的左节点 | （1）将父节点作为新的当前节点； <br/>（2）以新当节点为支点进行右旋； |
| 3）父节点为红色，叔叔节点为黑色，且当前节点是其父节点的右节点 | （1）将父节点设为黑色； <br/>（2）将祖父节点设为红色； <br/>（3）以祖父节点为支点进行左旋，进入下一次循环判断； |

```
/**
 * 插入再平衡
 *（1）每个节点或者是黑色，或者是红色。
 *（2）根节点是黑色。
 *（3）每个叶子节点（NIL）是黑色。（注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！）
 *（4）如果一个节点是红色的，则它的子节点必须是黑色的。
 *（5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。
 */
private void fixAfterInsertion(Entry<K,V> x) {
    // 插入的节点为红节点，x为当前节点
    x.color = RED;

    // 只有当插入节点不是根节点且其父节点为红色时才需要平衡（违背了特性4）
    while (x != null && x != root && x.parent.color == RED) {
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            // a）如果父节点是祖父节点的左节点
            // y为叔叔节点
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                // 情况1）如果叔叔节点为红色
                // （1）将父节点设为黑色
                setColor(parentOf(x), BLACK);
                // （2）将叔叔节点设为黑色
                setColor(y, BLACK);
                // （3）将祖父节点设为红色
                setColor(parentOf(parentOf(x)), RED);
                // （4）将祖父节点设为新的当前节点
                x = parentOf(parentOf(x));
            } else {
                // 如果叔叔节点为黑色
                // 情况2）如果当前节点为其父节点的右节点
                if (x == rightOf(parentOf(x))) {
                    // （1）将父节点设为当前节点
                    x = parentOf(x);
                    // （2）以新当前节点左旋
                    rotateLeft(x);
                }
                // 情况3）如果当前节点为其父节点的左节点（如果是情况2）则左旋之后新当前节点正好为其父节点的左节点了）
                // （1）将父节点设为黑色
                setColor(parentOf(x), BLACK);
                // （2）将祖父节点设为红色
                setColor(parentOf(parentOf(x)), RED);
                // （3）以祖父节点为支点进行右旋
                rotateRight(parentOf(parentOf(x)));
            }
        } else {
            // b）如果父节点是祖父节点的右节点
            // y是叔叔节点
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                // 情况1）如果叔叔节点为红色
                // （1）将父节点设为黑色
                setColor(parentOf(x), BLACK);
                // （2）将叔叔节点设为黑色
                setColor(y, BLACK);
                // （3）将祖父节点设为红色
                setColor(parentOf(parentOf(x)), RED);
                // （4）将祖父节点设为新的当前节点
                x = parentOf(parentOf(x));
            } else {
                // 如果叔叔节点为黑色
                // 情况2）如果当前节点为其父节点的左节点
                if (x == leftOf(parentOf(x))) {
                    // （1）将父节点设为当前节点
                    x = parentOf(x);
                    // （2）以新当前节点右旋
                    rotateRight(x);
                }
                // 情况3）如果当前节点为其父节点的右节点（如果是情况2）则右旋之后新当前节点正好为其父节点的右节点了）
                // （1）将父节点设为黑色
                setColor(parentOf(x), BLACK);
                // （2）将祖父节点设为红色
                setColor(parentOf(parentOf(x)), RED);
                // （3）以祖父节点为支点进行左旋
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    // 平衡完成后将根节点设为黑色
    root.color = BLACK;
}
```

插入元素举例

我们依次向红黑树中插入 4、2、3 三个元素，来一起看看整个红黑树平衡的过程。

三个元素都插入完成后，符合父节点是祖父节点的左节点，叔叔节点为黑色，且当前节点是其父节点的右节点，即情况2）。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/20201222173337.png)

情况2）需要做以下两步处理：

（1）将父节点作为新的当前节点；

（2）以新当节点为支点进行左旋，进入情况3）；

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/20201222173403.png)

情况3）需要做以下三步处理：

（1）将父节点设为黑色；

（2）将祖父节点设为红色；

（3）以祖父节点为支点进行右旋，进入下一次循环判断；

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/20201222173428.png)

下一次循环不符合父节点为红色了，退出循环，插入再平衡完成。

### 4.8 删除元素

删除元素本身比较简单，就是采用二叉树的删除规则。

（1）如果删除的位置有两个叶子节点，则从其右子树中取最小的元素放到删除的位置，然后把删除位置移到替代元素的位置，进入下一步。

（2）如果删除的位置只有一个叶子节点（有可能是经过第一步转换后的删除位置），则把那个叶子节点作为替代元素，放到删除的位置，然后把这个叶子节点删除。

（3）如果删除的位置没有叶子节点，则直接把这个删除位置的元素删除即可。

（4）针对红黑树，如果删除位置是黑色节点，还需要做再平衡。

（5）如果有替代元素，则以替代元素作为当前节点进入再平衡。

（6）如果没有替代元素，则以删除的位置的元素作为当前节点进入再平衡，平衡之后再删除这个节点。

```
public V remove(Object key) {
    Entry<K,V> p = getEntry(key);
    if (p == null)
        return null;

    V oldValue = p.value;
    deleteEntry(p);
    return oldValue;
}
```

```
//如果这个方法不理解可以查看二叉树节点删除视频。https://www.bilibili.com/video/BV1HE411f7ZX?from=search&seid=3003638974794138226
private void deleteEntry(Entry<K,V> p) {
    // 修改次数加1
    modCount++;
    // 元素个数减1
    size--;
	//情况3。如果有两个子节点，找到后继节点，用后继节点的值替换带删除位置的节点的值，然后删除后继节点。
    if (p.left != null && p.right != null) {
        // 如果当前节点既有左子节点，又有右子节点
        // 取其右子树中最小的节点
        Entry<K,V> s = successor(p);
        // 用右子树中最小节点的值替换当前节点的值
        p.key = s.key;
        p.value = s.value;
        // 把右子树中最小节点设为当前节点
        p = s;
        // 这种情况实际上并没有删除p节点，而是把p节点的值改了，实际删除的是p的后继节点
    }

    // 如果原来的当前节点（p）有2个子节点，则当前节点已经变成原来p的右子树中的最小节点了，也就是说其没有左子节点了
    // 到这一步，p肯定只有一个子节点了
    // 如果当前节点有子节点，则用子节点替换当前节点
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);
   //情况2 ，如果有一个子节点，则将该子节点作为替换节点，删除当前位值节点。
    if (replacement != null) {
        // 把替换节点直接放到当前节点的位置上（相当于删除了p，并把替换节点移动过来了）
        replacement.parent = p.parent;
        //如果父节点为空那么，替换节点变为根节点
        if (p.parent == null)
            root = replacement;
         
        else if (p == p.parent.left)//删除位置节点是其父类的左节点
            p.parent.left  = replacement; 
        else  //删除位置节点是其父类的右节点
            p.parent.right = replacement;

        // 将p的各项属性都设为空，断开与删除节点的连接
        p.left = p.right = p.parent = null;

        // 如果p是黑节点，则需要再平衡
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    } else if (p.parent == null) {
        // 如果当前节点就是根节点，则直接将根节点设为空即可
        root = null;
    } else {
        // 如果当前节点没有子节点且其为黑节点，则把自己当作虚拟的替换节点进行再平衡
        if (p.color == BLACK)
            fixAfterDeletion(p);
		//情况1，没有子节点，直接删除
        // 平衡完成后删除当前节点（与父节点断绝关系）
        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}
```

**后继节点**

​																																	8

​																															/                \

​																														6                       9

​																													/       \              /           \

​																												5            7        10            11

①如果节点有右子树，则该节点的后继节点就是往右子树出发，然后转到右子树的左子树，一直到左子树的左子树为空（即输入节点的右子树的最左子树，例如节点8，后继节点就是9）

②如果节点没有右子树，则向上寻找父节点，直到父节点的左子树等于当前节点，则该父节点就是后继节点（如节点7，没有右子树，则向上找，这时到6这个节点，因为6的左子树不等于7，所以继续向上找，这时到节点8，8的左子树等于当前节点6，所以返回8）

```
static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.right != null) {
     //p右侧（大于p的节点） 节点中最小的节点。
        Entry<K,V> p = t.right;
        while (p.left != null)
            p = p.left;
        return p;
    } else {
    	//如果右侧无节点，采用回溯法，向上寻找父节点，直到父节点的左子树等于当前节点
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```

#### 4.8.1 元素删除平衡

经过上面的处理，真正删除的肯定是黑色节点才会进入到再平衡阶段。

因为删除的是黑色节点，导致整颗树不平衡了，所以这里我们假设把删除的黑色赋予当前节点，这样当前节点除了它自已的颜色还多了一个黑色，那么：

（1）如果当前节点是根节点，则直接涂黑即可，不需要再平衡；

（2）如果当前节点是红+黑节点，则直接涂黑即可，不需要平衡；

（3）如果当前节点是黑+黑节点，则我们只要通过旋转把这个多出来的黑色不断的向上传递到一个红色节点即可，这又可能会出现以下四种情况：

**（假设当前节点为父节点的左子节点）**

| 情况                                                         | 策略                                                         |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| 1）x是黑+黑节点，x的兄弟是红节点                             | （1）将兄弟节点设为黑色； <br/>（2）将父节点设为红色； <br/>（3）以父节点为支点进行左旋； <br/>（4）重新设置x的兄弟节点，进入下一步； |
| 2）x是黑+黑节点，x的兄弟是黑节点，且兄弟节点的两个子节点都是黑色 | （1）将兄弟节点设置为红色； <br/>（2）将x的父节点作为新的当前节点，进入下一次循环； |
| 3）x是黑+黑节点，x的兄弟是黑节点，且兄弟节点的右子节点为黑色，左子节点为红色 | （1）将兄弟节点的左子节点设为黑色； <br/>（2）将兄弟节点设为红色； <br/>（3）以兄弟节点为支点进行右旋；<br/> （4）重新设置x的兄弟节点，进入下一步； |
| 4）x是黑+黑节点，x的兄弟是黑节点，且兄弟节点的右子节点为红色，左子节点任意颜色 | （1）将兄弟节点的颜色设为父节点的颜色；<br/> （2）将父节点设为黑色； <br/>（3）将兄弟节点的右子节点设为黑色； <br/>   (4）以父节点为支点进行左旋； <br/>（5）将root作为新的当前节点（退出循环）； |

**（假设当前节点为父节点的右子节点，正好反过来）**

| 情况                                                         | 策略                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1）x是黑+黑节点，x的兄弟是红节点                             | （1）将兄弟节点设为黑色； <br/>（2）将父节点设为红色； <br/>（3）以父节点为支点进行右旋； <br/>（4）重新设置x的兄弟节点，进入下一步； |
| 2）x是黑+黑节点，x的兄弟是黑节点，且兄弟节点的两个子节点都是黑色 | （1）将兄弟节点设置为红色； <br/>（2）将x的父节点作为新的当前节点，进入下一次循环； |
| 3）x是黑+黑节点，x的兄弟是黑节点，且兄弟节点的左子节点为黑色，右子节点为红色 | （1）将兄弟节点的右子节点设为黑色； <br/>（2）将兄弟节点设为红色； <br/>（3）以兄弟节点为支点进行左旋； <br/>（4）重新设置x的兄弟节点，进入下一步； |
| 4）x是黑+黑节点，x的兄弟是黑节点，且兄弟节点的左子节点为红色，右子节点任意颜色 | （1）将兄弟节点的颜色设为父节点的颜色； <br/>（2）将父节点设为黑色； <br/>（3）将兄弟节点的左子节点设为黑色； <br/>（4）以父节点为支点进行右旋； <br/>（5）将root作为新的当前节点（退出循环）； |

让我们来看看TreeMap中的实现：

```
![treemap-delete2](C:%5CUsers%5CAdministrator%5CPictures%5Ctreemap-delete2.png)/**
 * 删除再平衡
 *（1）每个节点或者是黑色，或者是红色。
 *（2）根节点是黑色。
 *（3）每个叶子节点（NIL）是黑色。（注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！）
 *（4）如果一个节点是红色的，则它的子节点必须是黑色的。
 *（5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。
 */
private void fixAfterDeletion(Entry<K,V> x) {
    // 只有当前节点不是根节点且当前节点是黑色时才进入循环
    while (x != root && colorOf(x) == BLACK) {
        if (x == leftOf(parentOf(x))) {
            // 如果当前节点是其父节点的左子节点
            // sib是当前节点的兄弟节点
            Entry<K,V> sib = rightOf(parentOf(x));

            // 情况1）如果兄弟节点是红色
            if (colorOf(sib) == RED) {
                // （1）将兄弟节点设为黑色
                setColor(sib, BLACK);
                // （2）将父节点设为红色
                setColor(parentOf(x), RED);
                // （3）以父节点为支点进行左旋
                rotateLeft(parentOf(x));
                // （4）重新设置x的兄弟节点，进入下一步
                sib = rightOf(parentOf(x));
            }

            if (colorOf(leftOf(sib))  == BLACK &&
                    colorOf(rightOf(sib)) == BLACK) {
                // 情况2）如果兄弟节点的两个子节点都是黑色
                // （1）将兄弟节点设置为红色
                setColor(sib, RED);
                // （2）将x的父节点作为新的当前节点，进入下一次循环
                x = parentOf(x);
            } else {
                if (colorOf(rightOf(sib)) == BLACK) {
                    // 情况3）如果兄弟节点的右子节点为黑色
                    // （1）将兄弟节点的左子节点设为黑色
                    setColor(leftOf(sib), BLACK);
                    // （2）将兄弟节点设为红色
                    setColor(sib, RED);
                    // （3）以兄弟节点为支点进行右旋
                    rotateRight(sib);
                    // （4）重新设置x的兄弟节点
                    sib = rightOf(parentOf(x));
                }
                // 情况4）
                // （1）将兄弟节点的颜色设为父节点的颜色
                setColor(sib, colorOf(parentOf(x)));
                // （2）将父节点设为黑色
                setColor(parentOf(x), BLACK);
                // （3）将兄弟节点的右子节点设为黑色
                setColor(rightOf(sib), BLACK);
                // （4）以父节点为支点进行左旋
                rotateLeft(parentOf(x));
                // （5）将root作为新的当前节点（退出循环）
                x = root;
            }
        } else { // symmetric
            // 如果当前节点是其父节点的右子节点
            // sib是当前节点的兄弟节点
            Entry<K,V> sib = leftOf(parentOf(x));

            // 情况1）如果兄弟节点是红色
            if (colorOf(sib) == RED) {
                // （1）将兄弟节点设为黑色
                setColor(sib, BLACK);
                // （2）将父节点设为红色
                setColor(parentOf(x), RED);
                // （3）以父节点为支点进行右旋
                rotateRight(parentOf(x));
                // （4）重新设置x的兄弟节点
                sib = leftOf(parentOf(x));
            }

            if (colorOf(rightOf(sib)) == BLACK &&
                    colorOf(leftOf(sib)) == BLACK) {
                // 情况2）如果兄弟节点的两个子节点都是黑色
                // （1）将兄弟节点设置为红色
                setColor(sib, RED);
                // （2）将x的父节点作为新的当前节点，进入下一次循环
                x = parentOf(x);
            } else {
                if (colorOf(leftOf(sib)) == BLACK) {
                    // 情况3）如果兄弟节点的左子节点为黑色
                    // （1）将兄弟节点的右子节点设为黑色
                    setColor(rightOf(sib), BLACK);
                    // （2）将兄弟节点设为红色
                    setColor(sib, RED);
                    // （3）以兄弟节点为支点进行左旋
                    rotateLeft(sib);
                    // （4）重新设置x的兄弟节点
                    sib = leftOf(parentOf(x));
                }
                // 情况4）
                // （1）将兄弟节点的颜色设为父节点的颜色
                setColor(sib, colorOf(parentOf(x)));
                // （2）将父节点设为黑色
                setColor(parentOf(x), BLACK);
                // （3）将兄弟节点的左子节点设为黑色
                setColor(leftOf(sib), BLACK);
                // （4）以父节点为支点进行右旋
                rotateRight(parentOf(x));
                // （5）将root作为新的当前节点（退出循环）
                x = root;
            }
        }
    }

    // 退出条件为多出来的黑色向上传递到了根节点或者红节点
    // 则将x设为黑色即可满足红黑树规则
    setColor(x, BLACK);
}
```

#### 4.8.2 删除元素举例

情况4： 当前节点为黑，兄弟节点为黑，兄弟节点的右子节点为红色，左子节点任意颜色。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/treemap-delete1.png)

1. 删除6号元素，则从右子树中找到了最小元素7，7是后继节点，6号元素的位置用值7替换，然后删除7号元素位置，7又没有子节点了，所以把7作为当前节点进行再平衡。

2. 我们看到7是黑节点，且其兄弟为黑节点，且其兄弟的两个子节点都是红色，满足情况4），平衡之后如下图所示。

   

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/treemap-delete2.png)







再删除7号元素，则从右子树中找到了最小元素8，8有子节点且为黑色，所以8的子节点9是替代节点，以9为当前节点进行再平衡。

我们发现9是红节点，则直接把它涂成黑色即满足了红黑树的特性，不需要再过多的平衡了。



![](https://gitee.com/zhouxiaoliang/img/raw/master/img/treemap-delete3.png)

把根节点删除，从右子树中找到了最小的元素5，5没有子节点，所以把5作为当前节点进行再平衡。

我们看到5是黑节点，且其兄弟为红色，符合情况1），平衡之后如下图所示，然后进入情况2）。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/treemap-delete4.png)

对情况2）进行再平衡后如下图所示。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/treemap-delete5.png)

然后进入下一次循环，发现不符合循环条件了，直接把x涂为黑色即可，退出这个方法之后会把旧x删除掉（见deleteEntry()方法），最后的结果就是下面这样。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/treemap-delete6.png)

上面只描述了 在父节点左侧的4种情况。   如果删除节点在父节点右侧，会有另外与之相反的另外4种情况。

### 4.9  遍历

#### 4.9.1 二叉树的遍历

二叉查找树的遍历有前序遍历、中序遍历、后序遍历。

（1）前序遍历，先遍历我，再遍历我的左子节点，最后遍历我的右子节点；

（2）中序遍历，先遍历我的左子节点，再遍历我，最后遍历我的右子节点；

（3）后序遍历，先遍历我的左子节点，再遍历我的右子节点，最后遍历我；

这里的前中后都是以“我”的顺序为准的，我在前就是前序遍历，我在中就是中序遍历，我在后就是后序遍历。

下面让我们看看经典的中序遍历是怎么实现的：

```
public class TreeMapTest {

    public static void main(String[] args) {
        // 构建一颗10个元素的树
        TreeNode<Integer> node = new TreeNode<>(1, null).insert(2)
                .insert(6).insert(3).insert(5).insert(9)
                .insert(7).insert(8).insert(4).insert(10);

        // 中序遍历，打印结果为1到10的顺序
        node.root().inOrderTraverse();
    }
}

/**
 * 树节点，假设不存在重复元素
 * @param <T>
 */
class TreeNode<T extends Comparable<T>> {
    T value;
    TreeNode<T> parent;
    TreeNode<T> left, right;

    public TreeNode(T value, TreeNode<T> parent) {
        this.value = value;
        this.parent = parent;
    }

    /**
     * 获取根节点
     */
    TreeNode<T> root() {
        TreeNode<T> cur = this;
        while (cur.parent != null) {
            cur = cur.parent;
        }
        return cur;
    }

    /**
     * 中序遍历
     */
    void inOrderTraverse() {
        if(this.left != null) this.left.inOrderTraverse();
        System.out.println(this.value);
        if(this.right != null) this.right.inOrderTraverse();
    }

    /**
     * 经典的二叉树插入元素的方法
     */
    TreeNode<T> insert(T value) {
        // 先找根元素
        TreeNode<T> cur = root();

        TreeNode<T> p;
        int dir;

        // 寻找元素应该插入的位置
        do {
            p = cur;
            if ((dir=value.compareTo(p.value)) < 0) {
                cur = cur.left;
            } else {
                cur = cur.right;
            }
        } while (cur != null);

        // 把元素放到找到的位置
        if (dir < 0) {
            p.left = new TreeNode<>(value, p);
            return p.left;
        } else {
            p.right = new TreeNode<>(value, p);
            return p.right;
        }
    }
}
```

#### 4.9.2 TreeMap的遍历

二叉树的遍历我们很明显地看到，它是通过递归的方式实现的，但是递归会占用额外的空间，直接到线程栈整个释放掉才会把方法中申请的变量销毁掉，所以当元素特别多的时候是一件很危险的事。

那么如何不采用递归来遍历二叉树呢？

TreeMap中的实现

```
@Override
public void forEach(BiConsumer<? super K, ? super V> action) {
    Objects.requireNonNull(action);
    // 遍历前的修改次数
    int expectedModCount = modCount;
      // 执行遍历，先获取第一个元素的位置，再循环遍历后继节点
    for (Entry<K, V> e = getFirstEntry(); e != null; e = successor(e)) {
      // 执行动作
        action.accept(e.key, e.value);
 // 如果发现修改次数变了，则抛出异常
        if (expectedModCount != modCount) {
            throw new ConcurrentModificationException();
        }
    }
}
```

（1）寻找第一个节点；

从根节点开始找最左边的节点，即最小的元素。

```
    final Entry<K,V> getFirstEntry() {
        Entry<K,V> p = root;
        // 从根节点开始找最左边的节点，即最小的元素
        if (p != null)
            while (p.left != null)
                p = p.left;
        return p;
    }
```

（2）循环遍历后继节点；

略，上面已经介绍过了





这种方式的时间复杂度

首先，寻找第一个元素，因为红黑树是接近平衡的二叉树，所以找最小的节点，相当于是从顶到底了，时间复杂度为O(log n)；

其次，寻找后继节点，因为红黑树插入元素的时候会自动平衡，最坏的情况就是寻找右子树中最小的节点，时间复杂度为O(log k)，k为右子树元素个数；

最后，需要遍历所有元素，时间复杂度为O(n)；

所以，总的时间复杂度为 O(log n) + O(n * log k) ≈ O(n)。

虽然遍历红黑树的时间复杂度是O(n)，

## 5.ConcurrentHashMap

（1）ConcurrentHashMap与HashMap的数据结构是否一样？

（2）HashMap在多线程环境下何时会出现并发安全问题？

（3）ConcurrentHashMap是怎么解决并发安全问题的？

（4）ConcurrentHashMap使用了哪些锁？

（5）ConcurrentHashMap中有哪些不常见的技术值得学习？

（6）扩容过程中，读访问能否访问到数据，怎么实现的？

（7）扩容过程中写访问如何处理？

（8）假设指定桶位形成红黑树，且当前红黑树正在进行自平衡，那么此时的读线程是被阻塞等待还是有其他的方案，详细说明？

（9）JDK1.8中统计当前散列表中元素的个数是如何实现的？为什么没有使用AtomicLong?

(10)  描述一下LastRun机制。

ConcurrentHashMap是HashMap的线程安全版本，内部也是使用（数组 + 链表 + 红黑树）的结构来存储元素。

相比于同样线程安全的HashTable来说，效率等各方面都有极大地提高。

HashMap 为什么会出现线程不安全的问题？ 

HashMap多线程并发问题分析  ---->   https://www.cnblogs.com/andy-zhou/p/5402984.html

### 5.1 锁介绍

（1）synchronized

java中的关键字，内部实现为监视器锁，主要是通过对象监视器在对象头中的字段来表明的。

synchronized从旧版本到现在已经做了很多优化了，在运行时会有三种存在方式：偏向锁，轻量级锁，重量级锁。

偏向锁，是指一段同步代码一直被一个线程访问，那么这个线程会自动获取锁，降低获取锁的代价。

轻量级锁，是指当锁是偏向锁时，被另一个线程所访问，偏向锁会升级为轻量级锁，这个线程会通过自旋的方式尝试获取锁，不会阻塞，提高性能。

重量级锁，是指当锁是轻量级锁时，当自旋的线程自旋了一定的次数后，还没有获取到锁，就会进入阻塞状态，该锁升级为重量级锁，重量级锁会使其他线程阻塞，性能降低。

（2）CAS（轻量级锁的实现原理）

CAS，Compare And Swap，它是一种乐观锁，认为对于同一个数据的并发操作不一定会发生修改，在更新数据的时候，尝试去更新数据，如果失败就不断尝试。



（3）volatile（非锁，内存可见性，保证代码执行顺序不会重排）

java中的关键字，当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。（通知所有其他线程，它们的变量值状态失效)

volatile只保证可见性，不保证原子性，比如 volatile修改的变量 i，针对i++操作，不保证每次结果都正确，因为i++操作是两步操作，相当于 i = i +1，先读取，再加1，这种情况volatile是无法保证的。

（4）自旋锁

自旋锁 即轻量级锁（CAS），是指尝试获取锁的线程不会阻塞，而是循环的方式不断尝试，这样的好处是减少线程的上下文切换带来的开锁，提高性能，缺点是循环会消耗CPU。

（5）分段锁

分段锁，是一种锁的设计思路，它细化了锁的粒度，主要运用在ConcurrentHashMap中，实现高效的并发操作，当操作不需要更新整个数组时，就只锁数组中的一项就可以了。

（5）ReentrantLock

可重入锁，是指一个线程获取锁之后再尝试获取锁时会自动获取锁，可重入锁的优点是避免死锁。

其实，synchronized也是可重入锁。

### 5.2 属性

#### 5.2.1 常量

```
最大容量
private static final int MAXIMUM_CAPACITY = 1 << 30;

初始化容量
private static final int DEFAULT_CAPACITY = 16;

虚拟机限制的最大数组长度，在ArrayList中有说过，jdk1.8新引入的，ConcurrentHashMap的主体代码中是不使用这个的，主要用在Collection.toArray两个方法中
static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

并发级别
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;

负载因子
private static final float LOAD_FACTOR = 0.75f;

一个hash桶中hash冲突的数目大于此值时，把链表转化为红黑树，加快hash冲突时的查找速度
static final int TREEIFY_THRESHOLD = 8;

一个hash桶中hash冲突的数目小于等于此值时，把红黑树转化为链表，当数目比较少时，链表的实际查找速度更快，也是为了查找效率
static final int UNTREEIFY_THRESHOLD = 6;

当table数组的长度小于此值时，不会把链表转化为红黑树。所以转化为红黑树有两个条件，还有一个是 TREEIFY_THRESHOLD
static final int MIN_TREEIFY_CAPACITY = 64;


 扩容操作中，transfer这个步骤是允许多线程的，这个常量表示一个线程执行transfer时，最少要对连续的16个hash桶进行transfer
 （不足16就按16算）
 也就是单线程执行transfer时的最小任务量，单位为一个hash桶，这就是线程的transfer的步进（stride）
 最小值是DEFAULT_CAPACITY，不使用太小的值，避免太小的值引起transfer时线程竞争过多，如果计算出来的值小于此值，就使用此值
 正常步骤中会根据CPU核心数目来算出实际的，一个核心允许8个线程并发执行扩容操作的transfer步骤，这个8是个经验值，不能调整的
 因为transfer操作不是IO操作，也不是死循环那种100%的CPU计算，CPU计算率中等，1核心允许8个线程并发完成扩容，理想情况下也算是比较合理的值
 一段代码的IO操作越多，1核心对应的线程就要相应设置多点，CPU计算越多，1核心对应的线程就要相应设置少一些
 表明：默认的容量是16，也就是默认构造的实例，第一次扩容实际上是单线程执行的，看上去是可以多线程并发（方法允许多个线程进入），
     但是实际上其余的线程都会被一些if判断拦截掉，不会真正去执行扩容

private static final int MIN_TRANSFER_STRIDE = 16;

用于生成每次扩容都唯一的生成戳的数，最小是6。（这个值不是常量，但是也不提供修改方法）
private static int RESIZE_STAMP_BITS = 16;

最大的扩容线程的数量
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

移位量，把生成戳移位后保存在sizeCtl中当做扩容线程计数的基数，相反方向移位后能够反解出生成戳
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;


下面几个是特殊的节点的hash值，正常节点的hash值在hash函数中都处理过了，不会出现负数的情况，特殊节点在各自的实现类中有特殊的遍历方法
ForwardingNode的hash值，ForwardingNode是一种临时节点，在扩进行中才会出现，并且它不存储实际的数据
如果旧数组的一个hash桶中全部的节点都迁移到新数组中，旧数组就在这个hash桶中放置一个ForwardingNode
读操作或者迭代读时碰到ForwardingNode时，将操作转发到扩容后的新的table数组上去执行，写操作碰见它时，则尝试帮助扩容
static final int MOVED     = -1; // hash for forwarding nodes

TreeBin的hash值，TreeBin是ConcurrentHashMap中用于代理操作TreeNode的特殊节点，持有存储实际数据的红黑树的根节点
因为红黑树进行写入操作，整个树的结构可能会有很大的变化，这个对读线程有很大的影响，
所以TreeBin还要维护一个简单读写锁，这是相对HashMap，这个类新引入这种特殊节点的重要原因
static final int TREEBIN   = -2; // hash for roots of trees

ReservationNode的hash值，ReservationNode是一个保留节点，就是个占位符，不会保存实际的数据，正常情况是不会出现的，
在jdk1.8新的函数式有关的两个方法computeIfAbsent和compute中才会出现
static final int RESERVED  = -3; // hash for transient reservations

用于和负数hash值进行 & 运算，将其转化为正数（绝对值不相等），Hashtable中定位hash桶也有使用这种方式来进行负数转正数
最高位是0，其余位是1 ，eg: 01111111_11111111_11111111_11111111
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash

CPU的核心数，用于在扩容时计算一个线程一次要干多少活
static final int NCPU = Runtime.getRuntime().availableProcessors();

// 在序列化时使用，这是为了兼容以前的版本
/** For serialization compatibility. */
private static final ObjectStreamField[] serialPersistentFields = {
    new ObjectStreamField("segments", Segment[].class),
    new ObjectStreamField("segmentMask", Integer.TYPE),
    new ObjectStreamField("segmentShift", Integer.TYPE)
};

```

#### 5.2.2 变量

```
/* ---------------- Fields -------------- */

/**
 * The array of bins. Lazily initialized upon first insertion.
 * Size is always a power of two. Accessed directly by iterators.
 */
transient volatile Node<K,V>[] table;

//扩容后的新的table数组，只有在扩容时才有用
//nextTable != null，说明扩容方法还没有真正退出，一般可以认为是此时还有线程正在进行扩容，
//极端情况需要考虑此时扩容操作只差最后给几个变量赋值（包括nextTable = null）的这个大的步骤，
//这个大步骤执行时，通过sizeCtl经过一些计算得出来的扩容线程的数量是0
private transient volatile Node<K,V>[] nextTable;




// 非常重要的一个属性，源码中的英文翻译，直译过来是下面的四行文字的意思
//     sizeCtl = -1，表示有线程正在进行真正的初始化操作
//     sizeCtl = -(1 + nThreads)，表示有nThreads个线程正在进行扩容操作
//     sizeCtl > 0，表示接下来的真正的初始化操作中使用的容量，或者初始化/扩容完成后的threshold
//     sizeCtl = 0，默认值，此时在真正的初始化操作中使用默认容量
// 但是，通过对源码的理解，这段注释实际上是有问题的，有问题的是第二句，sizeCtl = -(1 + nThreads)这个，网上好多都是用第二句的直接翻译去解释代码，这样理解是错误的
// 默认构造的16个大小的ConcurrentHashMap，只有一个线程执行扩容时，sizeCtl = -2145714174，但是照这段英文注释的意思，sizeCtl的值应该是 -(1 + 1) = -2
// sizeCtl在小于0时的确有记录有多少个线程正在执行扩容任务的功能，但是不是这段英文注释说的那样直接用 -(1 + nThreads)
// 实际中使用了一种生成戳，根据生成戳算出一个基数，不同轮次的扩容操作的生成戳都是唯一的，来保证多次扩容之间不会交叉重叠，
//     当有n个线程正在执行扩容时，sizeCtl在值变为 (基数 + n)
// 1.8.0_111的源码的383-384行写了个说明：A generation stamp in field sizeCtl ensures that resizings do not overlap.
private transient volatile int sizeCtl;

// 下一个transfer任务的起始下标index 加上1 之后的值，transfer时下标index从length - 1开始往0走
// transfer时方向是倒过来的，迭代时是下标从小往大，二者方向相反，尽量减少扩容时transefer和迭代两者同时处理一个hash桶的情况，
// 顺序相反时，二者相遇过后，迭代没处理的都是已经transfer的hash桶，transfer没处理的，都是已经迭代的hash桶，冲突会变少
// 下标在[nextIndex - 实际的stride （下界要 >= 0）, nextIndex - 1]内的hash桶，就是每个transfer的任务区间
// 每次接受一个transfer任务，都要CAS执行 transferIndex = transferIndex - 实际的stride，
// 保证一个transfer任务不会被几个线程同时获取（相当于任务队列的size减1）
// 当没有线程正在执行transfer任务时，一定有transferIndex <= 0，这是判断是否需要帮助扩容的重要条件（相当于任务队列为空）
private transient volatile int transferIndex;


/**
 * Base counter value, used mainly when there is no contention,
 * but also as a fallback during table initialization
 * races. Updated via CAS.
 */
private transient volatile long baseCount;
/**
 * CAS自旋锁标志位，用于初始化，或者counterCells扩容时
 */
private transient volatile int cellsBusy;

/**
 * Table of counter cells. When non-null, size is a power of 2.
 */
private transient volatile CounterCell[] counterCells;

// views
private transient KeySetView<K,V> keySet;
private transient ValuesView<K,V> values;
private transient EntrySetView<K,V> entrySet;
```

### 5.3 内部类(基础类)

#### 5.3.1 Node 基本节点/普通节点

此节点就是一个很普通的Entry，在链表形式保存才使用这种节点，它存储实际的数据，基本结构类似于1.8的HashMap.Node，和1.7的Concurrent.HashEntry。

```
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;
    省略···
    
    
    // 不支持来自ConcurrentHashMap外部的修改，跟1.7的一样，迭代操作需要通过另外一个内部类MapEntry来代理，迭代写会重新执行一次put操作
    // 迭代中可以改变value，是一种写操作，此时需要保证这个节点还在map中，
    //     因此就重新put一次：节点不存在了，可以重新让它存在；节点还存在，相当于replace一次
    // 设计成这样主要是因为ConcurrentHashMap并非为了迭代操作而设计，它的迭代操作和其他写操作不好并发，
    //     迭代时的读写都是弱一致性的，碰见并发修改时尽量维护迭代的一致性
    // 返回值V也可能是个过时的值，保证V是最新的值会比较困难，而且得不偿失
    public final V setValue(V value) {
            throw new UnsupportedOperationException();
    }
    
    
    // 从此节点开始查找k对应的节点
    // 这里的实现是专为链表实现的，一般作用于头结点，各种特殊的子类有自己独特的实现
    // 不过主体代码中进行链表查找时，因为要特殊判断下第一个节点，所以很少直接用下面这个方法，
    //     而是直接写循环遍历链表，子类的查找则是用子类中重写的find方法
     /** Virtualized support for map.get(); overridden in subclasses.*/
    Node<K,V> find(int h, Object k) {
            Node<K,V> e = this;
            if (k != null) {
                do {
                    K ek;
                    if (e.hash == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                } while ((e = e.next) != null);
            }
            return null;
   }
```

#### 5.3.2 TreeNode 红黑树节点 

在红黑树形式保存时才存在，它也存储有实际的数据，结构和1.8的HashMap的TreeNode一样，一些方法的实现代码也基本一样。不过，ConcurrentHashMap对此节点的操作，都会由TreeBin来代理执行。也可以把这里的TreeNode看出是有一半功能的HashMap.TreeNode，另一半功能在ConcurrentHashMap.TreeBin中。

红黑树节点本身保存有普通链表节点Node的所有属性，因此可以使用两种方式进行读操作。

```
/**
 * Nodes for use in TreeBins
 */
static final class TreeNode<K,V> extends Node<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
     // 新添加的prev指针是为了删除方便，删除链表的非头节点的节点，都需要知道它的前一个节点才能进行删除，所以直接提供一个prev指针
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;

    TreeNode(int hash, K key, V val, Node<K,V> next,
             TreeNode<K,V> parent) {
        super(hash, key, val, next);
        this.parent = parent;
    }

    Node<K,V> find(int h, Object k) {
        return findTreeNode(h, k, null);
    }

    /**
     * Returns the TreeNode (or null if not found) for the given key
     * starting at given root.
     */
      // 以当前节点 this 为根节点开始遍历查找，跟HashMap.TreeNode.find实现一样
    final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {
        if (k != null) {
            TreeNode<K,V> p = this;
            do  {
                int ph, dir; K pk; TreeNode<K,V> q;
                TreeNode<K,V> pl = p.left, pr = p.right;
                if ((ph = p.hash) > h)
                    p = pl;
                else if (ph < h)
                    p = pr;
                else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                    return p;
                else if (pl == null)
                    p = pr;
                else if (pr == null)
                    p = pl;
                else if ((kc != null ||
                          (kc = comparableClassFor(k)) != null) &&
                         (dir = compareComparables(kc, k, pk)) != 0)
                    p = (dir < 0) ? pl : pr;
                else if ((q = pr.findTreeNode(h, k, kc)) != null)// 对右子树进行递归查找
                    return q;
                else
                    p = pl; // 前面递归查找了右边子树，这里循环时只用一直往左边找
            } while (p != null);
        }
        return null;
    }
}
```

#### 5.3.3 ForwardingNode 转发节点

ForwardingNode是一种临时节点，它的使用主要是在扩容阶段，hash值固定为-1，并且它不存储实际的数据数据。如果旧数组的一个hash桶中全部的节点都迁移到新数组中，旧数组就在这个hash桶中放置一个ForwardingNode。读操作或者迭代读时碰到ForwardingNode时，将操作转发到扩容后的新的table数组上去执行，写操作碰见它时，则尝试帮助扩容。

```
/**
 * A node inserted at head of bins during transfer operations.
 */
static final class ForwardingNode<K,V> extends Node<K,V> {
    final Node<K,V>[] nextTable;
    ForwardingNode(Node<K,V>[] tab) {
        super(MOVED, null, null, null);
        this.nextTable = tab;
    }
	//ForwardingNode的查找操作，直接在新数组nextTable上去进行查找
    Node<K,V> find(int h, Object k) {
        // loop to avoid arbitrarily deep recursion on forwarding nodes
        使用循环，避免多次碰到ForwardingNode导致递归过深
        outer: for (Node<K,V>[] tab = nextTable;;) {
            Node<K,V> e; int n;
            if (k == null || tab == null || (n = tab.length) == 0 ||
                (e = tabAt(tab, (n - 1) & h)) == null)
                return null;
            for (;;) {
                int eh; K ek;
                // 第一个节点就是要找的节点，直接返回
                if ((eh = e.hash) == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
                if (eh < 0) {
                    if (e instanceof ForwardingNode) {// 继续碰见ForwardingNode的情况，这里相当于是递归调用一次本方法
                        tab = ((ForwardingNode<K,V>)e).nextTable;
                        continue outer;
                    }
                    else
                        return e.find(h, k);// 碰见特殊节点，调用其find方法进行查找
                }
                if ((e = e.next) == null) // 普通节点直接循环遍历链表
                    return null;
            }
        }
    }
}
```

那么为什么会引入ForwardingNode这个节点的概念呢？

ConcurrentHashMap 的JDK8与JDK7版本的并发实现相比，最大的区别在于JDK8 的锁粒度更细，理想情况下table数组元素的大小就是其支持并发的最大数，在JDK7里面最大并发个数就是Segment的个数，默认值是16，可以通过构造函数改变一经创建不可更改，这个值就是并发的粒度，每一个segment下面管理一个table数组，加锁的时候其实锁住的是整个segment，这样设计的好处在于数组的扩容是不会影响其他的segment的，简化了并发设计，不足之处在于并发的粒度稍粗，所以在JDK8里面，去掉了分段锁，将锁的级别控制在了更细粒度的table元素级别，也就是说只需要锁住链表的head节点，并不会影响其他的table元素的读写，好处在于并发的粒度更细，影响更小，进而并发效率更高，但是不足之处在于并发扩容的时候，由于操作的table元素都是同一个，不像JDK7中分段控制，所以这里需要等扩容完之后，所有的读写操作才能进行，所以**扩容的效率就成为了整个并发的一个瓶颈点**，好在Doug lea大神对扩容做了优化，本来在一个线程扩容的时候，如果影响了其他线程的数据，那么其他的线程的读写操作都应该阻塞，但Doug lea说你们闲着也是闲着，不如来一起参与扩容任务，这样人多力量大，办完事你们该干啥干啥，别浪费时间，于是在JDK8的源码里面就引入了一个ForwardingNode类，在一个线程发起扩容的时候，就会改变sizeCtl这个值，

sizeCtl ：默认为0，用来控制table的初始化和扩容操作，具体应用在后续会体现出来。 

-1 代表table正在初始化 

-N 表示有N-1个线程正在进行扩容操作 

其余情况： 

​	1、如果table未初始化，表示table需要初始化的大小。 

​	2、如果table初始化完成，表示table的容量，默认是table大小的0.75倍

扩容时候会判断这个值，如果超过阈值就要扩容，首先根据运算得到需要遍历的次数i，然后利用tabAt方法获得i位置的元素f，初始化一个forwardNode实例fwd，如果f == null，则在table中的i位置放入fwd，否则采用头插法的方式把当前旧table数组的指定任务范围的数据给迁移到新的数组中，然后 
给旧table原位置赋值fwd。直到遍历过所有的节点以后就完成了复制工作，把table指向nextTable，并更新sizeCtl为新数组大小的0.75倍 ，扩容完成。在此期间如果其他线程的有读写操作都会判断head节点是否为forwardNode节点，如果是就帮助扩容。

具体代码会在**transfer**方法中体现

**在扩容时读写操作如何进行**

(1)对于get读操作，如果当前节点有数据，还没迁移完成，此时不影响读，能够正常进行。 

如果当前链表已经迁移完成，那么头节点会被设置成fwd节点，此时get线程会帮助扩容。 


(2)对于put/remove写操作，如果当前链表已经迁移完成，那么头节点会被设置成fwd节点，此时写线程会帮助扩容，如果扩容没有完成，当前链表的头节点会被锁住，所以写线程会被阻塞，直到扩容完成。 

#### 5.3.4 TreeBin 红黑树对象（代理操作TreeNode的节点）

TreeBin的hash值固定为-2，它是ConcurrentHashMap中用于代理操作TreeNode的特殊节点，持有存储实际数据的红黑树的根节点。因为红黑树进行写入操作，整个树的结构可能会有很大的变化，这个对读线程有很大的影响，所以TreeBin还要维护一个简单读写锁，这是相对HashMap，这个类新引入这种特殊节点的重要原因。

```
// 红黑树节点TreeNode实际上还保存有链表的指针，因此也可以用链表的方式进行遍历读取操作
// 自身维护一个简单的读写锁，不用考虑写-写竞争的情况
// 不是全部的写操作都要加写锁，只有部分的put/remove需要加写锁
// 很多方法的实现和jdk1.8的ConcurrentHashMap.TreeNode里面的方法基本一样，可以互相参考
static final class TreeBin<K,V> extends Node<K,V> {
    // 红黑树结构的跟节点
    TreeNode<K,V> root;
    // 链表结构的头节点
    volatile TreeNode<K,V> first;
    // 最近的一个设置 WAITER 标识位的线程
    volatile Thread waiter;
    // 整体的锁状态标识位
    volatile int lockState;
    
    【
    // 重要的一点，红黑树的 读锁状态 和 写锁状态 是互斥的，但是从ConcurrentHashMap角度来说，读写操作实际上可以是不互斥的
    // 红黑树的 读、写锁状态 是互斥的，指的是以红黑树方式进行的读操作和写操作（只有部分的put/remove需要加写锁）是互斥的
    // 但是当有线程持有红黑树的 写锁 时，读线程不会以红黑树方式进行读取操作，而是使用简单的链表方式进行读取，此时读操作和写操作可以并发执行
    // 当有线程持有红黑树的 读锁 时，写线程可能会阻塞，不过因为红黑树的查找很快，写线程阻塞的时间很短
    // 另外一点，ConcurrentHashMap的put/remove/replace方法本身就会锁住TreeBin节点，这里不会出现写-写竞争的情况，因此这里的读写锁可以实现得很简单
    】
    // values for lockState
    // 二进制001，红黑树的 写锁状态
    static final int WRITER = 1; // set while holding write lock
    // 二进制010，红黑树的 等待获取写锁的状态
    static final int WAITER = 2; // set when waiting for write lock
     // 二进制100，红黑树的 读锁状态，读锁可以叠加，也就是红黑树方式可以并发读，每有一个这样的读线程，lockState都加上一个READER的值
    static final int READER = 4; // increment value for setting read lock
    
     // 在hashCode相等并且不是Comparable类时才使用此方法进行判断大小
     static int tieBreakOrder(Object a, Object b) {
            int d;
            if (a == null || b == null ||
                (d = a.getClass().getName().
                 compareTo(b.getClass().getName())) == 0)
                d = (System.identityHashCode(a) <= System.identityHashCode(b) ?
                     -1 : 1);
            return d;
        }
     // 用以b为头结点的链表创建一棵红黑树
     TreeBin(TreeNode<K,V> b) {
        super(TREEBIN, null, null, null);
        this.first = b;
        TreeNode<K,V> r = null;
        for (TreeNode<K,V> x = b, next; x != null; x = next) {
            next = (TreeNode<K,V>)x.next;
            x.left = x.right = null;
            if (r == null) {
                x.parent = null;
                x.red = false;
                r = x;
            }
            else {
                K k = x.key;
                int h = x.hash;
                Class<?> kc = null;
                for (TreeNode<K,V> p = r;;) {
                    int dir, ph;
                    K pk = p.key;
                    if ((ph = p.hash) > h)
                        dir = -1;
                    else if (ph < h)
                        dir = 1;
                    else if ((kc == null && (kc = comparableClassFor(k)) == null) || (dir = compareComparables(kc, k, pk)) == 0)
                        dir = tieBreakOrder(k, pk);
                        TreeNode<K,V> xp = p;
                    if ((p = (dir <= 0) ? p.left : p.right) == null) {
                        x.parent = xp;
                        if (dir <= 0)
                            xp.left = x;
                        else
                            xp.right = x;
                        r = balanceInsertion(r, x);
                        break;
                    }
                }
            }
        }
        this.root = r;
        assert checkInvariants(root);
    }
     	/**
         * Acquires write lock for tree restructuring.
         */
         // 对根节点加 写锁，红黑树重构时需要加上 写锁
        private final void lockRoot() {
            if (!U.compareAndSwapInt(this, LOCKSTATE, 0, WRITER)) // 先尝试获取一次 写锁，如果没获取到写锁执行下马的方法
                contendedLock(); // offload to separate method //如果CAS失败，以竞争的方式加锁,继续获取直到成功才返回
            //如果CAS成功，或者contendedLock方法返回，那么表示获取写锁成功，写锁是独占锁
        }

        /**
         * Releases write lock for tree restructuring.
         */
         // 释放 写锁
        private final void unlockRoot() {
            lockState = 0;
        }
		 /**
         * Possibly blocks awaiting root lock.
         */
         // 可能会阻塞写线程，当写线程获取到写锁时，才会返回
         
        private final void contendedLock() {
        	//waiting标志位，如果为true表示可以阻塞自己（写线程）；否则不能阻塞自己（写线程）
            boolean waiting = false;
            //开启一个死循环，尝试获取锁，获取到锁就结束循环
            for (int s;;) {
            // ~WAITER是对WAITER进行二进制取反，当此时没有线程持有 读锁（不会有线程持有 写锁）时，这个if为真
            // ~WAITER，即~2，即表示 -3 是一个固定值. -3的二进制补码是 11111111111111111111111111111101
            // ((s = lockState) & ~WAITER)==0 表示 lockState为0(二进制数全是0)或者2(二进制数为10)时，结果为0
            //lockState为0时，表示没有任何线程获取任何锁；
            //lockState为2时，表示只有一个写线程在等待获取锁，这也就是前面讲的find方法中，最后一个读线程释放了读锁并且还有写线程等待获取写锁的情况，实际上就是该线程
            //这个判断如果为true则表示：没有任何线程获取任何锁，或者只有一个写线程在等待获取锁（就是当前线程被唤醒之后的逻辑）。
                if (((s = lockState) & ~WAITER) == 0) {
               		//此时当前获取写锁的线程可以继续尝试获取写锁
                    if (U.compareAndSwapInt(this, LOCKSTATE, s, WRITER)) {
                        if (waiting)// 获取到写锁时，如果自己曾经注册过 WAITER 状态，将其清除,表示此时没有写线程在等待写锁
                            waiter = null;
                        return;//获取到了锁contendedLock方法结束
                    }
                }
                //WAITER=010 那么(s & WAITER) == 0，s=0 或 01(十进制>=1)   
                //WAITER固定为2 (二进制10),如果s & WAITER为0，即需要s & 2 =0，那么s(lockState)必须为1或者大于2的数，比如4、8等等
                //由于不存在写并发（外面对写操作加上了synchronized锁），因此lockState一定属于大于2的数，比如4、8等等
             	//这表示有线程获取到了读锁，此时写线程应该等待
                else if ((s & WAITER) == 0) {
                //尝试将lockState设置为s | WAITER  ，这里的s|WAITER就相当于s+WAITER，即将此时的lockState加上2，表示有写线程在等待获取写锁
                    if (U.compareAndSwapInt(this, LOCKSTATE, s, s | WAITER)) { // 尝试占据 WAITER 状态标识位
                        waiting = true; //如果CAS成功，那么waiting置为true
                        waiter = Thread.currentThread(); //waiter设置为当前线程
                    }
                }
                //否则，根据前面的判断此时的lockState一定是6、10、14等数
                //判断是否需要阻塞自己，如果waiting=true，表示需要阻塞
                else if (waiting)
                //调用park方法阻塞自己，此时写线程不再继续执行代码，而是等待被唤醒
           	 	//这里的park不会释放之前获取到的synchronized锁，因为park或者unpark方法根本就与“锁”无关
            	//如果被唤醒，那么可能是因为最后一个读锁也被释放了，或者是因为被中断，那么继续循环获取锁
            	//该循环的唯一出口就是获取到了写锁
                    LockSupport.park(this);
            }
        }
        // 从根节点开始遍历查找，找到“相等”的节点就返回它，没找到就返回null
    	// 当有写线程加上 写锁 时，使用链表方式进行查找
        final Node<K,V> find(int h, Object k) {
            if (k != null) {
                for (Node<K,V> e = first; e != null; ) {
                    int s; K ek;
                     // 两种特殊情况下以链表的方式进行查找
                	 // 1、有线程正持有 写锁，这样做能够不阻塞读线程
                	 // 2、WAITER时，不再继续加 读锁，能够让已经被阻塞的写线程尽快恢复运行，或者刚好让某个写线程不被阻塞
                	 
                	 //(WAITER|WRITER)=3(二进制11)，lockState&3！=0，那么lockState>=1,
                	 //有线程持有写锁(lockState=1)，或有写线程在等待写锁(lockState=2)，不再加读锁，而是以链表方式查找
                    if (((s = lockState) & (WAITER|WRITER)) != 0) {
                        if (e.hash == h &&
                            ((ek = e.key) == k || (ek != null && k.equals(ek))))
                            return e;
                        e = e.next;
                    }
                    else if (U.compareAndSwapInt(this, LOCKSTATE, s,
                                                 s + READER)) {    //否则，加读锁, 读线程数量加1，读状态进行累加
                        TreeNode<K,V> r, p;
                        try {
                            p = ((r = root) == null ? null :
                                 r.findTreeNode(h, k, null));// 以红黑树方式查找
                        } finally {
                            Thread w;
                            // 如果这是最后一个读线程，并且有写线程因为 读锁 而阻塞，那么要通知它，告诉它可以尝试获取写锁了
                            // U.getAndAddInt(this, LOCKSTATE, -READER)这个操作是在更新之后返回lockstate的旧值，
                        	// 不是返回新值，相当于先判断==，再执行减法
                        	//(w = waiter) != null，表示有写线程在等待
                        	//U.getAndAddInt(this, LOCKSTATE, -READER) == (READER|WAITER),表示最后一个读线程
                            if (U.getAndAddInt(this, LOCKSTATE, -READER) == (READER|WAITER) && (w = waiter) != null)
                                LockSupport.unpark(w); // 让被阻塞的写线程运行起来，重新去尝试获取 写锁
                        }
                        return p;
                    }
                }
            }
            return null;
        }
        
//Unsafe类
//获取内存地址为obj+offset的变量值, 并将该变量值加上delta
public final int getAndAddInt(Object obj, long offset, int delta) {
    int v;
    do {
    	//通过对象和偏移量获取变量的值
    	//由于volatile的修饰, 所有线程看到的v都是一样的
        v= this.getIntVolatile(obj, offset);
    /*
	while中的compareAndSwapInt()方法尝试修改v的值,具体地, 该方法也会通过obj和offset获取变量的值
	如果这个值和v不一样, 说明其他线程修改了obj+offset地址处的值, 此时compareAndSwapInt()返回false, 继续循环
	如果这个值和v一样, 说明没有其他线程修改obj+offset地址处的值, 此时可以将obj+offset地址处的值改为v+delta, compareAndSwapInt()返回true, 退出循环
	Unsafe类中的compareAndSwapInt()方法是原子操作, 所以compareAndSwapInt()修改obj+offset地址处的值的时候不会被其他线程中断
	*/
    } while(!this.compareAndSwapInt(obj, offset, v, v + delta));

    return v;
}


 // 用于实现ConcurrentHashMap.putVal
    final TreeNode<K,V> putTreeVal(int h, K k, V v) {
        Class<?> kc = null;
        boolean searched = false;
        for (TreeNode<K,V> p = root;;) {
            int dir, ph; K pk;
            if (p == null) {
                first = root = new TreeNode<K,V>(h, k, v, null, null);
                break;
            }
            else if ((ph = p.hash) > h)
                dir = -1;
            else if (ph < h)
                dir = 1;
            else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                return p;
            else if ((kc == null && (kc = comparableClassFor(k)) == null) || (dir = compareComparables(kc, k, pk)) == 0) {
                if (!searched) {
                    TreeNode<K,V> q, ch;
                    searched = true;
                    if (((ch = p.left) != null && (q = ch.findTreeNode(h, k, kc)) != null) ||
                        ((ch = p.right) != null && (q = ch.findTreeNode(h, k, kc)) != null))
                        return q;
                }
                dir = tieBreakOrder(k, pk);
            }
 
            TreeNode<K,V> xp = p;
            if ((p = (dir <= 0) ? p.left : p.right) == null) {
                TreeNode<K,V> x, f = first;
                first = x = new TreeNode<K,V>(h, k, v, f, xp);
                if (f != null)
                    f.prev = x;
                if (dir <= 0)
                    xp.left = x;
                else
                    xp.right = x;
                // 下面是有关put加 写锁 部分
                // 二叉搜索树新添加的节点，都是取代原来某个的NIL节点（空节点，null节点）的位置
                if (!xp.red) // xp是新添加的节点的父节点，如果它是黑色的，新添加一个红色节点就能够保证x这部分的一部分路径关系不变，
                             //     这是insert重新染色的最最简单的情况
                    x.red = true; // 因为这种情况就是在树的某个末端添加节点，不会改变树的整体结构，对读线程使用红黑树搜索的搜索路径没影响
                else { // 其他情况下会有树的旋转的情况出现，当读线程使用红黑树方式进行查找时，可能会因为树的旋转，导致多遍历、少遍历节点，影响find的结果
                    lockRoot(); // 除了那种最最简单的情况，其余的都要加 写锁，让读线程用链表方式进行遍历读取
                    try {
                        root = balanceInsertion(root, x);
                    } finally {
                        unlockRoot();
                    }
                }
                break;
            }
        }
        assert checkInvariants(root);
        return null;
    }

 	// 基本是同jdk1.8的HashMap.TreeNode.removeTreeNode，仍然是从链表以及红黑树上都删除节点
    // 两点区别：1、返回值，红黑树的规模太小时，返回true，调用者再去进行树->链表的转化；2、红黑树规模足够，不用变换成链表时，进行红黑树上的删除要加 写锁
    final boolean removeTreeNode(TreeNode<K,V> p) {
        TreeNode<K,V> next = (TreeNode<K,V>)p.next;
        TreeNode<K,V> pred = p.prev;  // unlink traversal pointers
        TreeNode<K,V> r, rl;
        if (pred == null)
            first = next;
        else
            pred.next = next;
        if (next != null)
            next.prev = pred;
        if (first == null) {
            root = null;
            return true;
        }
        if ((r = root) == null || r.right == null || (rl = r.left) == null || rl.left == null) // too small
            return true;
        lockRoot();
        try {
            TreeNode<K,V> replacement;
            TreeNode<K,V> pl = p.left;
            TreeNode<K,V> pr = p.right;
            if (pl != null && pr != null) {
                TreeNode<K,V> s = pr, sl;
                while ((sl = s.left) != null) // find successor
                    s = sl;
                boolean c = s.red; s.red = p.red; p.red = c; // swap colors
                TreeNode<K,V> sr = s.right;
                TreeNode<K,V> pp = p.parent;
                if (s == pr) { // p was s's direct parent
                    p.parent = s;
                    s.right = p;
                }
                else {
                    TreeNode<K,V> sp = s.parent;
                    if ((p.parent = sp) != null) {
                        if (s == sp.left)
                            sp.left = p;
                        else
                            sp.right = p;
                    }
                    if ((s.right = pr) != null)
                        pr.parent = s;
                }
                p.left = null;
                if ((p.right = sr) != null)
                    sr.parent = p;
                if ((s.left = pl) != null)
                    pl.parent = s;
                if ((s.parent = pp) == null)
                    r = s;
                else if (p == pp.left)
                    pp.left = s;
                else
                    pp.right = s;
                if (sr != null)
                    replacement = sr;
                else
                    replacement = p;
            }
            else if (pl != null)
                replacement = pl;
            else if (pr != null)
                replacement = pr;
            else
                replacement = p;
            if (replacement != p) {
                TreeNode<K,V> pp = replacement.parent = p.parent;
                if (pp == null)
                    r = replacement;
                else if (p == pp.left)
                    pp.left = replacement;
                else
                    pp.right = replacement;
                p.left = p.right = p.parent = null;
            }
 
            root = (p.red) ? r : balanceDeletion(r, replacement);
 
            if (p == replacement) {  // detach pointers
                TreeNode<K,V> pp;
                if ((pp = p.parent) != null) {
                    if (p == pp.left)
                        pp.left = null;
                    else if (p == pp.right)
                        pp.right = null;
                    p.parent = null;
                }
            }
        } finally {
            unlockRoot();
        }
        assert checkInvariants(root);
        return false;
    }
    //下面几个方法我们在HashMap中已经做过详细介绍了，记不清可以回溯。
 	static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root, TreeNode<K,V> p);//左旋
    static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root, TreeNode<K,V> p);//右旋
    static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root, TreeNode<K,V> x);//插入平衡
    static <K,V> TreeNode<K,V> balanceDeletion(TreeNode<K,V> root, TreeNode<K,V> x);//删除平衡
    
     // 递归检查一些关系，确保构造的是正确无误的红黑树
    static <K,V> boolean checkInvariants(TreeNode<K,V> t);
 	// Unsafe相关的初始化工作
    private static final sun.misc.Unsafe U;
    private static final long LOCKSTATE;
    static {
        try {
            U = sun.misc.Unsafe.getUnsafe();
            Class<?> k = TreeBin.class;
            //实时获取锁的状态变化
            LOCKSTATE = U.objectFieldOffset(k.getDeclaredField("lockState"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }

```

#### 5.3.5 ReservationNode 保留节点（占位符）

也叫空节点，computeIfAbsent和compute这两个函数式api中才会使用。它的hash值固定为-3，就是个占位符，不会保存实际的数据，正常情况是不会出现的，在jdk1.8新的函数式有关的两个方法computeIfAbsent和compute中才会出现。

为什么需要这个节点？

因为正常的写操作，都会想对hash桶的第一个节点进行加锁，但是null是不能加锁，所以就要new一个占位符出来，放在这个空hash桶中成为第一个节点，把占位符当锁的对象，这样就能对整个hash桶加锁了。put/remove不使用ReservationNode是因为它们都特殊处理了下，并且这种特殊情况实际上还更简单，put直接使用cas操作，remove直接不操作，都不用加锁。但是computeIfAbsent和compute这个两个方法在碰见这种特殊情况时稍微复杂些，代码多一些，不加锁不好处理，所以需要ReservationNode来帮助完成对hash桶的加锁操作。

```
static final class ReservationNode<K,V> extends Node<K,V> {
    ReservationNode() {
        super(RESERVED, null, null, null);
    }
	// 空节点代表这个hash桶当前为null，所以肯定找不到“相等”的节点
    Node<K,V> find(int h, Object k) {
        return null;
    }
}
```

### 5.4 构造方法

cap乘以负载因子loadFactor应该大于等于initialCapcity，即是，cap >= initialCapcity /loadFactor，这里是loadFactor = 0.75
即是，cap >= initialCapcity * 4/3 = initialCapcity + (1/3) *initialCapcity
构造方法*ConcurrentHashMap(int initialCapacity)*里，是根据initialCapacity + (initialCapacity >>> 1) + 1确立的cap,

initialCapacity >>> 1 = initialCapacity * (1/2) > initialCapacity *(1/3)     满足要求,后面还有+1，应该是避免initialCapcity 为0，毕竟要有元素。

 最后代入到tableSizeFor(int)方法里，使得最后的cap为2的n次方，如果输入的是10，最后的结果是16，大于且最贴近输入值



```
public ConcurrentHashMap() {
}
//创建一个新的空容器，其初始表大小适应指定数量的元素，不需要动态调整大小。
//那么(cap*loadFactor)应该大于等于initialCapcity，即扩容阈值应该大于initialCapcity，这样在初始化的时候才不会触发动态扩容。
//initialCapacity + (initialCapacity >>> 1)相当于负载因子loadFactor=(2/3)> (3/4)默认的负载因子0.75.那么扩展阈值会更大，不容易触发动态扩容。
public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
            MAXIMUM_CAPACITY :
            tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1)); // 求 2^n，initialCapacity靠近2^n的右侧的值
    this.sizeCtl = cap;
}

public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
    this.sizeCtl = DEFAULT_CAPACITY;
    putAll(m);
}

public ConcurrentHashMap(int initialCapacity, float loadFactor) {
    this(initialCapacity, loadFactor, 1);
}
// concurrencyLevel只是为了此方法能够兼容之前的版本，它并不是实际的并发级别，loadFactor也不是实际的加载因子了
// 这两个都失去了原有的意义，仅仅对初始容量有一定的控制作用
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)// 检查参数
        throw new IllegalArgumentException();
    if (initialCapacity < concurrencyLevel)   // Use at least as many bins
        initialCapacity = concurrencyLevel;   // as estimated threads
    //上面有解释
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);
    int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
    this.sizeCtl = cap;
}
```

构造方法与HashMap对比可以发现，没有了HashMap中的threshold,loadFactor属性,而是改用了sizeCtl来控制,而且只存储了容量在里面，那么它是怎么用的呢？官方给出的解释如下：

（1）-1，表示有线程正在进行初始化操作

（2）-(1 + nThreads)，表示有n个线程正在一起扩容

（3）0，默认值，后续在真正初始化的时候使用默认容量

（4）> 0，初始化或扩容完成后下一次的扩容门槛

### 5.5 基本方法

```
Class<?> ak = Node[].class;
//返回数组类型的第一个元素的偏移地址(基础偏移地址)。如果arrayIndexScale方法返回的比例因子不为0，你可以通过结合基础偏移地址和比例因子访问数组的所有元素
ABASE = U.arrayBaseOffset(ak);
//返回数组类型的比例因子(其实就是数据中元素偏移地址的增量，因为数组中的元素的地址是连续的)。
int scale = U.arrayIndexScale(ak);
if ((scale & (scale - 1)) != 0)
	throw new Error("data type scale not a power of two");//数据类型规模不是2的幂
//Integer.numberOfLeadingZeros(scale); 表示scale转换成32位二进制数后，最高位的1前面的0的个数。
//单个数组成员相对前一个成员的偏移量
ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);//它的作用是把scale 转换为二进制左移的需要的位数。例如，4 相当于 << 2，8 相当于 << 3。
// volatile读取table[i]
@SuppressWarnings("unchecked")
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    // ((long)i << ASHIFT) + ABASE 相当于  i*scale + ABASE
    //((long)i << ASHIFT)是指tab[i]中第i个元素在相对于数组第一个元素的偏移量
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}

```



```
// CAS更新table[i]，也就是Node链表的头节点，或者TreeBin节点（它持有红黑树的根节点）
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
```



### 5.6 添加元素

```
public V put(K key, V value) {
    return putVal(key, value, false);
}

/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
	// hash扰动函数，跟1.8的HashMap的基本一样，& HASH_BITS用于把hash值转化为正数，负数hash是有特别的作用的
    int hash = spread(key.hashCode());
    // 要插入的元素所在桶的元素个数
    int binCount = 0;
     // 死循环，结合CAS使用（如果CAS失败，则会重新取整个桶进行下面的流程）
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
        	// 如果桶未初始化或者桶个数为0，则初始化桶
            tab = initTable();//5.7详细介绍
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) { // 如果要插入的元素所在的桶还没有元素，则把这个元素插入到这个桶中
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                // 如果使用CAS插入元素时，发现已经有元素了，则进入下一次循环，重新操作
                // 如果使用CAS插入元素成功，则break跳出循环，流程结束
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
        // 如果要插入的元素所在的桶的第一个元素的hash是MOVED(ForwardingNode节点，有线程正处在扩容中)，则当前线程帮忙一起迁移元素
            tab = helpTransfer(tab, f);//5.8详细介绍
        else {
            V oldVal = null;
            // 如果这个桶不为空且不在迁移元素，则锁住这个桶（分段锁，与JDK1.7的区别）
            // 并查找要插入的元素是否在这个桶中
            // 存在，则替换值（onlyIfAbsent=false）
            // 不存在，则插入到链表结尾或插入树中
            synchronized (f) {
                if (tabAt(tab, i) == f) { // 再次检测第一个元素是否有变化，如果有变化则进入下一次循环，从头来过
                    if (fh >= 0) {
                     // 如果第一个元素的hash值大于等于0（说明不是在迁移，也不是树）
                     // 那就是桶中的元素使用的是链表方式存储
                        binCount = 1; // 桶中元素个数赋值为1
                        for (Node<K,V> e = f;; ++binCount) { // 遍历整个桶，每次结束binCount加1
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                // 如果找到了这个元素，则赋值了新值（onlyIfAbsent=false）
                                // 并退出循环
                                oldVal = e.val;//保留原值
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                            	// 如果到链表尾部还没有找到元素
                                // 就把它插入到链表结尾并退出循环
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;//退出循环
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {//当前桶的第一个节点是红黑树对象
                        Node<K,V> p;
                        binCount = 2; // 桶中元素个数赋值为2
                        // 调用红黑树的插入方法插入元素
                        // 如果成功插入则返回null
                        // 否则返回寻找到的节点
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {//5.3.4详细介绍
                            oldVal = p.val;
                            // 如果找到了这个元素，则赋值了新值（onlyIfAbsent=false）
                            // 并退出循环
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            //桶中的元素个数不为0
            if (binCount != 0) {
             	// 如果链表元素个数达到了8，则尝试树化
                // 因为上面把元素插入到树中时，binCount只赋值了2，并没有计算整个树中元素的个数，所以不会重复树化（容易忽略点）
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;// 退出外层大循环，流程结束
            }
        }
    }
    // 成功插入元素，元素个数加1（是否要扩容在这个里面）
    addCount(1L, binCount);
     // 成功插入元素返回null
    return null;
}
```

### 5.7 初始化容量

```
真正的初始化方法，使用保存在sizeCtl中的数据作为初始化容量
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)// 如果sizeCtl<0说明正在初始化或者扩容，让出CPU
       		//真正的初始化是要禁止并发的，保证tables数组只被初始化一次，但是又不能切换线程，所以用yeild()暂时让出CPU
            Thread.yield(); // lost initialization race; just spin，失去了初始化竞赛;只是自旋
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {//CAS更新sizeCtl标识为 "初始化" 状态
        	// 如果把sizeCtl原子更新为-1成功，则当前线程进入初始化
            // 如果原子更新失败则说明有其它线程先一步进入初始化了，则进入下一次循环
            // 如果下一次循环时还没初始化完毕，则sizeCtl<0进入上面if的逻辑让出CPU
            // 如果下一次循环更新完毕了，则table.length!=0，退出循环
            try {
            	// 检查table数组是否已经被初始化，没初始化就真正初始化
            	// 再次检查table是否为空，防止ABA问题
                if ((tab = table) == null || tab.length == 0) {
                 // 如果sc为0则使用默认值16
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    // 设置sc为数组长度的0.75倍
                    // n - (n >>> 2) = n - n/4 = 0.75n
                    // 可见这里装载因子和扩容门槛都是写死了的
                    // 这也正是没有threshold和loadFactor属性的原因
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

**ABA问题**：

ABA问题是指在CAS操作中带来的潜在问题

比如两个线程

- 线程1 查询A的值为a，与旧值a比较，
- 线程2 查询A的值为a，与旧值a比较，相等，更新为b值
- 线程2 查询A的值为b，与旧值b比较，相等，更新为a值
- 线程1 相等，更新B的值为c

可以看到这样的情况下，线程1 可以正常 进行CAS操作，将值从a变为c 但是在这之间，实际A值已经发了a->b  b->a的转换

仔细思考，这样可能带来的问题是，如果需要关注A值变化过程，是会漏掉一段时间窗口的监控

今天偶然看到一个ABA问题可能带来的问题

小明在提款机，提取了50元，因为提款机问题，
有两个线程，同时把余额从100变为50

线程1（提款机）：获取当前值100，期望更新为50，

线程2（提款机）：获取当前值100，期望更新为50，

线程1成功执行，线程2某种原因block了，这时，某人给小明汇款50



线程3（默认）：获取当前值50，期望更新为100，



这时候线程3成功执行，余额变为100，



线程2从Block中恢复，获取到的也是100，compare之后，继续更新余额为50！！！



此时可以看到，实际余额应该为100（100-50+50），但是实际上变为了50（100-50+50-50）

这就是ABA问题带来的成功提交

### 5.8 协助迁移元素（ForwardingNode）

```
/**
 * Helps transfer if a resize is in progress.如果有线程在扩容中，那么当前线程协助迁移元素
 */
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
    Node<K,V>[] nextTab; int sc;
    // 如果桶数组不为空，并且当前桶第一个元素为ForwardingNode类型，并且nextTab不为空
    // 说明当前桶已经迁移完毕了，才去帮忙迁移其它桶的元素
    // 扩容时会把旧桶的第一个元素置为ForwardingNode，并让其nextTab指向新桶数组
    if (tab != null && (f instanceof ForwardingNode) &&
        (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
        int rs = resizeStamp(tab.length);// 计数本次扩容的生成戳
        // 在判断一次是否正在执行扩容（这几个变量的值改变了，说明此次扩容结束了），sizeCtl<0，说明正在扩容
        while (nextTab == nextTable && table == tab &&
               (sc = sizeCtl) < 0) {
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                sc == rs + MAX_RESIZERS || transferIndex <= 0) // 判断下是否能真正帮助此次扩容（5.10集中说明）
                break;// 不能帮助，直接结束
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) { // 不满足前面4个条件时，尝试参与此次扩容，把正在执行transfer任务的线程数加1
                transfer(tab, nextTab);// 当前线程帮忙迁移元素，5.9详细介绍
                break;
            }
        }
        return nextTab;// 返回新数组
    }
    return table;// 返回新数组（执行这句说明一开始判断，就发现变量变化了，表明扩容已经结束了，table会被别的线程赋值为新数组）
}
	
```



### 5.9 迁移元素 

1.8的扩容可以多线程一起完成，因此扩容变得复杂了，但是效率提升了。这部分的内容比较多，分几点说下

#### 5.9.1一个transfer任务

对于一个大任务拆分成多个小任务供多线程执行，一般都要求这些小任务具有相似性，流程一致，并且很重要的一点，任务之间的相互影响尽量少。那么在扩容之中，是怎么划分这个任务的呢？

一般在扩容过程中分为2个步骤：

步骤一：创建一个2倍大小的数组，这个过程要求单线程完成，（多线程创建几个数组没有意义，容易出错），这个步骤每个HashMap、ConcurrentHashMap基本是都是同一套这里不再阐述。

步骤二：执行节点迁移，也就是rehash的过程，将旧的数组中的所有节点put到新的数组中。在JDK1.8的HashMap中,进行 n -> 2n 的扩容时，扩容前节点所在的hash桶的索引为index，这个节点迁移到新数组中只会有两种情况：在新数组的index位置，要么在新数组的index+n的位置。具体分配可以参考HashMap扩容过程。由此可以看出，旧table的每个桶位的节点在迁移过程中是互不影响的，那么这个特点对多线程的扩容是非常友好的。那么，每个桶的迁移都可以作为一个线程在扩容的时候的一个transfer任务。另外，每个线程要任务都不应该规模太小，因为扩容并不是IO型操作，节点迁移的执行速度本身很快，太多的线程来执行节点迁移，线程调度开销占比变大，反而降低了吞吐量。ConcurrentHashMap这里，会根据CPU的核心数目，来算出一个transfer任务包含的hash桶的数量。

```
if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
    stride = MIN_TRANSFER_STRIDE; // subdivide range
```

为了好说明，下面统一使用最小值MIN_TRANSFER_STRIDE，16，也就是1个线程的一次transfer任务要负责迁移16个hash桶。

#### 5.9.2 transfer任务的申请

在经典的生成者消费者模式中，会有一个任务队列。消费者向任务队列申请任务，申请到任务时，队列中元素数量减1，当任务队列中元素数量为0时，不能再申请任务。ConcurrentHashMap这里并没有使用额外的任务队列，因为table数组本身就可以当作是一个队列。上面已经说了，一个transfer任务 要负责迁移stride个hash桶,最简单的设计是16个hash桶都是连续的。为了记录还有多少个任务，使用了一个类变量transferIndex，可以把这个看成是任务队列的size，每一次申请任务，这个size减1。

迭代操作的下标是从小往大，也就是正向，为了减少扩容时的transfer和迭代的冲突，transfer使用反向，也就是下标从大到小。

多线程情况下，迭代和扩容同时进行时，顺序相反时，二者相遇过后，迭代没处理的都是已经transfer的hash桶，transfer没处理的，都是已经迭代的hash桶，冲突会变少。所以，任务队列的size减1，翻译过来就是，在table数组中的索引减stride。这个索引就是 transferIndex，用于标记整体的transfer进行到了哪里。因为transfer个数，从1开始，因此transferIndex也是从1开始，下标在[transferIndex - 实际的stride（下界要 >= 0）, transferIndex - 1]内的hash桶，就是每个transfer的任务区间。transferIndex <= 0 时，代表没有任务可以申请，此时无法帮助扩容。注意，NCPU不一定是2^n，因此最后一个任务中的hash桶的数量可能不足stride个，此时只执行余下的数量。

为了保证每个任务只被领取一次，transferIndex递减是用CAS操作完成的。

特殊情况下，会出现多线程扩容重叠，此时某个transfer任务虽然被领取了，但是却不能被执行，会被作废。因为transfer方法的代码中有考虑任务作废的情况。但是根据下面5.10的分析，扩容重叠这种特殊情况是有一个机制来避免的。

![image-20210110201130438](https://gitee.com/zhouxiaoliang/img/raw/master/img/image-20210110201130438.png)



```
执行节点迁移，准确地说是迁移内容，因为很多节点都需要进行复制，复制能够保证读操作尽量不受影响
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
      // 计算每个transfer任务中要负责迁移多少个hash桶
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
     // 如果nextTab为空，说明还没开始迁移
     // 就新建一个新桶数组
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")   // 新桶数组是原桶的两倍
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME   处理内存不足导致的OOM，以及table数组超过最大长度，这两种情况都实际上无法再进行扩容了
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        // 表明此时执行扩容的线程可以开始申请transfer任务了
        transferIndex = n;
    }
     // 新桶数组大小
    int nextn = nextTab.length;
     // 新建一个ForwardingNode类型的节点，并把新桶数组存储在里面
     // 转发节点，在旧数组的一个hash桶中所有节点都被迁移完后，放置在这个hash桶中，表明已经迁移完，对它的读操作会转发到新数组
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    //预处理标记
    boolean advance = true;
     // 扩容中收尾的线程把做个值设置为true，进行本轮扩容的收尾工作（两件事，1.重新检查一次所有hash桶，2.给属性赋新值）
    boolean finishing = false; // to ensure sweep before committing nextTab
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        // 整个while循环就是在算i的值
        // i的值会从n-1依次递减
        // 其中n是旧桶数组的大小，也就是说i从15开始一直减到1这样去迁移元素
        // while中的代码可以看成是预处理
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing) // 一次transfer任务还没有执行完毕
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {// transfer任务已经没有了，表明可以准备退出扩容了
                i = -1;
                advance = false;
            }
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {// 尝试申请一个transfer任务
                // 申请到任务后标记自己的任务区间
                bound = nextBound;
                //任务数减一
                i = nextIndex - 1;
                advance = false;
            }
        }
         // 这个分支中有处理 扩容重叠，5.10中会分析，到这里应该是不会出现扩容重叠的
         // i < 0 表明本次的transfer任务已经执行完毕了，此时需要准备退出这个方法
         // i >= n 表明扩容轮次跟预想的不一样（比如这个线程预想的是进行n -> 2n的扩容，实际nextTab是4n数组），此时不能进行节点迁移
          		   虽然申请到了任务，但是也不能执行，应该准备退出方法，此次任务作废，别的线程也不能领取了，只能让此轮扩容中最后一个线程在重新检查时处理掉
         // i + n >= nextn，此时前面两个条件为false，那么就有 0 < i < n，也就是 n < i + n < 2n，这个是一定成立的
         //     因为nextn最小也是2n，i + n 怎么也比2n小，所以我认为这个情况应该和i>=n属于同一种扩容情况。
 		 // 如果一次遍历完成了
         // 也就是整个map所有桶中的元素都迁移完成了
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
             // 如果全部迁移完成了，则替换旧桶数组
             // 并设置下一次扩容门槛为新桶数组容量的0.75倍
             // 执行本轮扩容的收尾工作
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {// 尝试把正在执行扩容的线程数减1，表明自己要退出扩容
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT) // 判断下自己是不是本轮扩容中的最后一个线程，如果不是，则直接退出。
                    return;
                // 如果自己是本轮扩容中的最后一个线程，那么要准备执行收尾工作，finishing为true才会走到上面的if条件
                finishing = advance = true;
                // recheck before commit 最后一个扩容的线程要重新检查一次旧数组的所有hash桶，看是否是都被正确迁移到新数组了。
                // 正常情况下，重新检查时，旧数组所有hash桶都应该是转发节点，此时这个重新检查的工作很快就会执行完。
                // 特殊情况，比如扩容重叠，那么会有线程申请到了transfer任务，但是参数错误（旧数组和新数组对不上，不是2倍长度的关系），
                // 此时这个线程领取的任务会作废，那么最后检查时，还要处理因为作废而没有被迁移的hash桶，把它们正确迁移到新数组中
 				// i重新赋值为n
                // 这样会再重新遍历一次桶数组，看看是不是都迁移完成了
                // 也就是第二次遍历都会走到下面的(fh = f.hash) == MOVED这个条件
                i = n; // recheck before commit
            }
        }
        else if ((f = tabAt(tab, i)) == null)   // 如果桶中无数据，直接放入ForwardingNode标记该桶已迁移
            advance = casTabAt(tab, i, null, fwd);
        else if ((fh = f.hash) == MOVED) 
        	// 如果桶中第一个元素的hash值为MOVED
            // 说明它是ForwardingNode节点
            // 也就是该桶已迁移
            advance = true; // already processed
        else {
            synchronized (f) {// 锁定该桶并迁移元素
            	// 再次判断当前桶第一个元素是否有修改
                // 也就是可能其它线程先一步迁移了元素
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    if (fh >= 0) {
                    	// 下面这段代码，使用高低位，跟1.6/1.7的使用 & 的效果基本一样
                    	// 第一个元素的hash值大于等于0
                        // 说明该桶中元素是以链表形式存储的
                        // 这里与HashMap迁移算法基本类似
                        // 唯一不同的是多了一步寻找lastRun
                        // 这里的lastRun是提取出链表后面不用处理再特殊处理的子链表
                        // 比如所有元素的hash值与桶大小n与操作后的值分别为 0 0 4 4 0 0 0
                        // 则最后后面三个0对应的元素肯定还是在同一个桶中
                        // 这时lastRun对应的就是倒数第三个节点
                        // 尽量重用Node链表尾部的一部分（起码能重用一个，实际情况下能重用比较多的节点，这时候就提高了效率）
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                       // 重用的是“低位”
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {// 重用的是“高位”

                            hn = lastRun;
                            ln = null;
                        }
                        // 遍历链表，把hash&n为0的放在低位链表中
                        // 不为0的放在高位链表中
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                         // 低位链表的位置不变
                        setTabAt(nextTab, i, ln);
                        // 高位链表的位置是原位置加n
                        setTabAt(nextTab, i + n, hn);
                        // 把旧table的hash桶中放置转发节点，表明此hash桶已经被处理
                        setTabAt(tab, i, fwd);
                        // advance为true，返回上面进行--i操作
                        advance = true;
                    }
                    else if (f instanceof TreeBin) {
                      	// 红黑树的情况，先使用链表的方式遍历，复制所有节点，根据高低位（1.8的HashMap中的做法)，
                        // 组装成两个链表，然后看下是否需要进行红黑树变换，最后放在新数组对应的hash桶中

                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                          // 遍历整颗树，根据hash&n是否为0分化成两颗树
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        // 如果分化的树中元素个数小于等于6，则退化成链表
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                            (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                            (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        // 低位树的位置不变    
                        setTabAt(nextTab, i, ln);
                        // 高位树的位置是原位置加n
                        setTabAt(nextTab, i + n, hn);
                         // 标记该桶已迁移
                        setTabAt(tab, i, fwd);
                          // advance为true，返回上面进行--i操作
                        advance = true;
                    }
                }
            }
        }
    }
}
```

### 5.10 resizeStamp以及扩容重叠相关

resizeStamp以下简称rs，下面是相关常量和变量

```
/**
 * The number of bits used for generation stamp in sizeCtl.
 * Must be at least 6 for 32bit arrays. 用于生成每次扩容都唯一的生成戳的数，最小是6
 */
private static int RESIZE_STAMP_BITS = 16;

/**
 * The maximum number of threads that can help resize.
 * Must fit in 32 - RESIZE_STAMP_BITS bits.可以帮助扩容的最大线程数。
 */
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

/**
 * The bit shift for recording size stamp in sizeCtl
 * 移位量，把生成戳移位后保存在sizeCtl中当做扩容线程计数的基数，相反方向移位后能够反解出生成戳
 */
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

	/**
     * Returns the stamp bits for resizing a table of size n.
     * Must be negative when shifted left by RESIZE_STAMP_SHIFT.
     */
    
// 返回与扩容有关的一个生成戳rs，每次新的扩容，都有一个不同的n，这个生成戳就是根据n来计算出来的一个数字，n不同，这个数字也不同
// 但是b = 32时MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1 = 0
    static final int resizeStamp(int n) {
        return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
    }
```

Integer.numberOfLeadingZeros(n)用于计算n转换成二进制后前面有几个0。这个有什么作用呢？
首先ConcurrentHashMap的容量必定是2的幂次方，所以不同的容量n前面0的个数必然不同，这样可以保证是在原容量为n的情况下进行扩容。

假设n = 2^x，那么有 Integer.numberOfLeadingZeros(n) = 31 - x < 32

```
0000 0000 0000 0000 0000 0000 0000 0000 ~ 0000 0000 0000 0000 0000 0000 0001 1111
0~31
```

可以记录扩容值。

(1 << (RESIZE_STAMP_BITS - 1)即是1<<15（默认情况），表示为二进制即是高16位为0，第16位为1：

```
0000 0000 0000 0000 1000 0000 0000 0000
```

因为RESIZE_STAMP_BITS 最小是6，因此这个运算，在不考虑变成负数的情况下，最小是 1<< 5 = 32，二进制最低的5位都是0,。

```
0000 0000 0000 0000 0000 0000 0010 0000
```

进行 | 运算后，即Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1)) ，可以知道二者的二进制不重叠，可以确定，n不同时，每个rs也不同。

所以resizeStamp(n)的返回值为：高16位置0，第16位为1，低15位存放当前容量n，用于表示是对n的扩容。

rs与RESIZE_STAMP_SHIFT配合可以求出新的sizeCtl的值，分情况如下：

1. sc < 0
   已经有线程在扩容，将sizeCtl+1并调用transfer()让当前线程参与扩容。
2. sc >= 0
   表示没有线程在扩容，使用CAS将sizeCtl的值改为(rs << RESIZE_STAMP_SHIFT) + 2。
   rs即resizeStamp(n)，记temp=rs << RESIZE_STAMP_SHIFT。如当前容量为8时rs的值：

```
//rs
0000 0000 0000 0000 1000 0000 0000 1000
//temp = rs << RESIZE_STAMP_SHIFT，即 temp = rs << 16，左移16后temp最高位为1，所以temp成了一个负数。
1000 0000 0000 1000 0000 0000 0000 0000
//sc = (rs << RESIZE_STAMP_SHIFT) + 2)
1000 0000 0000 1000 0000 0000 0000 0010

```

那么在扩容时sizeCtl值的意义：高15位为容量n ，低16位为并行扩容线程数+1



=================================================分割线==================================================================

rs是跟n绑定的，不同的n有不同的rs，并且rs能够反解出唯一的n，这里不能直接使用rs，因为 sizeCtl 大于0时表示的是扩容阈值，因此需要把rs处理成负数，并且还要保证不同扩容之间不会出现相同的sizeCtl。这两个是依靠移位操作 rs << RESIZE_STAMP_SHIFT 来完成的。

以下情况中都不考虑 RESIZE_STAMP_BITS = 32的情况，RESIZE_STAMP_BITS实际可以认为是常量，下面将RESIZE_STAMP_BITS当做b

假设正在执行扩容的线程数量为nThreads，令b = RESIZE_STAMP_BITS，table.length = 1 << x，其中6 <= b <=32，1 <= x <= 30 ，那么有：

RESIZE_STAMP_SHIFT = 32 - b，rs = (31 - x) | (1 << (b - 1))，

**sizeCtl** = sc = (rs << (32 - b)) + 1 + nThreads  （后面都用sc，看下面的addCount的代码，sc > 0时，尝试成为第一个扩容的线程的代码，+2表示有一个，+1表示有0个），为什么会是这个表达式，请查看5.2.2 sizeCtl的解释。

**MAX_RESIZERS** = (1 << (32 - b)) - 1 >= nThreads；



-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

b >= 6，那么(1 << (b-1)) >= 32，并且它的二进制中只有一位是1，(32 - b) + (b - 1) = 31 < 32，

 rs = (31 - x) | (1 << (b - 1)) 中的 或运算 可以转化为加法（因为不会进位），rs的移位运算可以转化为两部分移位的和 ，所以移位运算



(rs << (32 - b))  =  ((31 - x) << (32 - b))   +   (1 << (b - 1))<<(32 - b)；

(rs << (32 - b))  =  ((31 - x) << (32 - b))   +   (1 << ((b - 1)+(32 - b)))；

(rs << (32 - b))  =  ((31 - x) << (32 - b))   +   (1 << 31)；



31 - x < 32 = (1 << 5)，32 - b <= 26，

((31 - x) << (32 - b))   <    (1 << 5)<< (32 - b)

((31 - x) << (32 - b))   <    (1 <<( 5+(32 - b))==>最大1<<31

(1 << 31) = Integer.MIN_VALUE， 所以(rs << (32 - b))是负数，满足resizeStamp的设计；





sc = (rs << (32 - b)) + 1 + nThreads = ((31 - x) << (32 - b)) + (1 << 31) + 1 + nThreads，

因为32 - x < (1 << 5)，32 - b <=26，所以 ((32 - x) << (32 - b)) 是正数，并且绝对值小于 (1 << 31)， sc 不可能出现负数溢出变正数的情况，此时sc < 0；

nThreads取最小值0时，很明显也不可出现负数溢出，因此sc < 0；



结论：扩容完成前，使用rs时，sc总是负数，保证了基本的有效性；

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

sc反解出rs，通过反向移位

sc >>> RESIZE_STAMP_SHIFT = sc >>> (32 - b)，

右移位是把操作数当做无符号数进行普通的右移位，负数移位会变成正数，sc < 0，

在加法不产生进位时，把和的移位当做移位的和，那么有：

nThreads = 0时，

(sc >>> (32 - b)) = (rs << (32 - b)) + 1) >> (32 - b) 

(sc >>> (32 - b)) = ((31-x)+(1<<(b-1)) ) << (32 - b)) + 1) >> (32 - b) 

(sc >>> (32 - b)) = (((31-x) << (32 - b))+(1<<31)) + 1) >> (32 - b) 

(sc >>> (32 - b)) = ((31-x)+(1<<(b-1))) + 0= rs

正常情况（不发生扩容重叠）一定有 (sc >>> RESIZE_STAMP_SHIFT) >= rs；

nThreads = MAX_RESIZERS时，

MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

rs = (31-x)|(1<<(b-1)) ==>(31-x)+(1<<(b-1))

sc = (rs << (32 - b)) + 1 + nThreads

sc= (((31-x)+(1<<(b-1)))<<(32-b)) + 1 +  (1 << (32 - b)) - 1

sc=((31 - x) << (32 - b))  + ((1<<(b-1))<<(32-b))  + 1 +  (1 << (32 - b)) - 1

sc=((31 - x) << (32 - b))+   (1<<31) + 1 +  (1 << (32 - b)) - 1

(sc >>> (32 - b)) = (((31 - x) << (32 - b)) + (1 << 31) + 1 + (1 << (32 - b)) - 1) >>> (32 - b)，

(sc >>> (32 - b)) = (((31 - x) << (32 - b)) >>>(32-b) +  (1 << 31)>>>(32-b)   + (1 << (32 - b))>>>(32-b) )

(sc >>> (32 - b)) = ((31 - x)  +  (1 << (b-1))   + 1 ) = rs+1

sc>>> RESIZE_STAMP_SHIFT  = rs+1

因此正常情况（不发生扩容重叠）下一定有(sc >>> RESIZE_STAMP_SHIFT) <= rs + 1，当且仅当扩容线程为最大数时取等号；

------

再分析下 sc 的在每轮不同的扩容中的唯一性（这里没证明单调性）

sc 的值域为 [ ((31 - x) << (32 - b)) + (1 << 31) + 1, ((31 - x) << (32 - b)) + (1 << 31) + (1 << (32 - b)) ]；



另 x = x + 1，也就是进行下一轮扩容，那么 sc 的值域变为

 [ ((30 - x) << (32 - b)) + (1 << 31) + 1, ((30 - x) << (32 - b)) + (1 << 31) + (1 << (32 - b)) ]；

加法不进位时把移位的和当作和的移位，上界进行移位合并，那么x + 1时的值域为

 [ ((30 - x) << (32 - b)) + (1 << 31) + 1, ((31 - x) << (32 - b)) + (1 << 31)) ]；

很明显，x+1时上界，比x时的下界小1，所以两个值域中sc不可能相等；

这也就是说，n -> 2n扩容中的sc的任何可能取值，总是大于 2n -> 4n时sc的任何可能取值，因此sizeCtl具有扩容唯一性，每轮扩容的sizeCtl，都不可能和别的轮次的扩容的sizeCtl相同



那么 sc >>> RESIZE_STAMP_SHIFT 呢，这个就有点特殊了
x时，移位后的值域为 [ (31 - x) + (1 << (b - 1)), (32 - x) + (1 << (b - 1))]；
x+1时，移位后的值域为 [ (30 - x) + (1 << (b - 1)), (31 - x) + (1 << (b - 1))];
两者有一个重叠点。这时可能会绕过(sc >>> RESIZE_STAMP_SHIFT) != rs这个条件的检验，但是此时sc是不一样的，后面CAS更新时，依然不能使用旧的sc的值。



综合上面所说的，resizeStamp这个机制，能够避免sc = sizeCtl在不同轮次的扩容中出现ABA问题，进而避免 扩容重叠 问题出现。
特别的一点，b = 32这种极端情况下，resizeStamp机制会受到影响，并且MAX_RESIZERS = 0。那么这个b = 32应该是不会发生的，但是5个条件的if中又有处理b = 32的情况，又是一处代码跟实际不太对的上的情况。

------

**<5个条件的if>**，也就是这句 if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 || sc == rs + MAX_RESIZERS || (nt = nextTable) == null || transferIndex <= 0)

1.(nt = nextTable) == null：表示整个扩容过程已经结束，或者扩容过程处于一个单线程的阶段（transfer方法中创建nextTable是由单线程完成的），此时不能帮助扩容。

2.transferIndex <= 0：表示所有的transfer任务都被领取光了，没有剩余的hash桶给自己这个线程来transfer，此时线程不能再帮助扩容了。

3.(sc >>> RESIZE_STAMP_SHIFT) != rs：上面的那个证明中，有两个结论，

​	“正常情况（不发生扩容重叠）下一定有(sc >>> RESIZE_STAMP_SHIFT) <= rs + 1，当且仅当扩容线程为最大数时取等号”，

​	“正常情况（不发生扩容重叠）下一定有 (sc >>> RESIZE_STAMP_SHIFT) >= rs ”

​	rs + 1时是个特殊情况，此时扩容线程为最大值，是一定不能帮助扩容的。其他情况下等价于“正常情况（不发生扩容重叠）下一定有(sc >>> 						RESIZE_STAMP_SHIFT) == rs ”

​	所以它可以看成是一个个恒等式，正常情况下if中的这个判断条件一定为false。为true则表明sc 和 tab.length 对不上，不能互相反解，继续下去有可能发生扩容	重叠。因此为true时也不能帮助扩容，这个条件放在第一位置也是有道理的，它最先避免了扩容重叠的发生。

4.sc == rs + 1

​	第一次分配transfer任务时，把正在扩容线程数加1的操作是U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2)， +2代表有1个，+1代表有0个。再根据上下文，可以认为是此时此刻transfer任务都被执行完成了，本轮扩容操作实际上已经结束了。

5. sc == rs + MAX_RESIZERS,

   

   所以，目前ABC都有了合理解释，4 和 5 在目前的情况下，不能给出太合理的解释，只能认为它们两个是为了处理极端的 RESIZE_STAMP_BITS = 32的情况。在实际情况中，没有提供途径去修改 RESIZE_STAMP_BITS 的值，这样4和5一定是false。在这个层面上，可以认为它们两个没啥用

   

   https://bugs.java.com/bugdatabase/view_bug.do?bug_id=JDK-8214427 此链接说明了这是个bug

   在上述代码中，条件(sc == rs + 1 || sc == rs + MAX_RESIZERS)永远不会为真，因为rs的值是正的，sc的值是负的

   正确的条件应该是(sc >>> RESIZE_STAMP_SHIFT) == rs + 1 || (sc >>> RESIZE_STAMP_SHIFT) == rs + MAX_RESIZERS，这可以用来判断进程的大小调整是否完成或线程的大小调整是否达到最大值限制

   

   

   

### 5.11 扩容方法

```
/**
 * Adds to count, and if table is too small and not already
 * resizing, initiates transfer. If already resizing, helps
 * perform transfer if work is available.  Rechecks occupancy
 * after a transfer to see if another resize is already needed
 * because resizings are lagging additions.
 *
 * @param x the count to add
 * @param check if <0, don't check resize, if <= 1 only check if uncontended
 */
//添加到count中，如果数组太小并且还没有扩容，则启动transfer。
//如果已经扩容，则在可运行的情况下帮助执行transfer。
//在一次transfer后重新检查占用情况，看看是否已经需要另一次扩容，因为扩容是滞后于添加的。
上面是原始翻译

/**
 * 更改计数值，这部分相关是仿造LongAdder实现的，已经说过了
 * 检查是否触发了扩容，是否正在扩容，是否可以帮助扩容
 * 并且还要检查是否会触发下一次扩容，因为更改计数值的操作是不在加锁区域内的，扩容过程中可能还有别的线程添加了很多K-V
 * 参数check，用于指示计数操作是否会触发扩容，check < 0 代表一定不会触发，
 *    check <= 1时，只在没有计数时线程竞争才会触发扩容，check > 0 时，也表示的是hash桶中节点的数目
 * 普通的put可能会触发，Map拷贝构造中的putAll，因为事先扩容了，所以这个putAll不会触发扩容
 */
private final void addCount(long x, int check) {
	// 这里使用的思想跟LongAdder类是一模一样的（后面会讲）
    // 把数组的大小存储根据不同的线程存储到不同的段上（也是分段锁的思想）
    // 并且有一个baseCount，优先更新baseCount，如果失败了再更新不同线程对应的段
    // 这样可以保证尽量小的减少冲突
    CounterCell[] as; long b, s;
    if ((as = counterCells) != null ||
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {// 先尝试把数量加到baseCount上，如果失败再加到分段的CounterCell上
        CounterCell a; long v; int m;
        boolean uncontended = true;
        // 如果as为空或者长度为0
        // 或者当前线程所在的段为null
        // 或者在当前线程的段上加数量失败
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            !(uncontended =
              U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            // 强制增加数量（无论如何数量是一定要加上的，并不是简单地自旋）
            // 不同线程对应不同的段都更新失败了
            // 说明已经发生冲突了，那么就对counterCells进行扩容
            // 以减少多个线程hash到同一个段的概率
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)// 执行到这里，说明线程更新计数值时没有遇到线程竞争（cells != null已经被初始化），
                        //     check == 1时表示hash桶中原本只有一个节点，规模比较小，这次添加先不扩容
            return;
         // 计算元素个数
        s = sumCount();
    }
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        // 如果元素个数达到了扩容门槛，则进行扩容
        // 注意，正常情况下sizeCtl存储的是扩容门槛，即容量的0.75倍
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);  // rs是扩容时的一个邮戳标识
            if (sc < 0) { // sc<0说明正在扩容中
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    // 扩容已经完成了，退出循环
                    // 根据前面第3点分析的，这5个条件中主要有一个为true，就说明当前线程不能帮助此次扩容
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) 不满足前面5个条件时，尝试参与此次扩容，把正在执行transfer任务的线程数加1
                    transfer(tab, nt); // 去帮助执行transfer任务
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2)) // 试着让自己成为第一个执行transfer任务的线程
                transfer(tab, null);
            s = sumCount();
            // 重新计数，判断是否需要开启下一轮扩容
            // sizeCtl的高16位存储着rs这个扩容邮戳
            // sizeCtl的低16位存储着扩容线程数加1，即(1+nThreads)
            // 上面两个进入transfer方法的地方，都是把sizeCtl自增，这一点足够说明sizeCtl的英文注释表达的意思有误
            // 如果是 -(1 + nThreads) 表示，那么应该用减1，实际情况用的是加1
            // 代码中的加2，是因为逻辑中是用(rs << RESIZE_STAMP_SHIFT) + 1代表现在有0个线程
            // 下面的transfer方法中退出方法前的操作，也足够说明“sizeCtl注释错误”这一点
        }
    }
}
```

#### 5.11.1 计数操作

1.7及以前的ConcurrentHashMap中使用了Segment，Segment能够分担所有的针对单个K-V的写操作，包括put/replace。并且Segment自带一些数据，比如Segment.count，用于处理Map的计数要求，这样就可以像put/repalce一样，分担整个Map的并发计数压力。

但是1.8中没有再使用Segment来完成put/replace，虽然还是利用了锁分段的思想，但是使用的是自带的synchronized锁住hash桶中的第一个节点，没有新增别的数据。

因此计数操作被排除在外了，它无法享受synchronized实现的变种分段锁带来的高效率，单独使用一个Map.size来计数，线程竞争可能会很大，比使用Segment是效率低很多。

为了处理这个问题，jdk1.8中使用了一个仿造LongAdder实现的计数器，让计数操作额外使用别的基于分段并发思想的实现的类。ConcurrentHashMap中不直接使用LongAdder，而是自己拷贝代码实现一个内部的，主要为了方便。LongAdder的实现本身代码不是特别多，ConcurrentHashMap中的实现，基本和LongAdder一样，可以直接看做是LongAdder。

#### 5.11.2 LongAdder

LongAdder是jdk8新增的用于并发环境的计数器，目的是为了在高并发情况下，代替AtomicLong/AtomicInt，成为一个用于高并发情况下的高效的通用计数器。

高并发下计数，一般最先想到的应该是AtomicLong/AtomicInt，AtmoicXXX使用硬件级别的指令 CAS 来更新计数器的值，这样可以避免加锁，机器直接支持的指令，效率也很高。但是AtomicXXX中的 CAS 操作在出现线程竞争时，失败的线程会白白地循环一次，在并发很大的情况下，因为每次CAS都只有一个线程能成功，竞争失败的线程会非常多。失败次数越多，循环次数就越多，很多线程的CAS操作越来越接近 自旋锁（spin lock）。计数操作本来是一个很简单的操作，实际需要耗费的cpu时间应该是越少越好，AtomicXXX在高并发计数时，大量的cpu时间都浪费会在 自旋 上了，这很浪费，也降低了实际的计数效率。

```

// jdk1.8的AtomicLong的实现代码，这段代码在sun.misc.Unsafe中
// 当线程竞争很激烈时，while判断条件中的CAS会连续多次返回false，这样就会造成无用的循环，循环中读取volatile变量的开销本来就是比较高的
// 因为这样，在高并发时，AtomicXXX并不是那么理想的计数方式
public final long getAndAddLong(Object var1, long var2, long var4) {
    long var6;
    do {
        var6 = this.getLongVolatile(var1, var2);
    } while(!this.compareAndSwapLong(var1, var2, var6, var6 + var4));

    return var6;
}
```

说LongAdder比在高并发时比AtomicLong更高效，这么说有什么依据呢？

LongAdder是根据ConcurrentHashMap这些为并发设计的类的基本原理——锁分段，来实现的。它里面维护一组按需分配的计数单元，并发计数时，不同的线程可以在不同的计数单元上进行计数，这样减少了线程竞争，提高了并发效率。本质上是用空间换时间的思想，不过在实际高并发情况中消耗的空间可以忽略不计。现在，在处理高并发计数时，应该优先使用LongAdder，而不是继续使用AtomicLong。当然，线程竞争很低的情况下进行计数，使用Atomic还是更简单更直接，并且效率稍微高一些。其他情况，比如序号生成，这种情况下需要准确的数值，全局唯一的AtomicLong才是正确的选择，此时不应该使用LongAdder。

##### 5.11.2.1 类关系

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/LongAdder.png)

公共父类Striped64是实现中的核心，它实现一些核心操作，处理64位数据，很容易就能转化为其他基本类型，是个通用的类。

二元算术运算累积，指的是你可以给它提供一个二元算术方式，这个类按照你提供的方式进行算术计算，并保存计算结果。二元运算中第一个操作数是累积器中某个计数单元当前的值，另外一个值是外部提供的。

https://baike.baidu.com/item/%E4%BA%8C%E5%85%83%E8%BF%90%E7%AE%97/748189?fr=aladdin（二元运算）

举几个例子：

假设每次操作都需要把原来的数值加上某个值，那么二元运算为 (x, y) -> x+y，这样累积器每次都会加上你提供的数字y，这跟LongAdder的功能基本上是一样的；

假设每次操作都需要把原来的数值变为它的某个倍数，那么可以指定二元运算为 (x, y) -> x*y，累积器每次都会乘以你提供的数字y，y=2时就是通常所说的每次都翻一倍；

假设每次操作都需要把原来的数值变成它的5倍，再加上3，再除以2，再减去4，再乘以你给定的数，最后还要加上6，那么二元运算为 (x, y) -> ((x*5+3)/2 - 4)*y +6，累积器每次累积操作都会按照你说的做；

......

LongAccumulator是标准的实现类，LongAdder是特化的实现类，它的功能等价于LongAccumulator((x, y) -> x+y, 0L)。它们的区别很简单，前者可以进行任何二元算术操作，后者只能进行加减两种算术操作。

Double版本是Long版本的简单改装，相对Long版本，主要的变化就是用Double.longBitsToDouble 和Double.doubleToRawLongBits对底层的8字节数据进行long <---> double转换，存储的时候使用long型，计算的时候转化为double型。这是因为CAS是sun.misc.Unsafe中提供的操作，只对int、long、对象类型（引用或者指针）提供了这种操作，其他类型都需要转化为这三种类型才能进行CAS操作。这里的long型也可以认为是8字节的原始类型，因为把它视为long类型是无意义的。java中没有C语言中的 void* 无类型（或者叫原始类型），只能用最接近的long类型来代替。

##### 5.11.2.2 核心实现 Striped64

四个类的核心实现都在Striped64中，这个类使用分段的思想，来尽量平摊并发压力。类似1.7及以前版本的ConcurrentHashMap.Segment，Striped64中使用了一个叫Cell的类，是一个普通的二元算术累积单元，线程也是通过hash取模操作映射到一个Cell上进行累积。为了加快取模运算效率，也把Cell数组的大小设置为2^n，同时大量使用Unsafe提供的底层操作。基本的实现同1.7的ConcurrentHashMap非常像，而且更简单。

***1、累积单元Cell***

既然Cell这么简单，只有一个long型变量，为什么不直接用long value？

首先声明下，Unsafe提供的操作很强大，也能对数组的元素进行volatile读写，同时数组计算某个元素的offset偏移量本身就很简单，因此volatile、cas这种站不住脚。

```
/**
 * Padded variant of AtomicLong supporting only raw accesses plus CAS.
 *AtomicLong的填充变体，只支持原始访问和CAS。
 * JVM intrinsics note: It would be possible to use a release-only
 * form of CAS here, if it were provided.
 */
// 很简单的一个类，这个类可以看成是一个简化的AtomicLong
// 通过cas操作来更新value的值
// @sun.misc.Contended是一个高端的注解，代表使用缓存行填来避免伪共享，https://www.cnblogs.com/eycuii/p/11525164.html
@sun.misc.Contended static final class Cell {
    volatile long value;
    Cell(long x) { value = x; }
    final boolean cas(long cmp, long val) {
        return UNSAFE.compareAndSwapLong(this, valueOffset, cmp, val);
    }

    // Unsafe mechanics
    private static final sun.misc.Unsafe UNSAFE;
    private static final long valueOffset;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> ak = Cell.class;
            valueOffset = UNSAFE.objectFieldOffset
                (ak.getDeclaredField("value"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

***2、Stript64***

```
abstract class Striped64 extends Number {
    @sun.misc.Contended static final class Cell {
        ...
    }

    /** Number of CPUS, to place bound on table size */
    static final int NCPU = Runtime.getRuntime().availableProcessors();

    // cell数组，长度一样要是2^n，可以类比为jdk1.7的ConcurrentHashMap中的segments数组
    transient volatile Cell[] cells;

    // 累积器的基本值，在两种情况下会使用：
    // 1、没有遇到并发的情况，直接使用base，速度更快；
    // 2、多线程并发初始化table数组时，必须要保证table数组只被初始化一次，因此只有一个线程能够竞争成功，这种情况下竞争失败的线程会尝试在base上进行一次累积		  操作
    transient volatile long base;

   // 自旋标识，在对cells进行初始化，或者后续扩容时，需要通过CAS操作把此标识设置为1（busy，忙标识，相当于加锁），取消busy时可以直接使用cellsBusy = 0，相当于释放锁
    transient volatile int cellsBusy;

    /**
     * Package-private default constructor
     */
    Striped64() {
    }

    // 使用CAS更新base的值
    final boolean casBase(long cmp, long val) {
        return UNSAFE.compareAndSwapLong(this, BASE, cmp, val);
    }

    // 使用CAS将cells自旋标识更新为1
    // 更新为0时可以不用CAS，直接使用cellsBusy就行
    final boolean casCellsBusy() {
        return UNSAFE.compareAndSwapInt(this, CELLSBUSY, 0, 1);
    }

   // 下面这两个方法是ThreadLocalRandom中的方法，不过因为包访问关系，这里又重新写一遍
   //PROBE,它是ThreadLocalRandom里面的一个属性，在Striped64中可以把它理解为线程本身的hash值
    static final int getProbe() {
        return UNSAFE.getInt(Thread.currentThread(), PROBE);
    }

   // 相当于rehash，重新算一遍线程的hash值
    static final int advanceProbe(int probe) {
        probe ^= probe << 13;   // xorshift
        probe ^= probe >>> 17;
        probe ^= probe << 5;
        UNSAFE.putInt(Thread.currentThread(), PROBE, probe);
        return probe;
    }

     /**
     * 核心方法的实现，此方法建议在外部进行一次CAS操作（cell != null时尝试CAS更新base值，cells != null时，CAS更新hash值取模后对应的cell.value）
     * @param x the value 前面我说的二元运算中的第二个操作数，也就是外部提供的那个操作数
     * @param fn the update function, or null for add (this convention avoids the need for an extra field or function in LongAdder).
     *     外部提供的二元算术操作，实例持有并且只能有一个，生命周期内保持不变，null代表LongAdder这种特殊但是最常用的情况，可以减少一次方法调用
     * @param wasUncontended false if CAS failed before call 如果为false，表明调用者预先调用的一次CAS操作都失败了
     */
    final void longAccumulate(long x, LongBinaryOperator fn,
                              boolean wasUncontended) {
        int h;
        if ((h = getProbe()) == 0) {// 这个if相当于给线程生成一个非0的hash值
            ThreadLocalRandom.current(); // force initialization 强制初始化
            h = getProbe();
            wasUncontended = true;
        }
        //如果hash取模映射得到的Cell单元不是null，则为true，此值也可以看作是扩容意向，
        boolean collide = false;                // True if last slot nonempty 如果Cell[] 中最后一个槽为不为空，则为True
        for (;;) {
            Cell[] as; Cell a; int n; long v;
            if ((as = cells) != null && (n = as.length) > 0) {//Cell[]已经初始化过了
                if ((a = as[(n - 1) & h]) == null) {// hash取模映射得到的Cell单元还为null（为null表示还没有被使用）
                    if (cellsBusy == 0) {       // Try to attach new Cell，没有线程在执行扩容
                        Cell r = new Cell(x);   // Optimistically create 先创建新的累积单元
                        if (cellsBusy == 0 && casCellsBusy()) {//再次确认线程没有扩容并且当前线程已经获取到了锁
                            boolean created = false; 
                            try {               // Recheck under lock
                                Cell[] rs; int m, j;
                                if ((rs = cells) != null &&
                                    (m = rs.length) > 0 &&
                                    rs[j = (m - 1) & h] == null) { // 考虑别的线程可能执行了扩容，这里重新赋值（防止ABA问题）
         // 对没有使用的Cell单元进行累积操作（第一次赋值相当于是累积上一个操作数，求和时再和base执行一次运算就得到实际的结果）
                                    rs[j] = r;
                                    created = true;
                                }
                            } finally {
                                cellsBusy = 0;//清空自旋标识，释放锁
                            }
                            if (created)// 如果原本为null的Cell单元是由自己进行第一次累积操作，那么任务已经完成了，所以可以退出循环
                                break;
                            continue;           // Slot is now non-empty 不是自己进行第一次累积操作，重头再来
                        }
                    }
                    collide = false;// 执行这一句是因为cells被加锁了，不能往下继续执行第一次的赋值操作（第一次累积），所以还不能考虑扩容
                }
                else if (!wasUncontended)       // CAS already known to fail 前面一次CAS更新a.value（进行一次累积）的尝试已经失败了，说明已经发生了线程竞争
                    wasUncontended = true;      // Continue after rehash 情况失败标识，后面去重新算一遍线程的hash值，然后继续去调用CAS操作
                else if (a.cas(v = a.value, ((fn == null) ? v + x :
                                             fn.applyAsLong(v, x))))// 尝试CAS更新a.value（进行一次累积）           ------ 标记为分支A
                    break;// 成功了就完成了累积任务，退出循环
                else if (n >= NCPU || cells != as)// cell数组已经是最大的了，或者中途发生了扩容操作。因为NCPU不一定是2^n，所以这里用 >=
                    //CPU能够并行的CAS操作的最大数量是它的核心数（CAS在x86中对应的指令是cmpxchg，多核需要通过锁缓存来保证整体原子性），当n >= NCPU时，再出现几个线程映射到同一个Cell导致CAS竞争的情况，那就真不关扩容的事了，完全是hash值的锅了
                   collide = false;            // At max size or stale 
                else if (!collide)
                // 映射到的Cell单元不是null，并且尝试对它进行累积时，CAS竞争失败了，这时候把扩容意向设置为true
                // 下一次循环如果还是跟这一次一样，说明竞争很严重，那么就真正扩容
                // 把扩容意向设置为true，只有这里才会给collide赋值为true，也只有执行了这一句，才可能执行后面一个else if进行扩容
                    collide = true; 
                else if (cellsBusy == 0 && casCellsBusy()) { // 最后再考虑扩容，能到这一步说明竞争很激烈，尝试加锁进行扩容 ------ 标记为分支B
                    try {
                   // 检查下是否被别的线程扩容了（CAS更新锁标识，处理不了ABA问题，这里再检查一遍）
                        if (cells == as) {      // Expand table unless stale 
                            Cell[] rs = new Cell[n << 1]; // 执行2倍扩容
                            for (int i = 0; i < n; ++i)
                                rs[i] = as[i]; 转义元素
                            cells = rs;//重新赋值
                        }
                    } finally {
                        cellsBusy = 0;// 释放锁
                    }
                    collide = false;// 扩容意向为false
                    continue;                   // Retry with expanded table 扩容后重头再来
                }
                h = advanceProbe(h);// 重新给线程生成一个hash值，降低hash冲突，减少映射到同一个Cell导致CAS竞争的情况
            }
            else if (cellsBusy == 0 && cells == as && casCellsBusy()) {
            // cells没有被加锁，并且它没有被初始化，那么就尝试对它进行加锁，加锁成功进入这个else if
                boolean init = false;
                try {                           // Initialize table
                    if (cells == as) { // CAS避免不了ABA问题，这里再检测一次，如果还是null，或者空数组，那么就执行初始化
                        Cell[] rs = new Cell[2];// 初始化时只创建两个单元
                        rs[h & 1] = new Cell(x);// 对其中一个单元进行累积操作，另一个不管，继续为null
                        cells = rs;
                        init = true;
                    }
                } finally {
                    cellsBusy = 0;// 清空自旋标识，释放锁
                }
                if (init)// 如果某个原本为null的Cell单元是由自己进行第一次累积操作，那么任务已经完成了，所以可以退出循环，初始化完成
                    break;
            }
            else if (casBase(v = base, ((fn == null) ? v + x :
                                        fn.applyAsLong(v, x))))// cells正在进行初始化时，尝试直接在base上进行累加操作
                break;                          // Fall back on using base直接在base上进行累积操作成功了，任务完成，可以退出循环了
        }
    }

    /**
     * 跟long的逻辑基本上是一样的
     */
    final void doubleAccumulate(double x, DoubleBinaryOperator fn,
                                boolean wasUncontended) {
        int h;
        if ((h = getProbe()) == 0) {
            ThreadLocalRandom.current(); // force initialization
            h = getProbe();
            wasUncontended = true;
        }
        boolean collide = false;                // True if last slot nonempty
        for (;;) {
            Cell[] as; Cell a; int n; long v;
            if ((as = cells) != null && (n = as.length) > 0) {
                if ((a = as[(n - 1) & h]) == null) {
                    if (cellsBusy == 0) {       // Try to attach new Cell
                        Cell r = new Cell(Double.doubleToRawLongBits(x));
                        if (cellsBusy == 0 && casCellsBusy()) {
                            boolean created = false;
                            try {               // Recheck under lock
                                Cell[] rs; int m, j;
                                if ((rs = cells) != null &&
                                    (m = rs.length) > 0 &&
                                    rs[j = (m - 1) & h] == null) {
                                    rs[j] = r;
                                    created = true;
                                }
                            } finally {
                                cellsBusy = 0;
                            }
                            if (created)
                                break;
                            continue;           // Slot is now non-empty
                        }
                    }
                    collide = false;
                }
                else if (!wasUncontended)       // CAS already known to fail
                    wasUncontended = true;      // Continue after rehash
                else if (a.cas(v = a.value,
                               ((fn == null) ?
                                Double.doubleToRawLongBits
                                (Double.longBitsToDouble(v) + x) :
                                Double.doubleToRawLongBits
                                (fn.applyAsDouble
                                 (Double.longBitsToDouble(v), x)))))
                    break;
                else if (n >= NCPU || cells != as)
                    collide = false;            // At max size or stale
                else if (!collide)
                    collide = true;
                else if (cellsBusy == 0 && casCellsBusy()) {
                    try {
                        if (cells == as) {      // Expand table unless stale
                            Cell[] rs = new Cell[n << 1];
                            for (int i = 0; i < n; ++i)
                                rs[i] = as[i];
                            cells = rs;
                        }
                    } finally {
                        cellsBusy = 0;
                    }
                    collide = false;
                    continue;                   // Retry with expanded table
                }
                h = advanceProbe(h);
            }
            else if (cellsBusy == 0 && cells == as && casCellsBusy()) {
                boolean init = false;
                try {                           // Initialize table
                    if (cells == as) {
                        Cell[] rs = new Cell[2];
                        rs[h & 1] = new Cell(Double.doubleToRawLongBits(x));
                        cells = rs;
                        init = true;
                    }
                } finally {
                    cellsBusy = 0;
                }
                if (init)
                    break;
            }
            else if (casBase(v = base,
                             ((fn == null) ?
                              Double.doubleToRawLongBits
                              (Double.longBitsToDouble(v) + x) :
                              Double.doubleToRawLongBits
                              (fn.applyAsDouble
                               (Double.longBitsToDouble(v), x)))))
                break;                          // Fall back on using base
        }
    }

    // Unsafe mechanics
    private static final sun.misc.Unsafe UNSAFE;
    private static final long BASE;
    private static final long CELLSBUSY;
    private static final long PROBE;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> sk = Striped64.class;
            BASE = UNSAFE.objectFieldOffset
                (sk.getDeclaredField("base"));
            CELLSBUSY = UNSAFE.objectFieldOffset
                (sk.getDeclaredField("cellsBusy"));
            Class<?> tk = Thread.class;
            PROBE = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomProbe"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }

}
```

既然Cell这么简单，为什么不直接用long value？

标明的两个分支：分支A是用CAS更新对应的cell.value，是个写操作，分支B是进行扩容。

ConcurrentHashMap中，扩容和写操作是会严格处理的，在一个分段锁管辖区内，不会出现扩容和写操作并发：1.6和1.7的扩容操作都是在put内部执行的，put本身就会加锁，因此扩容进行时会阻塞对同一个Segment的写操作；1.8中扩容时，put/remove等方法如果碰见正在其他线程正在执行扩容，会去帮助扩容，扩容完成了之后才会去尝试加锁执行真正的写操作。

虽然B分支会进行”加锁“，但是A操作跟cellsBusy无关，”加锁“并不禁止A操作的执行。AB两个分支是不互斥的， 因此Striped64这里会出现A分支的写操作，和B分支扩容操作并发执行的情况。

那么问题是：为什么这么并发执行没问题？

A操作使用CAS更新Cell对象中的某个属性，并不改变数组持有的Cell对象的引用，扩容操作进行的是数组持有的Cell对象引用的复制，复制后引用指向的还是原来的那个Cell对象。

举个例子就是，旧的cell数组，叫作old，old[1] = cellA，cellA.value = 1，扩容后的新数组，叫作new，任然有new[1] = cellA。A分支实际上执行的是cellA.value = 2，无论分支A和B怎么并发执行，执行完成后新数组都能看到分支A对Cell的改变，扩容前后实际上数组持有的是同一群Cell对象。

这就是为什么不直接用long变量代替Cell对象。

long[]进行复制时，两个数组完完全全分离了，A分支直接作用在旧数组上，B分支扩容后，看不到串行复制执行后对旧数组同一位置的改变。举个例子就是，old[1]=10，A分支要把old[1]更新为11，这时候B分支已经复制到old[1]了，A分支执行完成后，B分支创建的新数组new[1]可能还是10（不管是多少，反正没记录A分支的操作），这样A分支的操作就被遗失了，程序会有问题。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/20170305230006646.png)





![](https://gitee.com/zhouxiaoliang/img/raw/master/img/20170305230028229.png)



##### 5.11.2.3 LongAdder

```
public class LongAdder extends Striped64 implements Serializable {
    private static final long serialVersionUID = 7249069246863182397L;

    // 构造方法，什么也不做，直接使用默认值，base = 0, cells = null
    public LongAdder() {
    }

    // add方法，根据父类的longAccumulate方法的要求，这里要进行一次CAS操作
    // （虽然这里有两个CAS，但是第一个CAS成功了就不会执行第二个，要执行第二个，第一个就被“短路”了不会被执行）
    // 在线程竞争不激烈时，这样做更快
    public void add(long x) {
        Cell[] as; long b, v; int m; Cell a;
        if ((as = cells) != null || !casBase(b = base, b + x)) {
            boolean uncontended = true;
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[getProbe() & m]) == null ||
                !(uncontended = a.cas(v = a.value, v + x)))
                longAccumulate(x, null, uncontended);
        }
    }

    /**
     * Equivalent to {@code add(1)}.
     */
    public void increment() {
        add(1L);
    }

    /**
     * Equivalent to {@code add(-1)}.
     */
    public void decrement() {
        add(-1L);
    }

    // 返回累加的和，也就是“当前时刻”的计数值
    // 此返回值可能不是绝对准确的，因为调用这个方法时还有其他线程可能正在进行计数累加，
    // 方法的返回时刻和调用时刻不是同一个点，在有并发的情况下，这个值只是近似准确的计数值
    // 高并发时，除非全局加锁，否则得不到程序运行中某个时刻绝对准确的值，但是全局加锁在高并发情况下是下下策
    // 在很多的并发场景中，计数操作并不是核心，这种情况下允许计数器的值出现一点偏差，此时可以使用LongAdder
    // 在必须依赖准确计数值的场景中，应该自己处理而不是使用通用的类
    （CAP思想---A）
    public long sum() {
        Cell[] as = cells; Cell a;
        long sum = base;
        if (as != null) {
            for (int i = 0; i < as.length; ++i) {
                if ((a = as[i]) != null)
                    sum += a.value;
            }
        }
        return sum;
    }

    // 重置计数器，只应该在明确没有并发的情况下调用，可以用来避免重新new一个LongAdder
    public void reset() {
        Cell[] as = cells; Cell a;
        base = 0L;
        if (as != null) {
            for (int i = 0; i < as.length; ++i) {
                if ((a = as[i]) != null)
                    a.value = 0L;
            }
        }
    }

    // 相当于sum()后再调用reset()
    public long sumThenReset() {
        Cell[] as = cells; Cell a;
        long sum = base;
        base = 0L;
        if (as != null) {
            for (int i = 0; i < as.length; ++i) {
                if ((a = as[i]) != null) {
                    sum += a.value;
                    a.value = 0L;
                }
            }
        }
        return sum;
    }

   。。。省略
}
```

jdk1.8的ConcurrentHashMap中，没有再使用Segment，使用了一个简单的仿造LongAdder实现的计数器，这样能够保证计数效率不低于使用Segment的效率。

#### 5.11.3 预先扩容

```
/**
 * Tries to presize table to accommodate the given number of elements.
 * 预先扩容，就是一个包含了初始化逻辑的扩容
 * 用于putAll，此时是需要考虑初始化；链表转化为红黑树中，不满足table容量条件时，进行一次扩容，此时就是普通的扩容
 * @param size number of elements (doesn't need to be perfectly accurate)
 */
private final void tryPresize(int size) {
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
        tableSizeFor(size + (size >>> 1) + 1);
    int sc;
    while ((sc = sizeCtl) >= 0) {
        Node<K,V>[] tab = table; int n;
        if (tab == null || (n = tab.length) == 0) {
            n = (sc > c) ? sc : c;
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if (table == tab) {
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
            }
        }
        else if (c <= sc || n >= MAXIMUM_CAPACITY)// c <= sc，说明已经被扩容过了；n >= MAXIMUM_CAPACITY说明table数组已经到了最大长度
            break;
        else if (tab == table) { // 可以扩容
            int rs = resizeStamp(n);// 计算本次扩容的生成戳
            if (sc < 0) {// sc < 0 表明此时有别的线程正在进行扩容
                Node<K,V>[] nt;
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0) // 这5个条件前面说了，用于判断是否能真正去帮助执行transfer任务
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) // 尝试参与此次扩容，把正在执行transfer任务的线程数加1
                    transfer(tab, nt);
            }
              // 试着让自己成为第一个执行transfer任务的线程，这个位运算前面分析了
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
        }
    }
}
```

### 5.12删除元素

删除元素跟添加元素一样，都是先找到元素所在的桶，然后采用分段锁的思想锁住整个桶，再进行操作。

```
public V remove(Object key) {
    // 调用替换节点方法
    return replaceNode(key, null, null);
}

final V replaceNode(Object key, V value, Object cv) {
    // 计算hash
    int hash = spread(key.hashCode());
    // 自旋
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0 ||
                (f = tabAt(tab, i = (n - 1) & hash)) == null)
            // 如果目标key所在的桶不存在，跳出循环返回null
            break;
        else if ((fh = f.hash) == MOVED)
            // 如果正在扩容中，协助扩容
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // 标记是否处理过
            boolean validated = false;
            synchronized (f) {
                // 再次验证当前桶第一个元素是否被修改过
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        // fh>=0表示是链表节点
                        validated = true;
                        // 遍历链表寻找目标节点
                        for (Node<K,V> e = f, pred = null;;) {
                            K ek;
                            if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                            (ek != null && key.equals(ek)))) {
                                // 找到了目标节点
                                V ev = e.val;
                                // 检查目标节点旧value是否等于cv
                                if (cv == null || cv == ev ||
                                        (ev != null && cv.equals(ev))) {
                                    oldVal = ev;
                                    if (value != null)
                                        // 如果value不为空则替换旧值
                                        e.val = value;
                                    else if (pred != null)
                                        // 如果前置节点不为空
                                        // 删除当前节点
                                        pred.next = e.next;
                                    else
                                        // 如果前置节点为空
                                        // 说明是桶中第一个元素，删除之
                                        setTabAt(tab, i, e.next);
                                }
                                break;
                            }
                            pred = e;
                            // 遍历到链表尾部还没找到元素，跳出循环
                            if ((e = e.next) == null)
                                break;
                        }
                    }
                    else if (f instanceof TreeBin) {
                        // 如果是树节点
                        validated = true;
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> r, p;
                        // 遍历树找到了目标节点
                        if ((r = t.root) != null &&
                                (p = r.findTreeNode(hash, key, null)) != null) {
                            V pv = p.val;
                            // 检查目标节点旧value是否等于cv
                            if (cv == null || cv == pv ||
                                    (pv != null && cv.equals(pv))) {
                                oldVal = pv;
                                if (value != null)
                                    // 如果value不为空则替换旧值
                                    p.val = value;
                                else if (t.removeTreeNode(p))
                                    // 如果value为空则删除元素
                                    // 如果删除后树的元素个数较少则退化成链表
                                    // t.removeTreeNode(p)这个方法返回true表示删除节点后树的元素个数较少
                                    setTabAt(tab, i, untreeify(t.first));
                            }
                        }
                    }
                }
            }
            // 如果处理过，不管有没有找到元素都返回
            if (validated) {
                // 如果找到了元素，返回其旧值
                if (oldVal != null) {
                    // 如果要替换的值为空，元素个数减1
                    if (value == null)
                        addCount(-1L, -1);
                    return oldVal;
                }
                break;
            }
        }
    }
    // 没找到元素返回空
    return null;
}
```

（1）计算hash；

（2）如果所在的桶不存在，表示没有找到目标元素，返回；

（3）如果正在扩容，则协助扩容完成后再进行删除操作；

（4）如果是以链表形式存储的，则遍历整个链表查找元素，找到之后再删除；

（5）如果是以树形式存储的，则遍历树查找元素，找到之后再删除；

（6）如果是以树形式存储的，删除元素之后树较小，则退化成链表；

（7）如果确实删除了元素，则整个map元素个数减1，并返回旧值；

（8）如果没有删除元素，则返回null；

### 5.13 获取元素

获取元素，根据目标key所在桶的第一个元素的不同采用不同的方式获取元素，关键点在于find()方法的重写。

```
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    // 计算hash
    int h = spread(key.hashCode());
    // 如果元素所在的桶存在且里面有元素
    if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
        // 如果第一个元素就是要找的元素，直接返回
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        else if (eh < 0)
            // hash小于0，说明是树或者正在扩容
            // 使用find寻找元素，find的寻找方式依据Node的不同子类有不同的实现方式
            return (p = e.find(h, key)) != null ? p.val : null;

        // 遍历整个链表寻找元素
        while ((e = e.next) != null) {
            if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

（1）hash到元素所在的桶；

（2）如果桶中第一个元素就是该找的元素，直接返回；

（3）如果是树或者正在迁移元素，则调用各自Node子类的find()方法寻找元素；

（4）如果是链表，遍历整个链表寻找元素；

（5）获取元素没有加锁；

#### 5.13.1 获取元素个数

元素个数的存储也是采用分段的思想，获取元素个数时需要把所有段加起来。LongAddr

```
public int size() {
    // 调用sumCount()计算元素个数
    long n = sumCount();
    return ((n < 0L) ? 0 :
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
                    (int)n);
}

final long sumCount() {
    // 计算CounterCell所有段及baseCount的数量之和
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```

（1）元素的个数依据不同的线程存在在不同的段里；（见addCounter()分析）

（2）计算CounterCell所有段及baseCount的数量之和；

（3）获取元素个数没有加锁；

### 5.14只读遍历器Traverser

1.8版本的扩容，采用了多线程扩容，并且很重要的一点，那就是扩容会对当前的table数组进行更改，扩容时会在transfer完一个hash桶中的所有节点后，把一个转发节点安放进去。这会影响正在进行的读操作。普通的读操作只读去一个hash桶中的内容，containsValue这种就会遍历读取所有hash桶，此时就不能使用之前版本的那种什么都不考虑的遍历读，因为此时读会跨越两个数组进行。

因此写了这个内部类，它考虑了遍历读与扩容并发的情况，专门处理1.8版本中的遍历读。

处理的逻辑很简单，就是碰到ForwardingNode时，保存当前正在遍历的数组以及索引信息，然后在FN.nextTable上进行遍历。如果继续碰到FN，就再保存一份信息，跳转到下下个数组进行hash桶节点的遍历。遍历完成后，返回最近一次记录的数组进行遍历，并清除掉最近一次保存的信息。这种行为跟入栈出栈很像，因此可以使用栈结构来保存。

这就是TableStack的由来，它是一个简化的栈，使用链表表示的。入栈就是在链表当前头结点的前面插入一个节点，并把头结点指针指向当前节点；出栈就是删除当前的头结点，把头结点指针指向删除的节点后面的一个节点。根据transfer的性质，在FN.nextTable上进行的一次遍历只用遍历两个hash桶，index 以及 index + index。在一次出栈操作执行前，需要遍历完这两个hash桶。

```
static final class TableStack<K,V> {
    int length;
    int index;
    Node<K,V>[] tab;
    TableStack<K,V> next;
}
```

```
static class Traverser<K,V> {
    Node<K,V>[] tab;        // current table; updated if resized 当前数组，也就是扩容完成后的旧数组
    Node<K,V> next;         // the next entry to use 新数组，扩容完成后使用的数组
    TableStack<K,V> stack, spare; // to save/restore on ForwardingNodes  用来 保存/恢复 转发节点
    int index;              // index of bin to use next 下一个要读取的hash桶的下标
    int baseIndex;          // current index of initial table 起始的下标，下界
    int baseLimit;          // index bound for initial table 终止的下标，上界
    final int baseSize;     // initial table size 数组的长度

    Traverser(Node<K,V>[] tab, int size, int index, int limit) {
        this.tab = tab;
        this.baseSize = size;
        this.baseIndex = this.index = index;
        this.baseLimit = limit;
        this.next = null;
    }

    /**
     * Advances if possible, returning next valid node, or null if none.
     */
     //遍历器的指针往前移动到下一个有实际数据节点，并返回这个节点，如果到头就返回null
    final Node<K,V> advance() {
        Node<K,V> e;
        if ((e = next) != null)// 如果已经进入了一个非空的hash桶，直接尝试获取它的下一个节点
            e = e.next;
        for (;;) {
            Node<K,V>[] t; int i, n;  // must use locals in checks
            if (e != null) //节点非null，直接返回
                return next = e; 
            //边界判断，遍历越界了表明没有了，可以直接返回null
            if (baseIndex >= baseLimit || (t = tab) == null ||
                (n = t.length) <= (i = index) || i < 0)
                return next = null;
            if ((e = tabAt(t, i)) != null && e.hash < 0) {// 处理特殊节点
                if (e instanceof ForwardingNode) { // 转发节点
                    tab = ((ForwardingNode<K,V>)e).nextTable;// 将遍历迁移到FN.nextTable新数组上进行
                    e = null;
                    pushState(t, i, n);// 入栈保存当前对tab数组的遍历信息
                    continue;// 开始新一次循环，遍历nextTable中对应的hash桶
                }
                else if (e instanceof TreeBin)// TreeBin时，获取红黑树所有节点的链表形式的头节点，使用链表的方式遍历，
                    e = ((TreeBin<K,V>)e).first;
                else
                    e = null;// 保留节点，没实际数据
            }
            if (stack != null)// 栈不为空
                recoverState(n); // 这里可以看做是出栈操作，得先遍历完FN.nextTable中的两个之后再出栈
            else if ((index = i + baseSize) >= n) // 栈为空，准备遍历下一个hash桶
                index = ++baseIndex; // visit upper slots if present
        }
    }

    /**
     * Saves traversal state upon encountering a forwarding node.
     */
      // 入栈操作，保存当前对tab的遍历信息
    private void pushState(Node<K,V>[] t, int i, int n) {
        TableStack<K,V> s = spare;  // reuse if possible
        if (s != null)
            spare = s.next;
        else
            s = new TableStack<K,V>();
        s.tab = t;
        s.length = n;
        s.index = i;
        s.next = stack;
        stack = s;
    }

    /**
     * Possibly pops traversal state.
     *
     * @param n length of current table
     */
     // 可能会出栈，不出栈时，更改索引，准备遍历的是FN.nextTable中对应的第二个hash桶
    private void recoverState(int n) {
        TableStack<K,V> s; int len;
        while ((s = stack) != null && (index += (len = s.length)) >= n) {
            n = len;
            index = s.index;
            tab = s.tab;
            s.tab = null;
            TableStack<K,V> next = s.next;
            s.next = spare; // save for reuse
            stack = next;
            spare = s;
        }
        if (s == null && (index += baseSize) >= n)
            index = ++baseIndex;
    }
}
```

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/20170305234602813.png)



视图（keySet/entrySet）的迭代器，基本上都是间接继承只读遍历器Traverser，用它来完成读操作。至于remove操作，直接用map.remove，remove方法（后面讲）中有处理碰见FN的情况，因此不需要迭代器额外处理。entry.set操作，依旧是跟1.6/1.7的一样，会执行一次map.put，保证节点在此时一定存在与map中。

### 5.15 总结

（1）ConcurrentHashMap是HashMap的线程安全版本；

（2）ConcurrentHashMap采用（数组 + 链表 + 红黑树）的结构存储元素；

（3）ConcurrentHashMap相比于同样线程安全的HashTable，效率要高很多；

（4）ConcurrentHashMap采用的锁有 synchronized，CAS，自旋锁，分段锁，volatile等；

（5）ConcurrentHashMap中没有threshold和loadFactor这两个字段，而是采用sizeCtl来控制；

（6）sizeCtl = -1，表示正在进行初始化；

（7）sizeCtl = 0，默认值，表示后续在真正初始化的时候使用默认容量；

（8）sizeCtl > 0，在初始化之前存储的是传入的容量，在初始化或扩容后存储的是下一次的扩容门槛；

（9）sizeCtl = (resizeStamp << 16) + (1 + nThreads)，表示正在进行扩容，高位存储扩容邮戳，低位存储扩容线程数加1；

（10）更新操作时如果正在进行扩容，当前线程协助扩容；

（11）更新操作会采用synchronized锁住当前桶的第一个元素，这是分段锁的思想；

（12）整个扩容过程都是通过CAS控制sizeCtl这个字段来进行的，这很关键；

（13）迁移完元素的桶会放置一个ForwardingNode节点，以标识该桶迁移完毕；

（14）元素个数的存储也是采用的分段思想，类似于LongAdder的实现；

（15）元素个数的更新会把不同的线程hash到不同的段上，减少资源争用；

（16）元素个数的更新如果还是出现多个线程同时更新一个段，则会扩容段（CounterCell）；

（17）获取元素个数是把所有的段（包括baseCount和CounterCell）相加起来得到的；

（18）查询操作是不会加锁的，所以ConcurrentHashMap不是强一致性的；

（19）ConcurrentHashMap中不能存储key或value为null的元素；

### 6.ConcurrentSkipListMap-跳表

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/ConcurrentSkipListMap.png)

#### 6.1跳表

**跳表是一个随机化的数据结构，实质就是一种可以进行二分查找的有序链表**。

跳表在原有的有序链表上面增加了多级索引，通过索引来实现快速查找。

跳表不仅能提高搜索性能，同时也可以提高插入和删除操作的性能。

##### 6.1.1跳表详解

**有序链表**

![image-20210119213238487](https://gitee.com/zhouxiaoliang/img/raw/master/img/image-20210119213238487.png)

考虑一个有序链表，我们要查找3、7、17这几个元素，我们只能从头开始遍历链表，直到查找到元素为止。考虑一个有序链表，我们要查找3、7、17这几个元素，我们只能从头开始遍历链表。

那么，有没有什么方法可以实现有序链表的二分查找呢？

答案是肯定的，那就是我们将要介绍的这种数据结构——跳表。

**跳表的演进**

我们把一些节点从有序表中提取出来，缓存一级索引，就组成了下面这样的结构：

![image-20210119214200320](https://gitee.com/zhouxiaoliang/img/raw/master/img/image-20210119214200320.png)

现在，我们要查找17这个元素是不是要快很多呢？

我们只要从一级索引往后遍历即可，只需要经过1、6、15、17这几个元素就可以找到17了。

那么，我们要查找11这个元素呢？

我们从一级索引的1开始，向右到6，再向右发现是15，它比11大，此路不通，从6往下走，再从下面的6往右走，到7，再到11。

同样地，一级索引也可以往上再提取一层，组成二级索引，如下：

![image-20210119214807788](https://gitee.com/zhouxiaoliang/img/raw/master/img/image-20210119214807788.png)

这时候我们再查找17这个元素呢？

只需要经过6、15、17这几个元素就可以找到17了。

这基本上就是跳表的核心思想了，其实这也是一个“空间换时间”的算法，通过向上提取索引增加了查找的效率。

##### 6.1.2 跳表的插入

上面讲的都是跳表的查询，那么，该如何向跳表中插入元素呢？

比如，我们要向上面这个跳表添加一个元素8。

首先，我们先根据投硬币的方式，决定8这个元素要占据的层数，没错就是扔硬币，

比如，层数level=2。

然后，找到8这个元素在下面两层的前置节点。

接着，就是链表的插入元素操作了，比较简单。

![image-20210119215458095](https://gitee.com/zhouxiaoliang/img/raw/master/img/image-20210119215458095.png)

##### 6.1.3 跳表的删除

首先，找到各层中包含元素x的节点。

然后，使用标准的链表删除元素的方法删除即可。

比如，要删除17这个元素。

![image-20210119220028499](https://gitee.com/zhouxiaoliang/img/raw/master/img/image-20210119220028499.png)

##### 6.1.4 标准化跳表

上面举的例子是完全随机的跳表，那么，如果我们每两个元素提取一个元素作为上一级的索引会怎么样呢？如下图。

![image-20210119220239358](https://gitee.com/zhouxiaoliang/img/raw/master/img/image-20210119220239358.png)

这是不是很像平衡二叉树，现在这颗树元素比较少，可能不太明显，我们来看个元素个数多的情况。

![image-20210119220206556](https://gitee.com/zhouxiaoliang/img/raw/master/img/image-20210119220206556.png)

可以看到，上一级元素的个数是下一级的一半，这样每次减少一半，就很接近平衡二叉树了。

##### 6.1.5 时间复杂度

我们知道单链表查询的时间复杂度为O(n)，而插入、删除操作需要先找到对应的位置，所以插入、删除的时间复杂度也是O(n)。

那么，跳表的时间复杂度是多少呢？

如果按照标准的跳表来看的话，每一级索引减少k/2个元素（k为其下面一级索引的个数），那么整个跳表的高度就是(log n)。

平衡二叉树的时间复杂度与树的高度成正比，即O(log n)。所以，这里跳表的时间复杂度也是O(log n)。

##### 6.1.6 总结

（1）跳表是可以实现二分查找的有序链表；

（2）每个元素插入时随机生成它的level；

（3）最低层包含所有的元素；

（4）如果一个元素出现在level(x)，那么它肯定出现在x以下的level中；

（5）每个索引节点包含两个指针，一个向下，一个向右；

（6）跳表查询、插入、删除的时间复杂度为O(log n)，与平衡二叉树接近；

##### 6.1.7 扩展

为什么Redis选择使用跳表而不是红黑树来实现有序集合？

首先，我们来分析下Redis的有序集合支持的操作：

1）插入元素

2）删除元素

3）查找元素

4）有序输出所有元素

5）查找区间内所有元素

其中，前4项红黑树都可以完成，且时间复杂度与跳表一致。

但是，最后一项，红黑树的效率就没有跳表高了。

在跳表中，要查找区间的元素，我们只要定位到两个区间端点在最低层级的位置，然后按顺序遍历元素就可以了，非常高效。

而红黑树只能定位到端点后，再从首位置开始每次都要查找后继节点，相对来说是比较耗时的。

此外，跳表实现起来很容易且易读，红黑树实现起来相对困难，所以Redis选择使用跳表来实现有序集合。

当然，**Redis**之所以用跳表来实现有序集合，还有其他原因，比如，跳表更容易代码实现。虽然跳表的实现也不简单，但比起红黑树来说还是好懂、好写多了，而简单就意味着可读性好，不容易出错。还有，跳表更加灵活，它可以通过改变索引构建策略，有效平衡执行效率和内存消耗。

#### 6.2 内部类

```
// 数据节点，典型的单链表结构
static final class Node<K,V> {
    final K key;
    // 注意：这里value的类型是Object，而不是V
    // 在删除元素的时候value会指向当前元素本身
    volatile Object value;
    volatile Node<K,V> next;

    Node(K key, Object value, Node<K,V> next) {
        this.key = key;
        this.value = value;
        this.next = next;
    }

    Node(Node<K,V> next) {
        this.key = null;
        this.value = this; // 当前元素本身(marker)
        this.next = next;
    }
}

// 索引节点，存储着对应的node值，及向下和向右的索引指针
static class Index<K,V> {
    final Node<K,V> node;
    final Index<K,V> down;
    volatile Index<K,V> right;

    Index(Node<K,V> node, Index<K,V> down, Index<K,V> right) {
        this.node = node;
        this.down = down;
        this.right = right;
    }
}

// 头索引节点，继承自Index，并扩展一个level字段，用于记录索引的层级
static final class HeadIndex<K,V> extends Index<K,V> {
    final int level;

    HeadIndex(Node<K,V> node, Index<K,V> down, Index<K,V> right, int level) {
        super(node, down, right);
        this.level = level;
    }
}
```

（1）Node，数据节点，存储数据的节点，典型的单链表结构；

（2）Index，索引节点，存储着对应的node值，及向下和向右的索引指针；

（3）HeadIndex，头索引节点，继承自Index，并扩展一个level字段，用于记录索引的层级；

#### 6.3 构造方法

```
public ConcurrentSkipListMap() {
    this.comparator = null;
    initialize();
}

public ConcurrentSkipListMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
    initialize();
}

public ConcurrentSkipListMap(Map<? extends K, ? extends V> m) {
    this.comparator = null;
    initialize();
    putAll(m);
}

public ConcurrentSkipListMap(SortedMap<K, ? extends V> m) {
    this.comparator = m.comparator();
    initialize();
    buildFromSorted(m);
}
```

四个构造方法里面都调用了initialize()这个方法，那么，这个方法里面有什么呢？

```
private static final Object BASE_HEADER = new Object();

private void initialize() {
    keySet = null;
    entrySet = null;
    values = null;
    descendingMap = null;
    // Node(K key, Object value, Node<K,V> next)
    // HeadIndex(Node<K,V> node, Index<K,V> down, Index<K,V> right, int level)
    head = new HeadIndex<K,V>(new Node<K,V>(null, BASE_HEADER, null),
                              null, null, 1);
}
```

可以看到，这里初始化了一些属性，并创建了一个头索引节点，里面存储着一个数据节点，这个数据节点的值是空对象，且它的层级是1。

所以，初始化的时候，跳表中只有一个头索引节点，层级是1，数据节点是一个空对象，down和right都是null。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/ConcurrentSkipList1.png)

通过内部类的结构我们知道，一个头索引指针包含node, down, right三个指针，为了便于理解，我们把指向node的指针用虚线表示，其它两个用实线表示，也就是虚线不是表明方向的。

#### 6.4 添加元素

通过对跳表的分析，我们知道跳表插入元素的时候会通过抛硬币的方式决定出它需要的层级，然后找到各层链中它所在的位置，最后通过单链表插入的方式把节点及索引插入进去来实现的。

整个插入过程分成三个部分：

------

Part I：找到目标节点的位置并插入

（1）这里的目标节点是数据节点，也就是最底层的那条链；

（2）寻找目标节点之前最近的一个索引对应的数据节点（数据节点都是在最底层的链表上）；

（3）从这个数据节点开始往后遍历，直到找到目标节点应该插入的位置；

（4）如果这个位置有元素，就更新其值（onlyIfAbsent=false）；

（5）如果这个位置没有元素，就把目标节点插入；

（6）至此，目标节点已经插入到最底层的数据节点链表中了；

------

Part II：随机决定是否需要建立索引及其层次，如果需要则建立自上而下的索引

（1）取个随机数rnd，计算(rnd & 0x80000001)；

（2）如果不等于0，结束插入过程，也就是不需要创建索引，返回；

（3）如果等于0，才进入创建索引的过程（只要正偶数才会等于0）；

（4）计算`while (((rnd >>>= 1) & 1) != 0)`，决定层级数，level从1开始；

（5）如果算出来的层级不高于现有最高层级，则直接建立一条竖直的索引链表（只有down有值），并结束Part II；

（6）如果算出来的层级高于现有最高层级，则新的层级只能比现有最高层级多1；

（7）同样建立一条竖直的索引链表（只有down有值）；

（8）将头索引也向上增加到相应的高度，结束Part II；

（9）也就是说，如果层级不超过现有高度，只建立一条索引链，否则还要额外增加头索引链的高度

------

Part III：将新建的索引节点（包含头索引节点）与其它索引节点通过右指针连接在一起（补上right指针）

（1）从最高层级的头索引节点开始，向右遍历，找到目标索引节点的位置；

（2）如果当前层有目标索引，则把目标索引插入到这个位置，并把目标索引前一个索引向下移一个层级；

（3）如果当前层没有目标索引，则把目标索引位置前一个索引向下移一个层级；

（4）同样地，再向右遍历，寻找新的层级中目标索引的位置，回到第（2）步；

（5）依次循环找到所有层级目标索引的位置并把它们插入到横向的索引链表中；

总结起来，一共就是三大步：

（1）插入目标节点到数据节点链表中；

（2）建立竖直的down链表；

（3）建立横向的right链表；

假设初始链表是这样：

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/ConcurrentSkipList2.png)

假如，我们现在要插入一个元素9。

（1）寻找目标节点之前最近的一个索引对应的数据节点，在这里也就是找到了5这个数据节点；

（2）从5开始向后遍历，找到目标节点的位置，也就是在8和12之间；

（3）插入9这个元素，Part I 结束；

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/ConcurrentSkipList3.png)

然后，计算其索引层级，假如是3，也就是level=3。

（1）建立竖直的down索引链表；

（2）超过了现有高度2，还要再增加head索引链的高度；

（3）至此，Part II 结束；

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/ConcurrentSkipList4.png)

最后，把right指针补齐。

（1）从第3层的head往右找当前层级目标索引的位置；

（2）找到就把目标索引和它前面索引的right指针连上，这里前一个正好是head；

（3）然后前一个索引向下移，这里就是head下移；

（4）再往右找目标索引的位置；

（5）找到了就把right指针连上，这里前一个是3的索引；

（6）然后3的索引下移；

（7）再往右找目标索引的位置；

（8）找到了就把right指针连上，这里前一个是5的索引；

（9）然后5下移，到底了，Part III 结束，整个插入过程结束；

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/ConcurrentSkipList5.png)

下面看详细代码：

寻找目标节点之前，最近的一个索引对应的数据节点

```
/**
 * Returns a base-level node with key strictly less than given key,
 * or the base-level header if there is no such node.  Also
 * unlinks indexes to deleted nodes found along the way.  Callers
 * rely on this side-effect of clearing indices to deleted nodes.
 * @param key the key
 * @return a predecessor of key
 */
private Node<K,V> findPredecessor(Object key, Comparator<? super K> cmp) {
    if (key == null)
        throw new NullPointerException(); // don't postpone errors
    for (;;) { // 自旋
    
    	// 从最高层头索引节点开始查找，先向右，再向下
        // 直到找到目标位置之前的那个索引
        for (Index<K,V> q = head, r = q.right, d;;) {
            if (r != null) { // 如果右节点不为空
                Node<K,V> n = r.node;  // 右节点对应的数据节点，为了方便，我们叫右节点的值
                K k = n.key;
                // 如果右节点的value为空
                // 说明其它线程把这个节点标记为删除了
                // 则协助删除
                if (n.value == null) {
                    if (!q.unlink(r))
                    	// 如果删除失败
                        // 说明其它线程先删除了，从头来过
                        break;           // restart
                    r = q.right;         // reread r 删除之后重新读取右节点
                    continue;
                }
                if (cpr(cmp, key, k) > 0) {  // 如果目标key比右节点还大，继续向右寻找
                    q = r; // 往右移
                    r = r.right; // 重新取右节点
                    continue;
                }
            }
            // 到这里说明当前层级已经到最右了
            // 两种情况：一是r==null，二是c<0
            // 再从下一级开始找
             // 如果没有下一级了，就返回这个索引对应的数据节点
            if ((d = q.down) == null)
                return q.node;
            // 往下移
            q = d;
             // 重新取右节点
            r = d.right;
        }
    }
}
```

协助删除元素

```
ConcurrentSkipListMap.Node
/**
 * Helps out a deletion by appending marker or unlinking from
 * predecessor. This is called during traversals when value
 * field seen to be null.
 * @param b predecessor
 * @param f successor
 */
void helpDelete(Node<K,V> b, Node<K,V> f) {
    /*
     * Rechecking links and then doing only one of the
     * help-out stages per call tends to minimize CAS
     * interference among helping threads.
     */
     // 这里的调用者this==n，三者关系是b->n->f
    if (f == next && this == b.next) {
        // 将n的值设置为null后，会先把n的下个节点设置为marker节点
        // 这个marker节点的值是它自己
        // 这里如果不是它自己说明marker失败了，重新marker
        if (f == null || f.value != f) // not already marked
            casNext(f, new Node<K,V>(f));
        else
         // marker过了，就把b的下个节点指向marker的下个节点
            b.casNext(this, f.next);
    }
}
marker --->参考6.5 删除元素的过程
```

```
// Index.class中的方法，删除succ节点
final boolean unlink(Index<K,V> succ) {
    // 原子更新当前节点指向下一个节点的下一个节点
    // 也就是删除下一个节点
    return node.value != null && casRight(succ, succ.right);
}

// Index.class中的方法，在当前节点与succ之间插入newSucc节点
final boolean link(Index<K,V> succ, Index<K,V> newSucc) {
    // 在当前节点与下一个节点中间插入一个节点
    Node<K,V> n = node;
    // 新节点指向当前节点的下一个节点
    newSucc.right = succ;
    // 原子更新当前节点的下一个节点指向新节点
    return n.value != null && casRight(succ, newSucc);
}
```

 添加元素



```
private V doPut(K key, V value, boolean onlyIfAbsent) {
    Node<K,V> z;             // added node
    if (key == null)
        throw new NullPointerException();
    Comparator<? super K> cmp = comparator;
    // Part I：找到目标节点的位置并插入
    // 这里的目标节点是数据节点，也就是最底层的那条链
    // 自旋
    outer: for (;;) {
   		// 寻找目标节点之前最近的一个索引对应的数据节点，存储在b中，b=before
        // 并把b的下一个数据节点存储在n中，n=next
        // 为了便于描述，我这里把b叫做当前节点，n叫做下一个节点
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
         	// 如果下一个节点不为空
            // 就拿其key与目标节点的key比较，找到目标节点应该插入的位置
            if (n != null) {
            	// v=value，存储节点value值
                // c=compare，存储两个节点比较的大小
                Object v; int c;
                // n的下一个数据节点，也就是b的下一个节点的下一个节点（孙子节点）
                Node<K,V> f = n.next;
                // 如果n不为b的下一个节点
                // 说明有其它线程修改了数据，则跳出内层循环
                // 也就是回到了外层循环自旋的位置，从头来过
                if (n != b.next)               // inconsistent read
                    break;
                if ((v = n.value) == null) {   // n is deleted  如果n的value值为空，说明该节点已删除，协助删除节点
                    n.helpDelete(b, f);
                    break;
                }
                // 如果b的值为空或者v等于n，说明b已被删除
                // 这时候n就是marker节点，那b就是被删除的那个
                if (b.value == null || v == n) // b is deleted
                    break;
                // 如果目标key与下一个节点的key大
                // 说明目标元素所在的位置还在下一个节点的后面
                if ((c = cpr(cmp, key, n.key)) > 0) {
                   // 就把当前节点往后移一位
                    // 同样的下一个节点也往后移一位
                    // 再重新检查新n是否为空，它与目标key的关系
                    b = n;
                    n = f;
                    continue;
                }
                // 如果比较时发现下一个节点的key与目标key相同
                // 说明链表中本身就存在目标节点
                if (c == 0) {
                    if (onlyIfAbsent || n.casValue(v, value)) {  // 则用新值替换旧值，并返回旧值（onlyIfAbsent=false）
                        @SuppressWarnings("unchecked") V vv = (V)v;
                        return vv;
                    }
                    break; // restart if lost race to replace value  如果替换旧值时失败，说明其它线程先一步修改了值，从头来过
                }
                // 如果c<0，就往下走，也就是找到了目标节点的位置
                // else c < 0; fall through
            }
 			// 有两种情况会到这里
            // 一是到链表尾部了，也就是n为null了
            // 二是找到了目标节点的位置，也就是上面的c<0
            
            
            // 新建目标节点，并赋值给z
            // 这里把n作为新节点的next
            // 如果到链表尾部了，n为null
            // 如果c<0，则n的key比目标key大，相当于在b和n之间插入目标节点z
            z = new Node<K,V>(key, value, n);
            
            if (!b.casNext(n, z)) // 原子更新b的下一个节点为目标节点z
                break;         // restart if lost race to append to b  如果更新失败，说明其它线程先一步修改了值，从头来过
            break outer; // 如果更新成功，跳出自旋状态
        }
    }
	
	// Part I，目标节点已经插入到有序链表中了

    // Part II：随机决定是否需要建立索引及其层次，如果需要则建立自上而下的索引
	
    int rnd = ThreadLocalRandom.nextSecondarySeed();  // 取个随机数
    // 0x80000001展开为二进制为10000000000000000000000000000001
    // 只有两头是1
    // 这里(rnd & 0x80000001) == 0
    // 相当于排除了负数（负数最高位是1），排除了奇数（奇数最低位是1）
    // 只有最高位最低位都不为1的数跟0x80000001做&操作才会为0
    // 也就是正偶数  （这个计算方法NB啊）
    if ((rnd & 0x80000001) == 0) { // test highest and lowest bits
        int level = 1, max;  // 默认level为1，也就是只要到这里了就会至少建立一层索引
        // 随机数从最低位的第二位开始，有几个连续的1则level就加几
        // 因为最低位肯定是0，正偶数嘛
        // 比如，1100110，level就加2
        while (((rnd >>>= 1) & 1) != 0)
            ++level;
        // 用于记录目标节点建立的最高的那层索引节点
        Index<K,V> idx = null;
        // 取头索引节点（这是最高层的头索引节点）
        HeadIndex<K,V> h = head;
        // 如果生成的层数小于等于当前最高层的层级
        // 也就是跳表的高度不会超过现有高度
        if (level <= (max = h.level)) {
            for (int i = 1; i <= level; ++i)
            // 从第一层开始建立一条竖直的索引链表
            // 这条链表使用down指针连接起来
            // 每个索引节点里面都存储着目标节点这个数据节点
            // 最后idx存储的是这条索引链表的最高层节点
                idx = new Index<K,V>(z, idx, null);
        }
        else { // try to grow by one level
         	// 如果新的层数超过了现有跳表的高度
            // 则最多只增加一层
             // 比如现在只有一层索引，那下一次最多增加到两层索引，增加多了也没有意义
            level = max + 1; // hold in array and later pick the one to use
            // idxs用于存储目标节点建立的竖起索引的所有索引节点
            // 其实这里直接使用idx这个最高节点也是可以完成的
            // 只是用一个数组存储所有节点要方便一些
            // 注意，这里数组0号位是没有使用的
            @SuppressWarnings("unchecked")Index<K,V>[] idxs =
                (Index<K,V>[])new Index<?,?>[level+1];
             // 从第一层开始建立一条竖的索引链表（跟上面一样，只是这里顺便把索引节点放到数组里面了）
            for (int i = 1; i <= level; ++i)
                idxs[i] = idx = new Index<K,V>(z, idx, null);
            for (;;) { // 自旋
                h = head;   // 旧的最高层头索引节点
                int oldLevel = h.level;  // 旧的最高层级
                // 再次检查，如果旧的最高层级已经不比新层级矮了
                // 说明有其它线程先一步修改了值，从头来过
                if (level <= oldLevel) // lost race to add level
                    break;
                 // 新的最高层头索引节点
                HeadIndex<K,V> newh = h;
                  // 头节点指向的数据节点
                Node<K,V> oldbase = h.node;
                 // 超出的部分建立新的头索引节点
                for (int j = oldLevel+1; j <= level; ++j)
                    newh = new HeadIndex<K,V>(oldbase, newh, idxs[j], j);
                if (casHead(h, newh)) {// 原子更新头索引节点
                    h = newh;// h指向新的最高层头索引节点
                    // 把level赋值为旧的最高层级的
                    // idx指向的不是最高的索引节点了
                    // 而是与旧最高层平齐的索引节点
                    idx = idxs[level = oldLevel];
                    break;
                }
            }
        }
        
        
        // 经过上面的步骤，有两种情况
        // 一是没有超出高度，新建一条目标节点的索引节点链
        // 二是超出了高度，新建一条目标节点的索引节点链，同时最高层头索引节点同样往上长

        // Part III：将新建的索引节点（包含头索引节点）与其它索引节点通过右指针连接在一起
        
        // find insertion points and splice in
        splice: for (int insertionLevel = level;;) {//这时level是等于旧的最高层级的，自旋
           // h为最高头索引节点
            int j = h.level;
            // 从头索引节点开始遍历
            // 为了方便，这里叫q为当前节点，r为右节点，d为下节点，t为目标节点相应层级的索引
            for (Index<K,V> q = h, r = q.right, t = idx;;) {
             	// 如果遍历到了最右边，或者最下边，
                // 也就是遍历到头了，则退出外层循环
                if (q == null || t == null)
                    break splice;
                if (r != null) {      // 如果右节点不为空
                    Node<K,V> n = r.node;// n是右节点的数据节点，为了方便，这里直接叫右节点的值
                    // compare before deletion check avoids needing recheck
                    int c = cpr(cmp, key, n.key);   // 比较目标key与右节点的值
                    if (n.value == null) { // 如果右节点的值为空了，则表示此节点已删除
                        if (!q.unlink(r))// 如果删除失败，说明有其它线程先一步修改了，从头来过
                            break;
                        r = q.right; // 删除成功后重新取右节点
                        continue;
                    }
                    if (c > 0) {// 如果比较c>0，表示目标节点还要往右
                   		 // 则把当前节点和右节点分别右移
                        q = r;
                        r = r.right;
                        continue;
                    }
                }
	 			// 到这里说明已经到当前层级的最右边了
                // 这里实际是会先走第二个if

                // 【第一个if】
                // j与insertionLevel相等了
                // 实际是先走的第二个if，j自减后应该与insertionLevel相等
                if (j == insertionLevel) {
                    if (!q.link(r, t))
                        break; // restart
                    if (t.node.value == null) {
                        findNode(key);
                        break splice;
                    }
                    if (--insertionLevel == 0)
                        break splice;
                }
				// 【第二个if】
                // j先自减1，再与两个level比较
                // j、insertionLevel和t(idx)三者是对应的，都是还未把右指针连好的那个层级
                if (--j >= insertionLevel && j < level)
                    t = t.down;// t往下移
                // 当前层级到最右边了
                // 那只能往下一层级去走了
                // 当前节点下移
                // 再取相应的右节点
                q = q.down;
                r = q.right;
            }
        }
    }
    return null;
}
```

#### 6.5 删除元素

删除元素，就是把各层级中对应的元素删除即可

（1）寻找目标节点之前最近的一个索引对应的数据节点（数据节点都是在最底层的链表上）；

（2）从这个数据节点开始往后遍历，直到找到目标节点的位置；

（3）如果这个位置没有元素，直接返回null，表示没有要删除的元素；

（4）如果这个位置有元素，先通过`n.casValue(v, null)`原子更新把其value设置为null；

（5）通过`n.appendMarker(f)`在当前元素后面添加一个marker元素标记当前元素是要删除的元素；

（6）通过`b.casNext(n, f)`尝试删除元素；

（7）如果上面两步中的任意一步失败了都通过`findNode(key)`中的`n.helpDelete(b, f)`再去不断尝试删除；

（8）如果上面两步都成功了，再通过`findPredecessor(key, cmp)`中的`q.unlink(r)`删除索引节点；

（9）如果head的right指针指向了null，则跳表高度降级；

假如初始跳表如下图所示，我们要删除9这个元素。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/ConcurrentSkipList6.png)

（1）找到9这个数据节点；

（2）把9这个节点的value值设置为null；

（3）在9后面添加一个marker节点，标记9已经删除了；

（4）让8指向12；

（5）把索引节点与它前一个索引的right断开联系；

（6）跳表高度降级；

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/ConcurrentSkipList7.png)

至于，为什么要有（2）（3）（4）这么多步骤呢？

因为多线程下如果直接让8指向12，在处理链表指向的时候，有可能其它线程先一步在9和12间插入了一个元素10，这时候就不对了。

如果第（2）步失败了，则直接重试；

如果第（3）或（4）步失败了，因为第（2）步是成功的，则通过helpDelete()不断重试去删除；

其实helpDelete()里面也是不断地重试（3）和（4）；

只有这三步都正确完成了，才能说明这个元素彻底被删除了。

#### 6.6 查询元素

（1）寻找目标节点之前最近的一个索引对应的数据节点（数据节点都是在最底层的链表上）；

（2）从这个数据节点开始往后遍历，直到找到目标节点的位置；

（3）如果这个位置没有元素，直接返回null，表示没有找到元素；

（4）如果这个位置有元素，返回元素的value值；



假如有如下图所示这个跳表，我们要查找9这个元素，它走过的路径是怎样的呢？



![](https://gitee.com/zhouxiaoliang/img/raw/master/img/ConcurrentSkipList6.png)

（1）寻找目标节点之前最近的一个索引对应的数据节点，这里就是5；

（2）从5开始往后遍历，经过8，到9；

（3）找到了返回；

整个路径如下图所示：



![](https://gitee.com/zhouxiaoliang/img/raw/master/img/ConcurrentSkipList8.png)





为啥不从9的索引直接过来呢？

从我实际打断点调试来看确实是按照上图的路径来走的。

因为findPredecessor()这个方法是插入、删除、查找元素多个方法共用的，在单链表中插入和删除元素是需要记录前一个元素的，而查找并不需要，这里为了兼容三者使得编码相对简单一点，所以就使用了同样的逻辑，而没有单独对查找元素进行优化。

JDK1.8 没有优化



```
private V doGet(Object key) {
    if (key == null)
        throw new NullPointerException();
    Comparator<? super K> cmp = comparator;
    outer: for (;;) {
        // 寻找目标节点之前最近的索引对应的数据节点
        // 为了方便，这里叫b为当前节点，n为下个节点，f为下下个节点
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            Object v; int c;
            // 如果链表到头还没找到元素，则跳出外层循环
            if (n == null)
                break outer;
            // 下下个节点
            Node<K,V> f = n.next;
            // 如果不一致读，从头来过
            if (n != b.next)                // inconsistent read
                break;
            // 如果n的值为空，说明节点已被其它线程标记为删除
            if ((v = n.value) == null) {    // n is deleted
                // 协助删除，再重试
                n.helpDelete(b, f);
                break;
            }
            // 如果b的值为空或者v等于n，说明b已被删除
            // 这时候n就是marker节点，那b就是被删除的那个
            if (b.value == null || v == n)  // b is deleted
                break;
            // 如果c==0，说明找到了元素，就返回元素值
            if ((c = cpr(cmp, key, n.key)) == 0) {
                @SuppressWarnings("unchecked") V vv = (V)v;
                return vv;
            }
            // 如果c<0，说明没找到元素
            if (c < 0)
                break outer;
            // 如果c>0，说明还没找到，继续寻找
            // 当前节点往后移
            b = n;
            // 下一个节点往后移
            n = f;
        }
    }
    return null;
}
```

