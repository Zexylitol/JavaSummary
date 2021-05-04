
# 1. 可变性

## String

**String是典型的Immutable（不可变）类**，被声明为`final class`,所有属性也都是`final`的。String类中使用`final`关键字修饰的数组来保存字符串，**所以String对象是不可变的**，类似拼接、裁剪字符串等动作，都会产生新的String对象。<br/>

Java 9 之前String类的实现使用字符数组来存储字符串：

```java
/** The value is used for character storage. */
private final char value[];
```

在Java 9之后String类的实现使用`byte`数组来存储字符串：

```java
private final byte value[];
```

## StringBuffer与StringBuilder

StringBuffer和StringBuilder都继承自AbstractStringBuilder类，在AbstractStringBuilder中也是使用字符数组（Java 9之后改为`byte`数组）存储字符串`char[] value`，但是没有用`final`关键字修饰，**所以这两种对象都是可变的。**

# 2. 线程安全性

## String

String是immutable类的典型实现，因为你无法对它内部数据进行任何修改，**原生的保证了基础线程安全**。

## StringBuffer

StringBuffer本质是一个**线程安全的可修改字符序列**，它的线程安全是通过对方法都加上`synchronized`关键字实现的，非常直白，也随之带来了额外的性能开销。

## StringBuilder

StringBuilder是Java 1.5中新增的，在能力上和StringBuffer没有本质区别，区别仅在于StringBuilder没有对方法加同步锁，**所以是非线程安全的**。







# 3. 应用场景

- 在字符串内容不经常发生变化的业务场景优先使用String类。例如：常量声明、少量的字符串拼接操作等。如果有大量的字符串内容拼接，避免使用String与String之间的“+”操作，因为这样会产生大量无用的中间对象，耗费空间且执行效率低下（新建对象、回收对象花费大量时间）
- 在频繁进行字符串的运算（如拼接、替换、删除等），并且运行在多线程环境下，建议使用StringBuffer，例如XML解析、HTTP参数解析与封装
- 在频繁进行字符串的运算（如拼接、替换、删除等），并且运行在单线程环境下，建议使用StringBuilder，例如SQL语句拼装、JSON解析等

# 总结

|            | String   | StringBuffer | StringBuilder |
| :--------- | -------- | ------------ | ------------- |
| 可变性     | 不可变   | 可变         | 可变          |
| 线程安全性 | 线程安全 | 线程安全     | 非线程安全    |

# Reference

- [JavaGuide](https://snailclimb.gitee.io/javaguide-interview/#/./docs/b-1%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93-Java%E5%9F%BA%E7%A1%80?id=_2110-string-stringbuffer-%e5%92%8c-stringbuilder-%e7%9a%84%e5%8c%ba%e5%88%ab%e6%98%af%e4%bb%80%e4%b9%88-string-%e4%b8%ba%e4%bb%80%e4%b9%88%e6%98%af%e4%b8%8d%e5%8f%af%e5%8f%98%e7%9a%84)
- [Java核心技术面试精讲](https://time.geekbang.org/column/intro/100006701)