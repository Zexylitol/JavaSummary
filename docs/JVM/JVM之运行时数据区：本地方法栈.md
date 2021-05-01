# 1. 本地方法接口(Native Method Interface, JNI)

## 1.1 什么是本地方法？

简单地讲，**一个Native Method就是一个Java调用非Java代码的接口**。一个Native Method是这样一个Java方法：该方法的实现由非Java语言实现，比如C。这个特征并非Java所特有，很多其它的编程语言都有这一机制，比如在C++中，你可以用`extern "C"`告 知C++编译器去调用一个C的函数。

> "A native method is a Java method whose implementation is provided by non-java code."

在定义一个native method时，并不提供实现体(有些像定义一个Java interface)，因为其实现体是由非java语言在外面实现的。

**本地接口的作用是融合不同的编程语言为Java所用，它的初衷是融合C/C++程序。**

```java
// Object中的hashCode()方法
public native int hashCode();
```

**标示符`native`可以与所有其它的java标示符连用，但是`abstract`除外**

- 因为`native`表示方法体由非java语言实现的，`abstract`表示没有方法体，两者矛盾，不能共用

# 2. 运行时数据区内部结构

![image-20210313100451279](https://img-blog.csdnimg.cn/img_convert/4c0926f71dffa52cd271f4dd59d82e1e.png)

# 3. 本地方法栈

- <font color="blue">**Java虚拟机栈用于管理Java方法的调用，而本地方法栈用于管理本地方法调用**</font>
- **本地方法栈也是线程私有的**
- 允许被实现成固定或者可动态扩展的内存大小（同Java虚拟机栈）

- **当某个线程调用一个本地方法时，它就进入了一个全新的并且不再受虚拟机限制的世界。它和虛拟机拥有同样的权限**
  - 本地方法可以通过本地方法接口来访问虚拟机内部的运行时数据区
  - 它甚至可以直接使用本地处理器中的寄存器
  - 直接从本地内存的堆中分配任意数量的内存

- **并不是所有的JVM都支持本地方法**。因为Java虚拟机规范并没有明确要求本地方法栈的使用语言、具体实现方式、数据结构等。如果JVM产品不打算支持native方法，也可以无需实现本地方法栈

- <font color="red">**在Hotspot JVM中， 直接将本地方法栈和虚拟机栈合二为一**</font>

# 4. HotSpot VM整体结构

![image-20210304202518367](https://img-blog.csdnimg.cn/img_convert/e18e23df6ac587d1b087a7a5182ea6a3.png)

# Reference

- [尚硅谷详解JVM虚拟机](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=15299793702369147437)