
# 1. 垃圾标记阶段：对象存活判断

## 1.1 两种方式

- 在堆里存放着几乎所有的Java对象实例，在GC执行垃圾回收之前，首先**需要区分内存中哪些是存活对象，哪些是已经死亡的对象**，只有标记为已经死亡的对象，GC才会在执行垃圾回收时，释放掉其所占用的内存空间。判断对象存活一般有两种方式：
  - **引用计数算法**
  - **可达性分析算法**

|      | 引用计数算法(Reference Counting)                             | 可达性分析算法<br/>(或根搜索算法、追踪性垃圾收集)<br/>(Tracing Garbage Collection) |
| :--: | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 简介 | 1. 对每个对象保存一个<font color="red">整型的引用计数器属性，用于记录对象被引用的情况</font><br/>2. 只要对象的引用计数器的值为0，即表示对象不可能再被使用，可进行回收 | 1. 此算法以`GC Roots`(根对象集合)为起始点，从上至下<font color="red">搜索被GC Roots所连接的目标对象是否可达</font><br/>2. 搜索所走过的路径称为<font color="red">引用链(Reference Chain)</font><br/>3. <font color="red">存活对象：</font>能够被`GC Roots`直接或间接连接的对象<br/>4. <font color="red">垃圾对象：</font>没有任何引用链相连的对象 |
| 优点 | <font color="blue">实现简单，垃圾对象便于辨识，判定效率高，回收没有延迟性</font> | <font color="blue">实现简单，执行高效，更重要的是有效解决了循环引用问题，防止内存泄露的发生</font> |
| 缺点 | 1. <font color="green">增加了存储空间的开销：</font>需要单独的字段存储计数器<br/>2. <font color="green">增加了时间开销：</font>每次赋值都需要更新计数器<br/>3. <font color="green">无法处理循环引用的情况：</font>致命缺陷，导致在Java的垃圾回收器中没有使用这类算法 | 为了保证分析工作的准确性，<font color="green">分析工作必须在一个能保障一致性的快照中进行，这也是导致GC时必须`“Stop The World”`的一个重要原因</font><br/>(即时是号称(几乎)不会发生停顿的CMS收集器中，<font color="red">枚举根节点时也是必须要停顿的</font>) |
| 总结 | 1. **Java并没有选择引用计数**，是因为很难处理循环引用关系<br/>2. Pyhton同时支持引用计数和垃圾收集机制，Python如何解决循环引用？<br/>2.1 手动解除：就是在合适的时机，解除引用关系<br/>2.2 使用弱引用weakref，weakref是Python提供的标准库，旨在解决循环引用 | **Java使用的就是可达性分析算法**                             |

## 1.2 循环引用

- 两个对象互为引用，如果采用引用计数法，这两个对象将不能被回收，因为他们的引用计数无法为零

```java
class A{
  public B b;
   
}
class B{
  public A a;
}
public class Main{
    public static void main(String[] args){
    A a = new A();
    B b = new B();
    a.b=b;
    b.a=a;
    }
}
```

## 1.3 GC Roots

- 所谓`GC Roots`根集合就是**一组必须活跃的引用**

- 在Java语言中，GC Roots包括以下几类元素：

  - | GC Root                                                      | 举例                                                         |
    | :----------------------------------------------------------- | :----------------------------------------------------------- |
    | 虚拟机栈中引用的对象                                         | 比如：各个线程被调用的方法中使用到的参数、局部变量等         |
    | 本地方法栈内JNI(本地方法)引用的对象                          |                                                              |
    | 方法区中类静态属性引用的对象                                 | 比如：Java类的引用类型静态变量                               |
    | 方法区中常量引用的对象                                       | 比如：字符串常量池里的引用                                   |
    | 所有被同步锁synchronized持有的对象                           |                                                              |
    | Java虚拟机内部的引用                                         | 比如：基本数据类型对应的Class对象，一些常驻的异常对象（如：<br/>NullPointerException、OutOfMemoryError），系统类加载器 |
    | 反映Java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等 |                                                              |

  - <font color="red">**堆空间周边比如虚拟机栈、本地方法栈、方法区、常量池里边的引用都可以考虑作为GC Roots**</font>

## 1.4 查看GC Roots的工具

- MAT(Memory Analyzer)
- 命令行：jps + jmap
- JVisualVM
- JProfiler

# Reference

- [Java垃圾回收之循环引用](https://www.cnblogs.com/lihaozy/archive/2013/06/08/3125974.html)
- [尚硅谷JVM详解](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=10244461765307948942)