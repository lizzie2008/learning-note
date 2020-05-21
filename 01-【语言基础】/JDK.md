# 集合



# Class.forName 和 ClassLoader

## 解释

在java中Class.forName()和ClassLoader都可以对类进行加载。ClassLoader就是遵循双亲委派模型最终调用启动类加载器的类加载器，实现的功能是“通过一个类的全限定名来获取描述此类的二进制字节流”，获取到二进制流后放到JVM中。Class.forName()方法实际上也是调用的ClassLoader来实现的。

Class.forName(String className)；这个方法的源码是

```java
@CallerSensitive
public static Class<?> forName(String className)
            throws ClassNotFoundException {
    Class<?> caller = Reflection.getCallerClass();
    return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
}
```

最后调用的方法是forName0这个方法，在这个forName0方法中的第二个参数被默认设置为了true，这个参数代表是否对加载的类进行初始化，设置为true时会类进行初始化，代表会执行类中的静态代码块，以及对静态变量的赋值等操作。

也可以调用Class.forName(String name, boolean initialize,ClassLoader loader)方法来手动选择在加载类的时候是否要对类进行初始化。Class.forName(String name, boolean initialize,ClassLoader loader)的源码如下：

```java
/* @param name       fully qualified name of the desired class
 * @param initialize if {@code true} the class will be initialized.
 *                   See Section 12.4 of <em>The Java Language Specification</em>.
 * @param loader     class loader from which the class must be loaded
 * @return           class object representing the desired class
 *
 * @exception LinkageError if the linkage fails
 * @exception ExceptionInInitializerError if the initialization provoked
 *            by this method fails
 * @exception ClassNotFoundException if the class cannot be located by
 *            the specified class loader
 *
 * @see       java.lang.Class#forName(String)
 * @see       java.lang.ClassLoader
 * @since     1.2     */
@CallerSensitive
public static Class<?> forName(String name, boolean initialize,
                               ClassLoader loader)
    throws ClassNotFoundException
{
    Class<?> caller = null;
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        // Reflective call to get caller class is only needed if a security manager
        // is present.  Avoid the overhead of making this call otherwise.
        caller = Reflection.getCallerClass();
        if (sun.misc.VM.isSystemDomainLoader(loader)) {
            ClassLoader ccl = ClassLoader.getClassLoader(caller);
            if (!sun.misc.VM.isSystemDomainLoader(ccl)) {
                sm.checkPermission(
                    SecurityConstants.GET\_CLASSLOADER\_PERMISSION);
            }
        }
    }
    return forName0(name, initialize, loader, caller);
}
```

源码中的注释只摘取了一部分，其中对参数initialize的描述是：if {@code true} the class will be initialized.意思就是说：如果参数为true，则加载的类将会被初始化。

## 举例

下面还是举例来说明结果吧：

一个含有静态代码块、静态变量、赋值给静态变量的静态方法的类

```java
public class ClassForName {

    //静态代码块
    static {
        System.out.println("执行了静态代码块");
    }
    //静态变量
    private static String staticFiled = staticMethod();

    //赋值静态变量的静态方法
    public static String staticMethod(){
        System.out.println("执行了静态方法");
        return "给静态字段赋值了";
    }

}
```

使用Class.forName()的测试方法：

```java
@Test
public void test45(){
    try {
        ClassLoader.getSystemClassLoader().loadClass("com.eurekaclient2.client2.ClassForName");
        System.out.println("#########-------------结束符------------##########");
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
}
```

运行结果：

```
执行了静态代码块执行了静态方法#########-------------结束符------------##########  
```

使用ClassLoader的测试方法：

```java
@Test
public void test45(){
    try {
        ClassLoader.getSystemClassLoader().loadClass("com.eurekaclient2.client2.ClassForName");
        System.out.println("#########-------------结束符------------##########");
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
}
```

运行结果：

```
#########-------------结束符------------##########
```

> 根据运行结果得出Class.forName加载类时将类进了初始化，而ClassLoader的loadClass并没有对类进行初始化，只是把类加载到了虚拟机中。

## 应用场景

在我们熟悉的Spring框架中的IOC的实现就是使用的ClassLoader。

而在我们使用JDBC时通常是使用Class.forName()方法来加载数据库连接驱动。这是因为在JDBC规范中明确要求Driver(数据库驱动)类必须向DriverManager注册自己。

以MySQL的驱动为例解释：

```java
public class Driver extends NonRegisteringDriver implements java.sql.Driver {  
    // ~ Static fields/initializers  
    // ---------------------------------------------  

    //  
    // Register ourselves with the DriverManager  
    //  
    static {  
        try {  
            java.sql.DriverManager.registerDriver(new Driver());  
        } catch (SQLException E) {  
            throw new RuntimeException("Can't register driver!");  
        }  
    }  

    // ~ Constructors  
    // -----------------------------------------------------------  

    /** 
     * Construct a new driver and register it with DriverManager 
     *  
     * @throws SQLException 
     *             if a database error occurs. 
     */  
    public Driver() throws SQLException {  
        // Required for Class.forName().newInstance()      }  

}
```

我们看到Driver注册到DriverManager中的操作写在了静态代码块中，这就是为什么在写JDBC时使用Class.forName()的原因了。

# 观察者模式与监听器模式

**观察者模式**：观察者(Observer)相当于事件监听者，被观察者(Observable)相当于事件源和事件，执行逻辑时通知observer即可触发oberver的update,同时可传被观察者和参数

**监听器模式**：事件源经过事件的封装传给监听器，当事件源触发事件后，监听器接收到事件对象可以回调事件的方法。

当事件源对象上发生操作时，将会调用事件监听器的一个方法，并在调用该方法时把事件对象传递过去。

![image-20200512104557197](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200512104604-259527.png) 

**观察者模式与监听模式的区别**

![image-20200512104644201](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200512104645-148939.png) 

## JDK自带的观察者模式

subject -> java.util.Observable(类)

```java
void addObserver(Observer o) 
如果观察者与集合中已有的观察者不同，则向对象的观察者集中添加此观察者。 
protected  void clearChanged() 
指示对象不再改变，或者它已对其所有的观察者通知了最近的改变，所以 hasChanged 方法将返回 false。 
int countObservers() 
返回 Observable 对象的观察者数目。 
void deleteObserver(Observer o) 
从对象的观察者集合中删除某个观察者。 
void deleteObservers() 
清除观察者列表，使此对象不再有任何观察者。 
boolean hasChanged() 
测试对象是否改变。 
void notifyObservers() 
如果 hasChanged 方法指示对象已改变，则通知其所有观察者，并调用 clearChanged 方法来指示此对象不再改变。 
void notifyObservers(Object arg) 
如果 hasChanged 方法指示对象已改变，则通知其所有观察者，并调用 clearChanged 方法来指示此对象不再改变。 
protected  void setChanged() 
标记此 Observable 对象为已改变的对象；现在 hasChanged 方法将返回 true。
```

observer -> java.util.Observer(接口)

```java
void update(Observable o, Object arg) 
只要改变了observable对象（主题角色）就调用此方法。
```

需要特别说明下setChanged()、clearChanged()和hasChanged()这3个方法：

参见上面Observable类的notifyObservers(Object arg)方法，hasChanged()为true才会通知观察者数据有变化，并且在通知完成之后调用clearChanged()修改hasChanged()为false，所以当主题数据改变时，需要先调用setChanged()方法使hasChanged为true

Observable的伪码应该如下所示：

```java
public class Observable {
    private boolean flag = false;
    private List<Observer> list = new ArrayList<Observer>();
    
    public boolean hasChanged(){
        return flag;
    }
    
    protected void setChanged(){
        flag = true;
    }
 
    protected void clearChanged(){
        flag = false;
    }
    
    public void addObserver(Observer o){
        if(!list.contain(o)){
            list.add(o);
        }
    }
    
    public void deleteObserver(Observer o){
        if(list.contain(o)){
            list.remove(o);
        }
    }
    
    public void notifyObservers(Object arg){
        if(hasChanged()){
            if(null != list && list.size > 0){
                for(Observer o : list){
                    o.update(this, arg);
                }
            }
        }
        clearChanged();
    }
}
```

```java
public class SpecialRepoter extends Observable {
    public void getNewNews(String msg){
        if(msg.length()>100){
            this.setChanged();
        }
        this.notifyObservers(msg);
    }
}
```

通过这段伪代码可以很清楚的了解这3个方法了，这3个方法使我们对何时进行push进行精确控制，在我们不想推送的时候，不调用setChanged()方法即可。

使用jdk自带的观察者模式重写一下以前的记者和报社例子：

想要进行通知，则必须调用Observable类的setChanged方法，但是Observable的setChanged方法为protected，故只能使用继承来实现自己的主题对象。

主题继承自Observable类，观察者实现Observer接口，并且主题需要的方法已经在Observable类中实现了

观察者：

```java
package com.dxz.observer3;

import java.util.Observable;
import java.util.Observer;

//新华社
public class XinhuaNewspaperObserver implements Observer {

    private String newspaperName = "新华社"; 
    
    @Override
    public void update(Observable o, Object arg) {
        System.out.println("'" + newspaperName + "'读取推送信息:" + arg);
        System.out.println("'" + newspaperName + "'读取拉取信息:" + ((RepoterObservable)o).getNews());
    }

}
package com.dxz.observer3;

import java.util.Observable;
import java.util.Observer;

//人民日报
public class PeopleDailyNewspaperObserver implements Observer {

    private String newspaperName = "人民日报"; 
    
    @Override
    public void update(Observable o, Object arg) {
        System.out.println("'" + newspaperName + "'读取推送信息:" + arg);
        System.out.println("'" + newspaperName + "'读取拉取信息:" + ((RepoterObservable)o).getNews());
    }
}
```

```java
package com.dxz.observer3;

import java.util.Observable;

public class RepoterObservable extends Observable {
    
    private String news;

    public String getNews() {
        return news;
    }
    public void setNews(String news) {
        this.news = news;
        setChanged();  // 必须调用这个方法来通知Observer状态发生了改变
        notifyObservers("you");
    }
}
```

```java
package com.dxz.observer3;
public class Client {
    public static void main(String[] args) {
        RepoterObservable subject = new RepoterObservable();
        PeopleDailyNewspaperObserver observer1 = new PeopleDailyNewspaperObserver();
        XinhuaNewspaperObserver observer2 = new XinhuaNewspaperObserver();
        
        subject.addObserver(observer1);
        subject.addObserver(observer2);

        subject.setNews("日韩贸易战最新消息,韩方一再要求撤回贸易管制 日方无松动迹象...");
    }
}
```

结果：

```
'新华社'读取推送信息:you
'新华社'读取拉取信息:日韩贸易战最新消息,韩方一再要求撤回贸易管制 日方无松动迹象...
'人民日报'读取推送信息:you
'人民日报'读取拉取信息:日韩贸易战最新消息,韩方一再要求撤回贸易管制 日方无松动迹象...
```

 **PUSH和PULL模式**

- notifyObserver(Object arg)：带参数的notifyObservers(Object arg):这个参数Object arg 其实就是 Observer接口中的update(Observable o, Object arg)方法中的第二个参数 。就是通知观察者所改变的数据对象。简单理解就是由主题主动的PUSH需要改变的数据对象给观察者。
- notifyObserver()：不带参数的方法，传递一个null数据对象给观察者，需要观察者主动到主题pull数据。

## JDK自带的监听器模式

**角色之：事件对象及事件对象里的事件源**

再来看看事件，即Event或EventObject结尾的那个类，里面含有getSource方法，返回的就是事件源，

```java
package java.util;

/**
 * <p>
 * The root class from which all event state objects shall be derived.
 * <p>
 * All Events are constructed with a reference to the object, the "source",
 * that is logically deemed to be the object upon which the Event in question
 * initially occurred upon.
 *
 * @since JDK1.1
 */

public class EventObject implements java.io.Serializable {

    private static final long serialVersionUID = 5516075349620653480L;

    /**
     * The object on which the Event initially occurred.
     */
    protected transient Object  source;

    /**
     * Constructs a prototypical Event.
     *
     * @param    source    The object on which the Event initially occurred.
     * @exception  IllegalArgumentException  if source is null.
     */
    public EventObject(Object source) {
        if (source == null)
            throw new IllegalArgumentException("null source");

        this.source = source;
    }

    /**
     * The object on which the Event initially occurred.
     *
     * @return   The object on which the Event initially occurred.
     */
    public Object getSource() {
        return source;
    }

    /**
     * Returns a String representation of this EventObject.
     *
     * @return  A a String representation of this EventObject.
     */
    public String toString() {
        return getClass().getName() + "[source=" + source + "]";
    }
}
```

**角色之：事件监听器**

```java
package java.util;

/**
 * A tagging interface that all event listener interfaces must extend.
 * @since JDK1.1
 */
public interface EventListener {
}
```

这个类也很简单，如果说观察者模式中的上层类和结果还带了不少逻辑不少方法的话，那么事件驱动模型中的上层类和接口简直看不到任何东西。没错，

> 事件驱动模型中，JDK的设计者们进行了最高级的抽象，就是让上层类只是代表了：我是一个事件（含有事件源），或，我是一个监听者！

示例：

```java
package com.dxz.listener2;

import java.util.HashSet;
import java.util.Iterator;
import java.util.Set;

/**
 * 事件源.
 */
public class EventSourceObject {
    private String name;
    // 监听器容器
    private Set<CusEventListener> listener;

    public EventSourceObject() {
        this.listener = new HashSet<CusEventListener>();
        this.name = "defaultname";
    }

    // 给事件源注册监听器
    public void addCusListener(CusEventListener cel) {
        this.listener.add(cel);
    }

    // 当事件发生时,通知注册在该事件源上的所有监听器做出相应的反应（调用回调方法）
    protected void notifies() {
        CusEventListener cel = null;
        Iterator<CusEventListener> iterator = this.listener.iterator();
        while (iterator.hasNext()) {
            cel = iterator.next();
            cel.fireCusEvent(new CusEvent(this));
        }
    }

    public String getName() {
        return name;
    }

    /**
     * 模拟事件触发器，当成员变量name的值发生变化时，触发事件。
     * @param name
     */
    public void setName(String name) {
        if (!this.name.equals(name)) {
            this.name = name;
            notifies();
        }
    }
}

package com.dxz.listener2;

import java.util.EventListener;

/**
 * 事件监听器，实现java.util.EventListener接口。定义回调方法，将你想要做的事放到这个方法下,因为事件源发生相应的事件时会调用这个方法。
 */
public class CusEventListener implements EventListener {
    // 事件发生后的回调方法
    public void fireCusEvent(CusEvent e) {
        EventSourceObject eObject = (EventSourceObject) e.getSource();
        System.out.println("My name has been changed!");
        System.out.println("I got a new name,named \"" + eObject.getName() + "\"");
    }
}

package com.dxz.listener2;

import java.util.EventObject;

/**
 * 事件类,用于封装事件源及一些与事件相关的参数.
 */
public class CusEvent extends EventObject {
    private static final long serialVersionUID = 1L;
    private Object source;// 事件源

    public CusEvent(Object source) {
        super(source);
        this.source = source;
    }

    public Object getSource() {
        return source;
    }

    public void setSource(Object source) {
        this.source = source;
    }
}

package com.dxz.listener2;

public class MainTest {

    public static void main(String[] args) {
        EventSourceObject object = new EventSourceObject();
        // 注册监听器
        object.addCusListener(new CusEventListener() {
            @Override
            public void fireCusEvent(CusEvent e) {
                super.fireCusEvent(e);
            }
        });
        // 触发事件
        object.setName("AiLu");
    }
}
```

结果：

```
My name has been changed!
I got a new name,named "AiLu"
```

1. 事件

   事件一般继承自java.util.EventObject类，封装了事件源对象及跟事件相关的信息。

2. 事件源
   事件源是事件发生的地方，由于事件源的某项属性或状态发生了改变(比如BUTTON被单击、TEXTBOX的值发生改变等等)导致某项事件发生。换句话说就是生成了相应的事件对象。因为事件监听器要注册在事件源上，所以事件源类中应该要有盛装监听器的容器(List、Set等等)。

3. 事件监听器
   事件监听器实现java.util.EventListener接口，注册在事件源上，当事件源的属性或状态改变时，取得相应的监听器调用其内部的回调方法。
   事件、事件源、监听器三者之间的联系
   事件源-----产生----->事件------>被事件监听器发现------>进入事件处理代码