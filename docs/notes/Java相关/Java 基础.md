## 1.String相关

### String概述

String 被声明为 final，因此它不可被继承。

在 Java 8 中，String 内部使用 char 数组存储数据。

``` java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}

``` 

在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 `coder` 来标识使用了哪种编码。

``` java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}

``` 

value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

### 不可变的好处

**1. 可以缓存 hash 值**

因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

**2. String Pool 的需要**

如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

**3. 安全性**

String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。

**4. 线程安全**

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

### String Pool

字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程中将字符串添加到 String Pool 中。

当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同字符串，而 s3 和 s4 是通过 s1.intern() 方法取得一个字符串引用。intern() 首先把 s1 引用的字符串放到 String Pool 中，然后返回这个字符串引用。因此 s3 和 s4 引用的是同一个字符串。

``` java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
String s4 = s1.intern();
System.out.println(s3 == s4);           // trueCopy to clipboardErrorCopied
``` 

如果是采用 "bbb" 这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。

``` java
String s5 = "bbb";
String s6 = "bbb";
System.out.println(s5 == s6);  // trueCopy to clipboardErrorCopied
``` 

在 Java 7 之前，String Pool 被放在运行时常量池中，它属于永久代。而在 Java 7，String Pool 被移到堆中。这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。

### String、StringBuilder、StringBuffer 

`StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类，在 `AbstractStringBuilder` 中也是使用字符数组保存字符串`char[]value` 但是没有用 `final` 关键字修饰，所以这两种对象都是可变的。

`StringBuilder` 与 `StringBuffer` 的构造方法都是调用父类构造方法也就是`AbstractStringBuilder` 实现的，大家可以自行查阅源码。

``` java
AbstractStringBuilder.java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;

    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
}
``` 

### 线程安全

`String` 中的对象是不可变的，也就可以理解为常量，线程安全。

`AbstractStringBuilder` 是 `StringBuilder` 与 `StringBuffer` 的公共父类，定义了一些字符串的基本操作，如 `expandCapacity`、`append`、`insert`、`indexOf` 等公共方法。

`StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。

`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。

### 性能

每次对 `String` 类型进行改变的时候，都会生成一个新的 `String` 对象，然后将指针指向新的 `String` 对象。`StringBuffer` 每次都会对 `StringBuffer` 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 `StringBuilder` 相比使用 `StringBuffer` 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

**对于三者使用的总结：**

1. 操作少量的数据: 适用 `String`
2. 单线程操作字符串缓冲区下操作大量数据: 适用 `StringBuilder`
3. 多线程操作字符串缓冲区下操作大量数据: 适用 `StringBuffer`

### String内存

参考：<https://www.bilibili.com/video/BV1gb411F76B?p=119>

![](https://gitee.com/dukangming/PicBedGitee/raw/master/img/20200901221233.png)

### new String("abc")

使用这种方式一共会创建两个字符串对象（前提是 String Pool 中还没有 "abc" 字符串对象）。

- "abc" 属于字符串字面量，因此编译时期会在 String Pool 中创建一个字符串对象，指向这个 "abc" 字符串字面量；
- 而使用 new 的方式会在堆中创建一个字符串对象。

## 2.内存分析

参考：



### 栈stack、堆heap、方法区method area

栈：

![](https://gitee.com/dukangming/PicBedGitee/raw/master/img/20200901163558.png)

堆：

![](https://gitee.com/dukangming/PicBedGitee/raw/master/img/20200901163800.png)

方法区：

![](https://gitee.com/dukangming/PicBedGitee/raw/master/img/20200901163903.png)

### 垃圾回收原理和算法

分代垃圾回收机制，是基于这样一个事实：不同的对象的生命周期是不一样的。因此，不同生命周期的对象可以采取不同的回收算法，以便提高回收效率。我们将对象分为三种状态：年轻代、年老代、持久代。JVM将堆内存划分为 Eden、Survivor 和 Tenured/Old 空间。

1. 年轻代

　　所有新生成的对象首先都是放在Eden区。 年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象，对应的是Minor GC，每次 Minor GC 会清理年轻代的内存，算法采用效率较高的复制算法，频繁的操作，但是会浪费内存空间。当“年轻代”区域存放满对象后，就将对象存放到年老代区域。

2. 年老代

　　在年轻代中经历了N(默认15)次垃圾回收后仍然存活的对象，就会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。年老代对象越来越多，我们就需要启动Major GC和Full GC(全量回收)，来一次大扫除，全面清理年轻代区域和年老代区域。

3. 持久代

　　用于存放静态文件，如Java类、方法等。持久代对垃圾回收没有显著影响。

![](https://gitee.com/dukangming/PicBedGitee/raw/master/img/20200901170126.png)

- Minor GC:

　　用于清理年轻代区域。Eden区满了就会触发一次Minor GC。清理无用对象，将有用对象复制到“Survivor1”、“Survivor2”区中(这两个区，大小空间也相同，同一时刻Survivor1和Survivor2只有一个在用，一个为空)

- Major GC：

　　用于清理老年代区域。

- Full GC：

　　用于清理年轻代、年老代区域。 成本较高，会对系统性能产生影响。

**垃圾回收过程：**

​    1、新创建的对象，绝大多数都会存储在Eden中，

​    2、当Eden满了（达到一定比例）不能创建新对象，则触发垃圾回收（GC），将无用对象清理掉，

​           然后剩余对象复制到某个Survivor中，如S1，同时清空Eden区

​    3、当Eden区再次满了，会将S1中的不能清空的对象存到另外一个Survivor中，如S2，

​          同时将Eden区中的不能清空的对象，也复制到S1中，保证Eden和S1，均被清空。

​    4、重复多次(默认15次)Survivor中没有被清理的对象，则会复制到老年代Old(Tenured)区中，

​    5、当Old区满了，则会触发一个一次完整地垃圾回收（FullGC），之前新生代的垃圾回收称为（minorGC）

## 3.Object 常见方法

Object 类是一个特殊的类，是所有类的父类。它主要提供了以下 11 个方法：

``` java
public final native Class<?> getClass()//native方法，用于返回当前运行时对象的Class对象，使用了final关键字修饰，故不允许子类重写。

public native int hashCode() //native方法，用于返回对象的哈希码，主要使用在哈希表中，比如JDK中的HashMap。
public boolean equals(Object obj)//用于比较2个对象的内存地址是否相等，String类对该方法进行了重写用户比较字符串的值是否相等。

protected native Object clone() throws CloneNotSupportedException//naitive方法，用于创建并返回当前对象的一份拷贝。一般情况下，对于任何对象 x，表达式 x.clone() != x 为true，x.clone().getClass() == x.getClass() 为true。Object本身没有实现Cloneable接口，所以不重写clone方法并且进行调用的话会发生CloneNotSupportedException异常。

public String toString()//返回类的名字@实例的哈希码的16进制的字符串。建议Object所有的子类都重写这个方法。

public final native void notify()//native方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。

public final native void notifyAll()//native方法，并且不能重写。跟notify一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。

public final native void wait(long timeout) throws InterruptedException//native方法，并且不能重写。暂停线程的执行。注意：sleep方法没有释放锁，而wait方法释放了锁 。timeout是等待时间。

public final void wait(long timeout, int nanos) throws InterruptedException//多了nanos参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上nanos毫秒。

public final void wait() throws InterruptedException//跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念

protected void finalize() throws Throwable { }//实例被垃圾回收器回收的时候触发的操作
``` 

### ==和equals

**==** : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)。

**equals()** : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

- 情况 1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
- 情况 2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来比较两个对象的内容是否相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。
- Integer、String均已重写equals()

```  java
@Test
public void test1() {
    Integer x = new Integer(1);
    Integer y = new Integer(1);
    System.out.println(x.equals(y)); // true
    System.out.println(x == y);      // false

    String a = new String("123");
    String b = new String("123");
    System.out.println(a.equals(b)); // true
    System.out.println(a == b);      // false

    String p = "123";
    String q = "123";
    System.out.println(p.equals(q)); // true
    System.out.println(p == q);      // true
}
``` 

### hashCode 与 equals 

- hashCode() 介绍

hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个 int 整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode() 定义在 JDK 的 Object.java 中，这就意味着 Java 中的任何类都包含有 hashCode() 函数。

散列表存储的是键值对(key-value)，它的特点是：能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！（可以快速找到所需要的对象）

- 为什么要有 hashCode

**我们先以“HashSet 如何检查重复”为例子来说明为什么要有 hashCode：** 当你把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与该位置其他已经加入的对象的 hashcode 值作比较，如果没有相符的 hashcode，HashSet 会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 `equals()`方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。（摘自我的 Java 启蒙书《Head first java》第二版）。这样我们就大大减少了 equals 的次数，相应就大大提高了执行速度。

通过我们可以看出：`hashCode()` 的作用就是**获取哈希码**，也称为散列码；它实际上是返回一个 int 整数。这个**哈希码的作用**是确定该对象在哈希表中的索引位置。**`hashCode()`在散列表中才有用，在其它情况下没用**。在散列表中 hashCode() 的作用是获取对象的散列码，进而确定该对象在散列表中的位置。

- hashCode（）与 equals（）的相关规定

1. 如果两个对象相等，则 hashcode 一定也是相同的
2. 两个对象相等,对两个对象分别调用 equals 方法都返回 true
3. 两个对象有相同的 hashcode 值，它们也不一定是相等的
4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

### 深浅拷贝clone

- **浅拷贝：**被复制对象的所有变量都含有与原来的对象相同的值，而所有的对其他对象的引用仍然指向原来的对象。换言之，浅拷贝仅仅复制所拷贝的对象，而不复制它所引用的对象。

   

  ![](https://gitee.com/dukangming/PicBedGitee/raw/master/img/shadow_copy2.png)

- **深拷贝：**对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。

  ![](https://gitee.com/dukangming/PicBedGitee/raw/master/img/deep_copy2.png)



## 4.关键字

### final（finally,finalize）

**1. 数据** 

声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

- 对于基本类型，final 使数值不变；
- 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。

``` java
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```

**2. 方法** 

声明方法不能被子类重写。

private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

**3. 类** 

声明类不允许被继承。

**4. finally**

在异常处理时提供 `finally` 块来执行任何清除操作。如果抛出一个异常，那么相匹配的 `catch` 子句就会执行，然后控制就会进入 `finally` 块（如果有的话）。

在以下 4 种特殊情况下，finally块不会被执行：

- 在 `finally` 语句块中发生了异常。
- 在前面的代码中用了 `System.exit()` 退出程序。
- 程序所在的线程死亡。
- 关闭 CPU 。

**5.finalize**

`finalize` ，是方法名。

Java 允许使用 `#finalize()` 方法，在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象调用的。

- 它是在 Object 类中定义的，因此所有的类都继承了它。
- 子类覆盖 `finalize()` 方法，以整理系统资源或者执行其他清理工作。
- `#finalize()` 方法，是在垃圾收集器删除对象之前对这个对象调用的。

一般情况下，我们在业务中不会自己实现这个方法，更多是在一些框架中使用，例如 [《Netty Using finalize() to release ByteBufs》](https://github.com/netty/netty/issues/4145) 。

### staitic

1. **静态变量：**

   - **静态变量：**类所有的实例都共享静态变量，静态变量在内存中只存在一份。被static 声明的成员变量属于静态成员变量，静态变量 存放在 Java 内存区域的方法区。调用格式：`类名.静态变量名`    `类名.静态方法名()`
   - **实例变量：**每创建一个实例就会产生一个实例变量，它与该实例同生共死。

2. **静态方法：**

   静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。

3. **静态代码块:**

   静态代码块定义在类中方法外, 静态代码块在非静态代码块之前执行(静态代码块—>非静态代码块—>构造方法)。 该类不管创建多少对象，静态代码块只在类初始化时运行一次。

   ``` java
   public class A {
       static {
           System.out.println("123");
       }
   
       public static void main(String[] args) {
           A a1 = new A();
           A a2 = new A();
       }
   }
   123
   ```

4. **静态内部类（static修饰类的话只能修饰内部类）：**  

   - 非静态内部类依赖于外部类的实例，而静态内部类不需要。

   - 静态内部类不能访问外部类的非静态的变量和方法。

     ``` java
     public class OuterClass {
     
         class InnerClass {
         }
     
         static class StaticInnerClass {
         }
     
         public static void main(String[] args) {
             // InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be referenced from a static context
             OuterClass outerClass = new OuterClass();
             InnerClass innerClass = outerClass.new InnerClass();
             StaticInnerClass staticInnerClass = new StaticInnerClass();
         }
     }
     ```

     

5. **静态导包(用来导入类中的静态资源，1.5之后的新特性):** 

   格式为：`import static` 这两个关键字连用可以指定导入某个类中的指定静态资源，并且不需要使用类名调用类中静态成员，可以直接使用类中静态成员变量和成员方法。

   ``` java
   import static com.xxx.ClassName.*
   
   ```

6. **初始化顺序：**

   静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于它们在代码中的顺序。

   ``` java
   public static String staticField = "静态变量";
   static {
       System.out.println("静态语句块");
   }
   public String field = "实例变量";
   {
       System.out.println("普通语句块");
   }
   
   ```

   最后才是构造函数的初始化。

   ``` java
   public InitialOrderTest() {
       System.out.println("构造函数");
   }
   ```

   存在继承的情况下，初始化顺序为：

   - 父类（静态变量、静态语句块）
   - 子类（静态变量、静态语句块）
   - 父类（实例变量、普通语句块）
   - 父类（构造函数）
   - 子类（实例变量、普通语句块）
   - 子类（构造函数）

