
# 1. 概述

针对HotSpot VM的实现，它里面的GC其实准确分类只有两大种：

- Partial GC：不是完整收集整个Java堆的垃圾收集

- - Young GC/Minor GC：只收集young gen的GC
  - Old GC/Major GC：只收集old gen的GC
    - 只有CMS GC会有单独收集老年代的行为
    - **很多时候Major GC会和Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收**
  - Mixed GC：收集整个young gen以及部分old gen的GC
    - 只有G1 GC会有这种行为

- Full GC：收集整个Java堆和方法区的垃圾收集

# 2. Minor GC

- 年轻代GC（Minor GC触发机制）
  - 当年轻代（Eden区）满时就会触发 Minor GC，这里的年轻代满指的是 Eden区满。**Survivor 满不会触发 Minor GC** 
  - Minor GC非常频繁，一般回收速度也比较快
  - Minor GC会引发STW

# 3. Full GC

- Full GC定义是相对明确的，就是针对Java整堆和方法区的全局范围的GC

- Full GC触发机制：
  - 调用`System.gc()`时，系统建议执行Full GC，但是不一定会执行 
  - 老年代空间不足
  - 方法区空间不足
  - 通过Minor GC后进入老年代的平均大小大于老年代的可用内存
  - 由Eden区、survivor space1（From Space）区向survivor space2（To Space）区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小 

# 4. 补充

- HotSpot VM的GC里，除了CMS的concurrent collection之外，其它能收集old gen的GC都会同时收集整个GC堆，包括young gen，不需要事先触发一次单独的young GC
  - Parallel Scavenge（-XX:+UseParallelGC）框架下，默认是在要触发full GC前先执行一次young GC，并且两次GC之间能让应用程序稍微运行一小下，以期降低full GC的暂停时间（因为young GC会尽量清理了young gen的死对象，减少了full GC的工作量）。控制这个行为的VM参数是-XX:+ScavengeBeforeFullGC。这是HotSpot VM里的奇葩
- Minor GC和Major GC是俗称，在Hotspot JVM实现的Serial GC, Parallel GC, CMS, G1 GC中大致可以对应到某个Young GC和Old GC算法组合
  - Hotspot JVM实现中几种GC算法组合：
    - Serial GC算法：Serial Young GC ＋ Serial Old GC （**实际上它是全局范围的Full GC**）
    - Parallel GC算法：Parallel Young GC ＋ 非并行的PS MarkSweep GC / 并行的Parallel Old GC（**这俩实际上也是全局范围的Full GC**），选PS MarkSweep GC 还是 Parallel Old GC 由参数UseParallelOldGC来控制
    - CMS算法：ParNew（Young）GC + CMS（Old）GC ＋ Full GC for CMS算法（出现**"Concurrent Mode Failure"**失败时，虚拟机将**启动后备预案：临时启用Serial Old收集器来重新进行老年代的垃圾收集**）
    - G1 GC：Young GC + mixed GC（新生代+部分老生代）＋ Full GC for G1 GC算法（回收阶段没有足够的To-space来存放晋升对象或者并发处理过程完成之前空间耗尽）
  - **各类GC算法的触发条件**：
    - 各种Young GC的触发原因都是eden区满了
    - Serial Old GC／PS MarkSweep GC／Parallel Old GC的触发则是在要执行Young GC时候预测其晋升的对象的总大小超过老生代剩余空间（分配担保机制）
    -  CMS GC的initial marking的触发条件是老生代使用比率超过某值
    - G1 GC的initial marking的触发条件是Heap使用比率超过某值
    -  Full GC for CMS算法和Full GC for G1 GC算法的触发原因：内存回收的速度赶不上内存分配的速度
    - PS MarkSweep GC／Parallel Old GC（Full GC）之前会跑一次Parallel Young GC，原因就是减轻Full GC 的负担













# Reference
- [尚硅谷JVM详解](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=17511287528542340392)
- [知乎](https://www.zhihu.com/question/41922036/answer/93079526)

- [JVM 垃圾回收之Minor GC、Major GC和Full GC之间的区别](https://xiaojin21cen.blog.csdn.net/article/details/87779487)