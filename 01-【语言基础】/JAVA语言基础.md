# 基础数据类型

## String类

### 为什么是final的

- 为了实现字符串常量池；
- 为了线程安全；
- 为了实现String可以创建HashCode不可变性。

# 注解（Annotation）

## 注解定义方式

直接上代码，看看spring中@Service注解的定义就知道了：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {
 
 String value() default "";
  
}
```

可以看到注解的定义和接口定义很像，但是多了@字符，注解的定义上有以下约定：

- 只能定义属性名，不能定义方法
- 属性的可见性只有public和default，不写则默认后者
- 属性的类型只能支持：基本数据类型、string、class、enum、Annotation类型及以上类型的数组
- 可以加上defult关键字指明默认值，当某字段不指明默认值时，必须在进行注解标注的时候进行此字段值的指定。
- 当使用value作为属性名称时，可以不显式指定value=“xxx”，如可以直接使用@Service("xxxService")

## 元注解

所谓元注解就是java中默认实现的专门对注解进行注解的注解。元注解的总数就5个，下面我们以上面讲到的@Service注解为例子各个击破：

**1.@Target**

此注解用于表示当前注解的使用范围，@Target({ElementType.TYPE})就代表着@Service这个注解是专门用来注解到类、接口、或者枚举类型上面的，当在方法上面加这个注解时，就会报错。可以看到注解位置是一个枚举类型，完整定义如下

```java
public enum ElementType {
  /** Class, interface (including annotation type), or enum declaration */
  TYPE,
 
  /** Field declaration (includes enum constants) */
  FIELD,
 
  /** Method declaration */
  METHOD,
 
  /** Formal parameter declaration */
  PARAMETER,
 
  /** Constructor declaration */
  CONSTRUCTOR,
 
  /** Local variable declaration */
  LOCAL_VARIABLE,
 
  /** Annotation type declaration */
  ANNOTATION_TYPE,
 
  /** Package declaration */
  PACKAGE,
 
  /**
   * Type parameter declaration
   *
   * @since 1.8
   */
  TYPE_PARAMETER,
 
  /**
   * Use of a type
   *
   * @since 1.8
   */
  TYPE_USE
}
```

**2.@Retention**

此注解用于表示当前注解的生命周期，说人话就是这个注解作用会保留到什么时候，如@Retention(RetentionPolicy.RUNTIME)就表示在程序运行期间依然有效，此时就可以通过反射拿到注解的信息，完整的枚举定义如下

```java
public enum RetentionPolicy {
  /**
   * Annotations are to be discarded by the compiler.
   */
  SOURCE,
 
  /**
   * Annotations are to be recorded in the class file by the compiler
   * but need not be retained by the VM at run time. This is the default
   * behavior.
   */
  CLASS,
 
  /**
   * Annotations are to be recorded in the class file by the compiler and
   * retained by the VM at run time, so they may be read reflectively.
   *
   * @see java.lang.reflect.AnnotatedElement
   */
  RUNTIME
}
```

**3.@Documented**

当被此注解所注解时，使用javadoc工具生成文档就会带有注解信息。

**4.@Inherited**

此注解与继承有关，当A注解添加此注解后，将A注解添加到某类上，此类的子类就会继承A注解。

```java
@Inherited
public @interface A{
}
 
@A
public class Parent{}
 
public class Son entends Parant{}//Son类继承了父类的A注解
```

**5.@Repeatable**

此注解顾名思义是拥有可以重复注解的能力。想象这样一个场景，我们需要定时执行某个任务，需要在每周一和周三执行，并且这个时间是可以灵活调整的，此时这个元注解就能派上用场：

```java
@Repeatable(Schedules.class)
public @interface Schedule {
  String date();
}
 
public @interface Schedules {
  Schedule[] value();
}
 
@Schedule(date = "周一")
@Schedule(date = "周三")
public class Executor {
}
```

注意看到此元注解后面括号里内容，在这指定的类叫做容器注解，意思是保存这多个注解的容器，故我们创建一个@Schedules注解作为@Schedule的容器注解，容器注解必须含有一个名字为value，返回类型为需放入此容器的注解数组的属性。

# 函数式编程

## 概述

### 函数式编程简介

我们最常用的面向对象编程（Java）属于**命令式编程**（Imperative Programming）这种编程范式。常见的编程范式还有**逻辑式编程**（Logic Programming），**函数式编程**（Functional Programming）。

函数式编程作为一种编程范式，在科学领域，是一种编写计算机程序数据结构和元素的方式，它把计算过程当做是数学函数的求值，而避免更改状态和可变数据。

函数式编程并非近几年的新技术或新思维，距离它诞生已有大概50多年的时间了。它一直不是主流的编程思维，但在众多的所谓顶级编程高手的科学工作者间，函数式编程是十分盛行的。

什么是函数式编程？简单的回答：一切都是数学函数。函数式编程语言里也可以有对象，但通常这些对象都是恒定不变的 —— 要么是函数参数，要什么是函数返回值。函数式编程语言里没有 for/next 循环，因为这些逻辑意味着有状态的改变。相替代的是，这种循环逻辑在函数式编程语言里是通过递归、把函数当成参数传递的方式实现的。

举个例子：

```java
a = a + 1
```

这段代码在普通成员看来并没有什么问题，但在数学家看来确实不成立的，因为它意味着变量值得改变。

### Lambda 表达式简介

Java 8的最大变化是引入了Lambda（Lambda 是希腊字母 λ 的英文名称）表达式——一种紧凑的、传递行为的方式。

先看个例子：

```java
button.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent event) {
        System.out.println("button clicked");
    }
});
```

这段代码使用了匿名类。`ActionListener` 是一个接口，这里 new 了一个类实现了 `ActionListener` 接口，然后重写了 `actionPerformed` 方法。`actionPerformed` 方法接收 `ActionEvent` 类型参数，返回空。

这段代码我们其实只关心中间打印的语句，其他都是多余的。所以使用 Lambda 表达式，我们就可以简写为：

```java
button.addActionListener(event -> System.out.println("button clicked"));
```

## Lambda 表达式

### Lambda 表达式的形式

Java 中 Lambda 表达式一共有五种基本形式，具体如下：

- 第1种，所示的 Lambda 表达式不包含参数,使用空括号 () 表示没有参数。该 Lambda 表达式 实现了 Runnable 接口,该接口也只有一个 run 方法,没有参数,且返回类型为 void。

```
Runnable noArguments = () -> System.out.println("Hello World");
```

- 第2种，所示的 Lambda 表达式包含且只包含一个参数,可省略参数的括号,这和例 2-2 中的 形式一样。Lambda 表达式的主体不仅可以是一个表达式,而且也可以是一段代码块,使用大括号 ({})将代码块括起来。

```java
ActionListener oneArgument = event -> System.out.println("button clicked");
```

- 第3种，Lambda 表达式的主体不仅可以是一个表达式,而且也可以是一段代码块,使用大括号 ({})将代码块括起来，该代码块和普通方法遵循的规则别无二致,可以用返 回或抛出异常来退出。只有一行代码的 Lambda 表达式也可使用大括号,用以明确 Lambda表达式从何处开始、到哪里结束。

```java
Runnable multiStatement = () -> {
    System.out.print("Hello");
    System.out.println(" World");
};
```

- 第4种，Lambda 表达式也可以表示包含多个参数的方法，这行代码并不是将两个数字相加,而是创建了一个函数,用来计算 两个数字相加的结果。变量 add 的类型是 BinaryOperator,它不是两个数字的和, 而是将两个数字相加的那行代码。

```java
BinaryOperator<Long> add = (x, y) -> x + y;
```

- 第5种，到目前为止,所有 Lambda 表达式中的参数类型都是由编译器推断得出的。这当然不错, 但有时最好也可以显式声明参数类型,此时就需要使用小括号将参数括起来,多个参数的 情况也是如此。

```java
BinaryOperator<Long> addExplicit = (Long x, Long y) -> x + y;
```

### 闭包

如果你以前使用过匿名内部类，也许遇到过这样的问题。当你需要匿名内部类所在方法里的变量，必须把该变量声明为 `final`。如下例子所示：

```java
final String name = getUserName();
button.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent event) {
        System.out.println("hi " + name);
    }
});
```

Java 8放松了这一限制，可以不必再把变量声明为 `final`，但其实该变量实际上仍然是 `final` 的。虽然无需将变量声明为 final，但在 Lambda 表达式中，也无法用作非终态变量。如果坚持用作非终态变量（即改变变量的值），编译器就会报错。

### 函数接口

上面例子里提到了 `ActionListener` 接口，我们看一下它的代码：

```java
public interface ActionListener extends EventListener {

    /**
     * Invoked when an action occurs.
     */
    public void actionPerformed(ActionEvent e);

}
```

`ActionListener` 只有一个抽象方法：`actionPerformed`，被用来表示行为:接受一个参数，返回空。记住，由于 `actionPerformed` 定义在一个接口里，因此 `abstract` 关键字不是必需的。该接口也继承自一个不具有任何方法的父接口：`EventListener`。

我们把这种接口就叫做函数接口。

JDK 8 中提供了一组常用的核心函数接口：

| 接口              | 参数   | 返回类型 | 描述                                                         |
| :---------------- | :----- | :------- | :----------------------------------------------------------- |
| Predicate<T>      | T      | boolean  | 用于判别一个对象。比如求一个人是否为男性                     |
| Consumer<T>       | T      | void     | 用于接收一个对象进行处理但没有返回，比如接收一个人并打印他的名字 |
| Function<T, R>    | T      | R        | 转换一个对象为不同类型的对象                                 |
| Supplier<T>       | None   | T        | 提供一个对象                                                 |
| UnaryOperator<T>  | T      | T        | 接收对象并返回同类型的对象                                   |
| BinaryOperator<T> | (T, T) | T        | 接收两个同类型的对象，并返回一个原类型对象                   |

其中 `Cosumer` 与 `Supplier` 对应，一个是消费者，一个是提供者。

`Predicate` 用于判断对象是否符合某个条件，经常被用来过滤对象。

`Function` 是将一个对象转换为另一个对象，比如说要装箱或者拆箱某个对象。

`UnaryOperator` 接收和返回同类型对象，一般用于对对象修改属性。`BinaryOperator` 则可以理解为合并对象。

如果以前接触过一些其他 Java 框架，比如 Google Guava，可能已经使用过这些接口，对这些东西并不陌生。所以，其实 Java 8 的改进并不是闭门造车，而是集百家之长。

## 集合处理

### Stream 简介

在程序编写过程中，集合的处理应该是很普遍的。Java 8 对于 `Collection` 的处理花了很大的功夫，如果从 JDK 7 过渡到 JDK 8，这一块也可能是我们感受最为明显的。

Java 8 中，引入了流（Stream）的概念，这个流和以前我们使用的 IO 中的流并不太相同。

所有继承自 `Collection` 的接口都可以转换为 `Stream`。还是看一个例子。

假设我们有一个 `List` 包含一系列的 `Person`，`Person` 有姓名 `name` 和年龄 `age` 连个字段。现要求这个列表中年龄大于 20 的人数。

通常按照以前我们可能会这么写：

```java
long count = 0;
for (Person p : persons) {
    if (p.getAge() > 20) {
        count ++;
    }
}
```

但如果使用 `stream` 的话，则会简单很多:

```java
long count = persons.stream()
                    .filter(person -> person.getAge() > 20)
                    .count();
```

这只是 `stream` 的很简单的一个用法。现在链式调用方法算是一个主流，这样写也更利于阅读和理解编写者的意图，一步方法做一件事。

### Stream 常用操作

`Stream` 的方法分为两类。一类叫惰性求值，一类叫及早求值。

判断一个操作是惰性求值还是及早求值很简单：只需看它的返回值。如果返回值是 Stream，那么是惰性求值。其实可以这么理解，如果调用惰性求值方法，`Stream` 只是记录下了这个惰性求值方法的过程，并没有去计算，等到调用及早求值方法后，就连同前面的一系列惰性求值方法顺序进行计算，返回结果。

通用形式为：

```java
Stream.惰性求值.惰性求值. ... .惰性求值.及早求值
```

整个过程和建造者模式有共通之处。建造者模式使用一系列操作设置属性和配置，最后调 用一个 build 方法，这时，对象才被真正创建。

- collect(toList())

`collect(toList())` 方法由 `Stream` 里的值生成一个列表，是一个及早求值操作。可以理解为 `Stream` 向 `Collection` 的转换。

> 注意这边的 `toList()` 其实是 `Collectors.toList()`，因为采用了静态倒入，看起来显得简洁。

```java
List<String> collected = Stream.of("a", "b", "c")
                               .collect(Collectors.toList());
assertEquals(Arrays.asList("a", "b", "c"), collected);
```

- map

如果有一个函数可以将一种类型的值转换成另外一种类型，`map` 操作就可以使用该函数，将一个流中的值转换成一个新的流。

```java
List<String> collected = Stream.of("a", "b", "hello")
                               .map(string -> string.toUpperCase())
                               .collect(toList());
assertEquals(asList("A", "B", "HELLO"), collected);
```

`map` 方法就是接受的一个 `Function` 的匿名函数类，进行的转换。

- filter

遍历数据并检查其中的元素时，可尝试使用 `Stream` 中提供的新方法 `filter`。

```java
List<String> beginningWithNumbers = 
        Stream.of("a", "1abc", "abc1")
              .filter(value -> isDigit(value.charAt(0)))
              .collect(toList());
assertEquals(asList("1abc"), beginningWithNumbers);
```

`filter` 方法就是接受的一个 `Predicate` 的匿名函数类，判断对象是否符合条件，符合条件的才保留下来。

- flatMap

`flatMap` 方法可用 `Stream` 替换值，然后将多个 `Stream` 连接成一个 `Stream`。

```java
List<Integer> together = Stream.of(asList(1, 2), asList(3, 4))
                               .flatMap(numbers -> numbers.stream())
                               .collect(toList());
assertEquals(asList(1, 2, 3, 4), together);
```

`flatMap` 最常用的操作就是合并多个 `Collection`。

- max和min

`Stream` 上常用的操作之一是求最大值和最小值。Stream API 中的 `max` 和 `min` 操作足以解决这一问题。

```java
List<Integer> list = Lists.newArrayList(3, 5, 2, 9, 1);
int maxInt = list.stream()
                 .max(Integer::compareTo)
                 .get();
int minInt = list.stream()
                 .min(Integer::compareTo)
                 .get();
assertEquals(maxInt, 9);
assertEquals(minInt, 1);
```

这里有 2 个要点需要注意：

1. `max` 和 `min` 方法返回的是一个 `Optional` 对象（对了，和 Google Guava 里的 Optional 对象是一样的）。`Optional` 对象封装的就是实际的值，可能为空，所以保险起见，可以先用 `isPresent()` 方法判断一下。`Optional` 的引入就是为了解决方法返回 `null` 的问题。
2. `Integer::compareTo` 也是属于 Java 8 引入的新特性，叫做 **方法引用（Method References）**。在这边，其实就是 `(int1, int2) -> int1.compareTo(int2)` 的简写，可以自己查阅了解，这里不再多做赘述。

- reduce

`reduce` 操作可以实现从一组值中生成一个值。在上述例子中用到的 `count`、`min` 和 `max` 方法,因为常用而被纳入标准库中。事实上，这些方法都是 `reduce` 操作。

```java
int result = Stream.of(1, 2, 3, 4)
                   .reduce(0, (acc, element) -> acc + element);
assertEquals(10, result);
```

注意 `reduce` 的第一个参数，这是一个初始值。`0 + 1 + 2 + 3 + 4 = 10`。

如果是累乘，则为：

```java
int result = Stream.of(1, 2, 3, 4)
                   .reduce(1, (acc, element) -> acc * element);
assertEquals(24, result);
```

因为任何数乘以 1 都为其自身嘛。`1 * 1 * 2 * 3 * 4 = 24`。

`Stream` 的方法还有很多，这里列出的几种都是比较常用的。`Stream` 还有很多通用方法，具体可以查阅 [Java 8 的 API 文档](https://docs.oracle.com/javase/8/docs/api/)。

### 数据并行化操作

`Stream` 的并行化也是 Java 8 的一大亮点。数据并行化是指将数据分成块，为每块数据分配单独的处理单元。这样可以充分利用多核 CPU 的优势。

并行化操作流只需改变一个方法调用。如果已经有一个 `Stream` 对象，调用它的 `parallel()` 方法就能让其拥有并行操作的能力。如果想从一个集合类创建一个流，调用 `parallelStream()` 就能立即获得一个拥有并行能力的流。

```java
int sumSize = Stream.of("Apple", "Banana", "Orange", "Pear")
                    .parallel()
                    .map(s -> s.length())
                    .reduce(Integer::sum)
                    .get();
assertEquals(sumSize, 21);
```

这里求的是一个字符串列表中各个字符串长度总和。

如果你去计算这段代码所花的时间，很可能比不加上 `parallel()` 方法花的时间更长。这是因为数据并行化会先对数据进行分块，然后对每块数据开辟线程进行运算，这些地方会花费额外的时间。并行化操作只有在 **数据规模比较大** 或者 **数据的处理时间比较长** 的时候才能体现出有事，所以并不是每个地方都需要让数据并行化，应该具体问题具体分析。

### 其他

- 收集器

`Stream` 转换为 `List` 是很常用的操作，其他 `Collectors` 还有很多方法，可以将 `Stream` 转换为 `Set`, 或者将数据分组并转换为 `Map`，并对数据进行处理。也可以指定转换为具体类型，如 `ArrayList`, `LinkedList` 或者 `HashMap`。甚至可以自定义 `Collectors`，编写自己的收集器。

[Collectors（收集器）](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html) 的内容太多，有兴趣的可以自己研究。

- 元素顺序

另外一个尚未提及的关于集合类的内容是流中的元素以何种顺序排列。一些集合类型中的元素是按顺序排列的，比如 List；而另一些则是无序的，比如 HashSet。增加了流操作后，顺序问题变得更加复杂。

总之记住。如果集合本身就是无序的，由此生成的流也是无序的。一些中间操作会产生顺序，比如对值做映射时，映射后的值是有序的，这种顺序就会保留 下来。如果进来的流是无序的，出去的流也是无序的。

如果我们需要对流中的数据进行排序，可以调用 `sorted` 方法：

```java
List<Integer> list = Lists.newArrayList(3, 5, 1, 10, 8);
List<Integer> sortedList = list.stream()
                               .sorted(Integer::compareTo)
                               .collect(Collectors.toList());
assertEquals(sortedList, Lists.newArrayList(1, 3, 5, 8, 10));
```

- @FunctionalInterface

我们讨论过函数接口定义的标准，但未提及 @FunctionalInterface 注释。事实上，每个用作函数接口的接口都应该添加这个注释。

但 Java 中有一些接口，虽然只含一个方法，但并不是为了使用 Lambda 表达式来实现的。比如，有些对象内部可能保存着某种状态，使用带有一个方法的接口可能纯属巧合。

该注释会强制 javac 检查一个接口是否符合函数接口的标准。如果该注释添加给一个枚举类型、类或另一个注释，或者接口包含不止一个抽象方法，javac 就会报错。重构代码时，使用它能很容易发现问题。

# 枚举

## 概念

`enum` 的全称为 enumeration， 是 JDK 1.5 中引入的新特性。

在Java中，被 `enum` 关键字修饰的类型就是枚举类型。形式如下：

```java
enum Color { RED, GREEN, BLUE }
```

如果枚举不添加任何方法，**枚举值默认为从0开始的有序数值**。以 Color 枚举类型举例，它的枚举常量依次为 `RED：0，GREEN：1，BLUE：2`。

**枚举的好处**：可以将常量组织起来，统一进行管理。

**枚举的典型应用场景**：错误码、状态机等。

**枚举类型的本质**

尽管 `enum` 看起来像是一种新的数据类型，事实上，**enum是一种受限制的类，并且具有自己的方法**。

创建enum时，编译器会为你生成一个相关的类，这个类继承自 `java.lang.Enum`。

`java.lang.Enum`类声明

```java
public abstract class Enum<E extends Enum<E>>
        implements Comparable<E>, Serializable { ... }
```

## 枚举的方法

在enum中，提供了一些基本方法：

`values()`：返回 enum 实例的数组，而且该数组中的元素严格保持在 enum 中声明时的顺序。

`name()`：返回实例名。

`ordinal()`：返回实例声明时的次序，从0开始。

`getDeclaringClass()`：返回实例所属的 enum 类型。

`equals()` ：判断是否为同一个对象。

可以使用 `==` 来比较`enum`实例。

此外，`java.lang.Enum`实现了`Comparable`和 `Serializable` 接口，所以也提供 `compareTo()` 方法。

**例：展示enum的基本方法**

```java
public class EnumMethodDemo {
    enum Color {RED, GREEN, BLUE;}
    enum Size {BIG, MIDDLE, SMALL;}
    public static void main(String args[]) {
        System.out.println("=========== Print all Color ===========");
        for (Color c : Color.values()) {
            System.out.println(c + " ordinal: " + c.ordinal());
        }
        System.out.println("=========== Print all Size ===========");
        for (Size s : Size.values()) {
            System.out.println(s + " ordinal: " + s.ordinal());
        }

        Color green = Color.GREEN;
        System.out.println("green name(): " + green.name());
        System.out.println("green getDeclaringClass(): " + green.getDeclaringClass());
        System.out.println("green hashCode(): " + green.hashCode());
        System.out.println("green compareTo Color.GREEN: " + green.compareTo(Color.GREEN));
        System.out.println("green equals Color.GREEN: " + green.equals(Color.GREEN));
        System.out.println("green equals Size.MIDDLE: " + green.equals(Size.MIDDLE));
        System.out.println("green equals 1: " + green.equals(1));
        System.out.format("green == Color.BLUE: %b\n", green == Color.BLUE);
    }
}
```

**输出**

```bash
=========== Print all Color ===========
RED ordinal: 0
GREEN ordinal: 1
BLUE ordinal: 2
=========== Print all Size ===========
BIG ordinal: 0
MIDDLE ordinal: 1
SMALL ordinal: 2
green name(): GREEN
green getDeclaringClass(): class org.zp.javase.enumeration.EnumDemo$Color
green hashCode(): 460141958
green compareTo Color.GREEN: 0
green equals Color.GREEN: true
green equals Size.MIDDLE: false
green equals 1: false
green == Color.BLUE: false
```

## 枚举的特性

枚举的特性，归结起来就是一句话：

> **除了不能继承，基本上可以将 `enum` 看做一个常规的类**。

但是这句话需要拆分去理解，让我们细细道来。

### 枚举可以添加方法

在概念章节提到了，**枚举值默认为从0开始的有序数值** 。那么问题来了：如何为枚举显示的赋值。

- Java 不允许使用 = 为枚举常量赋值

  如果你接触过C/C++，你肯定会很自然的想到赋值符号 `=` 。在C/C++语言中的enum，可以用赋值符号`=`显示的为枚举常量赋值；但是 ，很遗憾，**Java 语法中却不允许使用赋值符号 `=` 为枚举常量赋值**。

  **例：C/C++ 语言中的枚举声明**

  ```c++
  typedef enum{
      ONE = 1,
      TWO,
      THREE = 3,
      TEN = 10
  } Number;
  ```

- 枚举可以添加普通方法、静态方法、抽象方法、构造方法

  Java 虽然不能直接为实例赋值，但是它有更优秀的解决方案：**为 enum 添加方法来间接实现显示赋值**。

  创建 `enum` 时，可以为其添加多种方法，甚至可以为其添加构造方法。

  **注意一个细节：如果要为enum定义方法，那么必须在enum的最后一个实例尾部添加一个分号。此外，在enum中，必须先定义实例，不能将字段或方法定义在实例前面。否则，编译器会报错。**

  **例：全面展示如何在枚举中定义普通方法、静态方法、抽象方法、构造方法**

  ```java
  public enum ErrorCode {
      OK(0) {
          public String getDescription() {
              return "成功";
          }
      },
      ERROR_A(100) {
          public String getDescription() {
              return "错误A";
          }
      },
      ERROR_B(200) {
          public String getDescription() {
              return "错误B";
          }
      };
  
      private int code;
  
      // 构造方法：enum的构造方法只能被声明为private权限或不声明权限
      private ErrorCode(int number) { // 构造方法
          this.code = number;
      }
      public int getCode() { // 普通方法
          return code;
      } // 普通方法
      public abstract String getDescription(); // 抽象方法
      public static void main(String args[]) { // 静态方法
          for (ErrorCode s : ErrorCode.values()) {
              System.out.println("code: " + s.getCode() + ", description: " + s.getDescription());
          }
      }
  }
  ```

  注：上面的例子并不可取，仅仅是为了展示枚举支持定义各种方法。下面是一个简化的例子

  **例：一个错误码枚举类型的定义**

  本例和上例的执行结果完全相同。

  ```java
  public enum ErrorCodeEn {
      OK(0, "成功"),
      ERROR_A(100, "错误A"),
      ERROR_B(200, "错误B");
  
      ErrorCodeEn(int number, String description) {
          this.code = number;
          this.description = description;
      }
      private int code;
      private String description;
      public int getCode() {
          return code;
      }
      public String getDescription() {
          return description;
      }
      public static void main(String args[]) { // 静态方法
          for (ErrorCodeEn s : ErrorCodeEn.values()) {
              System.out.println("code: " + s.getCode() + ", description: " + s.getDescription());
          }
      }
  }
  ```

### 枚举可以实现接口

**`enum` 可以像一般类一样实现接口。**

同样是实现上一节中的错误码枚举类，通过实现接口，可以约束它的方法。

```java
public interface INumberEnum {
    int getCode();
    String getDescription();
}

public enum ErrorCodeEn2 implements INumberEnum {
    OK(0, "成功"),
    ERROR_A(100, "错误A"),
    ERROR_B(200, "错误B");

    ErrorCodeEn2(int number, String description) {
        this.code = number;
        this.description = description;
    }

    private int code;
    private String description;

    @Override
    public int getCode() {
        return code;
    }

    @Override
    public String getDescription() {
        return description;
    }
}
```

### 枚举不可以继承

**enum 不可以继承另外一个类，当然，也不能继承另一个 enum 。**

因为 `enum` 实际上都继承自 `java.lang.Enum` 类，而 Java 不支持多重继承，所以 `enum` 不能再继承其他类，当然也不能继承另一个 `enum`。

## 枚举的应用场景

### 组织常量

在JDK1.5 之前，在Java中定义常量都是`public static final TYPE a;` 这样的形式。有了枚举，你可以将有关联关系的常量组织起来，使代码更加易读、安全，并且还可以使用枚举提供的方法。

**枚举声明的格式**

**注：如果枚举中没有定义方法，也可以在最后一个实例后面加逗号、分号或什么都不加。**

下面三种声明方式是等价的：

```java
enum Color { RED, GREEN, BLUE }
enum Color { RED, GREEN, BLUE, }
enum Color { RED, GREEN, BLUE; }
```

### switch 状态机

我们经常使用switch语句来写状态机。JDK7以后，switch已经支持 `int`、`char`、`String`、`enum` 类型的参数。这几种类型的参数比较起来，使用枚举的switch代码更具有可读性。

```java
enum Signal {RED, YELLOW, GREEN}

public static String getTrafficInstruct(Signal signal) {
    String instruct = "信号灯故障";
    switch (signal) {
        case RED:
            instruct = "红灯停";
            break;
        case YELLOW:
            instruct = "黄灯请注意";
            break;
        case GREEN:
            instruct = "绿灯行";
            break;
        default:
            break;
    }
    return instruct;
}
```

### 组织枚举

可以将类型相近的枚举通过接口或类组织起来。

但是一般用接口方式进行组织。

原因是：Java接口在编译时会自动为enum类型加上`public static`修饰符；Java类在编译时会自动为 `enum` 类型加上static修饰符。看出差异了吗？没错，就是说，在类中组织 `enum`，如果你不给它修饰为 `public`，那么只能在本包中进行访问。

**例：在接口中组织 enum**

```java
public interface Plant {
    enum Vegetable implements INumberEnum {
        POTATO(0, "土豆"),
        TOMATO(0, "西红柿");

        Vegetable(int number, String description) {
            this.code = number;
            this.description = description;
        }

        private int code;
        private String description;

        @Override
        public int getCode() {
            return 0;
        }

        @Override
        public String getDescription() {
            return null;
        }
    }

    enum Fruit implements INumberEnum {
        APPLE(0, "苹果"),
        ORANGE(0, "桔子"),
        BANANA(0, "香蕉");

        Fruit(int number, String description) {
            this.code = number;
            this.description = description;
        }

        private int code;
        private String description;

        @Override
        public int getCode() {
            return 0;
        }

        @Override
        public String getDescription() {
            return null;
        }
    }
}
```

**例：在类中组织 enum**

本例和上例效果相同。

```java
public class Plant2 {
    public enum Vegetable implements INumberEnum {...}  // 省略代码
    public enum Fruit implements INumberEnum {...} // 省略代码
}
```

### 策略枚举

EffectiveJava中展示了一种策略枚举。这种枚举通过枚举嵌套枚举的方式，将枚举常量分类处理。

这种做法虽然没有switch语句简洁，但是更加安全、灵活。

**例：EffectvieJava中的策略枚举范例**

```java
enum PayrollDay {
    MONDAY(PayType.WEEKDAY), TUESDAY(PayType.WEEKDAY), WEDNESDAY(
            PayType.WEEKDAY), THURSDAY(PayType.WEEKDAY), FRIDAY(PayType.WEEKDAY), SATURDAY(
            PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    double pay(double hoursWorked, double payRate) {
        return payType.pay(hoursWorked, payRate);
    }

    // 策略枚举
    private enum PayType {
        WEEKDAY {
            double overtimePay(double hours, double payRate) {
                return hours <= HOURS_PER_SHIFT ? 0 : (hours - HOURS_PER_SHIFT)
                        * payRate / 2;
            }
        },
        WEEKEND {
            double overtimePay(double hours, double payRate) {
                return hours * payRate / 2;
            }
        };
        private static final int HOURS_PER_SHIFT = 8;

        abstract double overtimePay(double hrs, double payRate);

        double pay(double hoursWorked, double payRate) {
            double basePay = hoursWorked * payRate;
            return basePay + overtimePay(hoursWorked, payRate);
        }
    }
}
```

**测试**

```bash
System.out.println("时薪100的人在周五工作8小时的收入：" + PayrollDay.FRIDAY.pay(8.0, 100));
System.out.println("时薪100的人在周六工作8小时的收入：" + PayrollDay.SATURDAY.pay(8.0, 100));
```

## EnumSet和EnumMap

Java 中提供了两个方便操作enum的工具类——EnumSet 和 EnumMap。

`EnumSet` 是枚举类型的高性能 `Set` 实现。它要求放入它的枚举常量必须属于同一枚举类型。
`EnumMap` 是专门为枚举类型量身定做的 `Map` 实现。虽然使用其它的 Map 实现（如HashMap）也能完成枚举类型实例到值得映射，但是使用 EnumMap 会更加高效：它只能接收同一枚举类型的实例作为键值，并且由于枚举类型实例的数量相对固定并且有限，所以 EnumMap 使用数组来存放与枚举类型对应的值。这使得 EnumMap 的效率非常高。

```java
// EnumSet的使用
System.out.println("EnumSet展示");
EnumSet<ErrorCodeEn> errSet = EnumSet.allOf(ErrorCodeEn.class);
for (ErrorCodeEn e : errSet) {
    System.out.println(e.name() + " : " + e.ordinal());
}

// EnumMap的使用
System.out.println("EnumMap展示");
EnumMap<StateMachine.Signal, String> errMap = new EnumMap(StateMachine.Signal.class);
errMap.put(StateMachine.Signal.RED, "红灯");
errMap.put(StateMachine.Signal.YELLOW, "黄灯");
errMap.put(StateMachine.Signal.GREEN, "绿灯");
for (Iterator<Map.Entry<StateMachine.Signal, String>> iter = errMap.entrySet().iterator(); iter.hasNext();) {
    Map.Entry<StateMachine.Signal, String> entry = iter.next();
    System.out.println(entry.getKey().name() + " : " + entry.getValue());
}
```

# 泛型

## 泛型中的关键字

1. ? 表示通配符类型
2. <? extends T> 既然是extends，就是表示泛型参数类型的上界，说明参数的类型应该是T或者T的子类。
3. <? super T> 既然是super，表示的则是类型的下界，说明参数的类型应该是T类型的父类，一直到object。

- extends 可用于的返回类型限定，不能用于参数类型限定。
- super 可用于参数类型限定，不能用于返回类型限定。
- 希望只取出，不插入，就使用? extends
- 希望只插入，不取出，就使用? super
- 希望，即能插入，又能取出，就不要用通配符？

# 异常处理

## Java 异常类层次结构图

![image-20200414162954417](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200414162955-379202.png) 

1. 所有的异常都是从Throwable继承而来的，是所有异常的共同祖先。

2. Throwable有两个子类，Error和Exception。其中Error是错误，对于所有的编译时期的错误以及系统错误都是通过Error抛出的。这些错误表示故障发生于虚拟机自身、或者发生在虚拟机试图执行应用时，如Java虚拟机运行错误（Virtual MachineError）、类定义错误（NoClassDefFoundError）等。这些错误是不可查的，因为它们在应用程序的控制和处理能力之 外，而且绝大多数是程序运行时不允许出现的状况。对于设计合理的应用程序来说，即使确实发生了错误，本质上也不应该试图去处理它所引起的异常状况。在 Java中，错误通过Error的子类描述。

3. Exception，是另外一个非常重要的异常子类。它规定的异常是程序本身可以处理的异常。异常和错误的区别是，异常是可以被处理的，而错误是没法处理的。 

4. Checked Exception

   可检查的异常，这是编码时非常常用的，所有checked exception都是需要在代码中处理的。它们的发生是可以预测的，正常的一种情况，可以合理的处理。比如IOException，或者一些自定义的异常。除了RuntimeException及其子类以外，都是checked exception。

5. Unchecked Exception

   RuntimeException及其子类都是unchecked exception。比如NPE空指针异常，除数为0的算数异常ArithmeticException等等，这种异常是运行时发生，无法预先捕捉处理的。Error也是unchecked exception，也是无法预先处理的。

# hashCode 与 equals

hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个 int 整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode() 定义在 JDK 的 Object.java 中，这就意味着 Java 中的任何类都包含有 hashCode() 函数。

散列表存储的是键值对(key-value)，它的特点是：能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！（可以快速找到所需要的对象）

**我们先以“HashSet 如何检查重复”为例子来说明为什么要有 hashCode：** 当你把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与该位置其他已经加入的对象的 hashcode 值作比较，如果没有相符的 hashcode，HashSet 会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 `equals()`方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。（摘自我的 Java 启蒙书《Head first java》第二版）。这样我们就大大减少了 equals 的次数，相应就大大提高了执行速度。

通过我们可以看出：`hashCode()` 的作用就是**获取哈希码**，也称为散列码；它实际上是返回一个 int 整数。这个**哈希码的作用**是确定该对象在哈希表中的索引位置。**`hashCode()`在散列表中才有用，在其它情况下没用**。在散列表中 hashCode() 的作用是获取对象的散列码，进而确定该对象在散列表中的位置。

**hashCode（）与 equals（）的相关规定**：

1. 如果两个对象相等，则 hashcode 一定也是相同的
2. 两个对象相等,对两个对象分别调用 equals 方法都返回 true
3. 两个对象有相同的 hashcode 值，它们也不一定是相等的
4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）