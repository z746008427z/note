## lambda表达式 与 函数式接口

lambda 表达式，也称为闭包。允许将函数当成参数传递给某个方法，或者把代码本身当作数据处理。

```java
Arrays.asList("a", "b", "d").forEach(e -> System.out.println(e));
```

也可以显示指定该参数的类型。

```java
Arrays.asList("a", "b", "d").forEach((String e) -> System.out.println(e));
```

如果是更复杂的语句块，可以用 花括号 {} 将语句块括起来。类似 java 中的函数体。

```java
Arrays.asList("a", "b", "d").forEach((String e) -> {
            System.out.println(e);
            System.out.println(e);
        });
```

可以引用类成员和局部变量（会将这些变量隐式的转换成 final 的）。

```java
String separator = ",";
Arrays.asList("a", "b", "d").forEach(
    (String e) -> System.out.print(e + separator));

// 等同
final String separator = ",";
Arrays.asList("a", "b", "d").forEach(
    (String e) -> System.out.print(e + separator));
```

可以有返回值，返回值类型也由编译器推理得出，如果只有一行，可以不用使用 return 语句。

```java
Arrays.asList("a", "b", "d").sort((e1, e2) -> e1.compareTo(e2));
// 等同
Arrays.asList("a", "b", "d").sort((e1, e2) -> {
            int result = e1.compareTo(e2);
            return result;
        });
```



## 函数式接口

指只有一个函数的接口，这样的接口可以隐式转换为 lambda 表达式。（java.lang.Runnable）

函数式接口非常脆弱，只要接口中添加一个函数，该接口就不是函数式接口从而编译失败。为了克服脆弱性，显示说明某个接口是函数式接口，Java 8 提供了一个特殊的注解 **@FunctionalInterface**。

```java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```

默认方法和静态方法不会破坏函数式接口的定义。下面是合法的：

```java
@FunctionalInterface
public interface FunctionalDefaultMethods {
    void method();
        
    default void defaultMethod() {            
    }        
}
```



## 接口 默认方法 与 静态方法

#### 默认方法

默认方法可以在不破坏二进制兼容性的前提下，往现存接口中添加新的方法。不强制哪些实现了该接口的类也同事实现这个新加的方法。

```java
private interface Defaulable {
    // Interfaces now allow default methods, the implementer may or 
    // may not implement (override) them.
    default String notRequired() { 
        return "Default implementation"; 
    }        
}
        
private static class DefaultableImpl implements Defaulable {
}
    
private static class OverridableImpl implements Defaulable {
    @Override
    public String notRequired() {
        return "Overridden implementation";
    }
}
```

#### 静态方法

```java
private interface DefaulableFactory {
    // Interfaces now allow static methods
    static Defaulable create( Supplier< Defaulable > supplier ) {
        return supplier.get();
    }
}
```

由于 Jvm 上的默认方法的实现在字节码层面提供了支持，因此效率非常高。

默认方法允许在不打破现有集成体系的基础上改进接口。

该特性在官方库中的应用开始：给 **java.util.Colletion** 接口添加新的方法：**stream()**，**parallelStream()**，**forEach()**，**removeIf()** 等等。



## Java 官方库新特性

### Optional

Google Guava 引入了 Optional 类来解决 NullPointerException，从而避免源码被各种 null 检查污染。

Java 8 把 Optional 加入了官方库。

```java
Optional< String > fullName = Optional.ofNullable( null );
System.out.println( "Full Name is set? " + fullName.isPresent() );        
System.out.println( "Full Name: " + fullName.orElseGet( () -> "[none]" ) ); 
System.out.println( fullName.map( s -> "Hey " + s + "!" ).orElse( "Hey Stranger!" ) );
```

- 如果 **Optional** 实例持有一个非空值，则 isPresent() 方法返回 true，否则返回 false。
- **orElseGet()** 方法，**Optional** 实例持有 null，则可以接受一个 lambda 表达式生成的默认值；
- **map()** 方法可以将现有的 **Opetional** 实例的值转换成新的值；
- **orElse()** 方法与 **orElseGet()** 方法类似，但是在持有null的时候返回传入的默认值。



### Streams

```java
final Collection<Task> tasks = Arrays.asList(
                new Task(Status.OPEN, 5),
                new Task(Status.OPEN, 13),
                new Task(Status.CLOSED, 8)
        );

final long totalPointsOfOpenTasks = tasks
    .stream()
    .filter(task -> task.getStatus() == Status.OPEN)
    .mapToInt(Task::getPoints)
    .sum();
```

1. tasks 集合被转换成 stream 表示。
2. stream 上的 filter 操作回过滤掉所有 CLOSED 的task。
3. mapToInt 操作基于每个 task 实例的 Task:: getPoints 方法将 task 六砖城 Integer 集合。
4. 通过 sum 方法计算总和，得出最后结果。



#### Stream 中间操作 与 晚期操作。

**中期操作**回返回一个新的 stream——执行一个中间操作（filter）并不会执行实际的过滤操作，而是创建一个新的 stream，并将原 stream 中符合条件的元素放入新创建的 stream。

晚期操作（**forEach** / **sum**）遍历 stream 并得出结果或者附带结果。在执行晚期操作之后，stream 处理线已经处理完毕，在几乎所有情况下，晚期操作都是立刻堆 stream 进行遍历。

#### Stream 并行处理

支持并行处理（parallel processing）

```java
// Calculate total points of all tasks
final double totalPoints = tasks
                .stream()
                .parallel()
                .map(task -> task.getPoints()) // or map( Task::getPoints ) 
                .reduce(0, Integer::sum);

System.out.println("Total points (all tasks): " + totalPoints);
```

#### Stream 元素分组

```java
// Group tasks by their status
final Map<Status, List<Task>> map = tasks
                .stream()
                .collect(Collectors.groupingBy(Task::getStatus));
System.out.println(map);

{CLOSED=[[CLOSED, 8]], OPEN=[[OPEN, 5], [OPEN, 13]]}
```

#### Stream IO操作

```java
final Path path = new File(filename).toPath();
try (Stream<String> lines = Files.lines(path, StandardCharsets.UTF_8)) {
    lines.onClose(() -> System.out.println("Done!")).forEach(System.out::println);
}
```

stream 的方法 onClose 返回一个等价的有额外句柄的 stream，当 stream 的 close 方法被调用的时候这个句柄会被执行。











































