# 第10章 异常

## 第69条：只针对异常的情况才使用异常

不要用异常来做流程控制，只有真的有需要才做。

## 第70条：对可恢复的情况使用受检异常，对编程错误使用运行时异常

- **受检异常（Checked Exception）**：Exception 下的所有非 RuntimeException 异常。非本身的错误，而是外部因素导致的例如文件读写，数据库异常等造成的。
- **运行异常（Runtime Exception）**：Exception下面RuntimeException及其子类。

所以作为一个开发人员，只能自己去界定好是否能恢复，例如网络或者内存资源暂时不足等可能都属于能恢复。

## 第71条：避免不必要的使用受检异常

受检过多其实也会成为一种负担，因为 catch 或者 throw 都需要开发者去处理。

## 第72条：优先使用标准异常

## 第73条：抛出与抽象对应的异常

各个层次的异常要在不同层次处理

```java
try {
    ...
} catch (LowerLevelException cause) {
    throw new HighLevelException(cause);
}
```

## 第74条：每个方法抛出的所有异常都要建立文档

## 第75条：在细节消息中包含失败 - 捕获信息

捕获异常后，不要只打印一下异常的名字，最好写一下日志，记录一些必要的参数和域的值，方便排查和定位问题。

## 第76条：努力使失败保持原子性

程序执行过程中失败了，出现了异常，要做类似事务回滚一样的操作。

常见的三种方法保持失败原子性：

- 调整处理顺序，将可能会失败的计算部分，在改变对象状态之前进行处理，提前检查发现。
- 提前拷贝一份数据备份，用于失败之后可以保持原样。
- 编写恢复代码，由它拦截失败，并且回滚到之前的状态。

## 第77条：不要忽略异常



