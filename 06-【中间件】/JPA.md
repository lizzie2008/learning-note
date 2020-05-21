# 继承实体的映射策略

注：这里所说的实体指的是`@Entity`注解的类
继承映射使用`@Inheritance`来注解，它的strategy属性的取值由枚举`InheritanceType`来定义（包含`SINGLE_TABLE`、`TABLE_PER_CLASS`、`JOINED`，分别相应三种继承策略）。`@Inheritance`注解仅仅能作用于继承结构的超类上。假设不指定继承策略，默认使用SINGLE_TABLE。

JPA提供了三种继承映射策略：
- 一个类继承结构一个表的策略。这是继承映射的默认策略。即假设实体类B继承实体类A，实体类C也继承自实体A，那么仅仅会映射成一个表，这个表中包含了实体类A、B、C中全部的字段。JPA使用一个叫做“discriminator列”来区分某一行数据是应该映射成哪个实体。注解为：`@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`
- 联合子类策略。这样的情况下子类的字段被映射到各自的表中，这些字段包含父类中的字段。并运行一个join操作来实例化子类。注解为：`@Inheritance(strategy = InheritanceType.JOINED)`
- 每一个详细的类一个表的策略。注解为：`@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`