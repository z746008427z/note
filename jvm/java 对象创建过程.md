## Java 对象创建时机

一个对象在可以被使用之前必须要被正确地实例化。

- **使用 new 关键字创建对象**

  ​		最常见的也是最简单的创建对象的方式，通过这种方式可以调用任意的构造函数去创建对象。

  ```java
  Student student = new Student();
  ```

- **使用 Class 类的 newInstance 方法（反射机制）**

  ​		可以通过 Java 的反射机制使用 Class 类的 newInstance 方法来创建对象，事实上，这个 newInstance 方法调用无参的构造器创建对象。

  ```java
  Student student = (Student) Class.forName("Student类全限定名").newInstance();
  或
  Student student = Student.class.newInstance();
  ```

- **使用 Constructor 类的 newInstance 方法（反射机制）**

  ​		java.lang.relect.Constructor 类里也有一个 newInstance 方法可以创建对象，该方法和 Class 类中的newInstance 方法很像，但是相比之下，Constructor 类的 newInstance 方法更加强大些，我们可以通过这个 newInstance 方法调用有参数的和私有的构造函数。

  ```java
  public class Student {
  
      private int id;
  
      public Student(Integer id) {
          this.id = id;
      }
  
      public static void main(String[] args) throws Exception {
  
          Constructor<Student> constructor = Student.class
                  .getConstructor(Integer.class);
          Student stu3 = constructor.newInstance(123);
      }
  }
  ```

- **使用 clone 方法创建对象**

  ​		用 clone 方法创建对象的过程中并不会调用任何构造函数。简单而言，要想使用 clone 方法，我们就必须先实现 Cloneable 接口并实现其定义的 clone 方法，这也是原型模式的应用。

  ```java
  public class Student implements Cloneable{
  
      private int id;
  
      public Student(Integer id) {
          this.id = id;
      }
  
      @Override
      protected Object clone() throws CloneNotSupportedException {
          // TODO Auto-generated method stub
          return super.clone();
      }
  
      public static void main(String[] args) throws Exception {
  
          Constructor<Student> constructor = Student.class
                  .getConstructor(Integer.class);
          Student stu3 = constructor.newInstance(123);
          Student stu4 = (Student) stu3.clone();
      }
  }
  ```

- **使用（反）序列化机制创建对象**

  ​		当我们反序列化一个对象时，jvm 会给我们创建一个单独的对象，在此过程中，jvm 并不会调用任何构造函数。为了反序列化一个对象，我们需要让我们的类实现 Serializable 接口

  ```java
  public class Student implements Cloneable, Serializable {
  
      private int id;
  
      public Student(Integer id) {
          this.id = id;
      }
  
      @Override
      public String toString() {
          return "Student [id=" + id + "]";
      }
  
      public static void main(String[] args) throws Exception {
  
          Constructor<Student> constructor = Student.class
                  .getConstructor(Integer.class);
          Student stu3 = constructor.newInstance(123);
  
          // 写对象
          ObjectOutputStream output = new ObjectOutputStream(
                  new FileOutputStream("student.bin"));
          output.writeObject(stu3);
          output.close();
  
          // 读对象
          ObjectInputStream input = new ObjectInputStream(new FileInputStream(
                  "student.bin"));
          Student stu5 = (Student) input.readObject();
          System.out.println(stu5);
      }
  }
  ```

---



## Java 对象创建过程

​		当一个对象被创建时，虚拟机就会为其分配内存来存放对象自己的实例变量及其从父类继承过来的实例变量(即使这些从超类继承过来的实例变量有可能被隐藏也会被分配空间)。

​		**在为这些实例变量分配内存的同时，这些实例变量也会被赋予默认值(零值)。**在内存分配完成之后，Java虚拟机就会开始对新创建的对象按照程序猿的意志进行初始化。

​		在Java对象初始化过程中，主要涉及三种执行对象初始化的结构，分别是 **实例变量初始化**、**实例代码块初始化** 以及 **构造函数初始化**。



#### 实例变量初始化与实例代码块初始化

​		如果我们对实例变量直接赋值或者使用实例代码块赋值，那么编译器会将其中的代码放到类的构造函数中去，并且这些代码会被放在对超类构造函数的调用语句之后(还记得吗？Java要求构造函数的第一条语句必须是超类构造函数的调用语句)，构造函数本身的代码之前。



#### 构造函数初始化

​		每一个Java中的对象都至少会有一个构造函数，如果我们没有显式定义构造函数，那么它将会有一个默认无参的构造函数。在编译生成的字节码中，这些构造函数会被命名成<init>()方法，参数列表与Java语言书写的构造函数的参数列表相同。

**Java要求在实例化类之前，必须先实例化其超类，以保证所创建实例的完整性。**



---

## 类的初始化时机与过程

​		在类加载过程中，准备阶段是正式为类变量 (static 成员变量) **分配内存**并**设置类变量初始值（零值）**的阶段，而初始化阶段是真正开始**执行类中定义的 java 程序代码**(字节码)并按程序猿的意图去初始化类变量的过程。更直接地说，初始化阶段就是执行类构造器<clinit>()方法的过程。

​		类构造器<clinit>()与实例构造器<init>()不同，它不需要程序员进行显式调用，虚拟机会保证在子类类构造器<clinit>()执行之前，父类的类构造<clinit>()执行完毕。

​		在一个类的生命周期中，**类构造器<clinit>()最多会被虚拟机调用一次，而实例构造器<init>()则会被虚拟机调用多次**，只要程序员还在创建对象。



**类实例化一般过程**：

1. 父类的类构造器 <clinit>()
2. 子类的类构造器 <clinit>()
3. 父类的成员变量和实例代码块
4. 父类的构造函数
5. 子类的成员变量和实例代码块
6. 子类的构造函数。



> 引用：https://cloud.tencent.com/developer/article/1177048