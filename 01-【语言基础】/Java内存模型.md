

# 相关概念

- JVM内存结构：和Java虚拟机的运行时区域有关。
- Java内存模型：和Java的并发编程有关。
- Java对象模型：和Java对象在虚拟机中的表现形式有关。

# JVM内存结构

<img src="https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200227155255-361989.png" alt="image-20200227155254338" style="zoom:80%;" /> 

- 程序计数器 PC Register

　　每个线程都有一个程序计算器，就是一个指针，指向方法区中的方法字节码（下一个将要执行的指令代码），由执行引擎读取下一条指令，是一个非常小的内存空间，几乎可以忽略不记。

　　程序计数器（Program Counter Register）是一块较小的内存空间，它的作用可以看做是当前线程所执行的字节码的行号指示器。在虚拟机的概念模型里（仅是概念模型，各种虚拟机可能会通过一些更高效的方式去实现），字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。由于Java 虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间的计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。如果线程正在执行的是一个Java 方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Natvie 方法，这个计数器值则为空（Undefined）。此内存区域是唯一一个在Java 虚拟机规范中没有规定任何OutOfMemoryError 情况的区域。

- 本地方法栈 Native Method Stack

　　Native Method Stack中登记native方法，在Execution Engine执行时加载native libraies

　　本地方法栈与虚拟机栈基本类似，区别在于虚拟机栈为虚拟机执行的java方法服务，而本地方法栈则是为Native方法服务

- 方法区 Method Area

　　用于存储虚拟机加载的：静态变量+常量+类信息+运行时常量池 （类信息：类的版本、字段、方法、接口、构造函数等描述信息 ）

　　默认最小值为16MB，最大值为64MB，可以通过`-XX:PermSize` 和 `-XX:MaxPermSize` 参数限制方法区的大小

　　对于习惯在HotSpot 虚拟机上开发和部署程序的开发者来说，很多人愿意把方法区称为“永久代”（Permanent Generation），本质上两者并不等价，仅仅是因为HotSpot 虚拟机的设计团队选择把GC 分代收集扩展至方法区，或者说使用永久代来实现方法区而已。对于其他虚拟机（如BEA JRockit、IBM J9 等）来说是不存在永久代的概念的。即使是HotSpot 虚拟机本身，根据官方发布的路线图信息，现在也有放弃永久代并“搬家”至Native Memory 来实现方法区的规划了。Java 虚拟机规范对这个区域的限制非常宽松，除了和Java 堆一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择不实现垃圾收集。相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入了方法区就如永久代的名字一样“永久”存在了。这个区域的内存回收目标主要是针对常量池的回收和对类型的卸载，一般来说这个区域的回收“成绩”比较难以令人满意，尤其是类型的卸载，条件相当苛刻，但是这部分区域的回收确实是有必要的。在Sun 公司的BUG 列表中，曾出现过的若干个严重的BUG 就是由于低版本的HotSpot 虚拟机对此区域未完全回收而导致内存泄漏。根据Java 虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出OutOfMemoryError 异常。

- 栈 JVM Stack

　　编译器可知的各种基本数据类型(`boolean`、`byte`、`char`、`short`、`int`、`float`、`long`、`double`)、对象引用(引用指针，并非对象本身)

　　栈是java 方法执行的内存模型：

　　每个方法被执行的时候 都会创建一个“栈帧”用于存储局部变量表(包括参数)、操作栈、方法出口等信息。

　　每个方法被调用到执行完的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

　　（局部变量表：存放了编译器可知的各种基本数据类型(`boolean`、`byte`、`char`、`short`、`int`、`float`、`long`、`double`)、对象引用(引用指针，并非对象本身)，

　　其中64位长度的long和double类型的数据会占用2个局部变量的空间，其余数据类型只占1个。

　　局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在栈帧中分配多大的局部变量是完全确定的，在运行期间栈帧不会改变局部变量表的大小空间）

　　栈的生命期是跟随线程的生命期，线程创建时创建，线程结束栈内存也就释放，是线程私有的。

- 堆 Java Heap

　　所有的对象实例以及数组都要在堆上分配，此内存区域的唯一目的就是存放对象实例

　　堆是Java 虚拟机所管理的内存中最大的一块。Java 堆是被所有线程共享的一块内存区域，在虚拟机启动时创建

　　堆是理解Java GC机制最重要的区域，没有之一

　　结构：新生代（Eden区+2个Survivor区） 老年代  永久代（HotSpot有）

　　**新生代**：新创建的对象——>Eden区 

　　GC之后，存活的对象由Eden区 Survivor区0进入Survivor区1  

　　再次GC，存活的对象由Eden区 Survivor区1进入Survivor区0 

　　**老年代**：对象如果在新生代存活了足够长的时间而没有被清理掉（即在几次Young GC后存活了下来），则会被复制到老年代

　　如果新创建对象比较大（比如长字符串或大数组），新生代空间不足，则大对象会直接分配到老年代上（大对象可能触发提前GC，应少用，更应避免使用短命的大对象）

　　老年代的空间一般比新生代大，能存放更多的对象，在老年代上发生的GC次数也比年轻代少

　　**永久代**：可以简单理解为方法区（本质上两者并不等价）

　　如上文所说：对于习惯在HotSpot 虚拟机上开发和部署程序的开发者来说，很多人愿意把方法区称为“永久代”，本质上两者并不等价

　　仅仅是因为HotSpot 虚拟机的设计团队选择把GC 分代收集扩展至方法区，或者说使用永久代来实现方法区而已

　　对于其他虚拟机（如BEA JRockit、IBM J9 等）来说是不存在永久代的概念的

　　即使是HotSpot 虚拟机本身，根据官方发布的路线图信息，现在也有放弃永久代并“搬家”至Native Memory 来实现方法区的规划了

　　Jdk1.6及之前：常量池分配在永久代

　　Jdk1.7：有，但已经逐步“去永久代”

　　Jdk1.8及之后：没有永久代(java.lang.OutOfMemoryError: PermGen space,这种错误将不会出现在JDK1.8中)

- 直接内存 Direct Memor

　　直接内存并不是JVM管理的内存，可以这样理解，直接内存，就是JVM以外的机器内存，比如，你有4G的内存，JVM占用了1G，则其余的3G就是直接内存

　　JDK中有一种基于通道（Channel）和缓冲区（Buffer）的内存分配方式，将由C语言实现的native函数库分配在直接内存中，用存储在JVM堆中的DirectByteBuffer来引用

　　由于直接内存收到本机器内存的限制，所以也可能出现OutOfMemoryError的异常。

# Java对象模型

![image-20200227164749523](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200227164749-24431.png) 

- Java对象自身的存储模型
- JVM会给这个类创建一个instanceKlass ,保存在方法区,用来在JVM层表示该Java类。
- 当我们在Java代码中,使用new创建一个对象的时候, JVM会创建一个
- instanceOopDesc对象,这个对象中包含了对象头以及实例数据。

# Java内存模型

- Java Memory Model
- 为什么需要内存模型，C语言不存在内存模型的概念（导致的问题）
  - 依赖处理器,不同处理器结果不一样
  - 无法保证并发安全
  - 无法保证并发安全
  - 需要一个标准,让多线程运行的结果可预期
- JMM是规范
  - 是一组规范,需要各个JVM的实现来遵守JMM规范,以便于开发者可以利用这些规范,更方便地开发多线程程序
  - 如果没有这样的一个JMM内存模型来规范,那么很可能经过了不同JVM的不同规则的重排序之后，导致不同的虚拟机上运行的结果不一样那是很大的问题
- JMM是工具类和关键字的原理
  - volatile、synchronized、 Lock等的原理都是JMM
  - 如果没有JMM，那就需要我们自己指定什么时候用内存栅栏等，那是相当麻烦的；幸好有了JMM，让我们只需要用同步工具和关键字就可以开发并发程序
- 最重要的3点内容
  - 重排序
  - 可见性
  - 原子性

# 重排序

## 重排序的代码案例

```java
/**
 * 描述：     演示重排序的现象 “直到达到某个条件才停止”，测试小概率事件
 */
public class OutOfOrderExecution {

    private static int x = 0, y = 0;
    private static int a = 0, b = 0;

    public static void main(String[] args) throws InterruptedException {
        int i = 0;
        for (; ; ) {
            i++;
            x = 0;
            y = 0;
            a = 0;
            b = 0;

            CountDownLatch latch = new CountDownLatch(3);

            Thread one = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        latch.countDown();
                        latch.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    a = 1;
                    x = b;
                }
            });
            Thread two = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        latch.countDown();
                        latch.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    b = 1;
                    y = a;
                }
            });
            two.start();
            one.start();
            latch.countDown();
            one.join();
            two.join();

            String result = "第" + i + "次（" + x + "," + y + ")";
            if (x == 0 && y == 0) {
                System.out.println(result);
                break;
            } else {
                System.out.println(result);
            }
        }
    }
}
```

- 这4行代码的执行顺序决定了最终x和y的结果, 一般共有3种情况:

  1） a=1;x=b(0);b=1;y=a(1) ，最终结果是x=0, y=1

  2） b=1;y=a(0);a=1;x=b(1) ，最终结果是x=1, y=0

  3） b=1;a=1;x=b(1);y=a(1) ，最终结果是x=1, y=1

- 实际会出现x=0 , y=0，那是因为重排序发生了, 4行代码的执行顺序的其中一种可能：

  ​	y=a;a=1;X=b;b=1;

- 什么是重排序：在线程1内部的两行代码的实际执行顺序和代码在Java文件中的顺序不一致，代码指令并不是严格按照代码语句顺序执行的，它们的顺序被改变了，这就是重排序,这里被颠倒的是y= a和b=1这两行语句。

## 重排序的好处

重排序优化了指令，明显提高了处理速度

![image-20200228101922278](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200228101923-272568.png)  

## 重排序的3种情况

- 编译器优化：包括JVM， JIT编译器等
- CPU指令重排：就算编译器不发生重排，CPU也可能对指令进行重排
- 内存的"重排序”：线程A的修改线程B却看不到，引出可见性问题

# 可见性

## 可见性的代码案例

```java
/**
 * 描述：     演示可见性带来的问题
 */
public class FieldVisibility {

    int a = 1;
    int b = 2;

    private void change() {
        a = 3;
        b = a;
    }


    private void print() {
        System.out.println("b=" + b + ";a=" + a);
    }

    public static void main(String[] args) {
        while (true) {
            FieldVisibility test = new FieldVisibility();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test.change();
                }
            }).start();

            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test.print();
                }
            }).start();
        }
    }
}
```

四种普通情况：
1) a=3,b=2
2) a=1,b=2
3) a=3,b=3

还有一种特殊情况：

a=1,b=3，第二个线程b是可见的，打印出最新值，对a不可见，打印a的原始值

## 为什么会有可见性问题

![image-20200228104639534](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200228104639-45652.png)   

CPU有多级缓存,导致读的数据过期

- 如果所有个核心都只用一个缓存,那么也就不存在内存可见性问题了
- 每个核心都会将自己需要的数据读到独占缓存中，数据修改后也是写入到缓存中,然后等待刷入到主存中。所以会导致有些核心读取的值是一个过期的值

## JMM的抽象:主内存和本地内存

Java虚拟机规范中定义了Java内存模型（Java Memory Model，JMM），用于屏蔽掉各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的并发效果，JMM规范了Java虚拟机与计算机内存是如何协同工作的：规定了一个线程如何和何时可以看到由其他线程修改过后的共享变量的值，以及在必须时如何同步的访问共享变量。

- Java作为高级语言,屏蔽了这些底层细节，用JMM定义了一套读写内存数据的规范，虽然我们不再需要关心一级缓存和二级缓存的问题但是，JMM抽象了主内存和本地内存的概念。

- 这里说的本地内存并不是真的是一块给每个线程分配的内存，而是JMM的一-个抽象，是对于寄存器、一级缓存、二级缓存等的抽象。

![image-20200228105220657](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200228105222-73838.png)  

![image-20200228105241044](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200228105248-679494.png)  

JMM有以下规定:
- 所有的变量都存储在主内存中，同时每个线程也有自己独立的工作内存，工作内存中的变量内容是主内存中的拷贝
- 线程不能直接读写主内存中的变量而是只能操作自己工作内存中的变量，然后再同步到主内存中
- 主内存是多个线程共享的,但线程间不共享工作内存，如果线程间需要通信，必须借助主内存中转来完成
- 所有的共享变量存在于主内存中，每个线程有自己的本地内存,而且线程读写共享数据也是通过本地内存交换的，所以才导致了可见性问题

## Happens- Before原则

- 单线程规则
- **锁操作( synchronized和Lock )**
- **volatile变量**
- 线程启动
- 线程join
- 传递性
- 中断
- 构造方法
- 工具类的Happens-Before原则
  - 线程安全的容器get一定能看到在此之前的put等存入动作
  - CountDownLatch
  - Semaphore
  - Future
  - 线程池
  - CyclicBarrier

## volatile关键字

- volatile是什么

  - volatile是一种同步机制，比synchronized或者Lock相关类更轻量，因为使用volatile并不会发生上下文切换等开销很大的行为
  - 如果一个变量别修饰成volatile，那么JVM就知道了这个变量可能会被并发修改
  - 但是开销小，相应的能力也小，虽然说volatile是用来同步的保证线程安全的，但是volatile做不到synchronized那样的原子保护，volatile仅在很有限的场景下才能发挥作用

- volatile的适用场合

  - 不适合场合：a++
  - 适用场合1 : boolean flag，如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全
  - 适用场合2：作为刷新之前变量的触发器

- volatile的作用

  - 可见性：读-个volatile变量之前，需要先使相应的本地缓存失效，这样就必须到主内存读取最新值,写一个volatile属性会立即刷入到主内存
  - 禁止指令重排序优化：解决单例双重锁乱序问题

- volatile和synchronized的关系

  volatile在这方面可以看做是轻量版的synchronized：如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全。

- volatile小结

  - volatile修饰符适用于以下场景：某个属性被多个线程共享，其中有一个线程修改了此属性，其他线程可以立即得到修改后的值，比如boolean flag 或者作为触发器，实现轻量级同步
  - volatile属性的读写操作都是无锁的，它不能替代synchronized，因为它没有提供原子性和互斥性。因为无锁,不需要花费时间在获取锁和释放锁上，所以说它是低成本的
  - volatile只能作用于属性,我们用volatile修饰属性，这样compilers就不会对这个属性做指令重排序
  - volatile提供了可见性，任何一个线程对其的修改将立马对其他线程可见。volatile 属性不会被线程缓存，始终从主存中读取
  - volatile 提供了 happens-before 保证,对volatile变量 v 的写入happens-before 所有其他线程后续对 v 的读操作
  - volatile可以使得long和double的赋值是原子性

## 能保证可见性的措施

- 除了volatile可以让变量保证可见性外，synchronized、Lock、 并发集合、Thread.join()和Thread.start(等都可以保证的可见性
- 具体看happens-before原则的规定

## synchronized正确理解

- synchronized不仅保证了原子性，还保证了可见性
- synchronized不仅让被保护的代码安全，还近朱者赤

# 原子性

## 什么是原子性

一系列的操作，要么全部执行成功，要么全部不执行,不会出现执行一半的情况，是不可分割的。

## Java中的原子操作

- 除long和double之外的基本类型( int, byte, boolean, short, char,float )的赋值操作
- 所有引|用reference的赋值操作，不管是32位的机器还是64位的机器
- java.concurrent.Atomic.*包中所有类的原子操作

## long和double的原子性

官方描述：

> 出于Java编程语言存储器模型的目的，对非易失性long或double值的单个写入被视为两个单独的写入：每个32位半写一个。这可能导致线程从一次写入看到64位值的前32位，而从另一次写 入看到第二次32位的情况。
> volatile long和doublevalues的写入和读取始终是原子的。
> 对引用的写入和读取始终是原子的，无论它们是实现为32位还是64位值。

- 问题描述：官方文档、对于64位的值的写入,可以分为两个32位的操作进行写入、读取错误、使用volatile解决
- 结论：在32位上的JVM上，long和double的操作不是原子的，但是在64位的JVM上是原子的
- 实际开发中：商用Java虚拟机中不会出现

## 原子操作+原子操作!=原子操作

- 简单地把原子操作组合在一起，并不能保证整体依然具有原子性
- 全同步的HashMap也不完全安全

## JAVA生成对象非原子操作

- 新建一个空的对象
- 把这个对象的地址指向p
- 执行类的构造函数

# JMM应用实例-单例模式

## 单例模式的8种写法

1. 饿汉式(静态常量) [可用]

   ```java
   /**
    * 描述：     饿汉式（静态常量）（可用）
    */
   public class Singleton1 {
   
       private final static Singleton1 INSTANCE = new Singleton1();
   
       private Singleton1() {
   
       }
   
       public static Singleton1 getInstance() {
           return INSTANCE;
       }
   }
   ```

2. 饿汉式(静态代码块) [可用]

   ```java
   /**
    * 描述：     饿汉式（静态代码块）（可用）
    */
   public class Singleton2 {
   
       private final static Singleton2 INSTANCE;
   
       static {
           INSTANCE = new Singleton2();
       }
   
       private Singleton2() {
       }
   
       public static Singleton2 getInstance() {
           return INSTANCE;
       }
   }
   ```

3. 懒汉式(线程不安全)[不可用]

   ```java
   /**
    * 描述：     懒汉式（线程不安全）
    */
   public class Singleton3 {
   
       private static Singleton3 instance;
   
       private Singleton3() {
   
       }
   
       public static Singleton3 getInstance() {
           if (instance == null) {
               instance = new Singleton3();
           }
           return instance;
       }
   }
   ```

4. 懒汉式(线程安全,同步方法)[不推荐用]

   ```java
   /**
    * 描述：     懒汉式（线程安全）（不推荐）
    */
   public class Singleton4 {
   
       private static Singleton4 instance;
   
       private Singleton4() {
   
       }
   
       public synchronized static Singleton4 getInstance() {
           if (instance == null) {
               instance = new Singleton4();
           }
           return instance;
       }
   }
   ```

5. 懒汉式(线程不安全,同步代码块)[不可用]

   ```java
   /**
    * 描述：     懒汉式（线程不安全）（不推荐）
    */
   public class Singleton5 {
   
       private static Singleton5 instance;
   
       private Singleton5() {
   
       }
   
       public static Singleton5 getInstance() {
           if (instance == null) {
               synchronized (Singleton5.class) {
                   instance = new Singleton5();
               }
           }
           return instance;
       }
   }
   ```

6. **双重检查[推荐用]**

   - 优点：线程安全、延迟加载、效率较高

   - 为什么要double-check
     1.线程安全
     2.单check行不行?
     3.性能问题

   - 为什么要用volatile

     1.新建对象实际上有3个步骤
     2.重排序会带来NPE
     3.防止重排序

   ```java
   /**
    * 描述：     双重检查（推荐面试使用）
    */
   public class Singleton6 {
   
       private volatile static Singleton6 instance;
   
       private Singleton6() {
   
       }
   
       public static Singleton6 getInstance() {
           if (instance == null) {
               synchronized (Singleton6.class) {
                   if (instance == null) {
                       instance = new Singleton6();
                   }
               }
           }
           return instance;
       }
   }
   ```

7. 静态内部类[推荐用]

   ```java
   /**
    * 描述：     静态内部类方式，可用
    */
   public class Singleton7 {
   
       private Singleton7() {
       }
   
       private static class SingletonInstance {
   
           private static final Singleton7 INSTANCE = new Singleton7();
       }
   
       public static Singleton7 getInstance() {
           return SingletonInstance.INSTANCE;
       }
   }
   ```

8. **枚举[推荐用]**

   ```java
   /**
    * 描述：     枚举单例
    */
   public enum Singleton8 {
       INSTANCE;
   
       public void whatever() {
   
       }
   }
   ```

## 不同单例模式写法对比

- 饿汉：简单，但是没有lazy loading
- 懒汉：有线程安全问题
- 静态内部类：可用
- 双重检查：面试用
- 枚举：最好，Joshua Bloch大神在《Effective Java》中明确表达过的观点：“使用枚举实现单例的方法虽然还没有广泛采用，但是单元素的枚举类型已经成为实现Singleton的最佳方法
  - 写法简单
  - 线程安全有保障
  - 避免反序列化破坏单例

## 总结

- 最好的方法是利用枚举，因为还可以防止反序列化重新创建新的对象
- 非线程同步的方法不能使用
- 如果程序一开始要加载的资源太多，那么就应该使用懒加载
- 饿汉式如果是对象的创建需要配置文件就不适用
- 懒加载虽然好，但是静态内部类这种方式会引|入编程复杂性

