# Java锁的种类

面试过程时，经常会被问到各种各样的锁，如乐观锁、读写锁等等，非常繁多，在此做一个总结。介绍的内容如下：

- 乐观锁/悲观锁
- 独享锁/共享锁
- 互斥锁/读写锁
- 可重入锁
- 公平锁/非公平锁
- 分段锁
- 偏向锁/轻量级锁/重量级锁
- 自旋锁

以上是一些锁的名词，这些分类并不是全是指锁的状态，有的指锁的特性，有的指锁的设计，下面总结的内容是对每个锁的名词进行一定的解释。

## 乐观锁/悲观锁

乐观锁与悲观锁并不是特指某两种类型的锁，是人们定义出来的概念或思想，主要是指看待并发同步的角度。

乐观锁：顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量，在Java中java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式CAS(Compare and Swap 比较并交换)实现的。

悲观锁：总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。比如Java里面的同步原语synchronized关键字的实现就是悲观锁。

悲观锁适合写操作非常多的场景，乐观锁适合读操作非常多的场景，不加锁会带来大量的性能提升。

悲观锁在Java中的使用，就是利用各种锁。

乐观锁在Java中的使用，是无锁编程，常常采用的是CAS算法，典型的例子就是原子类，通过CAS自旋实现原子操作的更新。

### 乐观锁

乐观锁总是认为不存在并发问题，每次去取数据的时候，总认为不会有其他线程对数据进行修改，因此不会上锁。但是在更新时会判断其他线程在这之前有没有对数据进行修改，一般会使用“数据版本机制”或“CAS操作”来实现。

#### 数据版本机制

实现数据版本一般有两种，第一种是使用版本号，第二种是使用时间戳。以版本号方式为例。

版本号方式：一般是在数据表中加上一个数据版本号version字段，表示数据被修改的次数，当数据被修改时，version值会加一。当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，若刚才读取到的version值为当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功。
核心SQL代码：

```sql
update table set xxx=#{xxx}, version=version+1 where id=#{id} and version=#{version};
```

#### CAS操作

CAS（Compare and Swap 比较并交换），当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。

CAS操作中包含三个操作数——需要读写的内存位置(V)、进行比较的预期原值(A)和拟写入的新值(B)。如果内存位置V的值与预期原值A相匹配，那么处理器会自动将该位置值更新为新值B，否则处理器不做任何操作。

###  悲观锁

悲观锁认为对于同一个数据的并发操作，一定会发生修改的，哪怕没有修改，也会认为修改。因此对于同一份数据的并发操作，悲观锁采取加锁的形式。悲观的认为，不加锁并发操作一定会出问题。

在对任意记录进行修改前，先尝试为该记录加上排他锁（exclusive locking）。

如果加锁失败，说明该记录正在被修改，那么当前查询可能要等待或者抛出异常。具体响应方式由开发者根据实际需要决定。

如果成功加锁，那么就可以对记录做修改，事务完成后就会解锁了。

期间如果有其他对该记录做修改或加排他锁的操作，都会等待我们解锁或直接抛出异常。

## 独享锁/共享锁

独享锁是指该锁一次只能被一个线程所持有。

共享锁是指该锁可被多个线程所持有。

对于Java ReentrantLock而言，其是独享锁。但是对于Lock的另一个实现类ReadWriteLock，其读锁是共享锁，其写锁是独享锁。

读锁的共享锁可保证并发读是非常高效的，读写，写读，写写的过程是互斥的。

独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。

对于Synchronized而言，当然是独享锁。

## 互斥锁/读写锁

上面讲的独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。

互斥锁在Java中的具体实现就是ReentrantLock。

读写锁在Java中的具体实现就是ReadWriteLock。

## 可重入锁

可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。说的有点抽象，下面会有一个代码的示例。

对于Java ReetrantLock而言，从名字就可以看出是一个重入锁，其名字是Re entrant Lock 重新进入锁。

对于Synchronized而言，也是一个可重入锁。可重入锁的一个好处是可一定程度避免死锁。

```java
synchronized void setA() throws Exception{
　　Thread.sleep(1000);
　　setB();
}

synchronized void setB() throws Exception{
　　Thread.sleep(1000);
}
```

上面的代码就是一个可重入锁的一个特点。如果不是可重入锁的话，setB可能不会被当前线程执行，可能造成死锁。

## 公平锁/非公平锁

公平锁是指多个线程按照申请锁的顺序来获取锁。

非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。

对于Java ReetrantLock而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。

对于Synchronized而言，也是一种非公平锁。由于其并不像ReentrantLock是通过AQS的来实现线程调度，所以并没有任何办法使其变成公平锁。

## 分段锁

分段锁其实是一种锁的设计，并不是具体的一种锁，对于ConcurrentHashMap而言，其并发的实现就是通过分段锁的形式来实现高效的并发操作。

我们以ConcurrentHashMap来说一下分段锁的含义以及设计思想，ConcurrentHashMap中的分段锁称为Segment，它即类似于HashMap（JDK7和JDK8中HashMap的实现）的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表；同时又是一个ReentrantLock（Segment继承了ReentrantLock）。

当需要put元素的时候，并不是对整个hashmap进行加锁，而是先通过hashcode来知道他要放在哪一个分段中，然后对这个分段进行加锁，所以当多线程put的时候，只要不是放在一个分段中，就实现了真正的并行的插入。

但是，在统计size的时候，可就是获取hashmap全局信息的时候，就需要获取所有的分段锁才能统计。

分段锁的设计目的是细化锁的粒度，当操作不需要更新整个数组的时候，就仅仅针对数组中的一项进行加锁操作。

## 偏向锁/轻量级锁/重量级锁

synchronized锁有四种状态，无锁，偏向锁，轻量级锁，重量级锁，这几个状态会随着竞争状态逐渐升级，锁可以升级但不能降级，但是偏向锁状态可以被重置为无锁状态

**偏向锁**
为什么要引入偏向锁？

因为经过HotSpot的作者大量的研究发现，大多数时候是不存在锁竞争的，常常是一个线程多次获得同一个锁，因此如果每次都要竞争锁会增大很多没有必要付出的代价，为了降低获取锁的代价，才引入的偏向锁。

偏向锁原理和升级过程

当线程1访问代码块并获取锁对象时，会在java对象头和栈帧中记录偏向的锁的threadID，因为偏向锁不会主动释放锁，因此以后线程1再次获取锁的时候，需要比较当前线程的threadID和Java对象头中的threadID是否一致，如果一致（还是线程1获取锁对象），则无需使用CAS来加锁、解锁；如果不一致（其他线程，如线程2要竞争锁对象，而偏向锁不会主动释放因此还是存储的线程1的threadID），那么需要查看Java对象头中记录的线程1是否存活，如果没有存活，那么锁对象被重置为无锁状态，其它线程（线程2）可以竞争将其设置为偏向锁；如果存活，那么立刻查找该线程（线程1）的栈帧信息，如果还是需要继续持有这个锁对象，那么暂停当前线程1，撤销偏向锁，升级为轻量级锁，如果线程1 不再使用该锁对象，那么将锁对象状态设为无锁状态，重新偏向新的线程。

**轻量级锁**
为什么要引入轻量级锁？

轻量级锁考虑的是竞争锁对象的线程不多，而且线程持有锁的时间也不长的情景。因为阻塞线程需要CPU从用户态转到内核态，代价较大，如果刚刚阻塞不久这个锁就被释放了，那这个代价就有点得不偿失了，因此这个时候就干脆不阻塞这个线程，让它自旋这等待锁释放。

轻量级锁原理和升级过程

线程1获取轻量级锁时会先把锁对象的对象头MarkWord复制一份到线程1的栈帧中创建的用于存储锁记录的空间（称为DisplacedMarkWord），然后使用CAS把对象头中的内容替换为线程1存储的锁记录（DisplacedMarkWord）的地址；

如果在线程1复制对象头的同时（在线程1CAS之前），线程2也准备获取锁，复制了对象头到线程2的锁记录空间中，但是在线程2CAS的时候，发现线程1已经把对象头换了，线程2的CAS失败，那么线程2就尝试使用自旋锁来等待线程1释放锁。 自旋锁简单来说就是让线程2在循环中不断CAS

但是如果自旋的时间太长也不行，因为自旋是要消耗CPU的，因此自旋的次数是有限制的，比如10次或者100次，如果自旋次数到了线程1还没有释放锁，或者线程1还在执行，线程2还在自旋等待，这时又有一个线程3过来竞争这个锁对象，那么这个时候轻量级锁就会膨胀为重量级锁。重量级锁把除了拥有锁的线程都阻塞，防止CPU空转。

![image-20210422112525547](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20210422112527-139456.png) 

## 自旋锁

在Java中，自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU。

# 锁的相关知识

## AQS

AbstractQueuedSynchronized 抽象队列式的同步器，AQS定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的ReentrantLock/Semaphore/CountDownLatch…

![image-20200428143028869](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200428143030-4821.png) 

AQS维护了一个volatile int state(代表共享资源)和一个FIFO线程等待队列（多线程争用资源被阻塞时会进入此队列）。

state的访问方式有三种：

```
1 getState()
2 setState()
3 compareAndSetState()
```

AQS定义两种资源共享方式：Exclusive（独占，只有一个线程能执行，如ReentrantLock）和Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）。

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

```
1 isHeldExclusively():该线程是否正在独占资源。只有用到condition才需要去实现它。
2 tryAquire(int):独占方式。尝试获取资源，成功则返回true，失败则返回false。
3 tryRelease(int):独占方式。尝试释放资源，成功则返回true，失败则返回false。
4 tryAcquireShared(int):共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
5 tryReleaseShared(int):共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。
```

以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其他线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多少次，这样才能保证state是能回到零态的。

再以CountDownLatch为例，任务分为N个子线程去执行，state为初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS减1。等到所有子线程都执行完后（即state=0），会unpark()主调用线程，然后主调用线程就会await()函数返回，继续后余动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。

## CAS

CAS（Compare and Swap 比较并交换）是乐观锁技术，当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其他线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。

CAS操作中包含三个操作数——需要读写的内存位置（V）、进行比较的预期原值（A）和拟写入的新值（B）。如果内存位置V的值与预期原值A相匹配，那么处理器会自动将该位置值更新为新值B，否则处理器不做任何操作。无论哪种情况，它都会在CAS指令之前返回该位置的值（在CAS的一些特殊情况下将仅返回CAS是否成功，而不提取当前值）。CAS有效地说明了“我认为位置V应该包含值A；如果包含该值，则将B放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可”。这其实和乐观锁的冲突检查+数据更新的原理是一样的。

JAVA对CAS的支持：

在JDK1.5中新增java.util.concurrent包就是建立在CAS之上的。相对于synchronized这种阻塞算法，CAS是非阻塞算法的一种常见实现。所以java.util.concurrent包中的AtomicInteger为例，看一下在不使用锁的情况下是如何保证线程安全的。主要理解getAndIncrement方法，该方法的作用相当于++i操作。

```java
public class AtomicInteger extends Number implements java.io.Serializable{
　　private volatile int value;
　　public final int get(){
　　　　return value;
　　}

　 public final int getAndIncrement(){
　　　　for (;;){
　　　　　　int current = get();
　　　　　　int next = current + 1;
　　　　　　if (compareAndSet(current, next))
　　　　　　return current;
　　　　}
　　}
 
　　public final boolean compareAndSet(int expect, int update){
　　　　return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
　　}
}
```

# synchornized与lock区别

- lock是一个接口，而synchornized是java关键字，synchornized是内置的语言实现。
- synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而lock在发生异常的时候，如果没有主动释放锁unlock() ，很可能会造死锁索现象，因此使用lock() 后需要在finally中释放锁。
- lock可以让等待锁的线程相应中断，而synchornized却不行，使用synchornized时，等待的线程会一直等待下去，不会相应中断。
- 通过lock可以知道有没有成功获取锁，而synchornized却无法办到。

# 实战

## synchronized

synchronized可重入锁验证

```java
public class MyLockTest implements Runnable {
    public synchronized void get() {
        System.out.println("2 enter thread name-->" + Thread.currentThread().getName());
        //reentrantLock.lock();
        System.out.println("3 get thread name-->" + Thread.currentThread().getName());
        set();
        //reentrantLock.unlock();
        System.out.println("5 leave run thread name-->" + Thread.currentThread().getName());
    }

    public synchronized void set() {
        //reentrantLock.lock();
        System.out.println("4 set thread name-->" + Thread.currentThread().getName());
        //reentrantLock.unlock();
    }

    @Override
    public void run() {
        System.out.println("1 run thread name-->" + Thread.currentThread().getName());
        get();
    }

    public static void main(String[] args) {
        MyLockTest test = new MyLockTest();
        for (int i = 0; i < 10; i++) {
            new Thread(test, "thread-" + i).start();
        }
    }

}
```

运行结果

```bash
1 run thread name-->thread-0
2 enter thread name-->thread-0
3 get thread name-->thread-0
1 run thread name-->thread-1
1 run thread name-->thread-2
4 set thread name-->thread-0
5 leave run thread name-->thread-0
1 run thread name-->thread-3
2 enter thread name-->thread-2
3 get thread name-->thread-2
4 set thread name-->thread-2
5 leave run thread name-->thread-2
2 enter thread name-->thread-1
3 get thread name-->thread-1
4 set thread name-->thread-1
5 leave run thread name-->thread-1
2 enter thread name-->thread-3
3 get thread name-->thread-3
4 set thread name-->thread-3
5 leave run thread name-->thread-3
1 run thread name-->thread-5
2 enter thread name-->thread-5
3 get thread name-->thread-5
4 set thread name-->thread-5
5 leave run thread name-->thread-5
1 run thread name-->thread-7
1 run thread name-->thread-6
2 enter thread name-->thread-7
3 get thread name-->thread-7
4 set thread name-->thread-7
1 run thread name-->thread-4
5 leave run thread name-->thread-7
1 run thread name-->thread-8
2 enter thread name-->thread-8
3 get thread name-->thread-8
4 set thread name-->thread-8
5 leave run thread name-->thread-8
1 run thread name-->thread-9
2 enter thread name-->thread-4
3 get thread name-->thread-4
4 set thread name-->thread-4
5 leave run thread name-->thread-4
2 enter thread name-->thread-6
3 get thread name-->thread-6
4 set thread name-->thread-6
5 leave run thread name-->thread-6
2 enter thread name-->thread-9
3 get thread name-->thread-9
4 set thread name-->thread-9
5 leave run thread name-->thread-9
```

get()方法中顺利进入了set()方法，说明synchronized的确是可重入锁。分析打印Log，thread-0先进入get方法体，这个时候thread-1、thread-2、thread-3等待进入，但当thread-0离开时，thread-2却先进入了方法体，没有按照thread-1、thread-2、thread-3的顺序进入get方法体，说明sychronized的确是非公平锁。而且在一个线程进入get方法体后，其他线程只能等待，无法同时进入，验证了synchronized是独占锁。

## ReentrantLock

ReentrantLock既可以构造公平锁又可以构造非公平锁，默认为非公平锁，将上面的代码改为用ReentrantLock实现，再次运行。

```java
import java.util.concurrent.locks.ReentrantLock;

public class MyLockTest implements Runnable {

    private ReentrantLock reentrantLock = new ReentrantLock();

    public void get() {
        System.out.println("2 enter thread name-->" + Thread.currentThread().getName());
        reentrantLock.lock();
        System.out.println("3 get thread name-->" + Thread.currentThread().getName());
        set();
        reentrantLock.unlock();
        System.out.println("5 leave run thread name-->" + Thread.currentThread().getName());
    }

    public void set() {
        reentrantLock.lock();
        System.out.println("4 set thread name-->" + Thread.currentThread().getName());
        reentrantLock.unlock();
    }

    @Override
    public void run() {
        System.out.println("1 run thread name-->" + Thread.currentThread().getName());
        get();
    }

    public static void main(String[] args) {
        MyLockTest test = new MyLockTest();
        for (int i = 0; i < 10; i++) {
            new Thread(test, "thread-" + i).start();
        }
    }

}
```

运行结果

```bash
1 run thread name-->thread-0
2 enter thread name-->thread-0
1 run thread name-->thread-1
2 enter thread name-->thread-1
3 get thread name-->thread-0
4 set thread name-->thread-0
1 run thread name-->thread-3
2 enter thread name-->thread-3
3 get thread name-->thread-3
4 set thread name-->thread-3
5 leave run thread name-->thread-3
1 run thread name-->thread-4
2 enter thread name-->thread-4
3 get thread name-->thread-4
4 set thread name-->thread-4
5 leave run thread name-->thread-4
1 run thread name-->thread-5
2 enter thread name-->thread-5
3 get thread name-->thread-5
4 set thread name-->thread-5
5 leave run thread name-->thread-5
1 run thread name-->thread-7
2 enter thread name-->thread-7
3 get thread name-->thread-7
4 set thread name-->thread-7
5 leave run thread name-->thread-7
5 leave run thread name-->thread-0
3 get thread name-->thread-1
4 set thread name-->thread-1
5 leave run thread name-->thread-1
1 run thread name-->thread-2
2 enter thread name-->thread-2
3 get thread name-->thread-2
4 set thread name-->thread-2
5 leave run thread name-->thread-2
1 run thread name-->thread-9
2 enter thread name-->thread-9
3 get thread name-->thread-9
4 set thread name-->thread-9
5 leave run thread name-->thread-9
1 run thread name-->thread-6
1 run thread name-->thread-8
2 enter thread name-->thread-8
3 get thread name-->thread-8
4 set thread name-->thread-8
5 leave run thread name-->thread-8
2 enter thread name-->thread-6
3 get thread name-->thread-6
4 set thread name-->thread-6
5 leave run thread name-->thread-6
```

的确如其名，可重入锁，当然默认的确是非公平锁。thread-0持有锁期间，thread-1等待拥有锁，当thread-0释放锁时thread-3先获取到锁，并非按照先后顺序获取锁的。

将其构造为公平锁，看看运行结果是否符合预期。查看源码构造公平锁很简单，只要在构造器传入boolean值true即可。

```java
	/**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

修改上面例程的代码构造方法为：

```
1 ReentrantLock reentrantLock = new ReentrantLock(true);
```

ReentrantLock实现公平锁。

```java
 import java.util.concurrent.locks.ReentrantLock;

public class MyLockTest implements Runnable {

    private ReentrantLock reentrantLock = new ReentrantLock(true);

    public void get() {
        System.out.println("2 enter thread name-->" + Thread.currentThread().getName());
        reentrantLock.lock();
        System.out.println("3 get thread name-->" + Thread.currentThread().getName());
        set();
        reentrantLock.unlock();
        System.out.println("5 leave run thread name-->" + Thread.currentThread().getName());
    }

    public void set() {
        reentrantLock.lock();
        System.out.println("4 set thread name-->" + Thread.currentThread().getName());
        reentrantLock.unlock();
    }

    @Override
    public void run() {
        System.out.println("1 run thread name-->" + Thread.currentThread().getName());
        get();
    }

    public static void main(String[] args) {
        MyLockTest test = new MyLockTest();
        for (int i = 0; i < 10; i++) {
            new Thread(test, "thread-" + i).start();
        }
    }

}
```

运行结果

```bash
1 run thread name-->thread-0
2 enter thread name-->thread-0
3 get thread name-->thread-0
1 run thread name-->thread-2
2 enter thread name-->thread-2
4 set thread name-->thread-0
1 run thread name-->thread-3
2 enter thread name-->thread-3
1 run thread name-->thread-1
2 enter thread name-->thread-1
1 run thread name-->thread-5
2 enter thread name-->thread-5
3 get thread name-->thread-2
4 set thread name-->thread-2
5 leave run thread name-->thread-2
5 leave run thread name-->thread-0
3 get thread name-->thread-3
4 set thread name-->thread-3
5 leave run thread name-->thread-3
1 run thread name-->thread-9
2 enter thread name-->thread-9
3 get thread name-->thread-1
4 set thread name-->thread-1
5 leave run thread name-->thread-1
3 get thread name-->thread-5
4 set thread name-->thread-5
5 leave run thread name-->thread-5
3 get thread name-->thread-9
4 set thread name-->thread-9
5 leave run thread name-->thread-9
1 run thread name-->thread-6
2 enter thread name-->thread-6
3 get thread name-->thread-6
4 set thread name-->thread-6
1 run thread name-->thread-7
5 leave run thread name-->thread-6
2 enter thread name-->thread-7
3 get thread name-->thread-7
4 set thread name-->thread-7
5 leave run thread name-->thread-7
1 run thread name-->thread-4
2 enter thread name-->thread-4
3 get thread name-->thread-4
1 run thread name-->thread-8
2 enter thread name-->thread-8
4 set thread name-->thread-4
5 leave run thread name-->thread-4
3 get thread name-->thread-8
4 set thread name-->thread-8
5 leave run thread name-->thread-8
```

公平锁在多个线程想要同时获取锁的时候，会发现再排队，按照先来后到的顺序进行。

## ReentrantReadWriteLock

 读写锁的性能都会比排他锁要好，因为大多数场景读是多于写的。在读多于写的情况下，读写锁能够提供比排它锁更好的并发性和吞吐量。Java并发包提供读写锁的实现是ReentrantReadWriteLock。

| 特性       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 公平性选择 | 支持非公平(默认)和公平的锁获取方式，吞吐量还是非公平优于公平 |
| 重进入     | 该锁支持重进入，以读写线程为例：读线程在获取了读锁之后，能够再次获取读锁。而写线程在获取了写锁之后能够再次获取写锁，同时也可以获取读锁 |
| 锁降级     | 遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级成为读锁 |

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class MyLockTest {

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    Cache.put("key", new String(Thread.currentThread().getName() + " joke"));
                }
            }, "threadW-" + i).start();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Cache.get("key"));
                }
            }, "threadR-" + i).start();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    Cache.clear();
                }
            }, "threadC-" + i).start();
        }
    }
}

class Cache {
    static Map<String, Object> map = new HashMap<String, Object>();
    static ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    static Lock r = rwl.readLock();
    static Lock w = rwl.writeLock();

    // 获取一个key对应的value
    public static final Object get(String key) {
        r.lock();
        try {
            System.out.println("get " + Thread.currentThread().getName());
            return map.get(key);
        } finally {
            r.unlock();
        }
    }

    // 设置key对应的value，并返回旧有的value
    public static final Object put(String key, Object value) {
        w.lock();
        try {
            System.out.println("put " + Thread.currentThread().getName());
            return map.put(key, value);
        } finally {
            w.unlock();
        }
    }

    // 清空所有的内容
    public static final void clear() {
        w.lock();
        try {
            System.out.println("clear " + Thread.currentThread().getName());
            map.clear();
        } finally {
            w.unlock();
        }
    }
}
```

运行结果

```bash
put threadW-0
clear threadC-1
put threadW-1
get threadR-1
threadW-1 joke
put threadW-2
get threadR-0
threadW-2 joke
clear threadC-0
get threadR-2
null
clear threadC-4
clear threadC-2
clear threadC-3
put threadW-4
put threadW-3
get threadR-3
threadW-3 joke
put threadW-5
get threadR-4
threadW-5 joke
clear threadC-5
put threadW-6
put threadW-7
get threadR-7
threadW-7 joke
get threadR-5
threadW-7 joke
get threadR-6
threadW-7 joke
clear threadC-6
clear threadC-7
put threadW-8
clear threadC-8
put threadW-9
get threadR-9
threadW-9 joke
clear threadC-9
get threadR-8
null
```

可看到普通HashMap在多线程中数据可见性正常。