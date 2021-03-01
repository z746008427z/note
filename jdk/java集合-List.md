



# java List

## 1.List

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/Iterable.png)

list集合的元素是有序，可重复的，主要实现方式是动态数组和链表。

java提供的List的实现主要是 ArrayList、LinkedList、CopyOnWriteArrayList，另外还设有 Vector和Stack（线程安全，synchnorized）

关于List的问题：

1.ArrayList 和 LinkedList的区别？

​    ArrayList 有序数组，查询快，更新慢，栈方式，先进后出

​    LinkedList 链表，更新快，查询慢，链表方式，先入先出。 

2.ArrayList是如何扩容的。

   初始化0 ，添加第一个元素，长度设置为10

   元素个数超过10，扩展1.5倍

   最大长度2^32 -1

ArrayList内部使用数组存储元素，当数组长度不够时进行扩容，每次加一半的空间，ArrayList不会进行缩容；

3. ArrayList 插入，删除，查询元素的时间复杂度各是多少。

   ArrayList支持随机访问，通过索引访问元素极快，时间复杂度为O(1)；

   ArrayList添加元素到尾部极快，平均时间复杂度为O(1)；

   ArrayList添加元素到中间比较慢，因为要搬移元素，平均时间复杂度为O(n)；

   ArrayList从尾部删除元素极快，时间复杂度为O(1)；

   ArrayList从中间删除元素比较慢，因为要搬移元素，平均时间复杂度为O(n)；

4. 怎么求两个集合的交集，并集，差集。

   并集，add(Collection<? extends E> c)

   交集，retainAll(Collection<? extends E> c)

   差集，removeAll(Collection<? extends E> c)

5. ArrayList是怎么实现序列化和反序列化的。

   序列化：

   先调用java.io.ObjectOutputStream.defaultWriteObject()方法，写出非transient非static属性（会写出size属性），

    写出元素个数==>依次写出元素===> 如果有修改，抛出异常

   反序列化：

   声明空数组===》调用java.io.ObjectInputStream.defaultReadObject() 读入非transient非static属性（会读取size属性）==>按顺序来读==>根据size，设置容量。

6. 集合方法的toArray()有什么问题。

    不一定返回Object[] ，也可能返回字符串类型

7. 什么是fail-fast.

   快速失败，是在对集合进行迭代时，一旦发现集合有更新那么，快速失败。

8. LinkedList是单链表还是双链表。

   双链表

9. LinkedList除了作为List还有什么用处。

   栈，队列

10. LinkedList插入、删除、查询的时间复杂度是多少

    LinkedList 插入 首尾 O(1),插入 中间O（n）

    LinkedList 删除首尾O(1), 删除 中间O（n）

    LinkedList 查询 ，从中间分隔，根据查询对象，处于左右分区进行迭代，左分区，0~x ,右分区 x~n ，复杂度O（n）

11. 什么是随机访问

    RandomAccess接口

    数组是一种**线性表**数据的结构，他用一组**连续的内存空间**，来存储一组**相同数据类型**的数据。

    - 线性表:数据排列成一条线一样的结构。数据结构特点:存在一个唯一的没有前驱的（头）数据元素；存在一个唯一的没有后继的（尾）数据元素存在头和尾元素。像队列，链表，栈也是线性表结构。对应的还有非线性表结构（数据没有先后顺序的，二叉树，堆等）
    - 连续内存空间:计算机在分配内存空的时候都会对应分配一个内存地址，连续的内存空间对应的是指连续的内存地址，计算机是通过访问内存地址会获取内存中的值。
    - 相同的数据类型:相同的数据类型，换句话可以说数据存储所占用内存大小一样

    **随机访问**--->就是存取第N个数据时，不需要访问前(N-1)个数据,直接就可以对第N个数据操作(数组)

    ![](https://gitee.com/zhouxiaoliang/img/raw/master/img/1760039-20190806114345629-910014410.png)

    ## 为什么数组下标都是从0开始？

    - 从上面图示我们来分析：
      - 假设下标为1开始：我们要想获取第3个值得话 首地址（1000）+ （3-1）*4（数据类型占用的内存） = 1008 第三个内存地址的位置
      - 驾驶下标从0开始：我们想获取第3个值得花 首地址（1000）+ 2 *4（数据类型占用的内存） = 1008 省去了一个减的动作 提高了访问的效率。

12. 哪些集合支持随机访问,它们有哪些特性

    实现RandomAccess接口

13. CopyOnWriteArrayList是怎么保证并发安全的

    写操作， 加锁（CAS）  拷贝出副本，操作，然后将更新后的副本，覆盖源数组，释放锁

    加锁，1.8  ReentrantLock   11  synchronized(已优化，锁升级)

14. CopyOnWriteArrayList的实现采用的是什么思想

    CopyOnWriteArrayList采用读写分离的思想，读操作不加锁，写操作加锁，且写操作占用较大内存空间，所以适用于读多写少的场合；

15. CopyOnWriteArrayList是不是强一致性

​       不是，CopyOnWriteArrayList只保证最终一致性，不保证实时一致性；

3. CopyOnWriteArrayList适用于什么样的场景

   适用于读多写少的场合

4. CopyOnWriteArrayList插入、删除、查询元素的时间复杂度各是多少

   ​	CopyOnWriteArrayList的写操作 O(n)

      CopyOnWriteArrayList的读操作 O(1)

5. CopyOnWriteArrayList为什么没有size属性

   因为每次修改都是拷贝一份正好可以存储目标个数元素的数组，所以不需要size属性了，数组的长度就是集合的大小，而不像ArrayList数组的长度实际是要大于集合的大小的。比如，add(E e)操作，先拷贝一份n+1个元素的数组，再把新元素放到新数组的最后一位，这时新数组的长度为len+1了，也就是集合的size了。

6. 比较古老的Vector和Stack有什么缺陷，

   synchronized  可重入锁，性能低

### 1.1 ArrayList

ArrayList是一种以数组实现的List,具有扩展能力，又叫动态数组。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/ArrayList.png)

ArrayList 实现了 List,提供了基础的增删查，遍历的操作。

ArrayList 实现Serializable ，可序列化

ArrayList 实现RandomAccess，提供随机访问的能力

ArrayList 实现Cloneable，可被克隆（深浅克隆）

#### 1.1.1 属性

```
/**
 * 默认容量
 */
private static final int DEFAULT_CAPACITY = 10;

/**
 * 空数组，如果传入的容量为0时，使用此属性
 */
private static final Object[] EMPTY_ELEMENTDATA = {};

/**
 空数组，传入容量为0时使用，添加第一个元素的时候会重新初始化为默认容量大小
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 存储ArrayList元素的数组缓冲区。
 * ArrayList的容量是这个数组缓冲区的长度. 任何带有elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA的空ArrayList将在添加第一个元素时扩展为DEFAULT_CAPACITY。
 * 存储元素的数组
 */
transient Object[] elementData; // non-private to simplify nested class access

/**
 * The size of the ArrayList (the number of elements it contains).
 *集合中元素的个数
 * @serial
 
 */
private int size;
protected transient int modCount = 0;
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

（1）DEFAULT_CAPACITY

默认容量为10，也就是通过new ArrayList()创建时的默认容量。

（2）EMPTY_ELEMENTDATA

空的数组，这种是通过new ArrayList(0)创建时用的是这个空数组。

（3）DEFAULTCAPACITY_EMPTY_ELEMENTDATA

也是空数组，这种是通过new ArrayList()创建时用的是这个空数组，与EMPTY_ELEMENTDATA的区别是在添加第一个元素时使用这个空数组的会初始化为DEFAULT_CAPACITY（10）个元素。

（4）elementData

真正存放元素的地方，使用transient是为了不序列化这个字段。

至于没有使用private修饰，后面注释是写的“为了简化嵌套类的访问”，但是楼主实测加了private嵌套类一样可以访问。

*private表示是类私有的属性，只要是在这个类内部都可以访问，嵌套类或者内部类也是在类的内部，所以也可以访问类的私有成员。*

（5）size

真正存储元素的个数，而不是elementData数组的长度。

（6）modCount

#####  1.1.1.1 modCount

```
/**
 * The number of times this list has been <i>structurally modified</i>.
 * Structural modifications are those that change the size of the
 * list, or otherwise perturb it in such a fashion that iterations in
 * progress may yield incorrect results.
 *
 * <p>This field is used by the iterator and list iterator implementation
 * returned by the {@code iterator} and {@code listIterator} methods.
 * If the value of this field changes unexpectedly, the iterator (or list
 * iterator) will throw a {@code ConcurrentModificationException} in
 * response to the {@code next}, {@code remove}, {@code previous},
 * {@code set} or {@code add} operations.  This provides
 * <i>fail-fast</i> behavior, rather than non-deterministic behavior in
 * the face of concurrent modification during iteration.
 *
 * <p><b>Use of this field by subclasses is optional.</b> If a subclass
 * wishes to provide fail-fast iterators (and list iterators), then it
 * merely has to increment this field in its {@code add(int, E)} and
 * {@code remove(int)} methods (and any other methods that it overrides
 * that result in structural modifications to the list).  A single call to
 * {@code add(int, E)} or {@code remove(int)} must add no more than
 * one to this field, or the iterators (and list iterators) will throw
 * bogus {@code ConcurrentModificationExceptions}.  If an implementation
 * does not wish to provide fail-fast iterators, this field may be
 * ignored.
 */
protected transient int modCount = 0;
```

翻译：

modCount指的是list被结构性修改的次数。

结构性修改是指改变list的size大小，或者其他的情况导致正在迭代迭代的时候出现了错误的结果

这个字段用于迭代器和列表迭代器的实现类，由迭代器和列表迭代器返回。

如果这个值被意外改变，这个迭代器或者列表迭代器将会抛出异常来响应。add,next,remove,previous,st,add

在迭代过程中，它提供了fail-fast行为而不是不确定的行为来处理并发修改。

子类使用这个字段是可选的，如果子类希望提供fail-fast迭代器，它仅仅需要在add（int）,remove（int）方法(或者它重写的其他任何会修改这个列表的方法)中添加这个字段。

调用一次add(int,E)或者remove(int,E)方法仅仅给这个字段加1，否则迭代器会抛出异常。

如果一个实现类不希望提供fail-fast迭代器，则忽略这个字段。

**exceptedModCount**

expectedModCount = alist.modCount;

**modCount用法**

在线程不安全的集合中，在某些方法中，初始化迭代器会给modCount赋值，如果在遍历的过程中，一旦发现这个对象的modCount和迭代器存储的modCount不一样，就会报错。

**fail-fast机制**

在线程不安全的集合中，如果在使用迭代器的过程中，发现集合被修改，会抛出ConcurrentModificationExceptions错误，这就是快速失败机制。

对于集合进行结构性修改时，modCount都会增加，在初始化迭代器时，modCount的值会赋值给expectedModCount,在迭代过程中，只要modCount变了（不是线程安全的，其他线程可能修改）int expectedModCount=modCount就不成立了，迭代器监测到这一点，就会抛出错误：ConcurrentModificationExceptions

#### 1.1.2 构造方法

```
/**
* 把传入集合的元素初始化到ArrayList中
*/
public ArrayList(Collection<? extends E> c) {
		// 集合转数组
        Object[] a = c.toArray();
        if ((size = a.length) != 0) {
        	// 检查c是不是ArrayList类型，如果不是，重新拷贝成Object[].class类型
            if (c.getClass() == ArrayList.class) {
                elementData = a;
            } else {
                elementData = Arrays.copyOf(a, size, Object[].class);
            }
        } else {
            // 如果c的空集合，则初始化为空数组EMPTY_ELEMENTDATA
            elementData = EMPTY_ELEMENTDATA;
        }
}
```

传入集合并初始化elementData，这里会使用拷贝把传入集合的元素拷贝到elementData数组中，如果元素个数为0，则初始化为EMPTY_ELEMENTDATA空数组。

#### 1.1.3 add方法

##### 1.1.3.1 add(E e)

添加元素到末尾，平均时间复杂度为O(1)。

```
public boolean add(E e) {
    modCount++;//线程不安全
    add(e, elementData, size);
    return true;
}
```

```
/**
 * 这个helper方法从add(E)中分离出来，使方法字节码大小保持在35以下(-XX:MaxInlineSize默认值)，这有助于在c1编译循环中调用add(E)。
 */
private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}
```

size是当前的数组中有多少个元素，不包括新增的。

当发现此时元素数量和这个数组的长度相等，也就说明满了，将执行扩容操作，也就是grow()函数。如果没满，就赋值，然后元素的数量+1。

```
private Object[] grow() {
    return grow(size + 1);
}
private Object[] grow(int minCapacity) {
        return elementData = Arrays.copyOf(elementData,newCapacity(minCapacity));
}
```

看上面的第二个函数此时的minCapacity=size+1,执行拷贝函数返回结果给elementData,在拷贝函数里面有两个参数，第一个是老数组，第二个是要扩容的长度。新数组的长度是由newCapacity(minCapacity)决定。下面看此函数

```
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);    
    if (newCapacity - minCapacity <= 0) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return minCapacity;
    }
    return (newCapacity - MAX_ARRAY_SIZE <= 0)
        ? newCapacity
        : hugeCapacity(minCapacity);
}
```

oldCapacity为老数组的长度，newCapacity为老数组的1.5呗（位运算），下面的if条件判断此时扩容后的数组长度是否满足我要的数组长度，如果条件成立（newCapacity <=minCapacity ）,也就是说你这个扩容后还不行，还是太小啊，进入判断体，

这里的DEFAULTCAPACITY_EMPTY_ELEMENTDATA，上面有讲过，是个空数组，如果这个elementData是个空数组，则直接返回minCapacity这个长度作为新数组的长度，小于0抛出异常。下面我们再来看最下面的return 语句，如果此时扩容后的长度符合要求，这个 MAX_ARRAY_SIZE是啥呢？

```
/**
 * The maximum size of array to allocate (unless necessary).
 * Some VMs reserve some header words in an array.
 * Attempts to allocate larger arrays may result in
 * OutOfMemoryError: Requested array size exceeds VM limit
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

扩容后的长度比MAX_ARRAY_SIZE小，符合要求返回newCapacity，如果扩容后的长度都比最大的都大，执行hegeCapacity函数，下面来看

```
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE)
        ? Integer.MAX_VALUE
        : MAX_ARRAY_SIZE;
}
```

通过判断确定返回的是Integer.MAX_VALUE还是MAX_ARRAY_SIZE;

##### 1.1.3.2 add(int index, E element)

添加元素到指定位置，平均时间复杂度为O(n)。

```
public void add(int index, E element) {
 	//越界检查
    rangeCheckForAdd(index);
    //修改计数
    modCount++;
    final int s;
    Object[] elementData;
    //检查是否需要扩容
    if ((s = size) == (elementData = this.elementData).length)
        elementData = grow();
    //将index及其之后的元素的位置向后移动一位，那么index的位置空出
    System.arraycopy(elementData, index,
                     elementData, index + 1,
                     s - index);
    // 将元素插入到index的位置
    elementData[index] = element;
    // 大小增1
    size = s + 1;
}
```

System.arraycopy  详解 ：https://blog.csdn.net/qq_32440951/article/details/78357325

```
public static void arraycopy(Object src,
                             int srcPos,
                             Object dest,
                             int destPos,
                             int length)
其中：src表示源数组，srcPos表示源数组要复制的起始位置，desc表示目标数组，length表示要复制的长度。
```

##### 1.1.3.3 addAll

```
public boolean addAll(Collection<? extends E> c) {
	//将集合转换成数组
    Object[] a = c.toArray();
    //修改计数
    modCount++;
    int numNew = a.length;
    if (numNew == 0)
        return false;
    Object[] elementData;
    final int s;
    //检查是否需要扩容
    if (numNew > (elementData = this.elementData).length - (s = size))
        elementData = grow(s + numNew);
    //将c集合的元素全部拷贝到集合尾部
    System.arraycopy(a, 0, elementData, s, numNew);
    //长度增加
    size = s + numNew;
    return true;
}
```

```
public boolean addAll(int index, Collection<? extends E> c) {
	//检查越界
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    modCount++;
    int numNew = a.length;
    if (numNew == 0)
        return false;
    Object[] elementData;
    final int s;
    if (numNew > (elementData = this.elementData).length - (s = size))
        elementData = grow(s + numNew);
	
    int numMoved = s - index;
    //插入位置是否存在
    if (numMoved > 0)
    	//将index及其之后的元素的位置向后移动numNew位，那么index的位置（numNew）长度的空出
        System.arraycopy(elementData, index,
                         elementData, index + numNew,
                         numMoved);
     //将c集合的元素全部拷贝到从index开始numNew 个位置                  
    System.arraycopy(a, 0, elementData, index, numNew);
    size = s + numNew;
    return true;
}
```

#### 1.1.4 get(int index)

获取指定索引位置的元素，时间复杂度为O(1)。

```
public E get(int index) {
	//检查index是否在 集合范围内
    Objects.checkIndex(index, size);
    返回数组index位置的元素
    return elementData(index);
}
```

```
// Positional Access Operations

@SuppressWarnings("unchecked")
E elementData(int index) {
    return (E) elementData[index];
}
```

（1）检查索引是否越界，这里只检查是否越上界，如果越上界抛出IndexOutOfBoundsException异常，如果越下界抛出的是ArrayIndexOutOfBoundsException异常。

（2）返回索引位置处的元素；

#### 1.1.5  remove

##### 1.1.5.1 remove(int index)

删除指定索引位置的元素，时间复杂度为O(n)。

```
public E remove(int index) {
// 检查是否越界
    Objects.checkIndex(index, size);
    final Object[] es = elementData;

    @SuppressWarnings("unchecked") E oldValue = (E) es[index];
    fastRemove(es, index);
//返回删除的元素。
    return oldValue;
}
private void fastRemove(Object[] es, int i) {
        modCount++;
        final int newSize;
        // 如果index不是最后一位，则将index之后的元素往前挪一位
        if ((newSize = size - 1) > i)
            System.arraycopy(es, i + 1, es, i, newSize - i);
         // 将最后一个元素删除，帮助GC
        es[size = newSize] = null;
}

```



##### 1.1.5.2 remove(Object o)

删除指定索引位置的元素，时间复杂度为O(n)。

```
public boolean remove(Object o) {
    final Object[] es = elementData;
    final int size = this.size;
    int i = 0;
    found: {
        if (o == null) {
            for (; i < size; i++)
                if (es[i] == null)
                    break found;
        } else {
            for (; i < size; i++)
                if (o.equals(es[i]))
                    break found;
        }
        return false;
    }
    fastRemove(es, i);
    return true;
}
```

（1）找到第一个等于指定元素值的元素；java label 表达式

（2）快速删除；

*fastRemove(int index)相对于remove(int index)少了检查索引越界的操作，可见jdk将性能优化到极致。*

##### 1.1.5.3 removeAll(Collection<?> c)

```
public boolean removeAll(Collection<?> c) {
	//批量移除，c 移除集合；false 不补全移除的位置，，从0，到最后
    return batchRemove(c, false, 0, size);
}
 boolean batchRemove(Collection<?> c, boolean complement,
                        final int from, final int end) {
        Objects.requireNonNull(c);
        //源数组
        final Object[] es = elementData;
        int r;
        // Optimize for initial run of survivors
        //从起始位置遍历到末尾，在结束之前能够找到可移除的元素，则跳出循环，并记录第一个遍历到的移除的元素，否则返回false
        for (r = from;; r++) {
            if (r == end)
                return false;
            if (c.contains(es[r]) != complement)
                break;
        }
        //记录第一个移除元素在源数组中的位置
        int w = r++;
        try {
        	//从上面的位置开始遍历至结尾，将所有不移除的元素放到数组的前段（从0~n，n是分隔保存和移除元素的位置）
            for (Object e; r < end; r++)
                if (c.contains(e = es[r]) == complement)
                    es[w++] = e;
        } catch (Throwable ex) {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            System.arraycopy(es, r, es, w, end - r);
            w += end - r;
            throw ex;
        } finally {
        	//记录删除的元素个数到 modCount中
            modCount += end - w;
            shiftTailOverGap(es, w, end);
        }
        return true;
    }
    //通过向下滑动以下元素，将间隙从lo移到hi
    private void shiftTailOverGap(Object[] es, int lo, int hi) {
    	//原数组起始位置hi，目标数组起始位置lo,要copy的数组的长度
        System.arraycopy(es, hi, es, lo, size - hi);//数组不变
        //从w 开始循环，元素设置为空
        for (int to = size, i = (size -= hi - lo); i < to; i++)
            es[i] = null;
    }
```

#### 1.1.6 retainAll 交集

```
public boolean retainAll(Collection<?> c) {
//待处理集合c,complement 用来区分 找相同元素还是不同元素，true 找相同的元素，去掉不同的元素，false相反
    return batchRemove(c, true, 0, size);
}
```

（1）遍历elementData数组；

（2）如果元素在c中，则把这个元素添加到elementData数组的w位置并将w位置往后移一位；

（3）遍历完之后，w之前的元素都是两者共有的，w之后（包含）的元素不是两者共有的；

（4）将w之后（包含）的元素置为null，方便GC回收；

#### 1.1.7 transient  Object[] elementData

*elementData设置成了transient，那ArrayList是怎么把元素序列化的呢？*

这个关键字的作用: 将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会被序列化。

```
将{@code ArrayList}实例的状态保存到一个流中(即序列化它)。
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException {
    // Write out element count, and any hidden stuff
     // 防止序列化期间有修改
    int expectedModCount = modCount;
     // 写出非transient非static属性（会写出size属性）
    s.defaultWriteObject();

    // Write out size as capacity for behavioral compatibility with clone()
     // 写出元素个数
    s.writeInt(size);

    // Write out all elements in the proper order.
     // 依次写出元素
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }
 // 如果有修改，抛出异常
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```



```
从流中重新构造{@code ArrayList}实例(即反序列化它)。
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {

    // Read in size, and any hidden stuff
      // 读入非transient非static属性（会读取size属性）
    s.defaultReadObject();

    // Read in capacity
     // 读入元素个数，没什么用，只是因为写出的时候写了size属性，读的时候也要按顺序来读
    s.readInt(); // ignored

    if (size > 0) {
        // like clone(), allocate array based upon size not capacity
        // 计算容量，  // 检查是否需要扩容
        SharedSecrets.getJavaObjectInputStreamAccess().checkArray(s, Object[].class, size);
        Object[] elements = new Object[size];

        // Read in all elements in the proper order.
         // 依次读取元素到数组中
        for (int i = 0; i < size; i++) {
            elements[i] = s.readObject();
        }

        elementData = elements;
    } else if (size == 0) {
        elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new java.io.InvalidObjectException("Invalid size: " + size);
    }
}
```

查看writeObject()方法可知，先调用s.defaultWriteObject()方法，再把size写入到流中，再把元素一个一个的写入到流中。

一般地，只要实现了Serializable接口即可自动序列化，writeObject()和readObject()是为了自己控制序列化的方式，这两个方法必须声明为private，在java.io.ObjectStreamClass#getPrivateMethod()方法中通过反射获取到writeObject()这个方法。

在ArrayList的writeObject()方法中先调用了s.defaultWriteObject()方法，这个方法是写入非static非transient的属性，在ArrayList中也就是size属性。同样地，在readObject()方法中先调用了s.defaultReadObject()方法解析出了size属性。

elementData定义为transient的优势，自己根据size序列化真实的元素，而不是根据数组的长度序列化元素，减少了空间占用。

### 1.2 LinkedList

LinkedList是一个以双向链表实现的List，它除了作为List使用，还可以作为队列或者栈来使用

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/LinkedList.png)

通过继承体系，我们可以看到LinkedList不仅实现了List接口，还实现了Queue和Deque接口，所以它既能作为List使用，也能作为双端队列使用，当然也可以作为栈使用。

#### 1.2.1 属性

```
   元素个数
   transient int size = 0;

    /**
     * 链表首节点
     */
    transient Node<E> first;

    /**
     * 链表尾节点
     */
    transient Node<E> last;
```

#### 1.2.2 内部结构&构造方法

```
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

典型的双链表结构。

```
/**
 * Constructs an empty list.
 */
public LinkedList() {
}

/**
 * Constructs a list containing the elements of the specified
 * collection, in the order they are returned by the collection's
 * iterator.
 *
 * @param  c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

两个构造方法也很简单，可以看出是一个无界的队列。

#### 1.2.3 添加元素

作为一个双端队列，添加元素主要有两种，一种是在队列尾部添加元素，一种是在队列首部添加元素，这两种形式在LinkedList中主要是通过下面两个方法来实现的。

```
/**
 * 从队列首添加元素
 */
private void linkFirst(E e) {
	// 首节点
    final Node<E> f = first;
     // 创建新节点，新节点的next是首节点
    final Node<E> newNode = new Node<>(null, e, f);
    // 让新节点作为新的首节点
    first = newNode;
    // 判断是不是第一个添加的元素
    // 如果是就把last也置为新节点
    // 否则把原首节点的prev指针置为新节点
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
     // 元素个数加1
    size++;
     // 修改次数加1，说明这是一个支持fail-fast的集合
    modCount++;
}

/**
 * Links e as last element.
 */
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}

```



```
public void addFirst(E e) {
    linkFirst(e);
}

public void addLast(E e) {
    linkLast(e);
}

// 作为无界队列，添加元素总是会成功的
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}

public boolean offerLast(E e) {
    addLast(e);
    return true;
}
```

上面是作为双端队列来看，它的添加元素分为首尾添加元素，那么，作为List呢？

作为List，是要支持在中间添加元素的，主要是通过下面这个方法实现的。

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/LinkedList-1.png)

在succ节点之前增加 节点

```
/**
 * Inserts element e before non-null Node succ.
 */
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

获取指定位置的节点

```
/**
 * Returns the (non-null) Node at the specified element index.
 */
Node<E> node(int index) {
    // assert isElementIndex(index);
	// 因为是双链表
    // 所以根据index是在前半段还是后半段决定从前遍历还是从后遍历
    // 这样index在后半段的时候可以少遍历一半的元素
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

```
在指定index位置处添加元素
public void add(int index, E element) {
    // 判断是否越界
    checkPositionIndex(index);
    // 如果index是在队列尾节点之后的一个位置
    // 把新节点直接添加到尾节点之后
    // 否则调用linkBefore()方法在中间添加节点
    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
```

在队列首尾添加元素很高效，时间复杂度为O(1)。

在中间添加元素比较低效，首先要先找到插入位置的节点，再修改前后节点的指针，时间复杂度为O(n)。

#### 1.2.4 删除元素

![](https://gitee.com/zhouxiaoliang/img/raw/master/img/LinkedList-2.png)

在队列首尾删除元素很高效，时间复杂度为O(1)。

在中间删除元素比较低效，首先要找到删除位置的节点，再修改前后指针，时间复杂度为O(n)。

#### 1.2.5 栈

```
public void push(E e) {
    addFirst(e);
}

public E pop() {
    return removeFirst();
}
```

栈的特性是LIFO(Last In First Out)，所以作为栈使用也很简单，添加删除元素都只操作队列首节点即可。

### 1.3 CopyOnWriteArrayList

CopyOnWriteArrayList是ArrayList的线程安全版本，内部也是通过数组实现，每次对数组的修改都完全拷贝一份新的数组来修改，修改完了再替换掉老数组，这样保证了只阻塞写操作，不阻塞读操作，实现读写分离。

![diagram](https://gitee.com/zhouxiaoliang/img/raw/master/img/diagram.png)

CopyOnWriteArrayList实现了List, RandomAccess, Cloneable, java.io.Serializable等接口。

CopyOnWriteArrayList实现了List，提供了基础的添加、删除、遍历等操作。

CopyOnWriteArrayList实现了RandomAccess，提供了随机访问的能力。

CopyOnWriteArrayList实现了Cloneable，可以被克隆。

CopyOnWriteArrayList实现了Serializable，可以被序列化。

#### 1.3.1 属性

```
/** 用于修改时加锁 */
final transient ReentrantLock lock = new ReentrantLock();

/** 真正存储元素的地方，只能通过getArray()/setArray()访问 */
private transient volatile Object[] array;
```

jdk 9 中已经修改为：

```
/**
 * The lock protecting all mutators.  (We have a mild preference
 * for builtin monitors over ReentrantLock when either will do.)
 */
 保护所有变异体的锁，当两者都可以使用时，我们倾向于使用内置监视器而不是ReentrantLock。
final transient Object lock = new Object();

/** The array, accessed only via getArray/setArray. */
private transient volatile Object[] array;
```

重置锁方法也做了更新，9.0 新增VarHandle 类处理释放锁。

```
public Object clone() {
    try {
        @SuppressWarnings("unchecked")
        CopyOnWriteArrayList<E> clone =
            (CopyOnWriteArrayList<E>) super.clone();
        clone.resetLock();
        // Unlike in readObject, here we cannot visibility-piggyback on the
        // volatile write in setArray().
        VarHandle.releaseFence();
        return clone;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError();
    }
}
```

```
 /** Initializes the lock; for use when deserializing or cloning. */
    private void resetLock() {
        Field lockField = java.security.AccessController.doPrivileged(
            (java.security.PrivilegedAction<Field>) () -> {
                try {
                    Field f = CopyOnWriteArrayList.class
                        .getDeclaredField("lock");
                    f.setAccessible(true);
                    return f;
                } catch (ReflectiveOperationException e) {
                    throw new Error(e);
                }});
        try {
            lockField.set(this, new Object());
        } catch (IllegalAccessException e) {
            throw new Error(e);
        }
    }
}
```

jdk8.0

```
public Object clone() {
    try {
        @SuppressWarnings("unchecked")
        CopyOnWriteArrayList<E> clone =
            (CopyOnWriteArrayList<E>) super.clone();
        clone.resetLock();
        return clone;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError();
    }
}
```

```
// Support for resetting lock while deserializing
private void resetLock() {
    UNSAFE.putObjectVolatile(this, lockOffset, new ReentrantLock());
}
private static final sun.misc.Unsafe UNSAFE;
private static final long lockOffset;
static {
    try {
        UNSAFE = sun.misc.Unsafe.getUnsafe();
        Class<?> k = CopyOnWriteArrayList.class;
        lockOffset = UNSAFE.objectFieldOffset
            (k.getDeclaredField("lock"));
    } catch (Exception e) {
        throw new Error(e);
    }
}
```

知识点： UNSAFE 与 VarHandle

言归正传。

（1）lock

用于修改时加锁，使用transient修饰表示不自动序列化。

（2）array

真正存储元素的地方，使用transient修饰表示不自动序列化，使用volatile修饰表示一个线程对这个字段的修改另外一个线程立即可见。

#### 1.3.2 构造方法

创建空数组。

```
public CopyOnWriteArrayList() {
    // 所有对array的操作都是通过setArray()和getArray()进行
    setArray(new Object[0]);
}

final void setArray(Object[] a) {
    array = a;
}
```

如果c是CopyOnWriteArrayList类型，直接把它的数组赋值给当前list的数组，注意这里是浅拷贝，两个集合共用同一个数组。

如果c不是CopyOnWriteArrayList类型，则进行拷贝把c的元素全部拷贝到当前list的数组中。

```
public CopyOnWriteArrayList(Collection<? extends E> c) {
    Object[] elements;
    if (c.getClass() == CopyOnWriteArrayList.class)
        // 如果c也是CopyOnWriteArrayList类型
        // 那么直接把它的数组拿过来使用
        elements = ((CopyOnWriteArrayList<?>)c).getArray();
    else {
        // 否则调用其toArray()方法将集合元素转化为数组
        elements = c.toArray();
        // 这里c.toArray()返回的不一定是Object[]类型
        // 详细原因见ArrayList里面的分析
        if (elements.getClass() != Object[].class)
            elements = Arrays.copyOf(elements, elements.length, Object[].class);
    }
    setArray(elements);
}
```

把toCopyIn的元素拷贝给当前list的数组。

```
public CopyOnWriteArrayList(E[] toCopyIn) {
    setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
}
```

#### 1.3.3 添加元素

add(E e)方法

添加一个元素到末尾。

```
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        // 获取旧数组
        Object[] elements = getArray();
        int len = elements.length;
        // 将旧数组元素拷贝到新数组中
        // 新数组大小是旧数组大小加1
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // 将元素放在最后一位
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```

（1）加锁；

（2）获取元素数组；

（3）新建一个数组，大小为原数组长度加1，并把原数组元素拷贝到新数组；

（4）把新添加的元素放到新数组的末尾；

（5）把新数组赋值给当前对象的array属性，覆盖原数组；

（6）解锁；

add(int index, E element)方法

添加一个元素在指定索引处。

```
public void add(int index, E element) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        // 获取旧数组
        Object[] elements = getArray();
        int len = elements.length;
        // 检查是否越界, 可以等于len
        if (index > len || index < 0)
            throw new IndexOutOfBoundsException("Index: "+index+
                                                ", Size: "+len);
        Object[] newElements;
        int numMoved = len - index;
        if (numMoved == 0)
            // 如果插入的位置是最后一位
            // 那么拷贝一个n+1的数组, 其前n个元素与旧数组一致
            newElements = Arrays.copyOf(elements, len + 1);
        else {
            // 如果插入的位置不是最后一位
            // 那么新建一个n+1的数组
            newElements = new Object[len + 1];
            // 拷贝旧数组前index的元素到新数组中
            System.arraycopy(elements, 0, newElements, 0, index);
            // 将index及其之后的元素往后挪一位拷贝到新数组中
            // 这样正好index位置是空出来的
            System.arraycopy(elements, index, newElements, index + 1,
                             numMoved);
        }
        // 将元素放置在index处
        newElements[index] = element;
        setArray(newElements);
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```

（1）加锁；

（2）检查索引是否合法，如果不合法抛出IndexOutOfBoundsException异常，注意这里index等于len也是合法的；

（3）如果索引等于数组长度（也就是数组最后一位再加1），那就拷贝一个len+1的数组；

（4）如果索引不等于数组长度，那就新建一个len+1的数组，并按索引位置分成两部分，索引之前（不包含）的部分拷贝到新数组索引之前（不包含）的部分，索引之后（包含）的位置拷贝到新数组索引之后（不包含）的位置，索引所在位置留空；

（5）把索引位置赋值为待添加的元素；

（6）把新数组赋值给当前对象的array属性，覆盖原数组；

（7）解锁；

addIfAbsent(E e)方法

添加一个元素如果这个元素不存在于集合中。

```
public boolean addIfAbsent(E e) {
    // 获取元素数组, 取名为快照
    Object[] snapshot = getArray();
    // 检查如果元素不存在,直接返回false
    // 如果存在再调用addIfAbsent()方法添加元素
    return indexOf(e, snapshot, 0, snapshot.length) >= 0 ? false :
        addIfAbsent(e, snapshot);
}

private boolean addIfAbsent(E e, Object[] snapshot) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        // 重新获取旧数组
        Object[] current = getArray();
        int len = current.length;
        // 如果快照与刚获取的数组不一致
        // 说明有修改
        if (snapshot != current) {
            // 重新检查元素是否在刚获取的数组里
            int common = Math.min(snapshot.length, len);
            for (int i = 0; i < common; i++)
                // 到这个方法里面了, 说明元素不在快照里面
                if (current[i] != snapshot[i] && eq(e, current[i]))
                    return false;
            if (indexOf(e, current, common, len) >= 0)
                    return false;
        }
        // 拷贝一份n+1的数组
        Object[] newElements = Arrays.copyOf(current, len + 1);
        // 将元素放在最后一位
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```

（1）检查这个元素是否存在于数组快照中；

（2）如果存在直接返回false，如果不存在调用addIfAbsent(E e, Object[] snapshot)处理;

（3）加锁；

（4）如果当前数组不等于传入的快照，说明有修改，检查待添加的元素是否存在于当前数组中，如果存在直接返回false;

（5）拷贝一个新数组，长度等于原数组长度加1，并把原数组元素拷贝到新数组中；

（6）把新元素添加到数组最后一位；

（7）把新数组赋值给当前对象的array属性，覆盖原数组；

（8）解锁；

#### 1.3.4 获取元素

get(int index)

获取指定索引的元素，支持随机访问，时间复杂度为O(1)。

```
public E get(int index) {
    // 获取元素不需要加锁
    // 直接返回index位置的元素
    // 这里是没有做越界检查的, 因为数组本身会做越界检查
    return get(getArray(), index);
}

final Object[] getArray() {
    return array;
}

private E get(Object[] a, int index) {
    return (E) a[index];
}
```

（1）获取元素数组；

（2）返回数组指定索引位置的元素；

#### 1.3.5 移除元素

删除指定索引位置的元素。

```
public E remove(int index) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        // 获取旧数组
        Object[] elements = getArray();
        int len = elements.length;
        E oldValue = get(elements, index);
        int numMoved = len - index - 1;
        if (numMoved == 0)
            // 如果移除的是最后一位
            // 那么直接拷贝一份n-1的新数组, 最后一位就自动删除了
            setArray(Arrays.copyOf(elements, len - 1));
        else {
            // 如果移除的不是最后一位
            // 那么新建一个n-1的新数组
            Object[] newElements = new Object[len - 1];
            // 将前index的元素拷贝到新数组中
            System.arraycopy(elements, 0, newElements, 0, index);
            // 将index后面(不包含)的元素往前挪一位
            // 这样正好把index位置覆盖掉了, 相当于删除了
            System.arraycopy(elements, index + 1, newElements, index,
                             numMoved);
            setArray(newElements);
        }
        return oldValue;
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```

（1）加锁；

（2）获取指定索引位置元素的旧值；

（3）如果移除的是最后一位元素，则把原数组的前len-1个元素拷贝到新数组中，并把新数组赋值给当前对象的数组属性；

（4）如果移除的不是最后一位元素，则新建一个len-1长度的数组，并把原数组除了指定索引位置的元素全部拷贝到新数组中，并把新数组赋值给当前对象的数组属性；

（5）解锁并返回旧值；

#### 1.3.6 size方法

返回数组的长度。

```
public int size() {
    // 获取元素个数不需要加锁
    // 直接返回数组的长度
    return getArray().length;
}
```