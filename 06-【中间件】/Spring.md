# 基本概念

## SpringMVC和Struts2区别

1. Struts2是类级别的拦截， 一个类对应一个request上下文，SpringMVC是方法级别的拦截，一个方法对应一个request上下文，而方法同时又跟一个url对应,所以说从架构本身上SpringMVC就容易实现restful url,而struts2的架构实现起来要费劲，因为Struts2中Action的一个方法可以对应一个url，而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了。
2. 由上边原因，SpringMVC的方法之间基本上独立的，独享request response数据，请求数据通过参数获取，处理结果通过ModelMap交回给框架，方法之间不共享变量，而Struts2搞的就比较乱，虽然方法之间也是独立的，但其所有Action变量是共享的，这不会影响程序运行，却给我们编码 读程序时带来麻烦，每次来了请求就创建一个Action，一个Action对象对应一个request上下文。
3. 由于Struts2需要针对每个request进行封装，把request，session等servlet生命周期的变量封装成一个一个Map，供给每个Action使用，并保证线程安全，所以在原则上，是比较耗费内存的。
4. 拦截器实现机制上，Struts2有以自己的interceptor机制，SpringMVC用的是独立的AOP方式，这样导致Struts2的配置文件量还是比SpringMVC大。
5. SpringMVC的入口是servlet，而Struts2是filter（这里要指出，filter和servlet是不同的。以前认为filter是servlet的一种特殊），这就导致了二者的机制不同，这里就牵涉到servlet和filter的区别了。
6. SpringMVC集成了Ajax，使用非常方便，只需一个注解@ResponseBody就可以实现，然后直接返回响应文本即可，而Struts2拦截器集成了Ajax，在Action中处理时一般必须安装插件或者自己写代码集成进去，使用起来也相对不方便。
7. SpringMVC验证支持JSR303，处理起来相对更加灵活方便，而Struts2验证比较繁琐，感觉太烦乱。
8. Spring MVC和Spring是无缝的。从这个项目的管理和安全上也比Struts2高（当然Struts2也可以通过不同的目录结构和相关配置做到SpringMVC一样的效果，但是需要xml配置的地方不少）。
9. 设计思想上，Struts2更加符合OOP的编程思想， SpringMVC就比较谨慎，在servlet上扩展。
10. SpringMVC开发效率和性能高于Struts2。
11. SpringMVC可以认为已经100%零配置。

## 过滤器和拦截器

- **两者的作用**

   过滤器：

  是在java web中，你传入的request、response提前过滤掉一些信息，或者提前设置一些参数，然后再传入servlet或者struts的action进行业务逻辑，比如过滤掉非法url（不是login.do的地址请求，如果用户没有登陆都过滤掉），或者在传入servlet或者 struts的action前统一设置字符集，或者去除掉一些非法字符.。

   拦截器 ：

  是在面向切面编程的就是在你的service或者一个方法，前调用一个方法，或者在方法后调用一个方法比如动态代理就是拦截器的简单实现，在你调用方法前打印出字符串（或者做其它业务逻辑的操作），也可以在你调用方法后打印出字符串，甚至在你抛出异常的时候做业务逻辑的操作。

- 从具体实现区分

  过滤器是 servlet 的

  拦截器是 spring aop 的

- 细节区别

   ①拦截器是基于Java的反射机制的，而过滤器是基于函数回调。

   ②拦截器不依赖于servlet容器，过滤器依赖于servlet容器。

   ③拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。

   ④拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。

   ⑤在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。

   ⑥拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑。

- 工作流程及顺序

![image-20200416114943984](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200416114944-519403.png) 

## BeanPostProcessor接口

**ApplicationContextAwareProcessor**

`ApplicationContextAware` 通过它Spring容器会自动把上下文环境对象调用`ApplicationContextAware`接口中的`setApplicationContext`方法。

我们在`ApplicationContextAware`的实现类中，就可以通过这个上下文环境对象得到Spring容器中的Bean。

![image-20200513112837138](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200513112840-695426.png)  

**BeanValidationPostProcessor**

BeanValidationPostProcessor Bean内部有个boolean类型的属性afterInitialization，默认是false，如果是false，在postProcessBeforeInitialization过程中对bean进行验证，否则在postProcessAfterInitialization过程对bean进行验证

![image-20200513112807158](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200513112809-918188.png) 

**InitDestroyAnnotationBeanPostProcessor**

首先 InitDestroyAnnotationBeanPostProcessor 也是 BeanPostProcessors 的一个实现类，其对 BeanPostProcessor 的实现是通过同时实现 DestructionAwareBeanPostProcessor 和 MergedBeanDefinitionPostProcessor 这两个接口来完成的，这以为该类能够对 Bean 的销毁即其 MergedBeanDefinition 进行操作干预，同时通过上面的继承关系图我们可以看到它也实现了 Ordered 和 Serializable 接口，这表示该类具备所实现的功能可实现排序并且该类支持序列化。

该类的主要作用是调用带注释的 init 和 destroy 方法，其允许使用注解替代 Spring 的 InitializingBean 和 DisposableBean 回调接口（这点在前面的博文中介绍过，Spring 支持三种形式的初始化和销毁方法回调），并且可以通过 setInitAnnotationType 和 setDestroyAnnotationType 方法来配置此 BeanPostProcessor 检查的实际注解类型（因为这里的注解不需要属性参数，所以可以自定义任何形式注解）。同时，初始化和销毁注解可以应用于任何可见性的方法（public，package-protected，protected 或者 private 均可），并且支持同时注解多个此类方法，但是根据 Spring 的建议应尽量仅注解一个的 init 方法和一个 destroy 方法。

如果到这里你还是感觉云里雾里，那么举一个它最简单确实最常使用的应用即 @PostConstruct 和 @PreDestroy 注解，这两个注解分别是在 Bean 实例创建完成和销毁之前被调用的，也是开发中最简单的一种注解调用方式，而它的实现就是与 InitDestroyAnnotationBeanPostProcessor 和它的子类 CommonAnnotationBeanPostProcessor 密切相关（子类 CommonAnnotationBeanPostProcessor 中还提供了对于 @Resource 注解的支持）。但是它虽然是我们最常使用的注解，但可能我们却一直不知道它在 Spring 的内部是通过怎样得调用来发挥作用的，因此在下面的内容中我们将主要分析这两个注解在 Spring 当中的调用流程，以及它们与其它生命周期回调的关系。

![image-20200513112909305](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200513112910-176264.png) 

**AutowiredAnnotationBeanPostProcessor**

AutowiredAnnotationBeanPostProcessor实现了BeanPostProcessor接口，当 Spring 容器启动时，AutowiredAnnotationBeanPostProcessor 将扫描 Spring 容器中所有 Bean，当发现 Bean 中拥有@Autowired 注解时就找到和其匹配（默认按类型匹配）的 Bean，并注入到对应的地方中去。先来看下buildAutowiringMetadata方法

![image-20200513113130099](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200513113133-628420.png) 

# 自动装配

从Spring2.5开始，开始支持使用注解来自动装配Bean的属性。它允许更细粒度的自动装配，我们可以选择性的标注某一个属性来对其应用自动装配。

Spring支持几种不同的应用于自动装配的注解。

- Spring自带的@Autowired注解。
- JSR-330的@Inject注解。
- JSR-250的@Resource注解。

这里只重点关注Autowired注解，关于它的解析和注入过程。

使用@Autowired很简单，在需要注入的属性加入注解即可。

```java
@Autowired
UserService userService;
```

不过，使用它有几个点需要注意。

## 强制性

默认情况下，它具有强制契约特性，其所标注的属性必须是可装配的。如果没有Bean可以装配到Autowired所标注的属性或参数中，那么你会看到`NoSuchBeanDefinitionException`的异常信息。

```java
public Object doResolveDependency(DependencyDescriptor descriptor, String beanName,
            Set<String> autowiredBeanNames, TypeConverter typeConverter) throws BeansException {
    
    //查找Bean
    Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
    //如果拿到的Bean集合为空，且isRequired，就抛出异常。
    if (matchingBeans.isEmpty()) {
        if (descriptor.isRequired()) {
            raiseNoSuchBeanDefinitionException(type, "", descriptor);
        }
        return null;
    }
}
```

看到上面的源码，我们可以得到这一信息，Bean集合为空不要紧，关键`isRequired`条件不能成立，那么，如果我们不确定属性是否可以装配，可以这样来使用Autowired。

```java
@Autowired(required=false)
UserService userService;
```

## 装配策略

我记得曾经有个面试题是这样问的：Autowired是按照什么策略来自动装配的呢？

关于这个问题，不能一概而论，你不能简单的说按照类型或者按照名称。但可以确定的一点的是，它默认是按照类型来自动装配的，即byType。

- 默认按照类型装配

关键点`findAutowireCandidates`这个方法。

```java
protected Map<String, Object> findAutowireCandidates(
        String beanName, Class<?> requiredType, DependencyDescriptor descriptor) {
    
    //获取给定类型的所有bean名称，里面实际循环所有的beanName，获取它的实例
    //再通过isTypeMatch方法来确定
    String[] candidateNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
            this, requiredType, true, descriptor.isEager());
            
    Map<String, Object> result = new LinkedHashMap<String, Object>(candidateNames.length);
    
    //根据返回的beanName，获取其实例返回
    for (String candidateName : candidateNames) {
        if (!isSelfReference(beanName, candidateName) && isAutowireCandidate(candidateName, descriptor)) {
            result.put(candidateName, getBean(candidateName));
        }
    }
    return result;
}
```

- 按照名称装配

可以看到它返回的是一个列表，那么就表明，按照类型匹配可能会查询到多个实例。到底应该装配哪个实例呢？我看有的文章里说，可以加注解以此规避。比如`@qulifier、@Primary`等，实际还有个简单的办法。

比如，按照UserService接口类型来装配它的实现类。UserService接口有多个实现类，分为`UserServiceImpl、UserServiceImpl2`。那么我们在注入的时候，就可以把属性名称定义为Bean实现类的名称。

```java
@Autowired
UserService UserServiceImpl2;
```

这样的话，Spring会按照byName来进行装配。首先，如果查到类型的多个实例，Spring已经做了判断。

```java
public Object doResolveDependency(DependencyDescriptor descriptor, String beanName,
            Set<String> autowiredBeanNames, TypeConverter typeConverter) throws BeansException {
            
    //按照类型查找Bean实例
    Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
    //如果Bean集合为空，且isRequired成立就抛出异常
    if (matchingBeans.isEmpty()) {
        if (descriptor.isRequired()) {
            raiseNoSuchBeanDefinitionException(type, "", descriptor);
        }
        return null;
    }
    //如果查找的Bean实例大于1个
    if (matchingBeans.size() > 1) {
        //找到最合适的那个，如果没有合适的。。也抛出异常
        String primaryBeanName = determineAutowireCandidate(matchingBeans, descriptor);
        if (primaryBeanName == null) {
            throw new NoUniqueBeanDefinitionException(type, matchingBeans.keySet());
        }
        if (autowiredBeanNames != null) {
            autowiredBeanNames.add(primaryBeanName);
        }
        return matchingBeans.get(primaryBeanName);
    }   
}
```

可以看出，如果查到多个实例，`determineAutowireCandidate`方法就是关键。它来确定一个合适的Bean返回。其中一部分就是按照Bean的名称来匹配。

```java
protected String determineAutowireCandidate(Map<String, Object> candidateBeans, 
                DependencyDescriptor descriptor) {
    //循环拿到的Bean集合
    for (Map.Entry<String, Object> entry : candidateBeans.entrySet()) {
        String candidateBeanName = entry.getKey();
        Object beanInstance = entry.getValue();
        //通过matchesBeanName方法来确定bean集合中的名称是否与属性的名称相同
        if (matchesBeanName(candidateBeanName, descriptor.getDependencyName())) {
            return candidateBeanName;
        }
    }
    return null;
}
```

最后我们回到问题上，得到的答案就是：@Autowired默认使用byType来装配属性，如果匹配到类型的多个实例，再通过byName来确定Bean。

## 主和优先级

上面我们已经看到了，通过byType可能会找到多个实例的Bean。然后再通过byName来确定一个合适的Bean，如果通过名称也确定不了呢？
 还是`determineAutowireCandidate`这个方法，它还有两种方式来确定。

```dart
protected String determineAutowireCandidate(Map<String, Object> candidateBeans, 
                DependencyDescriptor descriptor) {
    Class<?> requiredType = descriptor.getDependencyType();
    //通过@Primary注解来标识Bean
    String primaryCandidate = determinePrimaryCandidate(candidateBeans, requiredType);
    if (primaryCandidate != null) {
        return primaryCandidate;
    }
    //通过@Priority(value = 0)注解来标识Bean value为优先级大小
    String priorityCandidate = determineHighestPriorityCandidate(candidateBeans, requiredType);
    if (priorityCandidate != null) {
        return priorityCandidate;
    }
    return null;
}
```

- Primary

它的作用是看Bean上是否包含@Primary注解，如果包含就返回。当然了，你不能把多个Bean都设置为@Primary，不然你会得到`NoUniqueBeanDefinitionException`这个异常。

```tsx
protected String determinePrimaryCandidate(Map<String, Object> candidateBeans, Class<?> requiredType) {
    String primaryBeanName = null;
    for (Map.Entry<String, Object> entry : candidateBeans.entrySet()) {
        String candidateBeanName = entry.getKey();
        Object beanInstance = entry.getValue();
        if (isPrimary(candidateBeanName, beanInstance)) {
            if (primaryBeanName != null) {
                boolean candidateLocal = containsBeanDefinition(candidateBeanName);
                boolean primaryLocal = containsBeanDefinition(primaryBeanName);
                if (candidateLocal && primaryLocal) {
                    throw new NoUniqueBeanDefinitionException(requiredType, candidateBeans.size(),
                            "more than one 'primary' bean found among candidates: " + candidateBeans.keySet());
                }
                else if (candidateLocal) {
                    primaryBeanName = candidateBeanName;
                }
            }
            else {
                primaryBeanName = candidateBeanName;
            }
        }
    }
    return primaryBeanName;
}
```

- Priority

你也可以在Bean上配置@Priority注解，它有个int类型的属性value，可以配置优先级大小。数字越小的，就被优先匹配。同样的，你也不能把多个Bean的优先级配置成相同大小的数值，否则`NoUniqueBeanDefinitionException`异常照样出来找你。

```dart
protected String determineHighestPriorityCandidate(Map<String, Object> candidateBeans, 
                                    Class<?> requiredType) {
    String highestPriorityBeanName = null;
    Integer highestPriority = null;
    for (Map.Entry<String, Object> entry : candidateBeans.entrySet()) {
        String candidateBeanName = entry.getKey();
        Object beanInstance = entry.getValue();
        Integer candidatePriority = getPriority(beanInstance);
        if (candidatePriority != null) {
            if (highestPriorityBeanName != null) {
                //如果优先级大小相同
                if (candidatePriority.equals(highestPriority)) {
                    throw new NoUniqueBeanDefinitionException(requiredType, candidateBeans.size(),
                        "Multiple beans found with the same priority ('" + highestPriority + "') " +
                            "among candidates: " + candidateBeans.keySet());
                }
                else if (candidatePriority < highestPriority) {
                    highestPriorityBeanName = candidateBeanName;
                    highestPriority = candidatePriority;
                }
            }
            else {
                highestPriorityBeanName = candidateBeanName;
                highestPriority = candidatePriority;
            }
        }
    }
    return highestPriorityBeanName;
}
```

最后，有一点需要注意。Priority的包在`javax.annotation.Priority;`，如果想使用它还要引入一个坐标。

```xml
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.2</version>
</dependency>
```

## @Qualifier

通过以上的探讨可知@Primary只是定义了Bean在自动装配时的一个优先级，如果优先级别相同则还是会产生歧义，这时就需要使用Spring提供的限定符策略，它可以对需要注入的Bean进行逐层筛选，最终选出唯一一个能满足限定条件的Bean。
@Qualifier默认使用
使用Spring声明的组件Bean默认都会有一个限定符，这个限定符的标识就是BeanID，也就是该Bean的类名称首字母小写之后的值。

```java
@Autowired
@Qualifier("fashionMusic")
public void setMusic(Music music) {
	this.music = music;
}
```

这里我们需要通过setter方法注入一个Music的实例，并通过@Qualifier(“fashionMusic”)限定只能注入限定符为“fashionMusic”的Bean组件。
这种使用限定符的方式将限定符标识与Bean的类名称紧密耦合，一旦我们的类名称发生变化，则必须相应修改依赖方的限定符标识。当然我们也可以为被依赖方这个Bean自定义一个修饰符，然后在依赖方使用自定义的修饰符进行限定，这样就完全将类名称与限定符标识完全解耦了。
@Qualifier自定义限定符使用

```java
@Component
@Primary
@Qualifier("fashion")
public class FashionMusic implements Music {
    @Override
    public String getMusicType() {
        return "流行音乐";
    }
}

@Autowired
@Qualifier("fashion")
public void setMusic(Music music) {
	this.music = music;
}
```

通过这种自定义限定符的方式来解决自动装配时产生的歧义安全性、扩展性更高，但是有个问题，假如RapMusic也自定义了和FashionMusic相同的限定符标识，这样还是会产生新的歧义，那么如何解决呢？

通常我们想到的肯定是继续使用@ Qualifier定义新的限定符标识，将范围限定至最小，但是Java不容许在同一个条目上出现多个相同类型的注解，此时我们就需要自定义限定符注解，来解决这种歧义问题
自定义限定符注解的使用

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Say {
}

@Component
@Primary
@Qualifier("fashion")
@Say
public class RapMusic implements Music {
    @Override
    public String getMusicType() {
        return "说唱音乐";
    }
}

@Autowired
@Qualifier("fashion")
@Say
public void setMusic(Music music) {
	this.music = music;
}
```

可见通过自定义限定符注解就可以解决由自定义限定符标识相同而导致的歧义性问题，至此有关Bean在自动装配过程中产生的歧义性问题已探讨完毕。

## @Resource|@Inject

@Resurce是基于JSR 250 规范的注解，@Inject是基于JSR 330规范的注解。
@Resource和@Autowired的主要区别是：@AutoWried按by type自动注入，而@Resource默认按byName自动注入。

@Resource有两个重要属性，分别是name和type：
两个属性都不指定时，@Resource默认按照属性名查找，如果查找不到就按照类型查找，如果找不大或找到多个会报异常。
只指定name属性，则按照指定的name名查找，如果找不大或找到多个会报异常。
只指定type属性，则按照属性信息查找，如果找不大或找到多个会报异常。
同时指定name属性和type属性，则按照属性信息和name名查找，如果找不大或找到多个会报异常。

@Inject注解和@Autowired注解的使用方式相同，只是@Inject没有required属性，也不能和@Qualifier和@Primary注解组合使用。

# AOP动态代理

## 动态代理的两种方式

一般而言，动态代理分为两种，一种是JDK反射机制提供的代理，另一种是CGLIB代理。在JDK代理，必须提供接口，而CGLIB则不需要提供接口。

- JDK原生动态代理是Java原生支持的，不需要任何外部依赖,但是它只能基于接口进行代理;
- CGLIB通过继承的方式进行代理，无论目标对象有没有实现接口都可以代理,但是无法处理final的情况。

**spring两种代理方式**

1. 若目标对象实现了若干接口，spring使用JDK的java.lang.reflect.Proxy类代理。 
优点：因为有接口，所以使系统更加松耦合 
缺点：为每一个目标类创建接口

2. 若目标对象没有实现任何接口，spring使用CGLIB库生成目标对象的子类。 
优点：因为代理类与目标类是继承关系，所以不需要有接口的存在。 
缺点：因为没有使用接口，所以系统的耦合性没有使用JDK的动态代理好

**1.JDK动态代理：**

```java
public interface Rent {
    public void rent();
}
```

```java
public class Landlord implements Rent{

    @Override
    public void rent() {
        System.out.println("房东要出租房子了！");
    }
}
```

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class Intermediary implements InvocationHandler{
    
    private Object post;
    
    Intermediary(Object post){
        this.post = post;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        Object invoke = method.invoke(post, args);
        System.out.println("中介：该房源已发布！");
        return invoke;
    }
}
```

```java
import java.lang.reflect.Proxy;

public class Test {
    public static void main(String[] args) {
        Rent rent = new Landlord();
        Intermediary intermediary = new Intermediary(rent);
        Rent rentProxy = (Rent) Proxy.newProxyInstance(rent.getClass().getClassLoader(), rent.getClass().getInterfaces(), intermediary);
        rentProxy.rent();
    }
}
```

**2.CGLIB动态代理：**

```java
public class Landlord {
    public void rent(){
        System.out.println("房东要出租房子了！");
    }
}
```

```java
import java.lang.reflect.Method;

import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class Intermediary implements MethodInterceptor {

    @Override
    public Object intercept(Object object, Method method, Object[] args,MethodProxy methodProxy) throws Throwable {
        Object intercept = methodProxy.invokeSuper(object, args);
        System.out.println("中介：该房源已发布！");
        return intercept;
    }
}
```

```java
import net.sf.cglib.proxy.Enhancer;

public class Test {
    public static void main(String[] args) {
        Intermediary intermediary = new Intermediary();
        
        Enhancer enhancer = new Enhancer();  
        enhancer.setSuperclass(Landlord.class);
        enhancer.setCallback(intermediary);
        
        Landlord rentProxy = (Landlord) enhancer.create();
        rentProxy.rent();
    }
}
```

## AOP编程

AOP【面向切面编程】：指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式；

AOP编程步骤：

1. 导入aop模块；Spring AOP：(spring-aspects)
2. 定义一个业务逻辑类（MathCalculator）；在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、方法出现异常，xxx）
3. 定义一个日志切面类（LogAspects）：切面类里面的方法需要动态感知MathCalculator.div运行到哪里然后执行；
   通知方法：
   前置通知(@Before)：logStart：在目标方法(div)运行之前运行
   后置通知(@After)：logEnd：在目标方法(div)运行结束之后运行（无论方法正常结束还是异常结束）
   返回通知(@AfterReturning)：logReturn：在目标方法(div)正常返回之后运行
   异常通知(@AfterThrowing)：logException：在目标方法(div)出现异常以后运行
   环绕通知(@Around)：动态代理，手动推进目标方法运行（joinPoint.procced()）
4. 给切面类的目标方法标注何时何地运行（通知注解）；
5. 将切面类和业务逻辑类（目标方法所在类）都加入到容器中;
6. 必须告诉Spring哪个类是切面类(给切面类上加一个注解：@Aspect)
7. 给配置类中加 @EnableAspectJAutoProxy 【开启基于注解的aop模式】，在Spring中很多的 @EnableXXX;

  总结三步：
  1）、将业务逻辑组件和切面类都加入到容器中；告诉Spring哪个是切面类（@Aspect）
  2）、在切面类上的每一个通知方法上标注通知注解，告诉Spring何时何地运行（切入点表达式）
  3）、开启基于注解的aop模式；@EnableAspectJAutoProxy

业务逻辑类：

```java
public class MathCalculator {
	
	public int div(int i,int j){
		System.out.println("MathCalculator...div...");
		return i/j;	
	}
}
```

切面类：

```java
@Aspect
public class LogAspects {
	
	//抽取公共的切入点表达式
	//1、本类引用
	//2、其他的切面引用
	@Pointcut("execution(public int com.atguigu.aop.MathCalculator.*(..))")
	public void pointCut(){};
	
	//@Before在目标方法之前切入；切入点表达式（指定在哪个方法切入）
	@Before("pointCut()")
	public void logStart(JoinPoint joinPoint){
		Object[] args = joinPoint.getArgs();
		System.out.println(""+joinPoint.getSignature().getName()+"运行。。。@Before:参数列表是：{"+Arrays.asList(args)+"}");
	}
	
	@After("com.atguigu.aop.LogAspects.pointCut()")
	public void logEnd(JoinPoint joinPoint){
		System.out.println(""+joinPoint.getSignature().getName()+"结束。。。@After");
	}
	
	//JoinPoint一定要出现在参数表的第一位
	@AfterReturning(value="pointCut()",returning="result")
	public void logReturn(JoinPoint joinPoint,Object result){
		System.out.println(""+joinPoint.getSignature().getName()+"正常返回。。。@AfterReturning:运行结果：{"+result+"}");
	}
	
	@AfterThrowing(value="pointCut()",throwing="exception")
	public void logException(JoinPoint joinPoint,Exception exception){
		System.out.println(""+joinPoint.getSignature().getName()+"异常。。。异常信息：{"+exception+"}");
	}
}
```

配置类：

```java
@EnableAspectJAutoProxy
@Configuration
public class MainConfigOfAOP {
	 
	//业务逻辑类加入容器中
	@Bean
	public MathCalculator calculator(){
		return new MathCalculator();
	}

	//切面类加入到容器中
	@Bean
	public LogAspects logAspects(){
		return new LogAspects();
	}
}
```

## AOP原理

**原理总结：**

1. `@EnableAspectJAutoProxy` 开启AOP功能
2. `@EnableAspectJAutoProxy` 会给容器中注册一个组件 `AnnotationAwareAspectJAutoProxyCreator`
3. `AnnotationAwareAspectJAutoProxyCreator`是一个后置处理器；
4. 容器的创建流程：
   - `registerBeanPostProcessors（）`注册后置处理器；创建`AnnotationAwareAspectJAutoProxyCreator`对象
   - `finishBeanFactoryInitialization（）`初始化剩下的单实例`bean`
     - 创建业务逻辑组件和切面组件
     - `AnnotationAwareAspectJAutoProxyCreator`拦截组件的创建过程
     - 组件创建完之后，判断组件是否需要增强，如果是：切面的通知方法，包装成增强器（`Advisor`）;给业务逻辑组件创建一个代理对象（`cglib`）；
5. 执行目标方法：
   - 代理对象执行目标方法
   - `CglibAopProxy.intercept()`；
     - 得到目标方法的拦截器链（增强器包装成拦截器`MethodInterceptor`）
     - 利用拦截器的链式机制，依次进入每一个拦截器进行执行；
     - 效果：
       正常执行：前置通知-》目标方法-》后置通知-》返回通知
       出现异常：前置通知-》目标方法-》后置通知-》异常通知

**详细源码分析：**

```
* AOP原理：【看给容器中注册了什么组件，这个组件什么时候工作，这个组件的功能是什么？】
*     @EnableAspectJAutoProxy；
* 1、@EnableAspectJAutoProxy是什么？
*     @Import(AspectJAutoProxyRegistrar.class)：给容器中导入AspectJAutoProxyRegistrar
*        利用AspectJAutoProxyRegistrar自定义给容器中注册bean；BeanDefinetion
*        internalAutoProxyCreator=AnnotationAwareAspectJAutoProxyCreator
* 
*     给容器中注册一个AnnotationAwareAspectJAutoProxyCreator；
* 
* 2、 AnnotationAwareAspectJAutoProxyCreator：
*     AnnotationAwareAspectJAutoProxyCreator
*        ->AspectJAwareAdvisorAutoProxyCreator
*           ->AbstractAdvisorAutoProxyCreator
*              ->AbstractAutoProxyCreator
*                    implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware
*                 关注后置处理器（在bean初始化完成前后做事情）、自动装配BeanFactory
* 
* AbstractAutoProxyCreator.setBeanFactory()
* AbstractAutoProxyCreator.有后置处理器的逻辑；
* 
* AbstractAdvisorAutoProxyCreator.setBeanFactory()-》initBeanFactory()
* 
* AnnotationAwareAspectJAutoProxyCreator.initBeanFactory()
*
*
* 流程：
*     1）、传入配置类，创建ioc容器
*     2）、注册配置类，调用refresh（）刷新容器；
*     3）、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；
*        1）、先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor
*        2）、给容器中加别的BeanPostProcessor
*        3）、优先注册实现了PriorityOrdered接口的BeanPostProcessor；
*        4）、再给容器中注册实现了Ordered接口的BeanPostProcessor；
*        5）、注册没实现优先级接口的BeanPostProcessor；
*        6）、注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；
*           创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】
*           1）、创建Bean的实例
*           2）、populateBean；给bean的各种属性赋值
*           3）、initializeBean：初始化bean；
*                 1）、invokeAwareMethods()：处理Aware接口的方法回调
*                 2）、applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization（）
*                 3）、invokeInitMethods()；执行自定义的初始化方法
*                 4）、applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization（）；
*           4）、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》aspectJAdvisorsBuilder
*        7）、把BeanPostProcessor注册到BeanFactory中；
*           beanFactory.addBeanPostProcessor(postProcessor);
* =======以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程========
* 
*        AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor
*     4）、finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单实例bean
*        1）、遍历获取容器中所有的Bean，依次创建对象getBean(beanName);
*           getBean->doGetBean()->getSingleton()->
*        2）、创建bean
*           【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截，InstantiationAwareBeanPostProcessor，会调用postProcessBeforeInstantiation()】
*           1）、先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建；
*              只要创建好的Bean都会被缓存起来
*           2）、createBean（）;创建bean；
*              AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例
*              【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】
*              【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回对象的】
*              1）、resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation
*                 希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续
*                 1）、后置处理器先尝试返回对象；
*                    bean = applyBeanPostProcessorsBeforeInstantiation（）：
*                       拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;
*                       就执行postProcessBeforeInstantiation
*                    if (bean != null) {
                     bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                  }
* 
*              2）、doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；和3.6流程一样；
*              3）、
*        
*     
* AnnotationAwareAspectJAutoProxyCreator【InstantiationAwareBeanPostProcessor】  的作用：
* 1）、每一个bean创建之前，调用postProcessBeforeInstantiation()；
*     关心MathCalculator和LogAspect的创建
*     1）、判断当前bean是否在advisedBeans中（保存了所有需要增强bean）
*     2）、判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean，
*        或者是否是切面（@Aspect）
*     3）、是否需要跳过
*        1）、获取候选的增强器（切面里面的通知方法）【List<Advisor> candidateAdvisors】
*           每一个封装的通知方法的增强器是 InstantiationModelAwarePointcutAdvisor；
*           判断每一个增强器是否是 AspectJPointcutAdvisor 类型的；返回true
*        2）、永远返回false
* 
* 2）、创建对象
* postProcessAfterInitialization；
*     return wrapIfNecessary(bean, beanName, cacheKey);//包装如果需要的情况下
*     1）、获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors
*        1、找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）
*        2、获取到能在bean使用的增强器。
*        3、给增强器排序
*     2）、保存当前bean在advisedBeans中；
*     3）、如果当前bean需要增强，创建当前bean的代理对象；
*        1）、获取所有增强器（通知方法）
*        2）、保存到proxyFactory
*        3）、创建代理对象：Spring自动决定
*           JdkDynamicAopProxy(config);jdk动态代理；
*           ObjenesisCglibAopProxy(config);cglib的动态代理；
*     4）、给容器中返回当前组件使用cglib增强了的代理对象；
*     5）、以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法的流程；
*     
*  
*  3）、目标方法执行  ；
*     容器中保存了组件的代理对象（cglib增强后的对象），这个对象里面保存了详细信息（比如增强器，目标对象，xxx）；
*     1）、CglibAopProxy.intercept();拦截目标方法的执行
*     2）、根据ProxyFactory对象获取将要执行的目标方法拦截器链；
*        List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
*        1）、List<Object> interceptorList保存所有拦截器 5
*           一个默认的ExposeInvocationInterceptor 和 4个增强器；
*        2）、遍历所有的增强器，将其转为Interceptor；
*           registry.getInterceptors(advisor);
*        3）、将增强器转为List<MethodInterceptor>；
*           如果是MethodInterceptor，直接加入到集合中
*           如果不是，使用AdvisorAdapter将增强器转为MethodInterceptor；
*           转换完成返回MethodInterceptor数组；
* 
*     3）、如果没有拦截器链，直接执行目标方法;
*        拦截器链（每一个通知方法又被包装为方法拦截器，利用MethodInterceptor机制）
*     4）、如果有拦截器链，把需要执行的目标对象，目标方法，
*        拦截器链等信息传入创建一个 CglibMethodInvocation 对象，
*        并调用 Object retVal =  mi.proceed();
*     5）、拦截器链的触发过程;
*        1)、如果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（指定到了最后一个拦截器）执行目标方法；
*        2)、链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行；
*           拦截器链的机制，保证通知方法与目标方法的执行顺序；
*     
```

![image-20200514171638363](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200514171641-717937.png)  

# 声明式事务

- 原子性（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
- 一致性（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
- 隔离性（Isolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
- 持久性（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。

## 传播行为

| 传播行为                  | 含义                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务 |
| PROPAGATION_SUPPORTS      | 表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行 |
| PROPAGATION_MANDATORY     | 表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常 |
| PROPAGATION_REQUIRED_NEW  | 表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NOT_SUPPORTED | 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NEVER         | 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常 |
| PROPAGATION_NESTED        | 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。注意各厂商对这种传播行为的支持是有所差异的。可以参考资源管理器的文档来确认它们是否支持嵌套事务 |

## 使用事务

- 导入相关依赖
  数据源、数据库驱动、Spring-jdbc模块

- 配置数据源、JdbcTemplate（Spring提供的简化数据库操作的工具）操作数据

- 给方法上标注 `@Transactional` 表示当前方法是一个事务方法

- `@EnableTransactionManagement` 开启基于注解的事务管理功能

- 配置事务管理器来控制事务;

  ```java
  @Bean
  public PlatformTransactionManager transactionManager()
  ```

## 事务原理

详细源码分析：

```
 * 原理：
 * 1）、@EnableTransactionManagement
 * 			利用TransactionManagementConfigurationSelector给容器中会导入组件
 * 			导入两个组件
 * 			AutoProxyRegistrar
 * 			ProxyTransactionManagementConfiguration
 * 2）、AutoProxyRegistrar：
 * 			给容器中注册一个 InfrastructureAdvisorAutoProxyCreator 组件；
 * 			InfrastructureAdvisorAutoProxyCreator：？
 * 			利用后置处理器机制在对象创建以后，包装对象，返回一个代理对象（增强器），代理对象执行方法利用拦截器链进行调用；
 * 
 * 3）、ProxyTransactionManagementConfiguration 做了什么？
 * 			1、给容器中注册事务增强器；
 * 				1）、事务增强器要用事务注解的信息，AnnotationTransactionAttributeSource解析事务注解
 * 				2）、事务拦截器：
 * 					TransactionInterceptor；保存了事务属性信息，事务管理器；
 * 					他是一个 MethodInterceptor；
 * 					在目标方法执行的时候；
 * 						执行拦截器链；
 * 						事务拦截器：
 * 							1）、先获取事务相关的属性
 * 							2）、再获取PlatformTransactionManager，如果事先没有添加指定任何transactionmanger
 * 								最终会从容器中按照类型获取一个PlatformTransactionManager；
 * 							3）、执行目标方法
 * 								如果异常，获取到事务管理器，利用事务管理回滚操作；
 * 								如果正常，利用事务管理器，提交事务
```

## 事务异常的处理

- unchecked  运行期Exception  spring默认会进行事务回滚    比如：RuntimeException
- checked    用户Exception    spring默认不会进行事务回滚  比如：Exception

如何改变spring的这种默认事务行为？可以通过在方法上

添加@Transactional(noRollbackFor=RuntimeException.class)让spring对于RuntimeException不回滚事务

添加@Transactional(RollbackFor=Exception.class)让spring对于Exception进行事务的回滚

在项目中，@Transactional(rollbackFor=Exception.class)，如果类加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚

# 扩展原理

## BeanFactoryPostProcessor

```
 * BeanFactoryPostProcessor：beanFactory的后置处理器；
 * 		在BeanFactory标准初始化之后调用，来定制和修改BeanFactory的内容；
 * 		所有的bean定义已经保存加载到beanFactory，但是bean的实例还未创建
 * 
 * BeanFactoryPostProcessor原理:
 * 1)、ioc容器创建对象
 * 2)、invokeBeanFactoryPostProcessors(beanFactory);
 * 	   如何找到所有的BeanFactoryPostProcessor并执行他们的方法；
 * 	   1）、直接在BeanFactory中找到所有类型是BeanFactoryPostProcessor的组件，并执行他们的方法
 * 	   2）、在初始化创建其他组件前面执行
```

```java
@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {

	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		System.out.println("MyBeanFactoryPostProcessor...postProcessBeanFactory...");
		int count = beanFactory.getBeanDefinitionCount();
		String[] names = beanFactory.getBeanDefinitionNames();
		System.out.println("当前BeanFactory中有"+count+" 个Bean");
		System.out.println(Arrays.asList(names));
	}
}
```

## BeanDefinitionRegistryPostProcessor

```
 *	BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor
 * 		postProcessBeanDefinitionRegistry();
 * 		在所有bean定义信息将要被加载，bean实例还未创建时；
 * 
 * 		优先于BeanFactoryPostProcessor执行；
 * 		利用BeanDefinitionRegistryPostProcessor给容器中再额外添加一些组件；
 * 
 * 	原理：
 *	1）、ioc创建对象
 *	2）、refresh()-》invokeBeanFactoryPostProcessors(beanFactory);
 *	3）、从容器中获取到所有的BeanDefinitionRegistryPostProcessor组件。
 *		1、依次触发所有的postProcessBeanDefinitionRegistry()方法
 *		2、再来触发postProcessBeanFactory()方法BeanFactoryPostProcessor；
 *	4）、再来从容器中找到BeanFactoryPostProcessor组件；然后依次触发postProcessBeanFactory()方法
```

```java
@Component
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor{

	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		// TODO Auto-generated method stub
		System.out.println("MyBeanDefinitionRegistryPostProcessor...bean的数量："+beanFactory.getBeanDefinitionCount());
	}

	//BeanDefinitionRegistry Bean定义信息的保存中心，以后BeanFactory就是按照BeanDefinitionRegistry里面保存的每一个bean定义信息创建bean实例；
	@Override
	public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
		// TODO Auto-generated method stub
		System.out.println("postProcessBeanDefinitionRegistry...bean的数量："+registry.getBeanDefinitionCount());
		//RootBeanDefinition beanDefinition = new RootBeanDefinition(Blue.class);
		AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(Blue.class).getBeanDefinition();
		registry.registerBeanDefinition("hello", beanDefinition);
	}
}
```

## ApplicationListener

```
 * ApplicationListener：监听容器中发布的事件。事件驱动模型开发；
 * 	  public interface ApplicationListener<E extends ApplicationEvent>
 * 		监听 ApplicationEvent 及其下面的子事件；
 * 
 * 	 步骤：
 * 		1）、写一个监听器（ApplicationListener实现类）来监听某个事件（ApplicationEvent及其子类）
 * 			@EventListener;
 * 			原理：使用EventListenerMethodProcessor处理器来解析方法上的@EventListener；
 * 
 * 		2）、把监听器加入到容器；
 * 		3）、只要容器中有相关事件的发布，我们就能监听到这个事件；
 * 				ContextRefreshedEvent：容器刷新完成（所有bean都完全创建）会发布这个事件；
 * 				ContextClosedEvent：关闭容器会发布这个事件；
 * 		4）、发布一个事件：
 * 				applicationContext.publishEvent()；
 * 	
 *  原理：
 *  	ContextRefreshedEvent、IOCTest_Ext$1[source=我发布的时间]、ContextClosedEvent；
 *  1）、ContextRefreshedEvent事件：
 *  	1）、容器创建对象：refresh()；
 *  	2）、finishRefresh();容器刷新完成会发布ContextRefreshedEvent事件
 *  2）、自己发布事件；
 *  3）、容器关闭会发布ContextClosedEvent；
 *  
 *  【事件发布流程】：
 *  	publishEvent(new ContextRefreshedEvent(this));
 *  	1）、获取事件的多播器（派发器）：getApplicationEventMulticaster()
 *  	2）、multicastEvent派发事件：
 *  	3）、获取到所有的ApplicationListener；
 *  		for (ApplicationListener<?> listener : getApplicationListeners(event, type)) 
 *  		1）、如果有Executor，可以支持使用Executor进行异步派发；
 *  			Executor executor = getTaskExecutor();
 *  		2）、否则，同步的方式直接执行listener方法；invokeListener(listener, event);
 *  			拿到listener回调onApplicationEvent方法；
 *  
 *  【事件多播器（派发器）】
 *  1）、容器创建对象：refresh();
 *  2）、initApplicationEventMulticaster();初始化ApplicationEventMulticaster；
 *  	1）、先去容器中找有没有id=“applicationEventMulticaster”的组件；
 *  	2）、如果没有this.applicationEventMulticaster = new SimpleApplicationEventMulticaster (beanFactory);
 *  		并且加入到容器中，我们就可以在其他组件要派发事件，自动注入这个applicationEventMulticaster；
 *  
 *  【容器中有哪些监听器】
 *  1）、容器创建对象：refresh();
 *  2）、registerListeners();
 *  	从容器中拿到所有的监听器，把他们注册到applicationEventMulticaster中；
 *  	String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
 *  	//将listener注册到ApplicationEventMulticaster中
 *		getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);	
```

```java
@Component
public class MyApplicationListener implements ApplicationListener<ApplicationEvent> {

	//当容器中发布此事件以后，方法触发
	@Override
	public void onApplicationEvent(ApplicationEvent event) {
		// TODO Auto-generated method stub
		System.out.println("收到事件："+event);
	}
}
```

当然，用`@EventListener`注解更为便捷，参照：`EventListenerMethodProcessor`，它实现了`SmartInitializingSingleton`的接口：

```
 * SmartInitializingSingleton 原理：->afterSingletonsInstantiated();
 * 1）、ioc容器创建对象并refresh()；
 * 2）、finishBeanFactoryInitialization(beanFactory);初始化剩下的单实例bean；
 * 		1）、先创建所有的单实例bean；getBean();
 *   	2）、获取所有创建好的单实例bean，判断是否是SmartInitializingSingleton类型的；
 *   		如果是就调用afterSingletonsInstantiated();
```

```java
@Service
public class UserService {
	
	@EventListener(classes={ApplicationEvent.class})
	public void listen(ApplicationEvent event){
		System.out.println("UserService。。监听到的事件："+event);
	}
}
```

# 容器创建过程

## 源码分析

```
======================================================================================
**********************Spring容器的refresh()【创建刷新】**********************************
======================================================================================
1、prepareRefresh()刷新前的预处理;
   1）、initPropertySources()初始化一些属性设置;子类自定义个性化的属性设置方法；
   2）、getEnvironment().validateRequiredProperties();检验属性的合法等
   3）、earlyApplicationEvents= new LinkedHashSet<ApplicationEvent>();保存容器中的一些早期的事件；
======================================================================================
2、obtainFreshBeanFactory();获取BeanFactory；
   1）、refreshBeanFactory();刷新【创建】BeanFactory；
	   创建了一个this.beanFactory = new DefaultListableBeanFactory();
	   设置id；
   2）、getBeanFactory();返回刚才GenericApplicationContext创建的BeanFactory对象；
   3）、将创建的BeanFactory【DefaultListableBeanFactory】返回；
======================================================================================
3、prepareBeanFactory(beanFactory);BeanFactory的预准备工作（BeanFactory进行一些设置）；
   1）、设置BeanFactory的类加载器、支持表达式解析器...
   2）、添加部分BeanPostProcessor【ApplicationContextAwareProcessor】
   3）、设置忽略的自动装配的接口EnvironmentAware、EmbeddedValueResolverAware、xxx；
   4）、注册可以解析的自动装配；我们能直接在任何组件中自动注入：
	   BeanFactory、ResourceLoader、ApplicationEventPublisher、ApplicationContext
   5）、添加BeanPostProcessor【ApplicationListenerDetector】
   6）、添加编译时的AspectJ；
   7）、给BeanFactory中注册一些能用的组件；
	   environment【ConfigurableEnvironment】、
	   systemProperties【Map<String, Object>】、
	   systemEnvironment【Map<String, Object>】
======================================================================================
4、postProcessBeanFactory(beanFactory);BeanFactory准备工作完成后进行的后置处理工作；
   子类通过重写这个方法来在BeanFactory创建并预准备完成以后做进一步的设置
======================================================================================
**********************以上是BeanFactory的创建及预准备工作**********************************
======================================================================================
5、invokeBeanFactoryPostProcessors(beanFactory);执行BeanFactoryPostProcessor的方法；
   BeanFactoryPostProcessor：BeanFactory的后置处理器。在BeanFactory标准初始化之后执行的；
   两个接口：BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor（子接口）
   1）、先执行BeanDefinitionRegistryPostProcessor
       1）、获取所有的BeanDefinitionRegistryPostProcessor；
	   2）、看先执行实现了PriorityOrdered优先级接口的BeanDefinitionRegistryPostProcessor、
		   postProcessor.postProcessBeanDefinitionRegistry(registry)
	   3）、在执行实现了Ordered顺序接口的BeanDefinitionRegistryPostProcessor；
		   postProcessor.postProcessBeanDefinitionRegistry(registry)
	   4）、最后执行没有实现任何优先级或者是顺序接口的BeanDefinitionRegistryPostProcessors；
		   postProcessor.postProcessBeanDefinitionRegistry(registry)					
   2）、再执行BeanFactoryPostProcessor的方法
	   1）、获取所有的BeanFactoryPostProcessor
	   2）、看先执行实现了PriorityOrdered优先级接口的BeanFactoryPostProcessor、
		   postProcessor.postProcessBeanFactory()
	   3）、在执行实现了Ordered顺序接口的BeanFactoryPostProcessor；
		   postProcessor.postProcessBeanFactory()
	   4）、最后执行没有实现任何优先级或者是顺序接口的BeanFactoryPostProcessor；
		   postProcessor.postProcessBeanFactory()	     ======================================================================================
6、registerBeanPostProcessors(beanFactory);注册BeanPostProcessor（Bean的后置处理器）【 intercept bean creation】
   不同接口类型的BeanPostProcessor；在Bean创建前后的执行时机是不一样的
   BeanPostProcessor、
   DestructionAwareBeanPostProcessor、
   InstantiationAwareBeanPostProcessor、
   SmartInstantiationAwareBeanPostProcessor、
   MergedBeanDefinitionPostProcessor【internalPostProcessors】、		
   1）、获取所有的 BeanPostProcessor;后置处理器都默认可以通过PriorityOrdered、Ordered接口来执行优先级
   2）、先注册PriorityOrdered优先级接口的BeanPostProcessor；
	   把每一个BeanPostProcessor；添加到BeanFactory中
       beanFactory.addBeanPostProcessor(postProcessor);
   3）、再注册Ordered接口的
   4）、最后注册没有实现任何优先级接口的
   5）、最终注册MergedBeanDefinitionPostProcessor；
   6）、注册一个ApplicationListenerDetector；来在Bean创建完成后检查是否是ApplicationListener，
   	   如果是:applicationContext.addApplicationListener((ApplicationListener<?>) bean);
======================================================================================
7、initMessageSource();初始化MessageSource组件（做国际化功能；消息绑定，消息解析）；
   1）、获取BeanFactory
   2）、看容器中是否有id为messageSource的，类型是MessageSource的组件
	   如果有赋值给messageSource，如果没有自己创建一个DelegatingMessageSource；
	   MessageSource：取出国际化配置文件中的某个key的值；能按照区域信息获取；
   3）、把创建好的MessageSource注册在容器中，以后获取国际化配置文件的值的时候，可以自动注入
       MessageSource；
	   beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);	
	   MessageSource.getMessage(String code, Object[] args, String defaultMessage, Locale locale);
======================================================================================
8、initApplicationEventMulticaster();初始化事件派发器；
   1）、获取BeanFactory
   2）、从BeanFactory中获取applicationEventMulticaster的ApplicationEventMulticaster；
   3）、如果上一步没有配置；创建一个SimpleApplicationEventMulticaster
   4）、将创建的ApplicationEventMulticaster添加到BeanFactory中，以后其他组件直接自动注入
======================================================================================
9、onRefresh();留给子容器（子类）
   子类重写这个方法，在容器刷新的时候可以自定义逻辑；
======================================================================================
10、registerListeners();给容器中将所有项目里面的ApplicationListener注册进来；
   1）、从容器中拿到所有的ApplicationListener
   2）、将每个监听器添加到事件派发器中；
      getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
   3）、派发之前步骤产生的事件；
======================================================================================
11、finishBeanFactoryInitialization(beanFactory);初始化所有剩下的单实例bean；
   1、beanFactory.preInstantiateSingletons();初始化后剩下的单实例bean
	  1）、获取容器中的所有Bean，依次进行初始化和创建对象
	  2）、获取Bean的定义信息；RootBeanDefinition
	  3）、Bean不是抽象的，是单实例的，不是懒加载；
		  1）、判断是否是FactoryBean；是否是实现FactoryBean接口的Bean；
		  2）、不是工厂Bean。利用getBean(beanName);创建对象
			  0、getBean(beanName)； ioc.getBean();
			  1、doGetBean(name, null, null, false);
			  2、先获取缓存中保存的单实例Bean。
			     如果能获取到说明这个Bean之前被创建过（所有创建过的单实例Bean都会被缓存起来）
				 从singletonObjects = new ConcurrentHashMap<String, Object>(256);获取
			  3、缓存中获取不到，开始Bean的创建对象流程；
			  4、标记当前bean已经被创建
			  5、获取Bean的定义信息；
			  6、【获取当前Bean依赖的其他Bean;如果有按照getBean()把依赖的Bean先创建出来；】
			  7、启动单实例Bean的创建流程；
				 1）、createBean(beanName, mbd, args);
				 2）、Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
				     让BeanPostProcessor先拦截返回代理对象；
					 【InstantiationAwareBeanPostProcessor】：提前执行；
					  先触发：postProcessBeforeInstantiation()；
					  如果有返回值：触发postProcessAfterInitialization()；
				 3）、如果前面的InstantiationAwareBeanPostProcessor没有返回代理对象；调用4）
				 4）、beanInstance = doCreateBean(beanName, mbdToUse, args);创建Bean
					 1）、【创建Bean实例】；createBeanInstance(beanName, mbd, args);
						  利用工厂方法或者对象的构造器创建出Bean实例；
					 2）、applyMergedBeanDefinitionPostProcessors();
						 调用MergedBeanDefinitionPostProcessor的
                         postProcessMergedBeanDefinition(mbd, beanType, beanName);
					 3）、【Bean属性赋值】populateBean(beanName, mbd, instanceWrapper);
						 	赋值之前：
						  1）、拿到InstantiationAwareBeanPostProcessor后置处理器；
						 	  postProcessAfterInstantiation()；
						  2）、拿到InstantiationAwareBeanPostProcessor后置处理器；
						 	  postProcessPropertyValues()；
						 	=====赋值之前：===
						  3）、应用Bean属性的值；为属性利用setter方法等进行赋值；
						 	  applyPropertyValues(beanName, mbd, bw, pvs);
					 4）、【Bean初始化】initializeBean(beanName, exposedObject, mbd);
						  1）、【执行Aware接口方法】invokeAwareMethods(beanName, bean);
						  	   执行xxxAware接口的方法
						 	   BeanNameAware\BeanClassLoaderAware\BeanFactoryAware
						  2）、【执行后置处理器初始化之前】
                               applyBeanPostProcessorsBeforeInitialization();
						 	   BeanPostProcessor.postProcessBeforeInitialization（）;
						  3）、【执行初始化方法】
						  	   invokeInitMethods(beanName, wrappedBean, mbd);
						 	   1）、是否是InitializingBean接口的实现；执行接口规定的初始化；
						 	   2）、是否自定义初始化方法；
						  4）、【执行后置处理器初始化之后】
						       applyBeanPostProcessorsAfterInitialization
						 	   BeanPostProcessor.postProcessAfterInitialization()；
					 5）、注册Bean的销毁方法；
				 5）、将创建的Bean添加到缓存中singletonObjects；
				     ioc容器就是这些Map；很多的Map里面保存了单实例Bean，环境信息。。。。；
所有Bean都利用getBean创建完成以后；
检查所有的Bean是否是SmartInitializingSingleton接口的；如果是；就执行
afterSingletonsInstantiated()；
======================================================================================
12、finishRefresh();完成BeanFactory的初始化创建工作；IOC容器就创建完成；
    1）、initLifecycleProcessor();初始化和生命周期有关的后置处理器；LifecycleProcessor
		 默认从容器中找是否有lifecycleProcessor的组件【LifecycleProcessor】；
		 如果没有new DefaultLifecycleProcessor();加入到容器；			
		 写一个LifecycleProcessor的实现类，可以在BeanFactory
		 void onRefresh();
		 void onClose();	
   2）、	getLifecycleProcessor().onRefresh();
		 拿到前面定义的生命周期处理器（BeanFactory）；回调onRefresh()；
   3）、publishEvent(new ContextRefreshedEvent(this));发布容器刷新完成事件；
   4）、liveBeansView.registerApplicationContext(this);
======================================================================================
**********************容器刷新流程结束**********************************
======================================================================================
```

## 过程总结

	1）、Spring容器在启动的时候，先会保存所有注册进来的Bean的定义信息；
		1）、xml注册bean；<bean>
		2）、注解注册Bean；@Service、@Component、@Bean、xxx
	2）、Spring容器会合适的时机创建这些Bean
		1）、用到这个bean的时候；利用getBean创建bean；创建好以后保存在容器中；
		2）、统一创建剩下所有的bean的时候；finishBeanFactoryInitialization()；
	3）、后置处理器；BeanPostProcessor
		1）、每一个bean创建完成，都会使用各种后置处理器进行处理；来增强bean的功能；
			AutowiredAnnotationBeanPostProcessor:处理自动注入
			AnnotationAwareAspectJAutoProxyCreator:来做AOP功能；
			xxx....
			增强的功能注解：
			AsyncAnnotationBeanPostProcessor
			....
	4）、事件驱动模型；
		ApplicationListener；事件监听；
		ApplicationEventMulticaster；事件派发：

# SpringMVC整合Servlet3.0

## 源码分析

```
1、web容器在启动的时候，会扫描每个jar包下的META-INF/services/javax.servlet.ServletContainerInitializer
2、加载这个文件指定的类SpringServletContainerInitializer
3、spring的应用一启动会加载感兴趣的WebApplicationInitializer接口的下的所有组件；
4、并且为WebApplicationInitializer组件创建对象（组件不是接口，不是抽象类）
	1）、AbstractContextLoaderInitializer：创建根容器；createRootApplicationContext()；
	2）、AbstractDispatcherServletInitializer：
			创建一个web的ioc容器；createServletApplicationContext();
			创建了DispatcherServlet；createDispatcherServlet()；
			将创建的DispatcherServlet添加到ServletContext中；
				getServletMappings();
	3）、AbstractAnnotationConfigDispatcherServletInitializer：注解方式配置的DispatcherServlet初始化器
			创建根容器：createRootApplicationContext()
					getRootConfigClasses();传入一个配置类
			创建web的ioc容器： createServletApplicationContext();
					获取配置类；getServletConfigClasses();

```

## 过程总结

以注解方式来启动SpringMVC：

- 继承AbstractAnnotationConfigDispatcherServletInitializer；
- 实现抽象方法指定DispatcherServlet的配置信息；

启动类：

```java
//web容器启动的时候创建对象；调用方法来初始化容器以前前端控制器
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	//获取根容器的配置类；（Spring的配置文件）   父容器；
	@Override
	protected Class<?>[] getRootConfigClasses() {
		// TODO Auto-generated method stub
		return new Class<?>[]{RootConfig.class};
	}

	//获取web容器的配置类（SpringMVC配置文件）  子容器；
	@Override
	protected Class<?>[] getServletConfigClasses() {
		// TODO Auto-generated method stub
		return new Class<?>[]{AppConfig.class};
	}

	//获取DispatcherServlet的映射信息
	//  /：拦截所有请求（包括静态资源（xx.js,xx.png）），但是不包括*.jsp；
	//  /*：拦截所有请求；连*.jsp页面都拦截；jsp页面是tomcat的jsp引擎解析的；
	@Override
	protected String[] getServletMappings() {
		// TODO Auto-generated method stub
		return new String[]{"/"};
	}

}
```
配置类：

```java
//Spring的容器不扫描controller;父容器
@ComponentScan(value="com.atguigu",excludeFilters={
		@Filter(type=FilterType.ANNOTATION,classes={Controller.class})
})
public class RootConfig {

}
```

```java
//SpringMVC只扫描Controller；子容器
//useDefaultFilters=false 禁用默认的过滤规则；
@ComponentScan(value="com.atguigu",includeFilters={
		@Filter(type=FilterType.ANNOTATION,classes={Controller.class})
},useDefaultFilters=false)
@EnableWebMvc
public class AppConfig  extends WebMvcConfigurerAdapter  {

	//定制
	
	//视图解析器
	@Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
		// TODO Auto-generated method stub
		//默认所有的页面都从 /WEB-INF/ xxx .jsp
		//registry.jsp();
		registry.jsp("/WEB-INF/views/", ".jsp");
	}
	
	//静态资源访问
	@Override
	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
		// TODO Auto-generated method stub
		configurer.enable();
	}
	
	//拦截器
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		// TODO Auto-generated method stub
		//super.addInterceptors(registry);
		registry.addInterceptor(new MyFirstInterceptor()).addPathPatterns("/**");
	}

}
```

控制器类：

```java
@Controller
public class HelloController {
	
	@Autowired
	HelloService helloService;
	
	
	@ResponseBody
	@RequestMapping("/hello")
	public String hello(){
		String hello = helloService.sayHello("tomcat..");
		return hello;
	}
	
	//  /WEB-INF/views/success.jsp
	@RequestMapping("/suc")
	public String success(){
		return "success";
	}
	
}
```

服务类：

```java
@Service
public class HelloService {
	
	public String sayHello(String name){
		
		return "Hello "+name;
	}

}
```

## 异步请求

**方案1：**控制器返回Callable

```java
/**
	 * 1、控制器返回Callable
	 * 2、Spring异步处理，将Callable 提交到 TaskExecutor 使用一个隔离的线程进行执行
	 * 3、DispatcherServlet和所有的Filter退出web容器的线程，但是response 保持打开状态；
	 * 4、Callable返回结果，SpringMVC将请求重新派发给容器，恢复之前的处理；
	 * 5、根据Callable返回的结果。SpringMVC继续进行视图渲染流程等（从收请求-视图渲染）。
	 * 
	 *  preHandle.../springmvc-annotation/async01
		主线程开始...Thread[http-bio-8081-exec-3,5,main]==>1513932494700
		主线程结束...Thread[http-bio-8081-exec-3,5,main]==>1513932494700
		=========DispatcherServlet及所有的Filter退出线程============================

		================等待Callable执行==========
		副线程开始...Thread[MvcAsync1,5,main]==>1513932494707
		副线程开始...Thread[MvcAsync1,5,main]==>1513932496708
		================Callable执行完成==========

		================再次收到之前重发过来的请求========
		preHandle.../springmvc-annotation/async01
		postHandle...（Callable的之前的返回值就是目标方法的返回值）
		afterCompletion...

		异步的拦截器:
			1）、原生API的AsyncListener
			2）、SpringMVC：实现AsyncHandlerInterceptor；
	 * @return
*/
@ResponseBody
@RequestMapping("/async01")
public Callable<String> async01(){
    System.out.println("主线程开始..."+Thread.currentThread()
                       +"==>"+System.currentTimeMillis());

    Callable<String> callable = new Callable<String>() {
        @Override
        public String call() throws Exception {
            System.out.println("副线程开始..."+Thread.currentThread()
                               +"==>"+System.currentTimeMillis());
            Thread.sleep(2000);
            System.out.println("副线程开始..."+Thread.currentThread()
                               +"==>"+System.currentTimeMillis());
            return "Callable<String> async01()";
        }
    };

    System.out.println("主线程结束..."+Thread.currentThread()
                       +"==>"+System.currentTimeMillis());
    return callable;
}
```

**方案2：**请求作为消息投递

![image-20200518154410905](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200518154420-672650.png) 

```java
@ResponseBody
@RequestMapping("/createOrder")
public DeferredResult<Object> createOrder(){
    DeferredResult<Object> deferredResult = new DeferredResult<>((long)3000, "create fail...");

    DeferredResultQueue.save(deferredResult);

    return deferredResult;
}

@ResponseBody
@RequestMapping("/create")
public String create(){
    //创建订单
    String order = UUID.randomUUID().toString();
    DeferredResult<Object> deferredResult = DeferredResultQueue.get();
    deferredResult.setResult(order);
    return "success===>"+order;
}

public class DeferredResultQueue {
	
	private static Queue<DeferredResult<Object>> queue = new ConcurrentLinkedQueue<DeferredResult<Object>>();
	
	public static void save(DeferredResult<Object> deferredResult){
		queue.add(deferredResult);
	}
	
	public static DeferredResult<Object> get( ){
		return queue.poll();
	}

}
```

# 实际应用

## 构造器注入

使用构造器注入的好处：

- 保证依赖不可变（final关键字）
- 保证依赖不为空（省去了我们对其检查）
- 保证返回客户端（调用）的代码的时候是完全初始化的状态
- 避免了循环依赖
- 提升了代码的可复用性

## 属性赋值

`@Value`和`@PropertySource`给类中的属性赋值：
使用@Value给属性赋值：

> 1、基本数值
> 2、可以写SpEL； #{}
> 3、可以写${}；取出配置文件【properties】中的值（在运行环境变量里面的值）

@PropertySource用于配置类，读取配置文件信息：

> @PropertySource(value={"classpath:/book.properties", "classpath:/person.properties"})，可同时读取多个配置文件

## 获取类自定义注解

Spring启动的时候，有关注解的实现类，将会以bean的形式存入容器中，而我们也编写了获取bean和获取相关类的方法。那么接下来，我们需要做的，是在启动时候，把实现类，放在map中，通过，Map<'人+内容',‘实现类’>的形式保存。在需要的时候直接getKey获取有关的实现类。
在这里我们需要实现ApplicationListener<ContextRefreshedEvent>接口，该接口的作用是在Spring启动的最后阶段，Spring自动执行该接口的方法onApplicationEvent。

```java
@Service
public class initContent implements ApplicationListener<ContextRefreshedEvent> {
 
	@Override
	public void onApplicationEvent(ContextRefreshedEvent event) {
		// 通过注解获取相关的类
		Map<String, Object> map = getContent.getMapbeanwithAnnotion(personInfo.class);
		for (Map.Entry<String, Object> entrymap : map.entrySet()) {
			try {
				// 通过反射获取相关的实现类的Object
				Object object = getContent.getTarget(entrymap.getValue());
				if (object != null) {
					PersonAnnotationService annotationService = (PersonAnnotationService) object;
					// 不为空的情况下，获取实现类的注解对象
					// 并把注解对象的注解字段当做map的Key,实现类Object当做值
					personInfo info = annotationService.getClass().getAnnotation(personInfo.class);
					getContent.getPersonbeanmap.put(info.personInfo(), object);
				}
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
}
```

## SpringBoot多数据源配置

### 首先是配置文件

- 采用yml配置文件，其他类型配置文件同理
- 配置了两个数据源，一个名字叫ds1数据源，一个名字叫ds2数据源，如果你想配置更多的数据源，继续加就行了

```yaml
spring:
 # 数据源配置
  datasource:
    ds1: #数据源1
      driver-class-name: com.mysql.jdbc.Driver # mysql的驱动你可以配置别的关系型数据库
      url: jdbc:mysql://ip:3306/db1 #数据源地址
      username: root # 用户名
      password: root # 密码
    ds2: # 数据源2
      driver-class-name: com.mysql.jdbc.Driver # mysql的驱动你可以配置别的关系型数据库
      url: jdbc:mysql://ip:3307/db2#数据源地址
      username: root # 用户名
      password: root # 密码
```

### 多数据源配置

增加一个Springboot的配置类

```java
/**
 * 多数据源配置
 */
@Configuration
public class DataSourceConfig {

    //主数据源配置 ds1数据源
    @Primary
    @Bean(name = "ds1DataSourceProperties")
    @ConfigurationProperties(prefix = "spring.datasource.ds1")
    public DataSourceProperties ds1DataSourceProperties() {
        return new DataSourceProperties();
    }

    //主数据源 ds1数据源
    @Primary
    @Bean(name = "ds1DataSource")
    public DataSource ds1DataSource(@Qualifier("ds1DataSourceProperties") DataSourceProperties dataSourceProperties) {
        return dataSourceProperties.initializeDataSourceBuilder().build();
    }

    //第二个ds2数据源配置
    @Bean(name = "ds2DataSourceProperties")
    @ConfigurationProperties(prefix = "spring.datasource.ds2")
    public DataSourceProperties ds2DataSourceProperties() {
        return new DataSourceProperties();
    }

    //第二个ds2数据源
    @Bean("ds2DataSource")
    public DataSource ds2DataSource(@Qualifier("ds2DataSourceProperties") DataSourceProperties dataSourceProperties) {
        return dataSourceProperties.initializeDataSourceBuilder().build();
    }

}
```

### JdbcTemplate多数据源配置

增加一个Springboot配置类

```java
/**
 * JdbcTemplate多数据源配置
 * 依赖于数据源配置
 *
 * @see DataSourceConfig
 */
@Configuration
public class JdbcTemplateDataSourceConfig {

    //JdbcTemplate主数据源ds1数据源
    @Primary
    @Bean(name = "ds1JdbcTemplate")
    public JdbcTemplate ds1JdbcTemplate(@Qualifier("ds1DataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    //JdbcTemplate第二个ds2数据源
    @Bean(name = "ds2JdbcTemplate")
    public JdbcTemplate ds2JdbcTemplate(@Qualifier("ds2DataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

### mybatis 多数据源配置

增加一个SpringBoot配置类
 mybatis多数据源的原理是根据不同包，调用不同的数据源，你只需要把你的mapper.java和mapper.xml(我喜欢叫dao.java和dao.xml)写在某个package中，springboot自动帮你实现数据源切换
 核心代码就这句
 `@MapperScan(basePackages ="com.web.ds2.**.dao", sqlSessionTemplateRef = "ds2SqlSessionTemplate")`
 用来指定包扫描指定sqlSessionTemplateRef
 和`sqlSessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath*:com/web/ds2/**/*.xml"));`
 用来指定mapper.xml的路径

**详细配置代码如下**

```java
/**
 * Mybatis主数据源ds1配置
 * 多数据源配置依赖数据源配置
 * @see  DataSourceConfig
 */
@Configuration
@MapperScan(basePackages ="com.web.ds1.**.dao", sqlSessionTemplateRef  = "ds1SqlSessionTemplate")
public class MybatisPlusConfig4ds1 {

    //主数据源 ds1数据源
    @Primary
    @Bean("ds1SqlSessionFactory")
    public SqlSessionFactory ds1SqlSessionFactory(@Qualifier("ds1DataSource") DataSource dataSource) throws Exception {
        MybatisSqlSessionFactoryBean sqlSessionFactory = new MybatisSqlSessionFactoryBean();
        sqlSessionFactory.setDataSource(dataSource);
        sqlSessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().
                        getResources("classpath*:com/web/ds1/**/*.xml"));  
        return sqlSessionFactory.getObject();
    }

    @Primary
    @Bean(name = "ds1TransactionManager")
    public DataSourceTransactionManager ds1TransactionManager(@Qualifier("ds1DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Primary
    @Bean(name = "ds1SqlSessionTemplate")
    public SqlSessionTemplate ds1SqlSessionTemplate(@Qualifier("ds1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

```java
/**
 * Mybatis  第二个ds2数据源配置
 * 多数据源配置依赖数据源配置
 * @see  DataSourceConfig
 */
@Configuration
@MapperScan(basePackages ="com.web.ds2.**.dao", sqlSessionTemplateRef  = "ds2SqlSessionTemplate")
public class MybatisPlusConfig4ds2 {

    //ds2数据源
    @Bean("ds2SqlSessionFactory")
    public SqlSessionFactory ds2SqlSessionFactory(@Qualifier("ds2DataSource") DataSource dataSource) throws Exception {
        MybatisSqlSessionFactoryBean sqlSessionFactory = new MybatisSqlSessionFactoryBean();
        sqlSessionFactory.setDataSource(dataSource);
        sqlSessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().
                getResources("classpath*:com/web/ds2/**/*.xml"));
        return sqlSessionFactory.getObject();
    }

//事务支持
    @Bean(name = "ds2TransactionManager")
    public DataSourceTransactionManager ds2TransactionManager(@Qualifier("ds2DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "ds2SqlSessionTemplate")
    public SqlSessionTemplate ds2SqlSessionTemplate(@Qualifier("ds2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

### mybatis-plus 多数据源配置

> mybatis-plus是mybatis的增强版，只增加，不影响。也就是说使用mybatis-plus兼容原来所有的mybatis代码和配置。然后又做了很多增加和简化使用，具体看官网教程[https://mybatis.plus/](https://links.jianshu.com/go?to=https%3A%2F%2Fmybatis.plus%2F)

相对于mybatis的多数据源配置就是改了下 SqlSessionFactory
 核心代码就是修改mybatis为mybatis-plus，如下

```java
    @Bean("ds2SqlSessionFactory")
    public SqlSessionFactory ds2SqlSessionFactory(@Qualifier("ds2DataSource") DataSource dataSource) throws Exception {
        MybatisSqlSessionFactoryBean sqlSessionFactory = new MybatisSqlSessionFactoryBean();
        sqlSessionFactory.setDataSource(dataSource);
        MybatisConfiguration configuration = new MybatisConfiguration();
        configuration.setDefaultScriptingLanguage(MybatisXMLLanguageDriver.class);
        configuration.setJdbcTypeForNull(JdbcType.NULL);
        sqlSessionFactory.setConfiguration(configuration);
        sqlSessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().
                getResources("classpath*:com/web/ds2/**/*.xml"));
        sqlSessionFactory.setPlugins(new Interceptor[]{
                new PaginationInterceptor(),
                new PerformanceInterceptor()
//                        .setFormat(true),
        });
        sqlSessionFactory.setGlobalConfig(new GlobalConfig().setBanner(false));
        return sqlSessionFactory.getObject();
    }
```

全部配置代码如下

```java
/**
 * Mybatis-plus ds2数据源配置
 * 多数据源配置依赖数据源配置
 * @see  DataSourceConfig
 */
@Configuration
@MapperScan(basePackages ="com.web.ds2.**.dao", sqlSessionTemplateRef  = "ds2SqlSessionTemplate")
public class MybatisPlusConfig4ds2{

    //ds2数据源
    @Bean("ds2SqlSessionFactory")
    public SqlSessionFactory ds2SqlSessionFactory(@Qualifier("ds2DataSource") DataSource dataSource) throws Exception {
        MybatisSqlSessionFactoryBean sqlSessionFactory = new MybatisSqlSessionFactoryBean();
        sqlSessionFactory.setDataSource(dataSource);
        MybatisConfiguration configuration = new MybatisConfiguration();
        configuration.setDefaultScriptingLanguage(MybatisXMLLanguageDriver.class);
        configuration.setJdbcTypeForNull(JdbcType.NULL);
        sqlSessionFactory.setConfiguration(configuration);
        sqlSessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().
                getResources("classpath*:com/web/ds2/**/*.xml"));
        sqlSessionFactory.setPlugins(new Interceptor[]{
                new PaginationInterceptor(),
                new PerformanceInterceptor()
//                        .setFormat(true),
        });
        sqlSessionFactory.setGlobalConfig(new GlobalConfig().setBanner(false));
        return sqlSessionFactory.getObject();
    }

    @Bean(name = "ds2TransactionManager")
    public DataSourceTransactionManager ds2TransactionManager(@Qualifier("ds2DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "ds2SqlSessionTemplate")
    public SqlSessionTemplate ds2SqlSessionTemplate(@Qualifier("ds2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

```java
/**
 * Mybatis-plus 主数据源ds1配置
 * 多数据源配置依赖数据源配置
 * @see  DataSourceConfig
 */
@Configuration
@MapperScan(basePackages ="com.web.ds1.**.dao", sqlSessionTemplateRef  = "ds1SqlSessionTemplate")
public class MybatisPlusConfig4ds1 {

    //主数据源 ds1数据源
    @Primary
    @Bean("ds1SqlSessionFactory")
    public SqlSessionFactory ds1SqlSessionFactory(@Qualifier("ds1DataSource") DataSource dataSource) throws Exception {
        MybatisSqlSessionFactoryBean sqlSessionFactory = new MybatisSqlSessionFactoryBean();
        sqlSessionFactory.setDataSource(dataSource);
        MybatisConfiguration configuration = new MybatisConfiguration();
        configuration.setDefaultScriptingLanguage(MybatisXMLLanguageDriver.class);
        configuration.setJdbcTypeForNull(JdbcType.NULL);
        sqlSessionFactory.setConfiguration(configuration);
        sqlSessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().
                        getResources("classpath*:com/ds1/web/ds1/**/*.xml"));
        sqlSessionFactory.setPlugins(new Interceptor[]{
                new PaginationInterceptor(),
                new PerformanceInterceptor()
//                        .setFormat(true),
        });
        sqlSessionFactory.setGlobalConfig(new GlobalConfig().setBanner(false));
        return sqlSessionFactory.getObject();
    }

    @Primary
    @Bean(name = "ds1TransactionManager")
    public DataSourceTransactionManager ds1TransactionManager(@Qualifier("ds1DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Primary
    @Bean(name = "ds1SqlSessionTemplate")
    public SqlSessionTemplate ds1SqlSessionTemplate(@Qualifier("ds1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

## 注解注册Bean的四种方式

给spring容器中注册bean有四种通过注解的方式：

1. 包扫描+组件标注注解
2. @Bean
3. @Import
4. 使用FactoryBean（工厂Bean）

下面逐个介绍他们的用法：

### 包扫描+组件标注注解

这种方式使我们最为常见的一种，通过两类注解配合使用。

@ComponentScan注解用来标在配置类的上方，表示了需要对哪些包进行扫描。也可以通过excludeFilters属性来指明那些类型是不被扫描的组件。

第二类注解是指@Controller/@Service/@Repository/@Component四个中的一个，这些注解一般标在自己写的类的上方，表示他们是需要注册到spring容器中的。

### @Bean注解

有时候我们需要将一些类，不属于我们编写的类注册进spring容器当中，这时候我们不能再通过上面那种方式，而是需要在注解类中的方法上面加上@Bean注解，这样就会将该方法所返回的对象注册进spring容器当中。

spring中注册时实例默认是单实例的，即singleton。在ioc容器启动会调用方法创建对象放到ioc容器中。以后每次获取就是直接从容器（map.get()）中拿。

当我们在@Bean注解相同位置再添加一个@Scope(“prototype”)注解时，它表示Bean是多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。每次获取的时候才会调用方法创建对象。

当然，我们也可以通过@Lazy注解加在@Bean注解相同位置来表示懒加载，这样即使是默认情况下的单例，容器启动的时候创建对象，也会变成容器启动不创建对象。第一次使用(获取)Bean创建对象，并初始化。

### @Import注解

@Import注解使用时直接标注在配置类的上方，表示向Spring容器中注册哪些Bean。其中的value属性可以传入三种类型的值，分别是：直接类对象、ImportSelector和ImportBeanDefinitionRegistrar。

直接传入类对象就表示注册这些Bean；

传入ImportSelector的实现类，需要实现其中的selectImports()方法，返回的字符串数组表示要注册进容器中的Bean。例如：

```java
public String[] selectImports(AnnotationMetadata importingClassMetadata) {
	//方法不要返回null值
	return new String[]{"com.gaishiHero.bean.Blue","com.gaishiHero.bean.Yellow"};
}
```

传入ImportBeanDefinitionRegistrar接口的实现类，需要实现其中的registerBeanDefinitions()方法，该方法第二个参数为BeanDefinitionRegistry，可以在该实现的方法中调用BeanDefinitionRegistry的registerBeanDefinition()方法，传入名称和BeanDefinition，就会将它们放入private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(64);中，就算注册完成。

```java
public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
	boolean definition = registry.containsBeanDefinition("com.gaishiHero.bean.Yellow");
	boolean definition2 = registry.containsBeanDefinition("com.gaishiHero.bean.Blue");
	if(definition && definition2){
		//定义一个BeanDefinition
		RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class);
		//注册一个Bean，指定bean名
		registry.registerBeanDefinition("rainBow", beanDefinition);
	}
}
```

### 使用FactoryBean

外形与@Bean注解类似，也是通过在配置类中写方法。不过不同点在于方法的返回值是FactoryBean的实例，此时，注册进容器中的Bean不是这个FactoryBean的实例，而是其中实现的方法getObject()的返回值。

补充说明：FactoryBean接口有三个方法需要被实现：getObject()、getObjectType()和isSingleton()。getObject()返回真实注册进IOC容器中的Bean，getObjectType()返回FactoryBean创造对象的类型，isSingleton()返回是否是单例方式，如果是单例方式，只会调用getObject()方法一次，如果不是单例，则会多次调用getObject()方法。

当然，如果我们真的需要注册进容器中的Bean就是FactoryBean的实例，则需要在获取Bean的时候给id前面加一个&，例如&colorFactoryBean

```java
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
//bean1是getObject()方法的返回值，bean2是colorFactoryBean对象本身
Object bean1 = applicationContext.getBean("colorFactoryBean");
Object bean2 = applicationContext.getBean("&colorFactoryBean");
```

## 定义Bean生命周期

### 使用@Bean注解

```java
/**
 *  指定组建的init方法和destroy的几种方法
 *  1：在配置类中 @Bean(initMethod = "init",destroyMethod = "destory")注解指定
 *  2：实现InitializingBean接口重写其afterPropertiesSet方法，实现DisposableBean接口重写destroy方法
 *  3：利用java的JSR250规范中的@PostConstruct标注在init方法上，@PreDestroy标注在destroy注解上
 */
```

需要注意的是：

- 单实例bean：容器启动时创建对象
- 多实例bean：每次获取时创建对象

- 初始化：
  - 对象创建完成，赋值完成，调用初始化方法
- 销毁：
  - 单实例：容器关闭时调用
  - 多实例：容器不会销毁，只能手动调用销毁方法

下面是具体代码

Car.java

```java
public class Car {
 
    public Car() {
        System.out.println("Car's Constructor..");
    }
 
    public void init(){
        System.out.println("Car's Init...");
    }
 
    public void destory(){
        System.out.println("Car's Destroy...");
    }
 
}
```

配置类

```java
@Bean(initMethod = "init",destroyMethod = "destory")
public Car car(){
    return new Car();
}
```

### 实现标记接口

在Spring中，`InitializingBean`和`DisposableBean`是两个标记接口，为Spring执行时bean的初始化和销毁某些行为时的有用方法。

1. 对于Bean实现 InitializingBean，它将运行 afterPropertiesSet()在所有的 bean 属性被设置之后。

2. 对于 Bean 实现了DisposableBean，它将运行 destroy()在 Spring 容器释放该 bean 之后。

**示例**

下面是一个例子，向您展示如何使用 InitializingBean 和 DisposableBean。一个 CustomerService bean来实现 InitializingBean和DisposableBean 接口，并有一个消息(message)属性。

```java
package com.yiibai.customer.services;

import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

public class CustomerService implements InitializingBean, DisposableBean
{
	String message;
	
	public String getMessage() {
	  return message;
	}

	public void setMessage(String message) {
	  this.message = message;
	}
	
	public void afterPropertiesSet() throws Exception {
	  System.out.println("Init method after properties are set : " + message);
	}
	
	public void destroy() throws Exception {
	  System.out.println("Spring Container is destroy! Customer clean up");
	}	
}
```

```java
package com.yiibai.common;

import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.yiibai.customer.services.CustomerService;

public class App 
{
    public static void main( String[] args )
    {
    	ConfigurableApplicationContext context = 
			new ClassPathXmlApplicationContext(new String[] {"applicationContext.xml"});
	
    	CustomerService cust = (CustomerService)context.getBean("customerService");
    	
    	System.out.println(cust);
    	
    	context.close();
    }
}
```

输出结果

```
Init method after properties are set : I'm property message 
com.yiibai.customer.services.CustomerService@4090c06f 
Spring Container is destroy! Customer clean up
```

### 使用JSR250注解

1.`@PostConstruct`说明

   被@PostConstruct修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器调用一次，类似于Serclet的inti()方法。被@PostConstruct修饰的方法会在构造函数之后，init()方法之前运行。

2.`@PreDestroy`说明

   被@PreDestroy修饰的方法会在服务器卸载Servlet的时候运行，并且只会被服务器调用一次，类似于Servlet的destroy()方法。被@PreDestroy修饰的方法会在destroy()方法之后运行，在Servlet被彻底卸载之前。（详见下面的程序实践）

**示例**

```java
package com.yygx.flowinterface.service.impl;
 
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
 
import net.zoneland.uniflow.client.impl.SyncProcessServiceClient;
import net.zoneland.uniflow.client.impl.SyncTaskServiceClient;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Repository;
 
/**
 *
 * 流程引擎客户端工具类
 * @author wuhda
 *
 */
@Repository
public class FlowClientUtil {
 
    private final static Logger LOGGER = LoggerFactory.getLogger(FlowClientUtil.class);
 
     
    /**
     * 流程类接口客户端
     */
    private SyncProcessServiceClient processClient;
     
    /**
     * 任务类接口客户端
     */
    private SyncTaskServiceClient taskClient;
     
     
    @PostConstruct
    public void init() {
        // 接入系统的账号密码，在流程管理平台（UniflowAdminConsole）上配置
        String username = "ehr";
        String password = "password";
         
        // Uniflow的服务地址
        String endpoint = "http://172.16.92.50:10081/UniflowControlCenter/rest";
         
        // 构建流程类接口客户端
        processClient = new SyncProcessServiceClient(username, password);
        processClient.setServerAddress( endpoint + "/RestProcessService" );
         
        // 构建任务类接口客户端
        taskClient = new SyncTaskServiceClient(username, password);
        taskClient.setServerAddress( endpoint + "/RestTaskService" );
    }
 
 
 
    @PreDestroy
    public void close() {
        System.out.println("close client ...");
        // 应用结束是最好关闭客户端（内部有HttpClient的连接池）
        processClient.close();
        taskClient.close();
    }
     
    public SyncProcessServiceClient getProcessClient() {
        return processClient;
    }
 
 
    public void setProcessClient(SyncProcessServiceClient processClient) {
        this.processClient = processClient;
    }
 
 
    public SyncTaskServiceClient getTaskClient() {
        return taskClient;
    }
 
 
    public void setTaskClient(SyncTaskServiceClient taskClient) {
        this.taskClient = taskClient;
    }     
}
```

### 实现后置处理器接口

有时候，我们希望在Spring IoC容器初始化受管Bean之前、属性设置之后对该Bean先做一些预处理，或者在容器销毁受管Bean之前自己释放资源。那么该如何实现呢？Spring IoC为我们提供了多种方法来实现受管Bean的预处理和后处理。
在Spring中定义了BeanPostProcessors接口，代码如下：

```java
public interface BeanPostProcessor {
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
    // 在调用bean的init方法（非构造方法）前调用
    // beanName 是在Spring IOC 容器的bean 名称，返回的对象需要会被直接使用
    // 切记！！！默认返回bean即可，不要无缘无故返回null，会出现各种NPE的情况
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
    // 在调用bean的init方法（非构造方法）后调用
}
```

如果这个接口的某个实现类被注册到某个容器，那么该容器的每个受管Bean在调用初始化方法之前，都会获得该接口实现类的一个回调。容器调用接口定义的方法时会将该受管Bean的实例和名字通过参数传入方法，进过处理后通过方法的返回值返回给容器。根据这个原理，我们就可以很轻松的自定义受管Bean。

上面提到过，要使用BeanPostProcessor回调，就必须先在容器中注册实现该接口的类，那么如何注册呢？BeanFactory和ApplicationContext容器的注册方式不大一样：若使用BeanFactory，则必须要显示的调用其addBeanPostProcessor()方法进行注册，参数为BeanPostProcessor实现类的实例；如果是使用ApplicationContext，那么容器会在配置文件在中自动寻找实现了BeanPostProcessor接口的Bean，然后自动注册，我们要做的只是配置一个BeanPostProcessor实现类的Bean就可以了。

注意，假如我们使用了多个的BeanPostProcessor的实现类，那么如何确定处理顺序呢？其实只要实现Ordered接口，设置order属性就可以很轻松的确定不同实现类的处理顺序了。

### 执行顺序总结

1. 首先执行bean的构造方法
2. BeanPostProcessor的postProcessBeforeInitialization方法
3. InitializingBean的afterPropertiesSet方法
4. @Bean注解的initMethod方法
5. BeanPostProcessor的postProcessAfterInitialization方法
6. DisposableBean的destroy方法
7. @Bean注解的destroyMethod方法

