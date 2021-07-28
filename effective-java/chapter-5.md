# 第5章

泛型，就是将类型参数化，类似定义一个方法的时候有形式参数，只有到真正的数据传入才知道实际参数是多少。而泛型就是将数据的类型也变成一个形式类型，就是在定义的时候并不去指定，只有等实际去创建的时候再去指定，这也就是泛型。

补充一个我当初刚开始学习常常遇到的问题：

下面是常见的泛型类型，但是我当初只认识`Comparable<T>` 类型，曾经一度认为 T 就是泛型的专有名词，后来遇到 E，K，V 等等就人傻了。

- `Comparable<T>`
- `Collection<E>`
- `Map<K,V>`

解一下惑：定义泛型的时候并不是必须使用 T E K V这几个值或者大写字母，前面的例子都是大家一种一般情况的约定而已：

- T：泛指一般的类
- E：泛指元素Element 
- K：泛指键Key
- V：泛指值Value

## 第26条：请不要使用原生态类型

| 术 语                | 范 例                            |
| -------------------- | -------------------------------- |
| **参数化的类型**     | `List<String>`                     |
| **实际类型参数**     | `String`                           |
| **泛型**             | `List<E>`                          |
| **形式类型参数**     | `E`                                |
| **无限制通配符类型** | `List<?>`                          |
| **原生态类型**       | `List`                             |
| **有限制类型参数**   | `<E extends Number>`               |
| **递归类型限制**     | `<T extends Comparable<T>>`        |
| **有限制通配符类型** | `List<? extends Number>`           |
| **泛型方法**         | `static <E> List<E> asList(E[] a)` |
| **类型令牌**         | `String.class`                     |

上述表格来自网络，其描述的是泛型中的一些官方的术语，找到这个规则的题目：请不要使用原生态类型，在表格中找到对应，即 List 这样的直接用，而要指定泛型。

## 第27条：消除非受检的警告

这一条很简单，只要尽可能的减少自己遇到的一些警告内容，然后修改代码，尽可能规范等去消除掉警告。

如果实在消除不掉，但是自己确能保证这个代码是正确安全的，也可以通过 @SuppressWarnings("unchecked") 进行避免警告。

## 第28条：列表优先于数组

结论：使用时，像 List 这样的列表优先于数组。

当不使用多态和泛型的时候，错误的赋值会在编译期就报错，列表 List 和数组都是这样的：

```java
Long[] array = new Long[1];
array[0] = "Hello Word";

List<Long> list = new ArrayList<Long>();
list.add("Hello Word");
```

而当使用了多态和泛型，数组类型会在运行时才会报错，而列表还是会在编译期就提醒。

```java
Object[] array = new Long[1];
array[0] = "Hello Word";

List<Object> list = new ArrayList<Long>();
list.add("Hello Word");
```

小结：

1. 在例如多态和泛型的情况下，数组有些时候在运行时才会报错，而列表会在编译期就给出体提示。
2. 数组是具体化的，运行时增强，无法创建泛型数组。
3. 而泛型是不可具体化的，只在编译阶段强化，到了运行时则会泛型擦除。

什么是泛型擦除？

- 首先，泛型这个概念是从 JDK 1.5 开始出现的，从兼容的角度考虑，泛型只会在编译器期间检查，到了运行时就会被消除掉，这就是泛型擦除。
- 例如你的泛型是 `List<String>` ，然后编译器就会在编译期判断你的类型是不是 `List<String>` ，等运行期间的时候，指定类型就会被擦除，仅仅作为一个 List 这样的容器来运行。

注：可以通过反射绕过运行期泛型的检查限制

## 第29条：优先考虑泛型

优先考虑泛型，但是在一些明显数组比较合适的情况下，参数全部用 Object 也不是特别合适，但是数组是没有办法通过泛型进行参数的创建的，所以会考虑在类的内部用 Object 数组，对外再用泛型进行规范。例如书中的例子：

## 第30条：优先考虑泛型方法

可以参考 java.util.Collections 这个工具类，其中使用的泛型方法都是很好用的。

首先泛型方法可以在编译期就保证了它是该类的子类。而且利用递归泛型还能避免强制类型转换的问题。

```java
// Generic stack using Object[] (Pages 130-3)
public class Stack<E> {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
 
    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }
 
    // Appropriate suppression of unchecked warning
    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
 
        // push requires elements to be of type E, so cast is correct
        @SuppressWarnings("unchecked") E result =
                (E) elements[--size];
 
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
 
    public boolean isEmpty() {
        return size == 0;
    }
 
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
 
    // Little program to exercise our generic Stack
    public static void main(String[] args) {
        Stack<String> stack = new Stack<>();
        for (String arg : args)
            stack.push(arg);
        while (!stack.isEmpty())
            System.out.println(stack.pop().toUpperCase());
    }
}
```

## 第31条：利用有限制通配符来提升API的灵活性（待修改）

## 第32条：谨慎并用泛型与可变参数

## 第33条：优先考虑类型安全的异构容器













