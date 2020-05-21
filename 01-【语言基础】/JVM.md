# 类加载器

- 启动类加载器：Bootstrap ClassLoader，负责加载存放在JDK\jre\lib(JDK代表JDK的安装目录，下同)下，或被-Xbootclasspath参数指定的路径中的，并且能被虚拟机识别的类库
- 扩展类加载器：Extension ClassLoader，该加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载DK\jre\lib\ext目录中，或者由java.ext.dirs系统变量指定的路径中的所有类库（如javax.*开头的类），开发者可以直接使用扩展类加载器。
- 应用程序类加载器：Application ClassLoader，该类加载器由sun.misc.Launcher$AppClassLoader来实现，它负责加载用户类路径（ClassPath）所指定的类，开发者可以直接使用该类加载器

# 双亲委派模式

Java是一门面向对象的语言，而对象又必然依托于类。类要运行，必须首先被加载到内存。我们可以简单地把类分为几类：

Java自带的核心类
Java支持的可扩展类
我们自己编写的类
如果所有的类都使用一个类加载器来加载，会出现什么问题呢？

假如我们自己编写一个类`java.util.Object`，它的实现可能有一定的危险性或者隐藏的bug。而我们知道Java自带的核心类里面也有`java.util.Object`，如果JVM启动的时候先行加载的是我们自己编写的`java.util.Object`，那么就有可能出现安全问题！

所以，Sun（后被Oracle收购）采用了另外一种方式来保证最基本的、也是最核心的功能不会被破坏。你猜的没错，那就是双亲委派模式！

双亲委派模式对类加载器定义了层级，每个类加载器都有一个父类加载器。在一个类需要加载的时候，首先委派给父类加载器来加载，而父类加载器又委派给祖父类加载器来加载，以此类推。如果父类及上面的类加载器都加载不了，那么由当前类加载器来加载，并将被加载的类缓存起来。

我们画一个图来辅助我们理解！

![image-20191205112531004](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20191205143618-250940.png)


类加载模型
Java自带的核心类 -- 由启动类加载器加载
Java支持的可扩展类 -- 由扩展类加载器加载
我们自己编写的类 -- 默认由应用程序类加载器或其子类加载
双亲委派模型解决了类错乱加载的问题，也设计得非常精妙。但它也不是万能的，在有些场景也会遇到它解决不了的问题。哪些场景呢？我们举一个例子来看看。

在Java核心类里面有`SPI（Service Provider Interface）`，它由Sun编写规范，第三方来负责实现。SPI需要用到第三方实现类。如果使用双亲委派模型，那么第三方实现类也需要放在Java核心类里面才可以，不然的话第三方实现类将不能被加载使用。但是这显然是不合理的！怎么办呢？`ContextClassLoader`（上下文类加载器）就来解围了。

在`java.lang.Thread`里面有两个方法，get/set上下文类加载器
```java
1. public void setContextClassLoader(ClassLoader cl)
2. public ClassLoader getContextClassLoader()
```
我们可以通过在SPI类里面调用`getContextClassLoader`来获取第三方实现类的类加载器。由第三方实现类通过调用`setContextClassLoader`来传入自己实现的类加载器。这样就变相地解决了双亲委派模式遇到的问题。但是很显然，这种机制破坏了双亲委派模式。

# 对象分配规则

- 对象优先分配在Eden区，如果Eden区没有足够的空间时，虚拟机执行一次Minor GC。
- 大对象直接进入老年代（大对象是指需要大量连续内存空间的对象）。这样做的目的是避免在Eden区和两个Survivor区之间发生大量的内存拷贝（新生代采用复制算法收集内存）。
- 长期存活的对象进入老年代。虚拟机为每个对象定义了一个年龄计数器，如果对象经过了1次Minor GC那么对象会进入Survivor区，之后每经过一次Minor GC那么对象的年龄加1，知道达到阀值对象进入老年区。
- 动态判断对象的年龄。如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代。
- 空间分配担保。每次进行Minor GC时，JVM会计算Survivor区移至老年区的对象的平均大小，如果这个值大于老年区的剩余值大小则进行一次Full GC，如果小于检查HandlePromotionFailure设置，如果true则只进行Monitor GC,如果false则进行Full GC。

# 类的生命周期

类的生命周期包括这几个部分，加载、连接、初始化、使用和卸载，其中前三部是类的加载的过程

- 加载，查找并加载类的二进制数据，在Java堆中也创建一个java.lang.Class类的对象
- 连接，连接又包含三块内容：验证、准备、初始化。 1）验证，文件格式、元数据、字节码、符号引用验证； 2）准备，为类的静态变量分配内存，并将其初始化为默认值； 3）解析，把类中的符号引用转换为直接引用
- 初始化，为类的静态变量赋予正确的初始值
- 使用，new出对象程序中使用
- 卸载，执行垃圾回收

# Java对象结构

Java对象由三个部分组成：对象头、实例数据、对齐填充。

对象头由两部分组成，第一部分存储对象自身的运行时数据：哈希码、GC分代年龄、锁标识状态、线程持有的锁、偏向线程ID（一般占32/64 bit）。第二部分是指针类型，指向对象的类元数据类型（即对象代表哪个类）。如果是数组对象，则对象头中还有一部分用来记录数组长度。

实例数据用来存储对象真正的有效信息（包括父类继承下来的和自己定义的）

对齐填充：JVM要求对象起始地址必须是8字节的整数倍（8字节对齐）

# 常见OOM异常和错误

- java.lang.StackOverflowError
- java.lang.OutOfMemoryError: Java heap space
- java.lang.OutOfMemoryError：GC overhead limit exceeded
- java.lang.OutOfMemoryError：Direct buffer memory
- java.lang.OutOfMemoryError：unable to create new native thread
- java.lang.OutOfMemoryError：MetaSpace

## java.lang.StackOverflowError

报这个错误一般是由于方法深层次的调用，默认的线程栈空间大小一般与具体的硬件平台有关。栈内存为线程私有的空间，每个线程都会创建私有的栈内存。栈空间内存设置过大，创建线程数量较多时会出现栈内存溢出StackOverflowError。同时，栈内存也决定方法调用的深度，栈内存过小则会导致方法调用的深度较小，如递归调用的次数较少。

Demo：

```java
public class StackOverFlowErrorDemo {

   static int i = 0;

    public static void main(String[] args) {
        stackOverflowErrorTest();
    }

    private static void stackOverflowErrorTest() {

        i++;
        System.out.println("这是第 "+i+" 次调用");
        stackOverflowErrorTest();

    }
}
//运行结果：
。。。。
这是第 6726 次调用
这是第 6727 次调用
Exception in thread "main" java.lang.StackOverflowError
。。。。
```

注意：这是一个Error!!!!

------

## java.lang.OutOfMemoryError: Java heap space

Heap size 设置 JVM堆的设置是指：java程序执行过程中JVM能够调配使用的内存空间的设置。JVM在启动的时候会自己主动设置Heap size的值，其**初始空间(即-Xms)是物理内存的1/64**，**最大空间(-Xmx)是物理内存的1/4**。能够利用JVM提供的-Xmn -Xms -Xmx等选项可进行设置。Heap size 的大小是Young Generation 和Tenured Generaion 之和。

Demo：

```java
public class OOMHeapSpaceDemo {
    public static void main(String[] args) {
        byte[] bytes = new byte[30*1024*1024];
    }
}
```

然后修改堆内存的初始容量和最大容量为5MB

运行程序，查看结果：

```java
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at jvm.OOMHeapSpaceDemo.main(OOMHeapSpaceDemo.java:7)
```

注意：这是一个Error!!!!

------

## java.lang.OutOfMemoryError：GC overhead limit exceeded

GC回收时间过长时会抛出的OutOfMemory。过长是指，超过98%的时间都在用来做GC并且回收了不到2%的堆内存。连续多次的GC，都回收了不到2%的极端情况下才会抛出。假如不抛出GC overhead limit 错误会发生什么事情呢？那就是GC清理出来的一点内存很快又会被再次填满，强迫GC再次执行，这样造成恶性循环，CPU的使用率一直很高，但是GC没有任何的进展。

Demo：

```java
/**
 * 调整虚拟机的参数：
 * -Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m
 */
public class GCOverHeadDemo {
    public static void main(String[] args) {
        int i= 0;
        List<String> list = new ArrayList<>();
        while (true){
            list.add(String.valueOf(++i).intern());
            System.out.println(i);

        }
    }
}


//运行结果：
[Full GC (Ergonomics) [PSYoungGen: 1024K->1024K(2048K)] [ParOldGen: 7101K->7099K(7168K)] 8125K->8123K(9216K), [Metaspace: 3264K->3264K(1056768K)], 0.0296282 secs] [Times: user=0.08 sys=0.00, real=0.03 secs] 

Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
```

可以看到GC前后内存占用时一样的，也就是说虽然在频繁的GC，但是并没有起到什么作用，并没有回收回来多少的内存。

------

## java.lang.OutOfMemoryError：Direct buffer memory

写NIO程序经常使用到ByteBuffer来读取或者写入数据，这是一种基于通道与缓冲区的I/O方式。它可以使用Native函数库直接分配堆外内存，然后通过一个存储在java堆里面的DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一些场景中提高性能，因为避免了java堆和Native堆中来回复制数据。

- ByteBuffer.allocate(capability) ：这种方式是分配JVM堆内存，属于GC管辖范围之内。由于需要拷贝，所以速度相对较慢；
- ByteBuffer.allocateDirect(capability)：这种方式是直接分配OS本地内存，不属于GC管辖范围之内，由于不需要内存拷贝所以速度相对较快。

但是如果不断分配本地内存，堆内存很少使用，那么JVM就不需要执行GC,DirectByteBuffer对象就不会被回收。这时候堆内存充足，但是本地内存已经用光了，再次尝试分配的时候就会出现OutOfMemoryError，那么程序就直接崩溃了。

Demo：

```java
/**
 * JVM配置参数：
 * -Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m
 */
public class DirectBufferMemoryDemo {
    public static void main(String[] args) {
        System.out.println("配置的MaxDirectMemorySize"+sun.misc.VM.maxDirectMemory()/(double)1024/1024+" MB");
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(6*1024*1024);
    }
}


//运行结果：
配置的MaxDirectMemorySize5.0 MB
[GC (System.gc()) [PSYoungGen: 1785K->488K(2560K)] 1785K->728K(9728K), 0.0019042 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 488K->0K(2560K)] [ParOldGen: 240K->640K(7168K)] 728K->640K(9728K), [Metaspace: 3230K->3230K(1056768K)], 0.0077924 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 

Exception in thread "main" java.lang.OutOfMemoryError: Direct buffer memory
```

------

## java.lang.OutOfMemoryError：unable to create new native thread

准确的说，这一个异常是和程序运行的平台相关的。导致的原因：

- 创建了太多的线程，一个应用创建多个线程，超过系统承载极限；
- 服务器不允许应用程序创建这么多的线程，Linux系统默认的允许单个进程可以创建的线程数量是1024个，当创建多线程数量多于这个数字的时候就会抛出此异常

如何解决呢？

- 想办法减少应用程序创建的线程的数量，分析应用是否真的需要创建这么多的线程。如果不是，改变代码将线程数量降到最低；
- 对于有的应用，确实需要创建很多的线程，远超过Linux限制的1024个 限制，那么可以通过修改Linux服务器的配置，扩大Linux的默认限制。

Demo：

```java
public class UnableCreateNewThreadDemo {
    public static void main(String[] args) {
        for (int i = 1; ;i++){
            System.out.println("i = " +i);
            new Thread(()->{
                try {
                    Thread.sleep(Integer.MAX_VALUE);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            },i+"").start();
        }
    }
}

//运行结果：
。。。。
i = 92916
i = 92917
Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
。。。。
```

------

## java.lang.OutOfMemoryError：MetaSpace

元空间的本质和永久代类似，都是对JVM规范中的方法区的实现。不过元空间与永久代之间最大的区别在于：元空间不在虚拟机中，而是使用的本地内存。因此，默认情况下，元空间的大小仅仅受到本地内存的限制 。

元空间存放了以下的内容：

- 虚拟机加载的类信息；
- 常量池；
- 静态变量；
- 即时编译后的代码

模拟MetaSpace空间溢出，我们不断生成类往元空间里灌，类占据的空间总是会超过MetaSpace指定的空间大小的

查看元空间的大小：`java -XX:+PrintFlagsInitial`

Demo：

```java
/**
 * JVM参数配置：
 * -XX:MetaSapceSize=5m
 */
public class MetaSpaceDemo {
    public static void main(String[] args) {
        for (int i = 0; i < 100_000_000; i++) {
            generate("eu.plumbr.demo.Generated" + i);
        }
    }
    public static Class generate(String name) throws Exception {
        ClassPool pool = ClassPool.getDefault();
        return pool.makeClass(name).toClass();
    }
}
```

# 如何判断对象可以被回收

判断对象是否存活一般有两种方式：

- 引用计数：每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，无法解决对象相互循环引用的问题。
- 可达性分析（Reachability Analysis）：从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的，不可达对象。

# 引用的分类

- 强引用：GC时不会被回收
- 软引用：描述有用但不是必须的对象，在发生内存溢出异常之前被回收
- 弱引用：描述有用但不是必须的对象，在下一次GC时被回收
- 虚引用（幽灵引用/幻影引用）:无法通过虚引用获得对象，用PhantomReference实现虚引用，虚引用用来在GC时返回一个通知。

## 强引用

强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出`OutOfM moryError`错误，使程序异常终止，也不会靠随意回收具有强引用 对象来解决内存不足的问题。

## 软引用

软引用是用来描述一些还有用但并非必须的对象。对于软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。

```csharp
/**
 * 软引用何时被收集
 * 运行参数 -Xmx200m -XX:+PrintGC
 * Created by ccr at 2018/7/14.
 */
public class SoftReferenceDemo {

    public static void main(String[] args) throws InterruptedException {
        //100M的缓存数据
        byte[] cacheData = new byte[100 * 1024 * 1024];
        //将缓存数据用软引用持有
        SoftReference<byte[]> cacheRef = new SoftReference<>(cacheData);
        //将缓存数据的强引用去除
        cacheData = null;
        System.out.println("第一次GC前" + cacheData);
        System.out.println("第一次GC前" + cacheRef.get());
        //进行一次GC后查看对象的回收情况
        System.gc();
        //等待GC
        Thread.sleep(500);
        System.out.println("第一次GC后" + cacheData);
        System.out.println("第一次GC后" + cacheRef.get());

        //在分配一个120M的对象，看看缓存对象的回收情况
        byte[] newData = new byte[120 * 1024 * 1024];
        System.out.println("分配后" + cacheData);
        System.out.println("分配后" + cacheRef.get());
    }

}

第一次GC前null
第一次GC前[B@7d4991ad
[GC (System.gc())  105728K->103248K(175104K), 0.0009623 secs]
[Full GC (System.gc())  103248K->103139K(175104K), 0.0049909 secs]
第一次GC后null
第一次GC后[B@7d4991ad
[GC (Allocation Failure)  103805K->103171K(175104K), 0.0027889 secs]
[GC (Allocation Failure)  103171K->103171K(175104K), 0.0016018 secs]
[Full GC (Allocation Failure)  103171K->103136K(175104K), 0.0089988 secs]
[GC (Allocation Failure)  103136K->103136K(199680K), 0.0009408 secs]
[Full GC (Allocation Failure)  103136K->719K(128512K), 0.0082685 secs]
分配后null
分配后null
```

从上面的示例中就能看出，软引用关联的对象不会被`GC`回收。`JVM`在分配空间时，若果`Heap`空间不足，就会进行相应的`GC`，但是这次`GC`并不会收集软引用关联的对象，但是在JVM发现就算进行了一次回收后还是不足（`Allocation Failure`），`JVM`会尝试第二次`GC`，回收软引用关联的对象。

像这种如果内存充足，`GC`时就保留，内存不够，`GC`再来收集的功能很适合用在缓存的引用场景中。在使用缓存时有一个原则，如果缓存中有就从缓存获取，如果没有就从数据库中获取，缓存的存在是为了加快计算速度，如果因为缓存导致了内存不足进而整个程序崩溃，那就得不偿失了。

## 弱引用

弱引用也是用来描述非必须对象的，他的强度比软引用更弱一些，被弱引用关联的对象，在垃圾回收时，如果这个对象只被弱引用关联（没有任何强引用关联他），那么这个对象就会被回收。

```csharp
/**
 * 弱引用关联对象何时被回收
 * Created by ccr at 2018/7/14.
 */
public class WeakReferenceDemo {
    public static void main(String[] args) throws InterruptedException {
        //100M的缓存数据
        byte[] cacheData = new byte[100 * 1024 * 1024];
        //将缓存数据用软引用持有
        WeakReference<byte[]> cacheRef = new WeakReference<>(cacheData);
        System.out.println("第一次GC前" + cacheData);
        System.out.println("第一次GC前" + cacheRef.get());
        //进行一次GC后查看对象的回收情况
        System.gc();
        //等待GC
        Thread.sleep(500);
        System.out.println("第一次GC后" + cacheData);
        System.out.println("第一次GC后" + cacheRef.get());

        //将缓存数据的强引用去除
        cacheData = null;
        System.gc();
        //等待GC
        Thread.sleep(500);
        System.out.println("第二次GC后" + cacheData);
        System.out.println("第二次GC后" + cacheRef.get());
    }
}
第一次GC前[B@7d4991ad
第一次GC前[B@7d4991ad
第一次GC后[B@7d4991ad
第一次GC后[B@7d4991ad
第二次GC后null
第二次GC后null
```

从上面的代码中可以看出，弱引用关联的对象是否回收取决于这个对象有没有其他强引用指向它。这个确实很难理解，既然弱引用关联对象的存活周期和强引用差不多，那直接用强引用好了，干嘛费用弄出个弱引用呢？其实弱引用存在必然有他的应用场景。

```dart
static Map<Object,Object> container = new HashMap<>();
public static void putToContainer(Object key,Object value){
    container.put(key,value);
}

public static void main(String[] args) {
    //某个类中有这样一段代码
    Object key = new Object();
    Object value = new Object();
    putToContainer(key,value);

    //..........
    /**
     * 若干调用层次后程序员发现这个key指向的对象没有用了，
     * 为了节省内存打算把这个对象抛弃，然而下面这个方式真的能把对象回收掉吗？
     * 由于container对象中包含了这个对象的引用,所以这个对象不能按照程序员的意向进行回收.
     * 并且由于在程序中的任何部分没有再出现这个键，所以，这个键 / 值 对无法从映射中删除。
     * 很可能会造成内存泄漏。
     */
    key = null;
}
```

下面一段话摘自《Java核心技术卷1》：

设计 `WeakHashMap`类是为了解决一个有趣的问题。如果有一个值，对应的键已经不再 使用了， 将会出现什么情况呢？ 假定对某个键的最后一次引用已经消亡，不再有任何途径引 用这个值的对象了。但是，由于在程序中的任何部分没有再出现这个键，所以，这个键 / 值 对无法从映射中删除。为什么垃圾回收器不能够删除它呢？ 难道删除无用的对象不是垃圾回 收器的工作吗？

遗憾的是，事情没有这样简单。垃圾回收器跟踪活动的对象。只要映射对象是活动的， 其中的所有桶也是活动的， 它们不能被回收。因此，需要由程序负责从长期存活的映射表中 删除那些无用的值。 或者使用 `WeakHashMap`完成这件事情。当对键的唯一引用来自散列条
 目时， 这一数据结构将与垃圾回收器协同工作一起删除键 / 值对。

下面是这种机制的内部运行情况。`WeakHashMap` 使用弱引用（`weak references`) 保存键。 `WeakReference` 对象将引用保存到另外一个对象中，在这里，就是散列键。对于这种类型的 对象，垃圾回收器用一种特有的方式进行处理。通常，如果垃圾回收器发现某个特定的对象 已经没有他人引用了，就将其回收。然而， 如果某个对象只能由 `WeakReference` 引用， 垃圾 回收器仍然回收它，但要将引用这个对象的弱引用放人队列中。`WeakHashMap`将周期性地检 查队列， 以便找出新添加的弱引用。一个弱引用进人队列意味着这个键不再被他人使用， 并 且已经被收集起来。于是， `WeakHashMap`将删除对应的条目。

除了`WeakHashMap`使用了弱引用，`ThreadLocal`类中也是用了弱引用。

## 虚引用

一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来获取一个对象的实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。虚引用和弱引用对关联对象的回收都不会产生影响，如果只有虚引用活着弱引用关联着对象，那么这个对象就会被回收。它们的不同之处在于弱引用的`get`方法，虚引用的`get`方法始终返回`null`,弱引用可以使用`ReferenceQueue`,虚引用必须配合`ReferenceQueue`使用。

`jdk`中直接内存的回收就用到虚引用，由于`jvm`自动内存管理的范围是堆内存，而直接内存是在堆内存之外（其实是内存映射文件，自行去理解虚拟内存空间的相关概念），所以直接内存的分配和回收都是有`Unsafe`类去操作，`java`在申请一块直接内存之后，会在堆内存分配一个对象保存这个堆外内存的引用，这个对象被垃圾收集器管理，一旦这个对象被回收，相应的用户线程会收到通知并对直接内存进行清理工作。

# 垃圾回收

## 基础篇

### 概述

　　在这篇文章中《[Jvm运行时数据区](https://www.cnblogs.com/chenpt/p/8953435.html)》介绍了Java内存运行时区域的各个部分，其中程序计数器、虚拟机栈、本地方法栈，3个区域随着线程的生存而生存的。内存分配和回收都是确定的。随着线程的结束内存自然就被回收了，因此不需要考虑垃圾回收的问题。而Java堆和方法区则不一样，各线程共享，内存的分配和回收都是动态的。因此垃圾收集器所关注的都是这部分内存。

　　接下来我们就讨论Jvm是怎么回收这部分内存的。在进行回收前垃圾收集器第一件事情就是确定哪些对象还存活，哪些已经死去。下面介绍两种基础的回收算法。

#### 引用计数算法

　　给对象添加一个引用计数器，每当有一个地方引用它时计数器就+1，当引用失效时计数器就-1,。只要计数器等于0的对象就是不可能再被使用的。

　　此算法在大部分情况下都是一个不错的选择，也有一些著名的应用案例。但是Java虚拟机中是没有使用的。

　　**优点**：实现简单、判断效率高。

　　**缺点**：很难解决对象之间循环引用的问题。例如下面这个例子

```
Object a = ``new` `Object();``Object b = ``new` `Object();``a=b;``b=a;``a=b=``null``; ``//这样就导致gc无法回收他们。　　
```

#### 可达性分析算法

　　通过一系列的称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链，当一个对象到GC Roots没有使用任何引用链时，则说明该对象是不可用的。

　　主流的商用程序语言（Java、C#等）在主流的实现中，都是通过可达性分析来判定对象是否存活的。

　　通过下图来清晰的感受gc root与对象展示的联系。所示灰色区域对象是存活的，Object5/6/7均是可回收的对象

![image-20200509102745178](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509102746-460810.png) 

**在Java语言中，可作为GC Roots 的对象包括下面几种**

- 虚拟机栈（栈帧中的本地变量表）中引用的对象
- 方法区中静态变量引用的对象
- 方法区中常量引用的对象
- 本地方法栈（即一般说的 Native 方法）中JNI引用的对象

　**优点**：更加精确和严谨，可以分析出循环数据结构相互引用的情况；

　**缺点**：实现比较复杂、需要分析大量数据，消耗大量时间、分析过程需要GC停顿（引用关系不能发生变化），即停顿所有Java执行线程（称为"Stop The World"，是垃圾回收重点关注的问题）。

### 引用

　在jdk1.2之后，Java对引用的概念进行了扩充，总体分为4类：强引用、软引用、弱引用、虚引用，这4中引用强度依次逐渐减弱。

- **强引用**：指在代码中普遍存在的，类似 Object obj = new Object(); 这类的引用，只有强引用还存在，GC就永远不会收集被引用的对象。
- **软引用**：指一些还有用但并非必须的对象。直到内存空间不够时（抛出OutOfMemoryError之前），才会被垃圾回收。采用SoftReference类来实现软引用
- **弱引用**：用来描述非必须对象。当垃圾收集器工作时就会回收掉此类对象。采用WeakReference类来实现弱引用。
- **虚引用**：一个对象是否有虚引用的存在，完全不会对其生存时间构成影响， 唯一目的就是能在这个对象被回收时收到一个系统通知， 采用PhantomRenference类实现

#### 判断一个对象生存还是死亡

　　宣告一个对象死亡，至少要经历两次标记。

　　**1、第一次标记**

　　如果对象进行可达性分析算法之后没发现与GC Roots相连的引用链，那它将会第一次标记并且进行一次筛选。

　　**筛选条件**：判断此对象是否有必要执行finalize()方法。

　　**筛选结果**：当对象没有覆盖finalize()方法、或者finalize()方法已经被JVM执行过，则判定为可回收对象。如果对象有必要执行finalize()方法，则被放入F-Queue队列中。稍后在JVM自动建立、低优先级的Finalizer线程（可能多个线程）中触发这个方法；　　

　　**2、第二次标记**

　　GC对F-Queue队列中的对象进行二次标记。

　　如果对象在finalize()方法中重新与引用链上的任何一个对象建立了关联，那么二次标记时则会将它移出“即将回收”集合。如果此时对象还没成功逃脱，那么只能被回收了。

　　**3、finalize() 方法**

　　finalize()是Object类的一个方法、一个对象的finalize()方法只会被系统自动调用一次，经过finalize()方法逃脱死亡的对象，第二次不会再调用；

　　特别说明：并不提倡在程序中调用finalize()来进行自救。建议忘掉Java程序中该方法的存在。因为它执行的时间不确定，甚至是否被执行也不确定（Java程序的不正常退出），而且运行代价高昂，无法保证各个对象的调用顺序（甚至有不同线程中调用）。

### 回收方法区　　

　　永久代的垃圾收集主要分为两部分内容：废弃常量和无用的类。

#### 回收废弃常量

　　回收废弃常量与Java堆的回收类似。下面举个栗子说明

　　假如一个字符串“abc” 已经进入常量池中，但当前系统没有一个string对象是叫做abc的，也就是说，没有任何string对象的引用指向常量池中的abc常量，也没用其他地方引用这个字面量。如果这是发生内存回收，那么这个常量abc将会被清理出常量池。常量池中的其他类（接口）、方法、字段的符号引用也与此类似。

#### 回收无用的类

　　需要同时满足下面3个条件的才能算是无用的类。

1. 该类所有的实例都已经被回收，也就是Java堆中无任何改类的实例。
2. 加载该类的ClassLoader已经被回收。
3. 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

　　虚拟机可以对同时满足这三个条件的类进行回收，但不是必须进行回收的。是否对类进行回收，HotSpot虚拟机提供了-Xnoclassgc参数进行控制。

## 算法篇

### 标记-清除算法

　　最基础的收集算法，总共分为‘ 标记 ’和‘ 清除 ’两个阶段

#### 标记

　　标记出所有需要回收的对象

　　在基础篇中说明了判断对象是否回收需要两次标记，现在我们再来回顾一下

　　**一次标记**：在经过可达性分析算法后，对象没有与GC Root相关的引用链，那么则被第一次标记。并且进行一次筛选：当对象有必要执行finalize()方法时，则把该对象放入F-Queue队列中。

　　**二次标记**：对F-Queue队列中的对象进行二次标记。在执行finalize()方法时，如果对象重新与GC Root引用链上的任意对象建立了关联，则把他移除出“ 即将回收 ”集合。否则就等着被回收吧！！！

　　对被第一次标记切被第二次标记的，就可以判定位可回收对象了。

#### 清除

　　两次标记后，还在“ 即将回收 ”集合的对象进行回收。

　　执行过程如下：

![image-20200509103238458](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509103239-566200.png) 

​       **优点**：基础最基础的可达性算法，后续的收集算法都是基于这种思想实现的

　　**缺点**：标记和清除效率不高，产生大量不连续的内存碎片，导致创建大对象时找不到连续的空间，不得不提前触发另一次的垃圾回收。	

### 复制算法

　　将可用内存按容量分为大小相等的两块，每次只使用其中一块，当这一块的内存用完了，就将还存活的对象复制到另外一块内存上，然后再把已使用过的内存空间一次清理掉。

　　复制算法执行过程如下：

![image-20200509103345938](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509103348-319525.png)  

​		**优点**：实现简单，效率高。解决了标记-清除算法导致的内存碎片问题。

　　**缺点**：代价太大，将内存缩小了一半。效率随对象的存活率升高而降低。

 现在的商业虚拟机都采用这种算法（需要改良1:1的缺点）来回收新生代。

#### HotSpot虚拟机的改良算法　

**弱代理论**　

　　分代垃圾收集基于弱代理论。具体描述如下：

- 大多说分配了内存的对象并不会存活太长时间，在处于年轻时代就会死掉。
- 很少有对象会从老年代变成年轻代。

 　其中IBM研究表明：新生代中98%的对象都是"朝生夕死"； 所以并不需要按1:1比例来划分内存（解决了缺点1）；

#### Hotspot虚拟机新生代内存布局及算法

　　新生代内存分配一块较大的Eden空间和两块较小的Survivor空间

　　每次使用Eden和其中一块Survivor空间

　　回收时将Eden和Survivor空间中存活的对象一次性复制到另一块Survivor空间上

　　最后清理掉Eden和使用过的Survivor空间。

Hotspot虚拟机默认Eden和Survivor的大小比例是8:1。

**分配担保**

　　如果另一块Survivor空间没有足够内存来存放上一次新生代收集下来的存活对象，那么这些对象则直接通过担保机制进入老年代。

关于分配担保的内容，我会在讲述垃圾收集器时详细描述。

### 标记-整理算法

　　标记-整理算法是根据老年代的特点应运而生。

#### 标记

　　标记过程和标记-清理算法一致（也是基于可达性分析算法）。

#### 整理

　　和标记-清理不同的是，该算法不是针对可回收对象进行清理，而是根据存活对象进行整理。让存活对象都向一端移动，然后直接清理掉边界以外的内存。

标记-整理算法示意图

![image-20200509103627418](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509103631-885453.png)  

　　**优点**：不会像复制算法那样随着存活对象的升高而降低效率，不像标记-清除算法那样产生不连续的内存碎片

　　**缺点**：效率问题，除了像标记-清除算法的标记过程外，还多了一步整理过程，效率更低。

### 分代收集算法

　　当前商业虚拟机的垃圾收集都是采用“ 分代收集 ”算法。

根据对象存活周期的不同将内存划分为几块。一般把java堆分为新生代和老年代。JVM根据各个年代的特点采用不同的收集算法。

新生代中，每次进行垃圾回收都会发现大量对象死去，只有少量存活，因此比较适合复制算法。只需要付出少量存活对象的复制成本就可以完成收集。

老年代中，因为对象存活率较高，没有额外的空间进行分配担保，所以适合标记-清理、标记-整理算法来进行回收。

## 终结篇

第一篇基础篇主要讲述了判断对象的生死？两种基础判断对象生死的算法、引用计数法、可达性分析算法，方法区的回收。在第二篇算法篇中主要介绍了垃圾回收的几种常用算法：标记-清除、复制算法、标记-整理算法、分代收集算法。那么接下来我们重点研究Jvm的垃圾收集器（serial收集器、parnew收集器、parallel scavenge收集器、serial old 收集器、parallel old收集器、cms收集器、g1收集器）。前面说了那么多就是为它做铺垫的。

**正式进入前先看下图解HotSpot虚拟机所包含的收集器：**

![image-20200509103954591](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509103958-690517.png)  

图中展示了7种作用于不同分代的收集器，如果两个收集器之间存在连线，则说明它们可以搭配使用。虚拟机所处的区域则表示它是属于新生代还是老年代收集器。

**新生代收集器**：Serial、ParNew、Parallel Scavenge

**老年代收集器**：CMS、Serial Old、Parallel Old

**整堆收集器**： G1

### 几个相关概念

**并行收集**：指多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态。

**并发收集**：指用户线程与垃圾收集线程同时工作（不一定是并行的可能会交替执行）。用户程序在继续运行，而垃圾收集程序运行在另一个CPU上。

**吞吐量**：即CPU用于运行用户代码的时间与CPU总消耗时间的比值（吞吐量 = 运行用户代码时间 / ( 运行用户代码时间 + 垃圾收集时间 )）。例如：虚拟机共运行100分钟，垃圾收集器花掉1分钟，那么吞吐量就是99%

### Serial 收集器

Serial收集器是最基本的、发展历史最悠久的收集器。

**特点：**单线程、简单高效（与其他收集器的单线程相比），对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程手机效率。收集器进行垃圾回收时，必须暂停其他所有的工作线程，直到它结束（Stop The World）。

**应用场景**：适用于Client模式下的虚拟机。

**Serial / Serial Old收集器运行示意图**

![image-20200509104102598](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509104106-377432.png) 

### ParNew收集器

ParNew收集器其实就是Serial收集器的多线程版本。

除了使用多线程外其余行为均和Serial收集器一模一样（参数控制、收集算法、Stop The World、对象分配规则、回收策略等）。

**特点**：多线程、ParNew收集器默认开启的收集线程数与CPU的数量相同，在CPU非常多的环境中，可以使用-XX:ParallelGCThreads参数来限制垃圾收集的线程数。

　　　和Serial收集器一样存在Stop The World问题

**应用场景**：ParNew收集器是许多运行在Server模式下的虚拟机中首选的新生代收集器，因为它是除了Serial收集器外，唯一一个能与CMS收集器配合工作的。

**ParNew/Serial Old组合收集器运行示意图如下：**

![image-20200509104128485](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509104131-466715.png) 

### Parallel Scavenge 收集器

与吞吐量关系密切，故也称为吞吐量优先收集器。

**特点**：属于新生代收集器也是采用复制算法的收集器，又是并行的多线程收集器（与ParNew收集器类似）。

该收集器的目标是达到一个可控制的吞吐量。还有一个值得关注的点是：GC自适应调节策略（与ParNew收集器最重要的一个区别）

**GC自适应调节策略**：Parallel Scavenge收集器可设置-XX:+UseAdptiveSizePolicy参数。当开关打开时不需要手动指定新生代的大小（-Xmn）、Eden与Survivor区的比例（-XX:SurvivorRation）、晋升老年代的对象年龄（-XX:PretenureSizeThreshold）等，虚拟机会根据系统的运行状况收集性能监控信息，动态设置这些参数以提供最优的停顿时间和最高的吞吐量，这种调节方式称为GC的自适应调节策略。

Parallel Scavenge收集器使用两个参数控制吞吐量：

- XX:MaxGCPauseMillis 控制最大的垃圾收集停顿时间
- XX:GCRatio 直接设置吞吐量的大小。

### Serial Old 收集器

Serial Old是Serial收集器的老年代版本。

**特点**：同样是单线程收集器，采用标记-整理算法。

**应用场景**：主要也是使用在Client模式下的虚拟机中。也可在Server模式下使用。

Server模式下主要的两大用途（在后续中详细讲解···）：

1. 在JDK1.5以及以前的版本中与Parallel Scavenge收集器搭配使用。
2. 作为CMS收集器的后备方案，在并发收集Concurent Mode Failure时使用。

Serial / Serial Old收集器工作过程图（Serial收集器图示相同）：

![image-20200509104208308](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509104209-757816.png) 

### **Parallel Old 收集器**

是Parallel Scavenge收集器的老年代版本。

**特点**：多线程，采用标记-整理算法。

**应用场景**：注重高吞吐量以及CPU资源敏感的场合，都可以优先考虑Parallel Scavenge+Parallel Old 收集器。

**Parallel Scavenge/Parallel Old收集器工作过程图：**

![image-20200509104320257](C:\Users\lancelot\AppData\Roaming\Typora\typora-user-images\image-20200509104320257.png) 

### CMS收集器

一种以获取最短回收停顿时间为目标的收集器。

**特点**：基于标记-清除算法实现。并发收集、低停顿。

**应用场景**：适用于注重服务的响应速度，希望系统停顿时间最短，给用户带来更好的体验等场景下。如web程序、b/s服务。

**CMS收集器的运行过程分为下列4步：**

**初始标记**：标记GC Roots能直接到的对象。速度很快但是仍存在Stop The World问题。

**并发标记**：进行GC Roots Tracing 的过程，找出存活对象且用户线程可并发执行。

**重新标记**：为了修正并发标记期间因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录。仍然存在Stop The World问题。

**并发清除**：对标记的对象进行清除回收。

 CMS收集器的内存回收过程是与用户线程一起并发执行的。

 CMS收集器的工作过程图：

![image-20200509104343649](C:\Users\lancelot\AppData\Roaming\Typora\typora-user-images\image-20200509104343649.png) 

CMS收集器的缺点：

- 对CPU资源非常敏感。
- 无法处理浮动垃圾，可能出现Concurrent Model Failure失败而导致另一次Full GC的产生。
- 因为采用标记-清除算法所以会存在空间碎片的问题，导致大对象无法分配空间，不得不提前触发一次Full GC。

### G1收集器

一款面向服务端应用的垃圾收集器。

**特点如下：**

并行与并发：G1能充分利用多CPU、多核环境下的硬件优势，使用多个CPU来缩短Stop-The-World停顿时间。部分收集器原本需要停顿Java线程来执行GC动作，G1收集器仍然可以通过并发的方式让Java程序继续运行。

分代收集：G1能够独自管理整个Java堆，并且采用不同的方式去处理新创建的对象和已经存活了一段时间、熬过多次GC的旧对象以获取更好的收集效果。

空间整合：G1运作期间不会产生空间碎片，收集后能提供规整的可用内存。

可预测的停顿：G1除了追求低停顿外，还能建立可预测的停顿时间模型。能让使用者明确指定在一个长度为M毫秒的时间段内，消耗在垃圾收集上的时间不得超过N毫秒。

**G1为什么能建立可预测的停顿时间模型？**

因为它有计划的避免在整个Java堆中进行全区域的垃圾收集。G1跟踪各个Region里面的垃圾堆积的大小，在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region。这样就保证了在有限的时间内可以获取尽可能高的收集效率。

**G1与其他收集器的区别**：

其他收集器的工作范围是整个新生代或者老年代、G1收集器的工作范围是整个Java堆。在使用G1收集器时，它将整个Java堆划分为多个大小相等的独立区域（Region）。虽然也保留了新生代、老年代的概念，但新生代和老年代不再是相互隔离的，他们都是一部分Region（不需要连续）的集合。

**G1收集器存在的问题：**

Region不可能是孤立的，分配在Region中的对象可以与Java堆中的任意对象发生引用关系。在采用可达性分析算法来判断对象是否存活时，得扫描整个Java堆才能保证准确性。其他收集器也存在这种问题（G1更加突出而已）。会导致Minor GC效率下降。

**G1收集器是如何解决上述问题的？**

采用Remembered Set来避免整堆扫描。G1中每个Region都有一个与之对应的Remembered Set，虚拟机发现程序在对Reference类型进行写操作时，会产生一个Write Barrier暂时中断写操作，检查Reference引用对象是否处于多个Region中（即检查老年代中是否引用了新生代中的对象），如果是，便通过CardTable把相关引用信息记录到被引用对象所属的Region的Remembered Set中。当进行内存回收时，在GC根节点的枚举范围中加入Remembered Set即可保证不对全堆进行扫描也不会有遗漏。

**如果不计算维护 Remembered Set 的操作，G1收集器大致可分为如下步骤：**

**初始标记**：仅标记GC Roots能直接到的对象，并且修改TAMS（Next Top at Mark Start）的值，让下一阶段用户程序并发运行时，能在正确可用的Region中创建新对象。（需要线程停顿，但耗时很短。）

**并发标记**：从GC Roots开始对堆中对象进行可达性分析，找出存活对象。（耗时较长，但可与用户程序并发执行）

**最终标记**：为了修正在并发标记期间因用户程序执行而导致标记产生变化的那一部分标记记录。且对象的变化记录在线程Remembered Set Logs里面，把Remembered Set Logs里面的数据合并到Remembered Set中。（需要线程停顿，但可并行执行。）

**筛选回收**：对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划。（可并发执行）

**G1收集器运行示意图：**

![image-20200509104415409](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509104417-205647.png) 

### 垃圾收集器组合选择

- 单CPU或小内存，单机程序
  -XX: +UseSerialGC
- 多CPU ，需要最大吞吐量，如后台计算型应用
  -XX: +UseParallelGC 或者
  -XX: +UseParallelOldGC
- 多CPU ，追求低停顿时间，需快速响应如互联网应用
  -XX: +UseConcMarkSweepGC
  -XX: +ParNewGC

![image-20200509114321059](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509114323-177870.png)  

![image-20200509114134648](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509114135-263723.png) 

# FullGC

堆内存划分为 Eden、Survivor 和 Tenured/Old 空间，如下图所示：

![image-20200416091815664](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200416091816-748126.png) 

## System.gc()方法的调用

此方法的调用是建议JVM进行Full GC,虽然只是建议而非一定,但很多情况下它会触发 Full GC,从而增加Full GC的频率,也即增加了间歇性停顿的次数。强烈影响系建议能不使用此方法就别使用，让虚拟机自己去管理它的内存，可通过通过-XX:+ DisableExplicitGC来禁止RMI调用System.gc。

## 老年代空间不足

老年代空间只有在新生代对象转入及创建大对象、大数组时才会出现不足的现象，当执行Full GC后空间仍然不足，则抛出如下错误：
java.lang.OutOfMemoryError: Java heap space 
为避免以上两种状况引起的Full GC，调优时应尽量做到让对象在Minor GC阶段被回收、让对象在新生代多存活一段时间及不要创建过大的对象及数组。

## 永生区空间不足

JVM规范中运行时数据区域中的方法区，在HotSpot虚拟机中又被习惯称为永生代或者永生区，Permanet Generation中存放的为一些class的信息、常量、静态变量等数据，当系统中要加载的类、反射的类和调用的方法较多时，Permanet Generation可能会被占满，在未配置为采用CMS GC的情况下也会执行Full GC。如果经过Full GC仍然回收不了，那么JVM会抛出如下错误信息：
java.lang.OutOfMemoryError: PermGen space 
为避免Perm Gen占满造成Full GC现象，可采用的方法为增大Perm Gen空间或转为使用CMS GC。

## CMS GC时出现promotion failed和concurrent mode failure

对于采用CMS进行老年代GC的程序而言，尤其要注意GC日志中是否有promotion failed和concurrent mode failure两种状况，当这两种状况出现时可能会触发Full GC。

具体原因和解决方案可以查看[使用CMS垃圾收集器产生的问题和解决方案](http://my.oschina.net/hosee/blog/674181)

## HandlePromotionFailure

在发生Minor GC之前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那么Minor GC可以确保是安全的。如果不成立，则虚拟机会查看HandlePromotionFailure设置值是否允许担保失败。如果允许，那么会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者HandlePromotionFailure设置不允许冒险，那这时也要改为进行一次Full GC。

统计得到的Minor GC晋升到老年代的平均大小大于老年代的剩余空间，这是一个较为复杂的触发情况，例如程序第一次触发Minor GC后，有6MB的对象晋升到老年代，那么当下一次Minor GC发生时，首先检查老年代的剩余空间是否大于6MB，如果小于6MB，则执行Full GC。
当新生代采用PS GC时，方式稍有不同，PS GC是在Minor GC后也会检查，例如上面的例子中第一次Minor GC后，PS GC会检查此时旧生代的剩余空间是否大于6MB，如小于，则触发对旧生代的回收。
除了以上4种状况外，对于使用RMI来进行RPC或管理的Sun JDK应用而言，默认情况下会一小时执行一次Full GC。可通过在启动时通过 java -Dsun.rmi.dgc.client.gcInterval=3600000来设置Full GC执行的间隔时间或通过-XX:+ DisableExplicitGC来禁止RMI调用System.gc。

## 堆中分配很大的对象

所谓大对象，是指需要大量连续内存空间的java对象，例如很长的数组，此种对象会直接进入老年代，而老年代虽然有很大的剩余空间，但是无法找到足够大的连续空间来分配给当前对象，此种情况就会触发JVM进行Full GC。

为了解决这个问题，CMS垃圾收集器提供了一个可配置的参数，即-XX:+UseCMSCompactAtFullCollection开关参数，用于在“享受”完Full GC服务之后额外免费赠送一个碎片整理的过程，内存整理的过程无法并发的，空间碎片问题没有了，但提顿时间不得不变长了，JVM设计者们还提供了另外一个参数 -XX:CMSFullGCsBeforeCompaction,这个参数用于设置在执行多少次不压缩的Full GC后,跟着来一次带压缩的

# Java常用工具

## javap命令

javap是jdk自带的反解析工具。它的作用就是根据class字节码文件，反解析出当前类对应的code区（汇编指令）、本地变量表、异常表和代码行偏移量映射表、常量池等等信息。
当然这些信息中，有些信息（如本地变量表、指令和代码行偏移量映射表、常量池中方法的参数名称等等）需要在使用javac编译成class文件时，指定参数才能输出，比如，你直接javac xx.java，就不会在生成对应的局部变量表等信息，如果你使用javac -g xx.java就可以生成所有相关信息了。如果你使用的eclipse，则默认情况下，eclipse在编译时会帮你生成局部变量表、指令和代码行偏移量映射表等信息的。
通过反编译生成的汇编代码，我们可以深入的了解java代码的工作机制。比如我们可以查看i++；这行代码实际运行时是先获取变量i的值，然后将这个值加1，最后再将加1后的值赋值给变量i。
通过局部变量表，我们可以查看局部变量的作用域范围、所在槽位等信息，甚至可以看到槽位复用等信息。

javap的用法格式：

```bash
javap <options> <classes>
```

其中classes就是你要反编译的class文件。
在命令行中直接输入javap或javap -help可以看到javap的options有如下选项：

```bash
javap -help
用法: javap <options> <classes>
其中, 可能的选项包括:
  -help  --help  -?        输出此用法消息
  -version                 版本信息
  -v  -verbose             输出附加信息
  -l                       输出行号和本地变量表
  -public                  仅显示公共类和成员
  -protected               显示受保护的/公共类和成员
  -package                 显示程序包/受保护的/公共类
                           和成员 (默认)
  -p  -private             显示所有类和成员
  -c                       对代码进行反汇编
  -s                       输出内部类型签名
  -sysinfo                 显示正在处理的类的
                           系统信息 (路径, 大小, 日期, MD5 散列)
  -constants               显示静态最终常量
  -classpath <path>        指定查找用户类文件的位置
```

一般常用的是-v -l -c三个选项。
javap -v classxx，不仅会输出行号、本地变量表信息、反编译汇编代码，还会输出当前类用到的常量池等信息。
javap -l 会输出行号和本地变量表信息。
javap -c 会对当前class字节码进行反编译生成汇编代码。
查看汇编代码时，需要知道里面的jvm指令，可以参考官方文档：
https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html
另外通过jclasslib工具也可以看到上面这些信息，而且是可视化的，效果更好一些。

运用jvm自带的命令可以方便的在生产监控和打印堆栈的日志信息帮忙我们来定位问题！虽然jvm调优成熟的工具已经有很多：jconsole、大名鼎鼎的VisualVM，IBM的Memory Analyzer等等，但是在生产环境出现问题的时候，一方面工具的使用会有所限制，另一方面喜欢装X的我们，总喜欢在出现问题的时候在终端输入一些命令来解决。所有的工具几乎都是依赖于jdk的接口和底层的这些命令，研究这些命令的使用也让我们更能了解jvm构成和特性。

Sun JDK监控和故障处理命令有jps jstat jmap jhat jstack jinfo下面做一一介绍

# Java启动参数

其一是**标准参数**（-），所有的JVM实现都必须实现这些参数的功能，而且向后兼容；
其二是**非标准参数**（-X），默认jvm实现这些参数的功能，但是并不保证所有jvm实现都满足，且不保证向后兼容；
其三是**非Stable参数**（-XX），此类参数各个jvm实现会有所不同，将来可能会随时取消，需要慎重使用；

## 常用标准参数

**verbose 
-verbose:class** 
 输出jvm载入类的相关信息，当jvm报告说找不到类或者类冲突时可此进行诊断。
**-verbose:gc** 
 输出每次GC的相关情况。
**-verbose:jni** 
 输出native方法调用的相关情况，一般用于诊断jni调用错误信息。

##  常用非标准参数

**-Xms**512m 设置JVM促使内存为512m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。

-**Xmx**512m ，设置JVM最大可用内存为512M。

**-Xmn**200m**：**设置年轻代大小为200M。整个堆大小=年轻代大小 + 年老代大小 + 持久代大小。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。

**-Xss**128k：

设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。

**-Xloggc:file** 

与-verbose:gc功能类似，只是将每次GC事件的相关情况记录到一个文件中，文件的位置最好在本地，以避免网络的潜在问题。
 若与verbose命令同时出现在命令行中，则以-Xloggc为准。
**-Xprof**

 跟踪正运行的程序，并将跟踪数据在标准输出输出；适合于开发环境调试。

## 非Stable参数

用-XX作为前缀的参数列表在jvm中可能是不健壮的，SUN也不推荐使用，后续可能会在没有通知的情况下就直接取消了；但是由于这些参数中的确有很多是对我们很有用的，比如我们经常会见到的-XX:PermSize、-XX:MaxPermSize等等；

首先来介绍**行为参数**：

| 参数及其默认值              | 描述                                                      |
| --------------------------- | --------------------------------------------------------- |
| -XX:-DisableExplicitGC      | 禁止调用System.gc()；但jvm的gc仍然有效                    |
| -XX:+MaxFDLimit             | 最大化文件描述符的数量限制                                |
| -XX:+ScavengeBeforeFullGC   | 新生代GC优先于Full GC执行                                 |
| -XX:+UseGCOverheadLimit     | 在抛出OOM之前限制jvm耗费在GC上的时间比例                  |
| **-XX:-UseConcMarkSweepGC** | **对老生代采用并发标记交换算法进行GC**                    |
| **-XX:-UseParallelGC**      | **启用并行GC**                                            |
| -XX:-UseParallelOldGC       | 对Full GC启用并行，当-XX:-UseParallelGC启用时该项自动启用 |
| **-XX:-UseSerialGC**        | **启用串行GC**                                            |
| -XX:+UseThreadPriorities    | 启用本地线程优先级                                        |

 上面表格中黑体的三个参数代表着jvm中GC执行的三种方式，即**串行、并行、并发**；
串行（**SerialGC**）是jvm的默认GC方式，一般适用于小型应用和单处理器，算法比较简单，GC效率也较高，但可能会给应用带来停顿；
并行（**ParallelGC**）是指GC运行时，对应用程序运行没有影响，GC和app两者的线程在并发执行，这样可以最大限度不影响app的运行；
并发（**ConcMarkSweepGC**）是指多个线程并发执行GC，一般适用于多处理器系统中，可以提高GC的效率，但算法复杂，系统消耗较大；

**性能调优**参数列表：

| 参数及其默认值                | 描述                                  |
| ----------------------------- | ------------------------------------- |
| -XX:LargePageSizeInBytes=4m   | 设置用于Java堆的大页面尺寸            |
| -XX:MaxHeapFreeRatio=70       | GC后java堆中空闲量占的最大比例        |
| **-XX:MaxNewSize=size**       | **新生成对象能占用内存的最大值**      |
| **-XX:MaxPermSize=64m**       | **老生代对象能占用内存的最大值**      |
| -XX:MinHeapFreeRatio=40       | GC后java堆中空闲量占的最小比例        |
| -XX:NewRatio=2                | 新生代内存容量与老生代内存容量的比例  |
| **-XX:NewSize=2.125m**        | **新生代对象生成时占用内存的默认值**  |
| -XX:ReservedCodeCacheSize=32m | 保留代码占用的内存容量                |
| -XX:ThreadStackSize=512       | 设置线程栈大小，若为0则使用系统默认值 |
| -XX:+UseLargePages            | 使用大页面内存                        |

 **调试参数**列表：

| 参数及其默认值                                 | 描述                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| -XX:-CITime                                    | 打印消耗在JIT编译的时间                                      |
| -XX:ErrorFile=./hs_err_pid<pid>.log            | 保存错误日志或者数据到文件中                                 |
| -XX:-ExtendedDTraceProbes                      | 开启solaris特有的dtrace探针                                  |
| **-XX:HeapDumpPath=./java_pid.hprof**          | **指定导出堆信息时的路径或文件名**                           |
| **-XX:-HeapDumpOnOutOfMemoryError**            | **当首次遭遇OOM时导出此时堆中相关信息**                      |
| -XX:OnError="<cmd args>;<cmd args>"            | 出现致命ERROR之后运行自定义命令                              |
| -XX:OnOutOfMemoryError="<cmd args>;<cmd args>" | 当首次遭遇OOM时执行自定义命令                                |
| -XX:-PrintClassHistogram                       | 遇到Ctrl-Break后打印类实例的柱状信息，与jmap -histo功能相同  |
| **-XX:-PrintConcurrentLocks**                  | **遇到Ctrl-Break后打印并发锁的相关信息，与jstack -l功能相同** |
| **-XX:-PrintCommandLineFlags**                 | **打印在命令行中出现过的标记**                               |
| -XX:-PrintCompilation                          | 当一个方法被编译时打印相关信息                               |
| -XX:-PrintGC                                   | 每次GC时打印相关信息                                         |
| **-XX:-PrintGCDetails**                        | **每次GC时打印详细信息**                                     |
| -XX:-PrintGCTimeStamps                         | 打印每次GC的时间戳                                           |
| -XX:-TraceClassLoading                         | 跟踪类的加载信息                                             |
| -XX:-TraceClassLoadingPreorder                 | 跟踪被引用到的所有类的加载信息                               |
| -XX:-TraceClassResolution                      | 跟踪常量池                                                   |
| -XX:-TraceClassUnloading                       | 跟踪类的卸载信息                                             |
| -XX:-TraceLoaderConstraints                    | 跟踪类加载器约束的相关信息                                   |

# GC日志查看

可以通过在java命令中加入参数来指定对应的gc类型，打印gc日志信息并输出至文件等策略。

GC的日志是以替换的方式(>)写入的，而不是追加(>>)，如果下次写入到同一个文件中的话，以前的GC内容会被清空。

对应的参数列表

```bash
-XX:+PrintGC 输出GC日志
-XX:+PrintGCDetails 输出GC的详细日志
-XX:+PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式）
-XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800）
-XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息
-Xloggc:../logs/gc.log 日志文件的输出路径
```

![image-20200507134215195](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200507134218-940820.png) 

# JVM调优命令

## jps

JVM Process Status Tool,显示指定系统内所有的HotSpot虚拟机进程。

### 命令格式

```bash
jps [options] [hostid]
```

### option参数

> - -l : 输出主类全名或jar路径
> - -q : 只输出LVMID
> - -m : 输出JVM启动时传递给main()的参数
> - -v : 输出JVM启动时显示指定的JVM参数

其中[option]、[hostid]参数也可以不写。

### 示例

```bash
$ jps -l -m
  28920 org.apache.catalina.startup.Bootstrap start
  11589 org.apache.catalina.startup.Bootstrap start
  25816 sun.tools.jps.Jps -l -m
```

## jstat

jstat(JVM statistics Monitoring)是用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。

### 命令格式

```bash
jstat [option] LVMID [interval] [count]
```

### 参数

> - [option] : 操作参数
> - LVMID : 本地虚拟机进程ID
> - [interval] : 连续输出的时间间隔
> - [count] : 连续输出的次数

#### option 参数总览

| Option           | Displays…                                                    |
| ---------------- | ------------------------------------------------------------ |
| class            | class loader的行为统计。Statistics on the behavior of the class loader. |
| compiler         | HotSpt JIT编译器行为统计。Statistics of the behavior of the HotSpot Just-in-Time compiler. |
| gc               | 垃圾回收堆的行为统计。Statistics of the behavior of the garbage collected heap. |
| gccapacity       | 各个垃圾回收代容量(young,old,perm)和他们相应的空间统计。Statistics of the capacities of the generations and their corresponding spaces. |
| gcutil           | 垃圾回收统计概述。Summary of garbage collection statistics.  |
| gccause          | 垃圾收集统计概述（同-gcutil），附加最近两次垃圾回收事件的原因。Summary of garbage collection statistics (same as -gcutil), with the cause of the last and |
| gcnew            | 新生代行为统计。Statistics of the behavior of the new generation. |
| gcnewcapacity    | 新生代与其相应的内存空间的统计。Statistics of the sizes of the new generations and its corresponding spaces. |
| gcold            | 年老代和永生代行为统计。Statistics of the behavior of the old and permanent generations. |
| gcoldcapacity    | 年老代行为统计。Statistics of the sizes of the old generation. |
| gcpermcapacity   | 永生代行为统计。Statistics of the sizes of the permanent generation. |
| printcompilation | HotSpot编译方法统计。HotSpot compilation method statistics.  |

#### option 参数详解

##### -class

监视类装载、卸载数量、总空间以及耗费的时间

```bash
$ jstat -class 11589
 Loaded  Bytes  Unloaded  Bytes     Time   
  7035  14506.3     0     0.0       3.67
```

> - Loaded : 加载class的数量
> - Bytes : class字节大小
> - Unloaded : 未加载class的数量
> - Bytes : 未加载class的字节大小
> - Time : 加载时间

##### -compiler

输出JIT编译过的方法数量耗时等

```bash
$ jstat -compiler 1262
Compiled Failed Invalid   Time   FailedType FailedMethod
    2573      1       0    47.60          1 org/apache/catalina/loader/WebappClassLoader findResourceInternal  
```

> - Compiled : 编译数量
> - Failed : 编译失败数量
> - Invalid : 无效数量
> - Time : 编译耗时
> - FailedType : 失败类型
> - FailedMethod : 失败方法的全限定名

##### -gc

垃圾回收堆的行为统计，常用命令

```bash
$ jstat -gc 1262
 S0C    S1C     S0U     S1U   EC       EU        OC         OU        PC       PU         YGC    YGCT    FGC    FGCT     GCT   
26112.0 24064.0 6562.5  0.0   564224.0 76274.5   434176.0   388518.3  524288.0 42724.7    320    6.417   1      0.398    6.815
```

C即Capacity 总容量，U即Used 已使用的容量

> - S0C : survivor0区的总容量
> - S1C : survivor1区的总容量
> - S0U : survivor0区已使用的容量
> - S1U : survivor1区已使用的容量
> - EC : Eden区的总容量
> - EU : Eden区已使用的容量
> - OC : Old区的总容量
> - OU : Old区已使用的容量
> - PC 当前perm的容量 (KB)
> - PU perm的使用 (KB)
> - YGC : 新生代垃圾回收次数
> - YGCT : 新生代垃圾回收时间
> - FGC : 老年代垃圾回收次数
> - FGCT : 老年代垃圾回收时间
> - GCT : 垃圾回收总消耗时间

```bash
$ jstat -gc 1262 2000 20
```

这个命令意思就是每隔2000ms输出1262的gc情况，一共输出20次

##### -gccapacity

同-gc，不过还会输出Java堆各区域使用到的最大、最小空间

```bash
$ jstat -gccapacity 1262
 NGCMN    NGCMX     NGC    S0C   S1C       EC         OGCMN      OGCMX      OGC        OC       PGCMN    PGCMX     PGC      PC         YGC    FGC 
614400.0 614400.0 614400.0 26112.0 24064.0 564224.0   434176.0   434176.0   434176.0   434176.0 524288.0 1048576.0 524288.0 524288.0    320     1  
```

> - NGCMN : 新生代占用的最小空间
> - NGCMX : 新生代占用的最大空间
> - OGCMN : 老年代占用的最小空间
> - OGCMX : 老年代占用的最大空间
> - OGC：当前年老代的容量 (KB)
> - OC：当前年老代的空间 (KB)
> - PGCMN : perm占用的最小空间
> - PGCMX : perm占用的最大空间

##### -gcutil

同-gc，不过输出的是已使用空间占总空间的百分比

```bash
$ jstat -gcutil 28920
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT   
 12.45   0.00  33.85   0.00   4.44  4       0.242     0    0.000    0.242
```

##### -gccause

垃圾收集统计概述（同-gcutil），附加最近两次垃圾回收事件的原因

```bash
$ jstat -gccause 28920
  S0     S1     E      O      P       YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC                 
 12.45   0.00  33.85   0.00   4.44      4    0.242     0    0.000    0.242   Allocation Failure   No GC  
```

> - LGCC：最近垃圾回收的原因
> - GCC：当前垃圾回收的原因

##### -gcnew

统计新生代的行为

```bash
$ jstat -gcnew 28920
 S0C      S1C      S0U        S1U  TT  MTT  DSS      EC        EU         YGC     YGCT  
 419392.0 419392.0 52231.8    0.0  6   6    209696.0 3355520.0 1172246.0  4       0.242
```

> - TT：Tenuring threshold(提升阈值)
> - MTT：最大的tenuring threshold
> - DSS：survivor区域大小 (KB)

##### -gcnewcapacity

新生代与其相应的内存空间的统计

```bash
$ jstat -gcnewcapacity 28920
  NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     S1C       ECMX        EC        YGC   FGC 
 4194304.0  4194304.0  4194304.0 419392.0 419392.0 419392.0 419392.0  3355520.0  3355520.0     4     0
```

> - NGC:当前年轻代的容量 (KB)
> - S0CMX:最大的S0空间 (KB)
> - S0C:当前S0空间 (KB)
> - ECMX:最大eden空间 (KB)
> - EC:当前eden空间 (KB)

##### -gcold

统计旧生代的行为

```bash
$ jstat -gcold 28920
   PC       PU        OC           OU       YGC    FGC    FGCT     GCT   
1048576.0  46561.7   6291456.0     0.0      4      0      0.000    0.242
```

##### -gcoldcapacity

统计旧生代的大小和空间

```bash
$ jstat -gcoldcapacity 28920
   OGCMN       OGCMX        OGC         OC         YGC   FGC    FGCT     GCT   
  6291456.0   6291456.0   6291456.0   6291456.0     4     0    0.000    0.242
```

##### -gcpermcapacity

永生代行为统计

```bash
$ jstat -gcpermcapacity 28920
    PGCMN      PGCMX       PGC         PC      YGC   FGC    FGCT     GCT   
 1048576.0  2097152.0  1048576.0  1048576.0     4     0    0.000    0.242
```

##### -printcompilation

hotspot编译方法统计

```bash
$ jstat -printcompilation 28920
    Compiled  Size  Type Method
    1291      78     1    java/util/ArrayList indexOf
```

> - Compiled：被执行的编译任务的数量
> - Size：方法字节码的字节数
> - Type：编译类型
> - Method：编译方法的类名和方法名。类名使用”/” 代替 “.” 作为空间分隔符. 方法名是给出类的方法名. 格式是一致于HotSpot - XX:+PrintComplation 选项

## jmap

jmap(JVM Memory Map)命令用于生成heap dump文件，如果不使用这个命令，还可以使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现OOM的时候·自动生成dump文件。 jmap不仅能生成dump文件，还阔以查询finalize执行队列、Java堆和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等。

### 命令格式

```bash
jmap [option] LVMID
```

### option参数

> - dump : 生成堆转储快照
> - finalizerinfo : 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
> - heap : 显示Java堆详细信息
> - histo : 显示堆中对象的统计信息
> - permstat : to print permanent generation statistics
> - F : 当-dump没有响应时，强制生成dump快照

### 示例

##### -dump

常用格式

```bash
-dump::live,format=b,file=<filename> pid 
```

dump堆到文件,format指定输出格式，live指明是活着的对象,file指定文件名

```bash
$ jmap -dump:live,format=b,file=dump.hprof 28920
  Dumping heap to /home/xxx/dump.hprof ...
  Heap dump file created
```

dump.hprof这个后缀是为了后续可以直接用MAT(Memory Anlysis Tool)打开。

##### -finalizerinfo

打印等待回收对象的信息

```bash
$ jmap -finalizerinfo 28920
  Attaching to process ID 28920, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 24.71-b01
  Number of objects pending for finalization: 0
```

可以看到当前F-QUEUE队列中并没有等待Finalizer线程执行finalizer方法的对象。

##### -heap

打印heap的概要信息，GC使用的算法，heap的配置及wise heap的使用情况,可以用此来判断内存目前的使用情况以及垃圾回收情况

```bash
$ jmap -heap 28920
  Attaching to process ID 28920, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 24.71-b01  

  using thread-local object allocation.
  Parallel GC with 4 thread(s)//GC 方式  

  Heap Configuration: //堆内存初始化配置
     MinHeapFreeRatio = 0 //对应jvm启动参数-XX:MinHeapFreeRatio设置JVM堆最小空闲比率(default 40)
     MaxHeapFreeRatio = 100 //对应jvm启动参数 -XX:MaxHeapFreeRatio设置JVM堆最大空闲比率(default 70)
     MaxHeapSize      = 2082471936 (1986.0MB) //对应jvm启动参数-XX:MaxHeapSize=设置JVM堆的最大大小
     NewSize          = 1310720 (1.25MB)//对应jvm启动参数-XX:NewSize=设置JVM堆的‘新生代’的默认大小
     MaxNewSize       = 17592186044415 MB//对应jvm启动参数-XX:MaxNewSize=设置JVM堆的‘新生代’的最大大小
     OldSize          = 5439488 (5.1875MB)//对应jvm启动参数-XX:OldSize=<value>:设置JVM堆的‘老生代’的大小
     NewRatio         = 2 //对应jvm启动参数-XX:NewRatio=:‘新生代’和‘老生代’的大小比率
     SurvivorRatio    = 8 //对应jvm启动参数-XX:SurvivorRatio=设置年轻代中Eden区与Survivor区的大小比值 
     PermSize         = 21757952 (20.75MB)  //对应jvm启动参数-XX:PermSize=<value>:设置JVM堆的‘永生代’的初始大小
     MaxPermSize      = 85983232 (82.0MB)//对应jvm启动参数-XX:MaxPermSize=<value>:设置JVM堆的‘永生代’的最大大小
     G1HeapRegionSize = 0 (0.0MB)  

  Heap Usage://堆内存使用情况
  PS Young Generation
  Eden Space://Eden区内存分布
     capacity = 33030144 (31.5MB)//Eden区总容量
     used     = 1524040 (1.4534378051757812MB)  //Eden区已使用
     free     = 31506104 (30.04656219482422MB)  //Eden区剩余容量
     4.614088270399305% used //Eden区使用比率
  From Space:  //其中一个Survivor区的内存分布
     capacity = 5242880 (5.0MB)
     used     = 0 (0.0MB)
     free     = 5242880 (5.0MB)
     0.0% used
  To Space:  //另一个Survivor区的内存分布
     capacity = 5242880 (5.0MB)
     used     = 0 (0.0MB)
     free     = 5242880 (5.0MB)
     0.0% used
  PS Old Generation //当前的Old区内存分布
     capacity = 86507520 (82.5MB)
     used     = 0 (0.0MB)
     free     = 86507520 (82.5MB)
     0.0% used
  PS Perm Generation//当前的 “永生代” 内存分布
     capacity = 22020096 (21.0MB)
     used     = 2496528 (2.3808746337890625MB)
     free     = 19523568 (18.619125366210938MB)
     11.337498256138392% used  

  670 interned Strings occupying 43720 bytes.
```

可以很清楚的看到Java堆中各个区域目前的情况。

##### -histo

打印堆的对象统计，包括对象数、内存大小等等 （因为在dump:live前会进行full gc，如果带上live则只统计活对象，因此不加live的堆大小要大于加live堆的大小 ）

```bash
$ jmap -histo:live 28920 | more
 num     #instances         #bytes  class name
----------------------------------------------
   1:         83613       12012248  <constMethodKlass>
   2:         23868       11450280  [B
   3:         83613       10716064  <methodKlass>
   4:         76287       10412128  [C
   5:          8227        9021176  <constantPoolKlass>
   6:          8227        5830256  <instanceKlassKlass>
   7:          7031        5156480  <constantPoolCacheKlass>
   8:         73627        1767048  java.lang.String
   9:          2260        1348848  <methodDataKlass>
  10:          8856         849296  java.lang.Class
  ....
```

仅仅打印了前10行

`xml class name`是对象类型，说明如下：

```bash
B  byte
C  char
D  double
F  float
I  int
J  long
Z  boolean
[  数组，如[I表示int[]
[L+类名 其他对象
```

##### -permstat

打印Java堆内存的永久保存区域的类加载器的智能统计信息。对于每个类加载器而言，它的名称、活跃度、地址、父类加载器、它所加载的类的数量和大小都会被打印。此外，包含的字符串数量和大小也会被打印。

```bash
$ jmap -permstat 28920
  Attaching to process ID 28920, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 24.71-b01
  finding class loader instances ..done.
  computing per loader stat ..done.
  please wait.. computing liveness.liveness analysis may be inaccurate ...
  
  class_loader            classes bytes   parent_loader           alive?  type  
  <bootstrap>             3111    18154296          null          live    <internal>
  0x0000000600905cf8      1       1888    0x0000000600087f08      dead    sun/reflect/DelegatingClassLoader@0x00000007800500a0
  0x00000006008fcb48      1       1888    0x0000000600087f08      dead    sun/reflect/DelegatingClassLoader@0x00000007800500a0
  0x00000006016db798      0       0       0x00000006008d3fc0      dead    java/util/ResourceBundle$RBClassLoader@0x0000000780626ec0
  0x00000006008d6810      1       3056      null          dead    sun/reflect/DelegatingClassLoader@0x00000007800500a0
```

##### -F

强制模式。如果指定的pid没有响应，请使用jmap -dump或jmap -histo选项。此模式下，不支持live子选项。

## jhat

jhat(JVM Heap Analysis Tool)命令是与jmap搭配使用，用来分析jmap生成的dump，jhat内置了一个微型的HTTP/HTML服务器，生成dump的分析结果后，可以在浏览器中查看。在此要注意，一般不会直接在服务器上进行分析，因为jhat是一个耗时并且耗费硬件资源的过程，一般把服务器生成的dump文件复制到本地或其他机器上进行分析。

### 命令格式

```bash
jhat [dumpfile]
```

### 参数

> - -stack false|true 关闭对象分配调用栈跟踪(tracking object allocation call stack)。 如果分配位置信息在堆转储中不可用. 则必须将此标志设置为 false. 默认值为 true.>
> - -refs false|true 关闭对象引用跟踪(tracking of references to objects)。 默认值为 true. 默认情况下, 返回的指针是指向其他特定对象的对象,如反向链接或输入引用(referrers or incoming references), 会统计/计算堆中的所有对象。>
> - -port port-number 设置 jhat HTTP server 的端口号. 默认值 7000.>
> - -exclude exclude-file 指定对象查询时需要排除的数据成员列表文件(a file that lists data members that should be excluded from the reachable objects query)。 例如, 如果文件列列出了 java.lang.String.value , 那么当从某个特定对象 Object o 计算可达的对象列表时, 引用路径涉及 java.lang.String.value 的都会被排除。>
> - -baseline exclude-file 指定一个基准堆转储(baseline heap dump)。 在两个 heap dumps 中有相同 object ID 的对象会被标记为不是新的(marked as not being new). 其他对象被标记为新的(new). 在比较两个不同的堆转储时很有用.>
> - -debug int 设置 debug 级别. 0 表示不输出调试信息。 值越大则表示输出更详细的 debug 信息.>
> - -version 启动后只显示版本信息就退出>
> - -J< flag > 因为 jhat 命令实际上会启动一个JVM来执行, 通过 -J 可以在启动JVM时传入一些启动参数. 例如, -J-Xmx512m 则指定运行 jhat 的Java虚拟机使用的最大堆内存为 512 MB. 如果需要使用多个JVM启动参数,则传入多个 -Jxxxxxx.

### 示例

```bash
$ jhat -J-Xmx512m dump.hprof
  eading from dump.hprof...
  Dump file created Fri Mar 11 17:13:42 CST 2016
  Snapshot read, resolving...
  Resolving 271678 objects...
  Chasing references, expect 54 dots......................................................
  Eliminating duplicate references......................................................
  Snapshot resolved.
  Started HTTP server on port 7000
  Server is ready.
```

中间的-J-Xmx512m是在dump快照很大的情况下分配512M内存去启动HTTP服务器，运行完之后就可在浏览器打开Http://localhost:7000进行快照分析 堆快照分析主要在最后面的Heap Histogram里，里面根据class列出了dump的时候所有存活对象。

分析同样一个dump快照，MAT需要的额外内存比jhat要小的多的多，所以建议使用MAT来进行分析，当然也看个人偏好。

### 分析

打开浏览器Http://localhost:7000，该页面提供了几个查询功能可供使用：

```bash
All classes including platform
Show all members of the rootset
Show instance counts for all classes (including platform)
Show instance counts for all classes (excluding platform)
Show heap histogram
Show finalizer summary
Execute Object Query Language (OQL) query
```

一般查看堆异常情况主要看这个两个部分： Show instance counts for all classes (excluding platform)，平台外的所有对象信息。如下图：

![image-20200415135132675](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200415135132-702710.png)  

Show heap histogram 以树状图形式展示堆情况。如下图：

![image-20200415135200504](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200415135204-254146.png) 

具体排查时需要结合代码，观察是否大量应该被回收的对象在一直被引用或者是否有占用内存特别大的对象无法被回收。
一般情况，会down到客户端用工具来分析

## jstack

jstack用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。 如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息, 如果现在运行的java程序呈现hung的状态，jstack是非常有用的。

### 命令格式

```bash
jstack [option] LVMID
```

### option参数

> - -F : 当正常输出请求不被响应时，强制输出线程堆栈
> - -l : 除堆栈外，显示关于锁的附加信息
> - -m : 如果调用到本地方法的话，可以显示C/C++的堆栈

### 示例

```bash
$ jstack -l 11494|more
2016-07-28 13:40:04
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.71-b01 mixed mode):

"Attach Listener" daemon prio=10 tid=0x00007febb0002000 nid=0x6b6f waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"http-bio-8005-exec-2" daemon prio=10 tid=0x00007feb94028000 nid=0x7b8c waiting on condition [0x00007fea8f56e000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000cae09b80> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None
      .....
```

### 分析

这里有一篇文章解释的很好 [分析打印出的文件内容](http://www.hollischuang.com/archives/110)

## jinfo

jinfo(JVM Configuration info)这个命令作用是实时查看和调整虚拟机运行参数。 之前的jps -v口令只能查看到显示指定的参数，如果想要查看未被显示指定的参数的值就要使用jinfo口令

### 命令格式

```bash
jinfo [option] [args] LVMID
```

### option参数

> - -flag : 输出指定args参数的值
> - -flags : 不需要args参数，输出所有JVM参数的值
> - -sysprops : 输出系统属性，等同于System.getProperties()

### 示例

```bash
$ jinfo -flag 11494
-XX:CMSInitiatingOccupancyFraction=80
```