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
```
#!scala
object Map {
  def apply(kv: Tuple*): Map = new HashMap(kv)
}
```
调用时:
```
#!scala
Map("China" -> "Beijing", "US" -> "Washington D.C")
```

### 第2条: 遇到多个构造器参数时要考虑用构造器

因为scala 支持 Named and Default Arguments, 相当于语言级支持了该条目内容,如:

```
#!scala
class Student(name: String, age: Int = 8, gender: Int = 0)
```

调用时:

```
#!scala
val xiaoming = new Student("Xiaoming", 8)
val leilei = new Student("Leilei", gender = 1)
```

### 第3条: 用私有构造器或者枚举类型强化Singlenton属性
### 第4条: 用私有构造器强化不可实例化的能力

这两条内容可以参考Singlenton设计模式.
在scala中, Singlenton是语言级支持的, 如:

```
#!scala
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
### 第9条: 覆盖equals时总要覆盖hashCode

这两条描述了java对象相等性的定义及机制.
在程序的世界里, 默认情况下, 两个对象实例相等, 意味着他们 指向同一个指针, 所以:
```
#!java
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
```
#!scala
case class Cat(name: String, age: Int)
```

如果反编译生成的class, 会发现该class, 已经override了equals, hashCode, toString

一般情况下, 使用case class 是最好的选择

#### 2. class
这种情况下, 和java class 是一致的,需要自己override equals, hashCode, 如:
```
#!scala
class Cat(name: String, age: Int) {
  
}
```

### 第10条: 始终要覆盖toString

### 第11条: 谨慎地覆盖clone

### 第12条: 考虑实现Comparable接口

