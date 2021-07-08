# 第2章 创建和销毁对象

这一章的主题是创建和销毁对象，何时以及如何创建｜避免创建｜销毁对象，以及销毁前的各种清理动作。

## 第1条：考虑用静态工厂方法代替构造器

对于类而言，最常用的获取实例的方法有两种：

1. 提供一个公有的构造器（new 出来）
2. **提供一个公有的静态工厂方法 (static factory method)，返回类的实例。（通过工厂实例化）**

### 1. 公有构造器方式的缺点

1. 只能通过 new className() 的方式实例化对象
2. 每次都会返回一个新的对象
3. 返回类型就是该类的类型

### 2. 静态工厂方法的优点

#### 2.1 静态工厂方法能够很好的解决公有构造器带来的问题

静态工厂的方法有名称

- 构造器有时候不能很好的描述正被返回的对象，因此具有适当名称的静态工厂会更适合。
- 当一个类需要多个带有相同签名的构造器的时候，就使用静态工厂方法来代替构造器，使用不同的方法名称来突出它们的区别，例如 `BigInteger.probalePrime()` 这个静态工厂方法就能很好的向用户描述，当前返回的实例是一个素数。

示例说明：

```java
BigInteger(String)
BigInteger(int, Random)
BigInteger(int, int, Random)
```

- 如果只看到这三个方法的名字，不查看源码或者注释，其实很难区分三者的入参内容，例如 ` BigInteger(int, int, Random)` 可能回返回素数，通过 `BigInteger.probalePrime()` 就会好很多。

#### 2.2 不必在每次调用它们的时候都创建一个新的对象

- 如果经常需要创建相同的对象，同时创建的消耗和代价又比较大，可以通过静态工厂方法将对象缓存起来，达到重复利用的目的，每次调用工厂方法时返回同一对象。

```java
public static final Boolean TRUE = new Boolean(true);
public static final Boolean FALSE = new Boolean(false);
 
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

#### 2.3 可以返回原返回类型的任何子类型（待解决）

#### 2.4 静态工厂方法可以根据方法的参数值，返回不同的对象类

例如：EnumSet.class中，`noneOf(Class<E>)` 方法，如果 universe.length<=64，返回 RegularEnumSet，其他则返回 JumboEnumSet

- 这也就是说明：这两个实现类对于客户端来说是不可见的，客户端永远不知道也不关心它们从工厂方法中的到的对象，它们只关心它们是 EnumSet 的某个子类 

#### 2.5 静态工厂方法返回的对象所属的类，在编写包含静态工厂方法时可以不存在（待解决）

### 3. 静态工厂方法的缺点

##### 3.1 类如果不含有公有的或者受保护的构造器，就不能被子类化

- 但是这也有好处，鼓励程序员使用复合而不是继承

##### 3.2 **如果不看源码，很难发现哪些方法是静态工厂实现的**

- 不过如果遵循标准的命名规范可以进行一定的弥补

```java
// from：类型转换方法，单个参数，返回该类型的一个相应的实例
Date d = Date.from(instant);
// of：聚合方法，带有多个参数，返回该类型的一个实例，并把它们合并起来
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
// valueOf：比 from 和 of 更繁琐的一种替换方法
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
// instance或getInstance：返回的实例是通过方法参数来描述的，但是不能说与参数具有同样的值
StackWalker luke = StackWalker.getInstance(options);
// create或者newInstance ：类似 instance或getInstance 但是能保证每次调用都返回一个新的实例
Object newArray = Array.newInstance(classObject, arrayLen);
// getType：类似 getInstance 但是在工厂方法处于不同的类中的时候使用，type 表示工厂方法所返回的对象类型
FileStore fs = Files.getFileStore(path);
// newType：类似 newInstance 但是在工厂方法处于不同的类中的时候使用，type 表示工厂方法所返回的对象类型
BufferedReader br = Files.newBufferedReader(path);
// type：getType 和 newType 的简洁版
List<Complaint> litany = Collections.list(legacyLitany);
```

## 第2条：遇到多个构造器参数时要考虑使用构造器

静态工厂和构造器有一个共同的局限性，即：它们都不能很好地扩展到大量的可选参数

- 也就是说，在一个类有很多参数的时候，一些可能是必须项，还有一些只是可选项，但是不同的实例可能在可选项中也会有非 null 非 0 的值，对于这种场景，处理方式有如下几种

### 1. 利用构造函数生成对象

```java
// Telescoping constructor pattern - does not scale well! (Pages 10-11)
public class NutritionFacts {
    private final int servingSize;  // (mL)            required
    private final int servings;     // (per container) required
    private final int calories;     // (per serving)   optional
    private final int fat;          // (g/serving)     optional
    private final int sodium;       // (mg/serving)    optional
    private final int carbohydrate; // (g/serving)     optional
 
    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }
 
    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }
 
    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }
 
    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }
 
    public static void main(String[] args) {
        NutritionFacts cocaCola =
                new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
}
```

- 这种方法包含一个含有所有必须属性的最短构造器。
- 针对非必须属性，就需要依次给出各种可能的情况，例如排列组合一样。
- 如果给出的选择没有合适的构造器，就只能选择包含有所需属性的构造器，但是就会有多余的部分用不到，只能被迫赋一个 null 或 0 属实不是很好。

### 2. 利用 JavaBean 实现

```java
// JavaBeans Pattern - allows inconsistency, mandates mutability  (pages 11-12)
public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize  = -1; // Required; no default value
    private int servings     = -1; // Required; no default value
    private int calories     = 0;
    private int fat          = 0;
    private int sodium       = 0;
    private int carbohydrate = 0;
 
    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)     { servings = val; }
    public void setCalories(int val)     { calories = val; }
    public void setFat(int val)          { fat = val; }
    public void setSodium(int val)       { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
 
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```

- 这种方式其实是最常用的实现方式，但是这种模式有个非常严重的问题，即：JavaBean 可能处于不一致的状态。
- 简单的说就是，对象是被先一步创建出来的，但是需要不断的赋值才能达到最终的结果，不像构造器属于一步到位，可能出现的问题就是，如果你的属性赋值还没有结束，已经有别的程序或者其他线程在使用这个对象了。所以就把类的不可变的可能性变得不复存在。这就需要程序员付出额外的努力去保证线程安全。

### 3. 建造者模式

```java
// Builder Pattern  (Page 13)
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
 
    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;
 
        // Optional parameters - initialized to default values
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;
 
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }
 
        public Builder calories(int val)
        { calories = val;      return this; }
        public Builder fat(int val)
        { fat = val;           return this; }
        public Builder sodium(int val)
        { sodium = val;        return this; }
        public Builder carbohydrate(int val)
        { carbohydrate = val;  return this; }
 
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
 
    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
 
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
}
```

- 通过调用的方式和实体类来看，实体类不再直接提供 JavaBean 了，而是利用一个内部静态类，将赋值的部分独立了出来，这样这个实体类对象实例化的过程就不能被拆分了，而且 builder 的赋值方法会返回到 builder 本身，形成了一个流式的调用链。

同样在类的层次结构中，建造者模式也是适用的

```java
// Builder pattern for class hierarchies (Page 14)
 
// Note that the underlying "simulated self-type" idiom  allows for arbitrary fluid hierarchies, not just builders
 
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    // HAM, MUSHROOM, ONION, PEPPER, SAUSAGE  火腿，蘑菇，洋葱，胡椒，香肠
    final Set<Topping> toppings;
 
    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }
 
        abstract Pizza build();
 
        // Subclasses must override this method to return "this"
        protected abstract T self();
    }
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // See Item 50
    }
}
 
// Subclass with hierarchical builder (Page 15)
// 纽约披萨，可以指定尺寸
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;
 
    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;
 
        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }
 
        @Override public NyPizza build() {
            return new NyPizza(this);
        }
 
        @Override protected Builder self() { return this; }
    }
 
    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
 
    @Override public String toString() {
        return "New York Pizza with " + toppings;
    }
}
 
// Subclass with hierarchical builder (Page 15)
// 卡尔佐内披萨，可以指定馅料是内置还是外置
public class Calzone extends Pizza {
    private final boolean sauceInside;
 
    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // Default
 
        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }
 
        @Override public Calzone build() {
            return new Calzone(this);
        }
 
        @Override protected Builder self() { return this; }
    }
 
    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
 
    @Override public String toString() {
        return String.format("Calzone with %s and sauce on the %s",
                toppings, sauceInside ? "inside" : "outside");
    }
}
 
// Using the hierarchical builder (Page 16)
public class PizzaTest {
    public static void main(String[] args) {
        NyPizza pizza = new NyPizza.Builder(SMALL)
                .addTopping(SAUSAGE).addTopping(ONION).build();
        Calzone calzone = new Calzone.Builder()
                .addTopping(HAM).sauceInside().build();
        
        System.out.println(pizza);
        System.out.println(calzone);
    }
}
```

- 父类中 addTopping 调用了抽象方法 self，这样就可以让 NyPizza、Calzone 子类实现返回各自类型的 Builder（即：NyPizz.Builder 返回 NyPizz 类型， Calzone.Builder 返回 Calzone 类型）。build方法可以构建各自子类类型：子类方法声明返回超级类中声明的返回类型的子类型，这被称为协变返回类型。它允许客户端无须转换类型就能使用这些构建器。

但是建造者模式也有它的几个缺点：

- 创建对象之前，必须创建它的构造器，如果在性能至上的情况下，其实就会有一定的影响
- 相比较于 JavaBean 来说，代码量明显变多了，在开发人员追求进度的情况下，可能大概率不会被选择。

## 第3条：用私有构造器或者枚举类型强化Singleton属性

本身这一点就是从单例模式出发，一步一步讲到枚举类型对与单例模式的应用，这是我以前的文章，写的很详细，就不再多提了。

参考：[【设计模式】第二篇：单例模式的几种实现And反射对其的破坏](https://juejin.cn/post/6890324208766287880)

## 第4条：通过私有构造器强化不可实例化能力

有些时候，我们遇到的情况是这样的：有一些工具类，并不希望被实例化，因为没有什么意义，但是如果我们不去管它，就会默认被生成一个无参构造函数，所以我们会考虑主动显式的去声明一个私有的构造器，防止工具类可以被实例化。

## 第5条：优先考虑依赖注入来引用资源

这一点也不再多说了，毕竟像 Spring 这种依赖注入的框架也已经很常见了。

## 第6条：避免创建不必要的对象

如果每次都去创建相同功能的新对象，着实有点浪费，如果对象是不可变的，那么它就可以被重用。

1. **在同一个虚拟机下，包含相同字符串的字面常量对象会被重用。**

   - 例如使用：`String s = "bikini";` 而不是 `String s = new String("bikini");`

2. **在同时提供了静态工厂方法和构造方法的类中，优先使用静态工厂。**

   - 例如使用：`Boolean.valueOf()` 而不是 `Boolean(String)` ，并且后者在 Java 9 中被废弃掉了。

3. **使用 `String.matches()` 做字符串正则匹配检查的时候，会有较大的性能问题**

   - 因为它在内部创建了一个 Pattern ,但是却只使用了一次，之后就可以回收了，如果重复创建它成本会很高，因为需要将正则表达式编译成一个有限状态机。

   - 改进：类初始化的时候，创建一个 static 类型的 Pattern 对象，然后重复利用。

4. **如果这个方法永远不会被调用，也不需要初始化相关的的字段，就可以把它们放到方法第一次被调用的时候。**

   - 不是特别推荐，性能上并没有显著的提高，而且会让方法变得复杂。

5. **JDK 1.5 后出现的自动装箱会创建对象，所以程序中优先使用基本类型**

6. **小对象的构造器只做很少量的显式工作，创建和回收都是很廉价的，所以通过创建附加的对象提升程序的清晰简洁性也是好事**

7. **通过维护自己的对象池（object pool）来避免创建对象并不是一种好的做法（代码, 内存）， 除非池中的对象是非常重量级的。**

## 第7条：消除过期的对象引用

Java 相较于 C++ 的一大特点就是垃圾回收机制（GC机制），因为 Java 并不需要你手动的去管理内存，对象生命周期结束后，也不需要自己去处理。

但是越是这种自动化的操作，其实你就需要更加谨慎，因为很可能你在不经意间，就未内存泄漏等问题埋下了种子。

- 例如：一个栈，先增长，后收缩，那么从栈中弹出来的元素并不会被回收，这是因为栈的内部维护着这些对象的过期引用（即指永远不会解除引用）

所以在对象引用过期后，一定要清空这些引用，可以参考一下 ArrayList 的代码是怎么设计的

- 当 remove 或者 clear 一个数据的时候，它都会有 `elementData[xxx] = null;` 这样一个操作，并且从注释也可以看出来，`// clear to let GC do its work`  这里是为了进行 GC 操作。

```java
    /**
     * Removes the first occurrence of the specified element from this list,
     * if it is present.  If the list does not contain the element, it is
     * unchanged.  More formally, removes the element with the lowest index
     * <tt>i</tt> such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>
     * (if such an element exists).  Returns <tt>true</tt> if this list
     * contained the specified element (or equivalently, if this list
     * changed as a result of the call).
     *
     * @param o element to be removed from this list, if present
     * @return <tt>true</tt> if this list contained the specified element
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    /*
     * Private remove method that skips bounds checking and does not
     * return the value removed.
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }

    /**
     * Removes all of the elements from this list.  The list will
     * be empty after this call returns.
     */
    public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
```

## 第8条：避免使用终结方法和清除方法

- 终结方法：finalize

- 清除方法：cleaner

终结方法通常是不可预测的，也是很危险的，一般情况下是不必要的。使用终结方法会导致行为的不稳定、性能降低，以及可移植性的问题。

在 JDK 9 中，用清除方法（cleanner）代替了终结方法（finalize）

虽然说清除方法没有终结方法那么危险，但是仍然是不可预测、运行缓慢、一般情况下也不推荐使用。

## 第9条：try-with-resources 优先于 try-catch-finally

这一条引用了我之前写过的文章：[005-JavaIO和异常.md](https://github.com/ideal-20/Java-Ideal-Interview/blob/main/docs/java/javase-basis/005-JavaIO%E5%92%8C%E5%BC%82%E5%B8%B8.md)

> 面对必须要关闭的资源，我们总是应该优先使用 try-with-resources 而不是try-finally。随之产生的代码更简短，更清晰，产生的异常对我们也更有用。try-with-resources语句让我们更容易编写必须要关闭的资源的代码，若采用try-finally则几乎做不到这点。—— Effecitve Java

Java 从 JDK1.7 开始引入了 try-with-resources ，在其中定义的变量只要实现了 AutoCloseable 接口，这样在系统可以自动调用它们的close方法，从而替代了finally中关闭资源的功能。

使用 try-catch-finally 你可能会这样做

```java
try {
    // 假设这里是一组关于 文件 IO 操作的代码
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (s != null) {
        s.close();
    }
}
```

但现在你可以这样做

```java
try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(new File("test.txt")))) {
    // 假设这里是操作代码
} catch (IOException e) {
    e.printStackTrace();
}
```

如果有多个资源需要 close ，只需要在 try 中，通过分号间隔开即可

```java
try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(new File("test.txt")));
     BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(new File("test.txt")))) {
    // 假设这里是操作代码
} catch (IOException e) {
    e.printStackTrace();
}
```

可参考 [JDK1.8中的try-with-resources声明](https://blog.csdn.net/weixin_40255793/article/details/80812961)























