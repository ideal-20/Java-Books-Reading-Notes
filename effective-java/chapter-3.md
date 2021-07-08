# 第3章：对于所有对象都通用的方法

尽管 Object 是一个具体的类，但是它设计的主要目的还是为了扩展它所有非 final 的方法（equals、hashCode、toStrin、clone 和 finalize）任何一个类在覆盖这些方法的时候，都有责任要遵守一些通用的规定，如果做不到这一点，那么就没办法和其他遵守这些约定的类一起工作，例如（HashMap 和 HashSet）

finalize 在第2章已经讨论过了，所以不再叙述，而 Comparable.compareTo 因为具有相似的特征，所以也在此进行讨论。

## 第10条：覆盖 equals 时请遵守通用约定

equals 主要围绕的问题就是：比值还是比地址，如果我们不重写 equals 方法，最后都会根据 Object 类中所定义的那般（`return (this == obj);`），去比较地址。如果想要比较值，就需要覆盖（重写）equals 方法

### 1. 什么时候不需要覆盖 equals

- **类的每个实例本质上都是唯一的。**
  - 对于代表活动实体而不是值的类来说（例如：Thread）Object 提供的 equals 实现对于这些类来说是正确的行为。
- **类没有必要提供“逻辑相等”功能。**
  - 例如 java.util.regex.Pattern 可以覆盖 equals，用来检查两个 Pattern 实例是否一致，但是设计者认为并没有这种必要。
- **超类（父类）已经覆盖了equals，超类的行为对于这个类也是合适的。**
- **类是私有的，或者是包级私有的，可以确定它的equals方法永远不会被调用。**

### 2. 什么时候需要覆盖 equals

说起来很简单：就是比值的时候

详细来说就是：类之间具有 "逻辑相等" 的概念，这与 “对象相等“ 是有区别的，并且超类还没有覆盖 equals ，这种类型一般可以叫做 “值类”，也就是一个表示值的类，例如 Integer 和 String ，程序员更想知道两个对象在值的上面是否相同，这也就是逻辑上的相同，而不是它们是否指向到同一个对象。

### 3. 覆盖 equals 需要遵守的约定

- **自反性**：对于任何非 null 的引用值 x，x.equals(x) 必须返回 true。
- **对称性**：对于任何非 null 的引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 必须返回 true。
- **传递性**：对于任何非 null 的引用值 x、y 和 z，如果 x.equals(y) 返回 true，并且 y.equals(z) 也返回 true，那么 x.equals(z) 也必须返回 true。
- **一致性**：对于任何非 null 的引用值  x 和 y，只要 equals 的比较操作在对象中所用的信息没有被修改，多次调用 x.equals(y) 就会一致的返回 true，或者一致的返回 false。
- **非空型**：对于任何非 null 的引用值 x，x.equals(null) 必须返回 false。

#### 3.1 实现较高质量 equals 方法的诀窍

- 使用`==`操作符检查参数是否为这个对象的引用，如果是，则返回true。
- 使用`instanceof`操作符检查参数是否为正确的类型，如果不是，则返回 false。
- 把参数转换成正确的类型。
- 对于该类中的每个关键域，检查参数中的域是否与该对象中对应的域相匹配。
- 当你编写完成了`equals`方法之后，应该问自己三个问题：它是否满足对称性、传递性、一致性（其他两个特性通常会自动满足）。

#### 3.2 必须实现 equals 时的告诫

- 覆盖 equals 时总要覆盖 hashCode（下一条会提到）。
- 不要企图让equals方法过于智能。
- 不要将 equals 申明中的 Object 对象替换成其他的类型（不然就不是重写 Object 中的 equals 方法）。
- 如果有可能，建议用 Lombok 的 @EqualsAndHashCode(callSuper=true) 直接实现。

## 第11条：覆盖equals时总要覆盖hashCode

选自（个人原创）：[Java-Idea-Interview|003-Java常见对象](https://github.com/ideal-20/Java-Ideal-Interview/blob/main/docs/java/javase-basis/003-Java%E5%B8%B8%E8%A7%81%E5%AF%B9%E8%B1%A1.md#25-%E4%B8%BA%E4%BB%80%E4%B9%88%E9%87%8D%E5%86%99-tostring-%E6%96%B9%E6%B3%95)

如果重写了 equals() 而未重写 hashcode() 方法，可能就会出现两个字面数据相同的对象（例如下面 stu1 和 stu2） equals 相同（因为 equals 都是根据对象的特征进行重写的），但 hashcode 不相同的情况。

```java
public class Student {
    private String name;
    public int age;
    // get set ... 
    // 重写 equals() 不重写 hashcode()
}
--------------------------------------------
Student stu1 = new Student("BWH_Steven",22);
Student stu2 = new Student("BWH_Steven",22);
--------------------------------------------
stu1.equals(stu2); // true
stu1.hashCode();  // 和 stu2.hashCode(); 结果不一致
stu2.hashCode();
```

如果把对象保存到 HashTable、HashMap、HashSet 等中（不允许重复），这种情况下，去查找的时候，由于都是先使用 hashCode() 去对比，如果返回的 hashCode 不同，则会认为对象不同。可以存储，从内容上看，明显就重复了。

> 所以一般的地方不需要重写 hashcode() ，只有当类需要放在 HashTable、HashMap、HashSet 等hash 结构的集合时才会去重写。

> **补充：阿里巴巴 Java 开发手册关于 hashCode 和 equals 的处理遵循规则：**
>
> - 只要重写 equals，就必须重写 hashCode。
> - 因为 Set 存储的是不重复的对象，依据 hashCode 和 equals 进行判断，所以 Set 存储的对象必须重写这两个方法。
> - 如果自定义对象做为 Map 的键，那么必须重写 hashCode 和 equals。
> - String 重写了 hashCode 和 equals 方法，所以我们可以非常愉快地使用 String 对象作为 key 来使用。

## 第12条：始终要覆盖toString

选自（个人原创）：[Java-Idea-Interview|003-Java常见对象](https://github.com/ideal-20/Java-Ideal-Interview/blob/main/docs/java/javase-basis/003-Java%E5%B8%B8%E8%A7%81%E5%AF%B9%E8%B1%A1.md#25-%E4%B8%BA%E4%BB%80%E4%B9%88%E9%87%8D%E5%86%99-tostring-%E6%96%B9%E6%B3%95)

主要目的还是为了简化输出

1. 在类中重写toString()后，输出类对象就变得有了意义（输出s 和 s.toString()是一样的 ，不写也会默认调用），变成了我们实实在在的信息 ，例如 Student{name='admin', age=20}，而不是上面的 cn.ideal.pojo.Student@1b6d3586
2. 如果我们想要多次输出 类中的成员信息，就需要多次书写 get 方法（每用一次就得写）

> toString() 方法，返回该对象的字符串表示。
>
> `Object` 类的 `toString` 方法返回一个字符串，该字符串由类名（对象是该类的一个实例）at 标记符 `@` 和此对象哈希码的无符号十六进制表示组成。换句话说，该方法返回一个字符串，它的值等于：
>
> 代码：`getClass().getName()+ '@' + Integer.toHexString(hashCode())`
>
> 通常我们希望， `toString` 方法会返回一个“以文本方式表示” 此对象的字符串。结果应是一个简明但易于读懂的信息表达式。因此建议所有子类都重写此方法。

## 第13条：谨慎地覆盖 clone（待修改）

Cloneable 接口的目的就是为了表示这个对象是允许克隆的，但是这个接口没能成功达成这个目的，主要就是因为它缺少了一个 clone 方法，而 Object的clone方法是受保护的。如果不借助反射，就不能仅仅因为一个对象实现了Colneable就可以钓鱼clone方法，即使是反射调用也不能保证这个对象一定具有可访问clone方法

既然Cloneable并没有包含任何方法，那么它到底有什么用呢？它其实觉得了Object中受保护的clone方法实现的行为，如果一个类实现了Cloneable那么Object的clone方法就返回该对象的逐域拷贝，否则会抛出CloneNotSupportedException。但真说接口一种极端非典型用法，不值得提倡。

如果实现Cloneable接口是要对某个类起到作用，类和它的所有超类都必须遵守一个一定协议，言外之意就是无需调用构造器就可以创建对象。

——停止，待修改

## 第14条：考虑实现Comparable接口

compareTo 方法是 Comparable 接口中的唯一方法，主要用在比较数字类型的值，与方法参数中的值是否相等，与 equals 相同的是，它也满足自反性、对称性和传递性。

第二版中的描述是：

>比较整数型基本类型的域, 可以用关系操作符`<`和`>`. 
>浮点域用`Double.compare`或`Float.compare`. (浮点值没有遵守compareTo的通用约定.)

而 Jdk 7开始，所有的基本类型的装箱类型都提供了静态的 compare 方法，所以不再建议使用 `<` 或 `>` 这样的关系符，尽可能的使用 Double.compare、Float.compare 这种装箱类型去比较基本类型

同时，不推荐利用数值相减的结果与0进行判断，因为十分有可能造成整数溢出。建议使用下面推荐的两种方法

```java
// 不推荐，容易造成整数溢出
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
}

// 推荐方法1
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
}
// 推荐方法2
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```









