# 第4章 类和接口

## 第15条：使类和成员的可访问性最小化

这一条没什么，其实就是封装的概念，即：隐藏掉内部的细节，对外提供接口。通过 privite、default、protected、public 等关键字来控制访问的范围。

## 第16条：要在公有中使用访问方法而非公有域

这一点和第15条有所关联，简单的说就是，将类的成员变量私有化，对外提供 get、set 方法。

- 但是如果类是包级私有的，或者是私有的嵌套类，那么直接暴露出公有的数据域也没什么问题。

## 第17条：使可变性最小化

如果对象不可变，其实它就可以看作是线程安全的，被多个线程共享也没有关系。

如果类没办法设计成不可变的，那么也应该尽可能的限制它的可变性。

总而言之，坚决不要为每个 get 方法都编写一个对应的 set 方法，除非有着很好的理由要让类变成可变的，否则它就应该是不可变的。

**让类不可变的 5 条规则：**

1. 不要提供任何会修改对象状态的方法（即set设值方法）
2. 保证类不会被扩展。防止子类化、可以声明类为final。
3. 声明所有的域都是final的。
4. 声明所有的域都是私有的。
5. 确保对于任何可变组件的互斥访问。

## 第18条：复合优先于继承

[【设计模式】第一篇：概述、耦合、UML、七大原则，详细分析总结（基于Java）—— (七) 合成复用原则](https://juejin.cn/post/6887718131607797773#heading-34)

在设计模式中，存在七大设计原则，其中合成复用原则就是这一点：

**定义：在软件复用时，要尽量先使用组合或者聚合等关联关系来实现，其次才考虑使用继承关系来实现**

书中的例子就很能说明问题：

**示例需求：统计 Set 中曾经新增元素多少个**

```java
// Broken - Inappropriate use of inheritance! (Page 87)
public class InstrumentedHashSet<E> extends HashSet<E> {
    // The number of attempted element insertions
    private int addCount = 0;
 
    public InstrumentedHashSet() {
    }
 
    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }
 
    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
 
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
 
    public int getAddCount() {
        return addCount;
    }
 
    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("Snap", "Crackle", "Pop"));
        System.out.println(s.getAddCount());
    }
}
```

在主函数的测试中，通过 `List.of("Snap", "Crackle", "Pop")` 添加了三个元素（这个方法是在 Java 9 才有的，低一点的版本可以选择 Arrays.asList 方法代替），但是最终的 addCount 的结果并不是 3 却是 6。

查看 HashSet 的继承体系，发现在 AbstractCollection<E> 中 addAll 方法的定义就是循环调用了 add 方法，所以 addAll 和 add 方法中都进行了计数，所以重复了。

```java
public abstract class AbstractCollection<E> implements Collection<E> {
    ...
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
    ...
}
```

**解决方案如下：**

```java
// Reusable forwarding class (Page 90)
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }
 
    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
 
// Wrapper class - uses composition in place of inheritance  (Page 90)
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
 
    public InstrumentedSet(Set<E> s) {
        super(s);
    }
 
    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }
 
    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("Snap", "Crackle", "Pop"));
        System.out.println(s.getAddCount());
    }
}
```

可以看到，我们在原先的 `InstrumentedHashSet<E>` 类 和 `HashSet<E> `类之间，加了一个新的 `ForwardingSet<E> ` 类。

- `InstrumentedHashSet<E>`  继承 `ForwardingSet<E> `  ， `ForwardingSet<E> `   再去实现 `Set<E>`

其实复合的这个特点，并不体现在这个新的继承关系上，而是在 `ForwardingSet<E> `  中，没有继承 `AbstractCollection<E>` 而是在自己的成员变量中定义了`private final Set<E> s;` ，所以在 `InstrumentedHashSet<E>` 中调用 add、addAll 方法的时候，是使用自己的成员 `s`，调用 addAll 的时候，依旧是循环调用 `AbstractCollection<E>` 中的 add 方法，但是却不会再进入到 `InstrumentedSet<E>` 中定义的add方法。

- 这种方式也叫做转发。
- 其实完全可以在自己的这个 `InstrumentedSet<E>` 类中进行 复合，但是书中引入了一个新类 `ForwardingSet<E>` d的意义在于，如果以后你还想增加一个关于 set 的新方法，你就可以直接继承 ForwardingSet 就好了，不必再把 set 原有的方法再新类的方法再转发实现一次。
- 这种设计手法就是装饰器模式。
- 但是这种手法不适合回调框架（需要将自身的引用传递给其他对象）



## 第19条：要么设计继承并提供文档说明，要么禁止继承（待修改）

- 对于专门为了继承而设计的类, 需要具有良好的文档

- 类必须通过某种形式提供适当的钩子（hook）
- 对于为了继承而设计的类，唯一的测试方法就是编写子类
- 为了允许继承，类还必须遵守其他的一些约束
- 对于那些并非为了安全地进行子类化而设计和编写文档的类, 要禁止子类化.

## 第20条：接口优于抽象类

[Java-Ideal-Interview —— 3.3.2 抽象类和接口的区别（重要）](https://github.com/ideal-20/Java-Ideal-Interview/blob/main/docs/java/javase-basis/002-Java面向对象.md#332-抽象类和接口的区别重要)

## 第21条：为后代设计接口

如果针对现存的接口进行功能扩展

- Java8 之前：把所有实现了这个接口的类重新实现新的方法，但是如果这个接口是对外提供的，就会牵扯很广，很麻烦。
- Java8 之后：8中接口的新特性，允许接口拥有了默认的缺省实现，之前实现这个接口的子类就可以用这个默认实现。

> 相比之前要相对简单一点，可以考虑采用java8接口的新特性，缺省实现来完成扩展。但是需要注意的是，就算接口拥有缺省实现了，也是一件拥有风险的事情。
>
> 例如，java.util.Collection<E>中新增了removeIf，用于函数式编程。但是该书撰写直到出版时，貌似org.apache.commons.collections4.Collection.SynchronizedCollection中，这个类来自Apache Commons类库，但是并没有针对removeIf做同步锁的实装。这样一来，如果在那个版本使用了removeIf，由于java.util.Collection<E>中的removeIf并没有加锁控制删除，如果另一个线程也同时操作了该集合，就会导致ConcurrentModificationException。而这由于是缺省方法来拓展的，编译并不会报错，只有到了真正运行时才会发现。
>
> [csdn@微笑小个子](https://blog.csdn.net/weixin_44453385/article/details/117962790?spm=1001.2014.3001.5501)

## 第22条：接口只用于定义类型

接口只用定义类型，也就是说，其他任何目的而顶定义接口都是不恰当的。

例如书中提到的：用接口定义常量，其实这是一种不良的设计。

- 暴露了实现细节到该类的导出API中。
- 实现常量接口对于类的用户来说没有价值。
- 如果以后的发行版本中不需要其中的常量了, 依然必须实现这个接口。
- 所有子类的命名空间也会被接口中的常量污染。

## 第23条：类层次优先于标签类

```java
// Tagged class - vastly inferior to a class hierarchy! (Page 109)
class Figure {
    enum Shape { RECTANGLE, CIRCLE };
 
    // Tag field - the shape of this figure
    final Shape shape;
 
    // These fields are used only if shape is RECTANGLE
    double length;
    double width;
 
    // This field is used only if shape is CIRCLE
    double radius;
 
    // Constructor for circle
    // 圆
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }
 
    // Constructor for rectangle
    // 矩形
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }
 
    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

- Figure类 内含 Shape 枚举, 包含圆形和方形，通过构造函数来区分形状，如果再增加其他形状，就会变得很乱。这也就是标签类的形式。

```java
// Class hierarchy replacement for a tagged class  (Page 110-11)
abstract class Figure {
    abstract double area();
}
 
// Class hierarchy replacement for a tagged class  (Page 110-11)
// 圆
class Circle extends Figure {
    final double radius;
 
    Circle(double radius) { this.radius = radius; }
 
    @Override double area() { return Math.PI * (radius * radius); }
}
 
// Class hierarchy replacement for a tagged class  (Page 110-11)
// 矩形
class Rectangle extends Figure {
    final double length;
    final double width;
 
    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}
```

- 如果使用具有层次结构的继承，就会更加清晰，后期扩展也很舒服。

```java
// Class hierarchy replacement for a tagged class  (Page 110-11)
// 正方形
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```

## 第24条：优先考虑静态成员类

- 类、类变量、类方法等均优先使用静态

## 第25条：限制源文件为单个顶级类

源文件：自己平时开发的 Java 文件

这一点也就是为什么我们从一开始学习 Java 的时候就提到，一个 Java 文件中只能有一个文件名和类名一致的类。



