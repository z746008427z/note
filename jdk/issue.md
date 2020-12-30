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



## 面向对象三个基本特征

### 封装

把客观事物封装成抽象的类，并且类可以把自己的数据和方法只让可信的类或者对象操作，对不可信的进行信息隐藏。

### 继承

可以使用现有类的所有功能，并在无需重新编写原来的类的情况下对这些功能进行扩展。

继承创建的新类称为“子类”或“派生类”。

被继承的类称为“基类”、“父类”或“超类”。

#### 特征

1. **可传递可扩展**。若类C继承类B，类B继承类A（多继承），则类C既有从类B那里继承下来的属性与方法，也有从类A那里继承下来的属性与方法，还可以有自己新定义的属性和方法。继承来的属性和方法尽管是隐式的，但仍是类C的属性和方法。
2. **可复用**。若类B继承类A，那么建立类B时只需要再描述与基类(类A)不同的少量特征(数据成员和成员方法)即可。这种做法能减小代码和数据的冗余度，大大增加程序的重用性。、
3. **可维护性**。继承通过增强一致性来减少模块间的接口和界面，大大增加了程序的易维护性。

### 多态

允许一个接口被多个同类动作使用的特性，通过传递给父类对象引用不同的子类对象从而表现出不同的行为，多态可为程序提供更好的可扩展性，同样也可以代码重用。

父类中的一个方法只有在父类中定义而在子类中没有重写的情况下，才可以被父类类型的引用调用；

父类中定义的方法，如果子类中重写了该方法，那么父类类型的引用将会调用子类中的这个方法，这就是动态连接。



## 重写 与 重载

#### 重写

重写是子类对父类的允许访问的方法的实现过程进行重新编写, 返回值和形参都不能改变。即外壳不变，核心重写！

#### 重载

重载(overloading) 是在一个类里面，方法名字相同，而参数不同。返回类型可以相同也可以不同。每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。

#### 区别

![image-20201229164042706](issue.assets/image-20201229164042706.png)

































