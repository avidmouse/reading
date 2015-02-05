# Effective Java 中文版（第2版）

------

## 第一章 引言

多年后重读此书, 依旧收获颇丰, 故做笔记于此, 以便加深理解和记忆.
并从scala的角度, 去观察这些条目.

------

## 第二章 创建和销毁对象

### 第1条: 考虑用静态工厂方法替代构造器

优势:
> * 有名称
> * 不必每次调用时创建一个新对象
> * 可以返回返回类型的任意子类型
> * 创建泛型实例时,代码更简洁(java 7中已添加泛型实例创建时的类型推断)

缺点:
> * 如果没有可继承的构造器, 不能被子类化
> * 于其他静态方法没有区别

如果使用scala, 因为支持apply, 可以很好的避免这两个缺点, 如:
```scala
object Map {
  def apply(kv: Tuple*): Map = new HashMap(kv)
}
```
调用时:
```scala
Map("China" -> "Beijing", "US" -> "Washington D.C")
```

### 第2条: 遇到多个构造器参数时要考虑用构造器

因为scala 支持 Named and Default Arguments, 相当于语言级支持了该条目内容,如:

```scala
class Student(name: String, age: Int = 8, gender: Int = 0)
```

调用时:

```scala
val xiaoming = new Student("Xiaoming", 8)
val leilei = new Student("Leilei", gender = 1)
```

### 第3条: 用私有构造器或者枚举类型强化Singlenton属性
### 第4条: 用私有构造器强化不可实例化的能力

这两条内容可以参考Singlenton设计模式.
在scala中, Singlenton是语言级支持的, 如:

```scala
object Elvis {
  def leaveTheBuilding()
}
```

### 第5条: 避免创建不必要的对象

这条是一个通用的原则, 即: 程序设计时, 要考虑对象共用性, 尽量避免创建不必要的对象;
但, 不是说创建对象就是坏事, 就应该避免;
总结: 要考虑性能, 避免一些浅显的错误即可, 但不要太考虑性能, 不要过早优化;

### 第6条: 消除过期的对象引用

如果理解了java GC的运作机制, 理解该条目便顺理成章了.
一般而言, 如果类自己管理内存并且自己的生命周期贯穿整个进程, 则要自己负责消除过期的对象引用,  如缓存.

### 第7条: 避免使用终结方法

finalize在实际应用中, 基本不会涉及.

------

## 第三章 对于所有对象都通用的方法

### 第8条: 覆盖equals时请遵守通用约定

约定如下:
> * 自反性(reflexive): 非 null 引用值[x], `x.equals(x) = true`;
> * 对称性(symmetric): 非 null 引用值[x, y], `if (y.equals(x)) x.equals(y) = true`;
> * 传递性(transitive): 非 null 引用值[x, y, z], `if (x.equals(y) && y.equals(z)) x.equals(z)`;
> * 一致性(consistent): 非 null 引用值[x, y], 如果未修改, `x.equals(y)` 的返回值永远一致;

需要主意的陷阱:

> - 定义equals方法时采用了错误的方法签名;
> - 修改equals方法但没有同时修改hashCode;
> - 用可变字段定义 equals方法(这条来自[programming in scala]);
> - 未能按同关系定义equals 方法;

### 第9条: 覆盖equals时总要覆盖hashCode

hashCode的约定是:
> * 对象equals所用到的信息未变, hashCode也应该不变;
> * 对象[x, y], if (x.equals(y)) x.hashCode == y.hashcode;
> * 对象[x, y], if (!x.equals(y)) x.hashCode == y.hashcode 或可以不等, 但不等的话, 有可能提高hash table 性能;

这两条描述了java对象相等性的定义及机制.
在程序的世界里, 默认情况下是物理相等, 所以 Object 的定义是:
```java
public class Object {
    public boolean equals(Object obj) {
        return (this == obj);
    }
}
```

当我们要的是逻辑相等时, 就需要告诉程序两个实例什么情况下才是相等的, 这个看似简单, 但实现时, 还是有许多规则的.

在scala中,有两种情况,
#### 1. case class

当定义class为case class 时, 编译器会帮我们override equals, hashCode, toString, 如:

```scala
case class Cat(name: String, age: Int)
```

如果反编译生成的class, 会发现该class, 已经override了equals, hashCode, toString

一般情况下, 使用case class 是最好的选择

#### 2. class

这种情况下, 和java class 是一致的,需要自己override equals, hashCode, 如:

```scala
class Cat(name: String, age: Int) {
  override def equals(other: Any): Boolean = 
    other match {
      case that: Cat =>
        (that canEqual this) &&
        name == that.name &&
        age == that.age
      case _ => false
    }
  def canEqual(other: Any): Boolean = other.isInstanceOf[Cat]
  
  override def hashCode: Int =
    41 * (
      41 +  name.hashCode
    ) + age
    
  override def toString = s"Cat($name, $age)"
}
```

### 第10条: 始终要覆盖toString

本条是一个良好的约定吧, 与人方便, 自己方便.

### 第11条: 谨慎地覆盖clone

还是使用拷贝工厂吧.
对于scala, 使用case class提供的copy方法吧.


### 第12条: 考虑实现Comparable接口

如果对象有明显的大小关系, 实现Comparable, 否则, 排序时传递Comparator更合适,  尤其是java8支持lambda后,写起来更方便了.

```java
//java
Collections.sort(names, (a, b) -> a.compareTo(b));
```

scala呢?

```scala
//scala
names.sortWith((a, b) -> a > b)
```

------

## 第四章 类和接口

### 第13条: 使类和成员的可访问性最小化

API的基本设计原则, 但一定要在方便性的基础上.

### 第14条: 在公有类中使用访问方法而非公有域

scala提供的getter setter,可以很方便的修改field的访问方式, 如:

```scala
class Point(var x: Int, var y: Int)
```

当要添加约束时, 可以修改为:

```scala
class Point(xv: Int, yv: Int) {
  x = xy
  var _x: Int = _

  def x = _x

  def x_=(v: Int) {
    require(v > 0)
    _x = v
  }
}
```

### 第15条: 使可变性最小化

对于scala, 该条目就是: 尽量使用case class

### 第16条: 复合优先于继承

这是架构的通用原则, 应该适用于所有面向对象语言吧!

### 第17条: 要么为继承而设计, 并提供文档说明, 要么就禁止继承

每个人的行为都是自主的, 不可预期的, 所以不要期望别人能完全理解你的意图, 并按你的约定扩展类.

### 第18条: 接口优于抽象类

当java8 引入default 关键字, 可在接口定义默认实现时, 这条更无可厚非了.

### 第19条: 接口只用于定义类型

常量还是放在相应的object里或enum里吧.

### 第20条: 类层次优于标签类

如果代码中充斥着样板代码, 别忘了面向对象.

### 第21条: 用函数对象表示策略

java8中已经添加lambda支持, 拥抱函数式编程吧.

### 第22条: 优先考虑静态成员类


------

## 第五章 泛型

### 第23条: 请不用在新代码中使用原生态类型

原生态类型只是为了兼容, 所以, 不要开历史的倒车了.
scala 是完全没有原生态类型的, 没有历史包袱感觉真好.

### 第24条: 消除非受检警告

认真对待编译器警告.

### 第25条: 列表优先于数组

scala中, 数组也是对象, 不存在协变(covariant)问题.

### 第26条: 优先考虑泛型

### 第27条: 优先考虑泛型方法

### 第28条: 利用有限制通配符来提升API的灵活性

### 第29条: 优先考虑类型安全的异构容器

------

## 第六章 枚举和注解


