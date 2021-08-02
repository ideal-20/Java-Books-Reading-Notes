# 第7章 Lambda和Stream

## 第42条：Lambda优先于匿名类

下例创建了一个list，目的就是想做一个排序处理，先来看一下有多少种写法。

```java
public class SortFourWays {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("111", "666", "222", "555");
      	// ... 想要做排序处理
    }
}
```

1. 匿名内部类的方式

```java
// Anonymous class instance as a function object - obsolete! (Page 193)
Collections.sort(words, new Comparator<String>() {
  public int compare(String s1, String s2) {
    return Integer.compare(s1.length(), s2.length());
  }
});

// IDEA：Anonymous new Comparator<String>() can be replaced with lambda
```

2. Lambda 带入参方式

```java
// Lambda expression as function object (replaces anonymous class) (Page 194)
Collections.sort(words,
                 (s1, s2) -> Integer.compare(s1.length(), s2.length()));

// IDEA：Can be replaced with Comparator.comparingInt
```

3. Lambda 使用 Comparator.comparingInt 方式

```java
// Comparator construction method (with method reference) in place of lambda (Page 194)
Collections.sort(words, comparingInt(String::length));
```

4. 直接使用 List 对象的 sort 方法

```java
// Default method List.sort in conjunction with comparator construction method (Page 194)
words.sort(comparingInt(String::length));
```

首先可以看到，一个最明显的改观就是代码量的减少，但是就语义的理解上来看，也没有很明显的混淆。而且最重要的是，除了第一种匿名类的方式外，其余方法都不需要指定参数的类型，这其实和编译器的类型推导有关，它会根据上下文自动的推导参数的类型。

虽然简单的 Lambda 语义还算清晰，虽然但是如果想要保证极高的阅读性，那么使用大篇幅 Lambda 其实并不算特别合适。

## 第43条：方法引用优先于Lambda

在上面 42 条的例子 `words.sort(comparingInt(String::length));` 中的 `String::length`  部分虽然总被误认为是 Lambda 的一部分，但它其实有一个自己的名字 —— 方法引用。

| 方法引用类型 | 范例                   | Lambda等式                                              |
| ------------ | ---------------------- | ------------------------------------------------------- |
| 静态         | Integer::parseInt      | str -> Integer.parseInt(str)                            |
| 有限制       | Instant.now()::isAfter | Instant then = Instant.now();<br />t -> then.isAfter(t) |
| 无限制       | String::toUpperCase    | str -> str.toUpperCase()                                |
| 类构造器     | TreeMap<K, V>::new     | () -> new TreeMap<K, V>                                 |
| 数组构造器   | int[]::new             | len -> new int[len]                                     |

但是方法引用也不是一直都无脑用的，例如自定义一个类，类名很长 XxxxxxxxxxxxClass 去执行其中的 aaa 方法的时候

1. 如果使用方法引用：service.execute(XxxxxxxxxxxxClass::aaa)
2. 如果使用 Lambda：service.execute(() -> aaa());

所以我们只需要自行观察用哪个更简洁即可。

## 第44条：坚持使用标准的函数接口

函数接口是一个只含有一个抽象方法的接口，它的作用就是为了给函数式编程使用，可以通过 @FunctionalInterface 表明。

- 而在 java.util.function 下定义了的函数接口，叫做标准函数接口。
- 通过 @FunctionalInterface 注解就能表明这个接口是为了 Lambda 设计的，而且这个接口只允许有一个抽象方法。

| 接口                           | 函数签名            | 范例                | 备注                                                         |
| ------------------------------ | ------------------- | ------------------- | ------------------------------------------------------------ |
| 一元运算符  `UnaryOperator<T>` | T apply(T t)        | String::toUpperCase | Unary代表一个入参、Operator代表返回结果与参数一致的函数。※继承自Function<T, R>接口，统一了泛型类型 |
| 二元运算符 `BinaryOperator<T>` | T apply(T t1, T t2) | BigInteger::add     | Binary代表两个入参、Operator代表返回结果与参数一致的函数。※继承自BiFunction<T, U, R>接口，统一了泛型类型 |
| 谓词 `Predicate<T>`            | boolean test(T t)   | Collection::isEmpty | Predicate代表有一个入参，并返回布尔值的函数                  |
| 函数 `Function<T,R>`           | R apply(T t)        | Arrays::asList      | Function代表返回类型与入参类型不一致的函数                   |
| 提供者 `Supplier<T>`           | T get()             | Instant::now        | Supplier代表无参数，返回一个值的函数                         |
| 消费者 `Consumer<T>`           | void accept(T t)    | System.out::println | Consumer代表无返回值，有一个参数的函数                       |

## 第45条：谨慎使用Stream

记住一点：Stream 不是万能的，要合理使用。

```java
// Overuse of streams - don't do this! (page 205)
public class StreamAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
 
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```

上面的代码，可读性其实很低，而适当的把按字母排序的逻辑调整封装到一个 alphabetize 方法中，就会好很多。

```java
// Tasteful use of streams enhances clarity and conciseness (Page 205)
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
 
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }
 
    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

**一些适合 Lambda 的场景**

- 代码块中，可以读取和修改范围内，任意局部变量；Lambda只能读取final变量，而且无法改值
- 代码块中，可以return、break、continue，或者抛出异常；Lambda都不可以

**一些不适合 Lambda 的场景**

- 统一转换元素的序列
- 过滤元素的序列
- 利用单个操作（如添加、连接或者计算其最小值）合并元素顺序
- 将元素的序列存放到一个集合中，根据某些公共属性进行分组
- 搜索满足某些条件的元素的序列

## 第46条：优先选择Stream中无副作用的函数

尽可能的把一切操作都选择为无状态的，无副作用的方法。

## 第47条：Stream要优先用Collection作为返回类型

关于 Stream 返回类型，如果只关注循环，那么用迭代器，如果返回的是基本数据类型，而且对性能有一定的要求，推荐使用数组，其它的情况，返回 Collection 就可以了。

## 第48条：谨慎使用 Stream 并行

Java5 引入的 JUC，Java7 引入的 Fork-Join，而 Java8 引入的 Stream，只需要通过 parallel 就可以实现并行了，内部其实就是封装的 ForkJoinPool。

但是需要注意**安全性**和**活性失败**的问题。

- 活性失败：A修改了共享变量，而 B 未知，考虑加锁或者 volatile。
