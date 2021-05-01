
# 1. 运行时数据区内部结构

![image-20210313100451279](https://img-blog.csdnimg.cn/img_convert/4c0926f71dffa52cd271f4dd59d82e1e.png)



# 2. 堆

**一个进程就对应着一个JVM实例，一个JVM实例就有一个运行时数据区，只有一个方法区和堆，一个进程中的多个线程共享方法区和堆空间，每个线程有自己独立的程序计数器、本地方法栈、虚拟机栈**

## 2.1 概述

- 堆区在JVM启动时即被创建，空间大小也随之确定（可调节）

- 《Java虚拟机规范》规定：堆可以处于物理上不连续的内存空间中，但逻辑上应该被视为连续的

- **所有的线程共享Java堆，在这里还可以划分线程私有的缓冲区**（Thread Local Allocation Buffer, TLAB）

- 《Java虚拟机规范》中对Java堆的描述是:所有的对象实例以及数组都应当在运行时分配在堆上。

  - > The heap is the run-time data area from which memory for all class instances and arrays is allocated 

  - 我要说的是:**几乎**所有的对象实例都在这里分配内存-------从实际使用角度看的

- 数组和对象可能永远不会存储在栈上，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置

- **在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除**

- **堆，是GC ( Garbage Collection, 垃圾收集器)执行垃圾回收的重点区域**

## 2.2 内存细分

- 现代垃圾收集器大部分都基于**分代收集理论设计**，堆空间细分为：
  - Java 7及以前堆内存逻辑上分为三部分：**新生代+老年代+<font color="red">永久代</font>**
    - Young Generation Space 又被划分为Eden区和Survivor区
    - Tenure Generation Space
    - Permanent Space
  - Java 8及以后堆内存逻辑上分为三部分：**新生代+老年代+<font color="red">元空间</font>**
    - Young Generation Space 又被划分为Eden区和Survivor区
    - Tenure Generation Space
    - Meta Space
  - 约定：
    - 新生区 = 新生代 = 年轻代
    - 养老区 = 老年区 = 老年代
    - 永久区 = 永久代

## 2.3 堆空间内部结构

### 2.3.1 JDK7

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210313204444251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


### 2.3.2 JDK8
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210313204501920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


## 2.4 年轻代与老年代

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210313204518515.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


- Java堆区进一步细分的话，可以划分为年轻代(YoungGen)和老年代(OldGen)
  - 其中年轻代又可以划分为Eden、Survivor0和Survivor1(Survivor0和Survivor1有时也叫做from区和to区)
- 存储在JVM中的Java对象可以被划分为两类：
  - **一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速**
  - **另一类对象的生命周期却非常长，在某些极端的情况下还能够与JVM的生命周期保持一致**

- **几乎所有的**Java对象都是在Eden区被new出来的
- 绝大部分的Java对象的销毁都在新生代进行
  - 新生代中80%的对象都是“朝生夕死”的

## 2.5 为什么需要把Java堆分代？

- 经研究，不同对象的生命周期不同，70%-99%的对象是临时对象
  - 新生代：有Eden、两块大小相同的Survivor(又称为from/to，s0和s1)构成，to区总为空
  - 老年代：存放新生代中经历多次GC仍然存活的对象
- **分代的唯一理由就是优化GC性能**，如果没有分代，那所有的对象都在一块，GC时就会对堆的所有区域进行扫描；而很多对象都是朝生夕死的，如果分代的话，把新创建的对象放到某一地方，当GC的时候先把这块存储“朝生夕死”对象的区域进行回收，这样就会腾出很大的空间出来

# HotSpot VM整体结构

![image-20210304202518367](https://img-blog.csdnimg.cn/img_convert/e18e23df6ac587d1b087a7a5182ea6a3.png)

# Reference

- [尚硅谷详解JVM虚拟机](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=15299793702369147437)