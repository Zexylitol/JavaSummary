
# 简介

|          | CMS垃圾回收器                                                |
| :------: | :----------------------------------------------------------- |
|   简介   | JDK1.5时期，HotSpot推出了一款在<font color="red">强交互应用中</font>有划时代意义的垃圾收集器：CMS(Concurrent-Mark-Sweep)收集器，<font color="red">这款收集器是HotSpot虚拟机中第一款真正意义上的**并发**收集器，它第一次实现了让垃圾收集线程与用户线程同时工作</font> |
| 回收方式 | <font color="blue">标记-清除算法、并发回收和"Stop-the-world"机制</font> |
| 回收区域 | 老年代                                                       |
| 应用场景 | **CMS收集器的关注点是尽可能缩短垃圾收集时用户线程的停顿时间**。停顿时间越短(低延迟)就越适合与用户交互的程序，良好的响应速度能提升用户体验<br/><font color="blue">目前很大一部分的Java应用集中在互联网站或B/S系统的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短</font>，以给用户带来较好的体验，CMS收集器非常符合这类应用的需求 |
|   优点   | 1. 并发收集<br/>2. 低延迟                                    |
|   弊端   | 1. <font color="blue">会产生内存碎片</font>，导致并发清除后，用户线程可用的空间不足。在无法分配大对象的情况下，不得不提前触发Full GC<br/>2. <font color="blue">CMS收集器对CPU资源非常敏感</font>。在并发阶段，它虽然不会导致用户停顿，但是会因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低<br/>3. <font color="blue">CMS收集器无法处理浮动垃圾(Floating Garbage)</font>，可能出现"Concurrent Mode Failure"失败而导致另一次完全"Stop-the-world"的Full GC的产生。**在并发标记和并发清理阶段由于程序的工作线程和垃圾收集线程是同时运行或者交叉运行的，那么如果产生新的垃圾对象，CMS将无法对这些垃圾对象进行处理**，最终会导致这些新产生的垃圾对象**没有被及时回收**，从而只能在下一次执行GC时释放这些之前未被回收的内存空间，这一部分垃圾就被称为浮动垃圾 |

- CMS作为老年代的收集器，却无法与JDK 1.4.0 中已经存在的新生代收集器Parallel Scavenge配合工作，**所以在JDK 1. 5中使用CMS来收集老年代的时候，新生代只能选择ParNew或者Serial收集器中的一个**

- 在G1出现之前，CMS使用还是非常广泛的，直到今天，仍然有很多系统使用CMS GC

- JDK8默认垃圾回收器：ParallelGC + ParallelOldGC
- JDK9默认垃圾回收器：G1
- JDK14默认的垃圾回收器：G1

# CMS工作原理

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021041120273935.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


- CMS整个过程比之前的收集器要复杂，整个过程分为4个主要阶段，即**初始标记阶段**、**并发标记阶段**、**重新标记阶段**和**并发清除阶段**

|              阶段              | 说明                                                         |
| :----------------------------: | :----------------------------------------------------------- |
|     初始(Initial-Mark)阶段     | 这个阶段中，程序中所有的工作线程都将会因为**"Stop-the-world"**机制而出现短暂的暂停，这个阶段的<br/>主要任务<font color="red">仅仅只是标记出GC Roots能直接关联到的对象</font>，一旦标记完成之后就会恢复之前被暂停的所有应用线程，由于直接关联对象比较小，所以这里的<font color="red">速度非常快</font> |
| 并发标记(Concurrent-Mark)阶段  | 从GC Roots的<font color="red">直接关联对象开始遍历整个对象图</font>，这个过程<font color="red">耗时较长但不需要停顿用户线程</font>，可以与垃圾收集线程一起并发运行 |
|      重新标记(remark)阶段      | 需要**"Stop-the-world"**<br/>是为了修正并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，**这个阶段的停顿时间通常会比初始标记阶段稍长一些，但也远比并发标记阶段的时间短** |
| 并发清除(concurrent sweep)阶段 | 清理掉标记阶段判断的已经死亡的对象，由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的 |

- 尽管CMS收集器采用的是并发回收(非独占式)，但是在其<font color="blue">初始化标记和重新标记这两个阶段中仍然需要执行"Stop-the-world"机制</font>暂停程序中的工作线程，不过暂停时间不会太长，因此目前所有的垃圾收集器都做不到完全不需要"Stop-the-world"，只是尽可能的缩短暂停时间
- <font color="blue">由于最耗费时间的并发标记与并发清除阶段都不需要暂停工作，所以整体的回收是低停顿的</font>
- **由于在垃圾收集阶段用户线程没有中断，所以在CMS回收过程中，还应该确保应用程序用户线程有足够的内存可用**。因此，CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，而是**当堆内存使用率达到某一阈值时，便开始进行回收**，以确保应用程序在CMS工作过程中依然有足够的空间支持应用程序运行，**要是CMS运行期间预留的内存无法满足程序需要**，就会出现一次**"Concurrent Mode Failure"**失败，这时虚拟机将**启动后备预案：临时启用Serial Old收集器来重新进行老年代的垃圾收集**，这样停顿时间就很长了
- **为什么CMS使用标记-清除算法而不是标记-压缩算法？**
  - 因为当并发清除时，用户线程并没有中断，用Compact整理内存的话，原来的用户线程使用的内存还怎么用呢？要保证用户线程能继续执行，前提是它的运行的资源不受影响。**Mark Compact更适合"Stop-the-world"这种场景下使用**

# 总结

- 想最小化使用内存和并行开销，选Serial GC
- 想要最大化应用程序的吞吐量，选Parallel GC
- 想要最小化GC的中断或停顿时间，选CMS GC

# CMS变化

- JDK9：CMS被标记为Deprecated
- JDK14：删除CMS垃圾回收器

# Reference

- [尚硅谷JVM详解](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=11705427724146495636)