# 简介

> 《深入理解Java虚拟机》一书中这样定义ZGC：
>
> Z Garbage Collector，即ZGC，是一款**基于Region内存布局**，（暂时）**不设置分代**，使用了读屏障、染色指针和内存多重映射等技术来实现**可并发的标记-整理算法**的，以**低延迟为首要目标**的一款垃圾收集器
>
> 未来将成为**服务端、大内存、低延迟应用**的首选垃圾收集器

为了满足如下目标进行设计：

- **停顿时间不会超过10ms**
- 停顿时间不会随着堆的增大而增大（不管多大的堆都能保持在10ms以下）
- 可支持几百M，甚至几T的堆大小（最大支持4T）

> **The Z Garbage Collector**, also known as **ZGC**, is a scalable low latency garbage collector designed to meet the following goals:
>
> - Pause times **do not** exceed **10ms**
> - Pause times **do not** increase with the heap or live-set size
> - Handle heaps ranging from a **few hundred megabytes** to **multi terabytes** in size

# 特性进度表

- JDK11，2018年 9月
  - ZGC发布
  - 不支持类的卸载。`-XX:+ClassUnloading` 不生效
- JDK12，2019年 3月
  - 支持并发的类卸载
  - 暂停时间进一步缩短
- JDK13，2019年 9月
  - 最大堆内存从4TB -> 16TB
  - 支持归还未使用的内存   uncommitting unused memory
  - 支持Linux与/AArch64平台
  - 减少时间到一个固定的时间点之下   (Reduced Time-To-Safepoint)
  - 支持  `-XX:SoftMaxHeapSize` ，当设置这个参数的时候，ZGC会尽量在指定的内存大小之下，除非为了避免内存溢出
- JDK 14，2020年3月
  - 增加稳定性
  - 支持不连续的地址空间

ZGC支持的平台：

| 平台          | 是否支持 | 当前进度     |
| :------------ | :------- | :----------- |
| Linux/x64     | Y        | Since JDK 11 |
| Linux/AArch64 | Y        | Since JDK 13 |
| macOS         | Y        | Since JDK 14 |
| Windows       | Y        | Since JDK 14 |

# ZGC特性

- Concurrent（并发执行）
- Region-based（内存基于region，类似G1）
- Compacting（压缩）
- NUMA-aware（Numa架构，一种内存页合并的硬件技术）
- Using colored pointers（着色指针）
- Using load barriers（读屏障）

### （1）Concurrent

ZGC只有短暂的STW，大部分的过程都是和应用线程并发执行，比如最耗时的并发标记和并发移动过程。

### （2）Region-based

ZGC中没有新生代和老年代的概念，只有一块一块的内存区域page，这和G1的region有点像，但和G1不一样的是，region的大小更加灵活和动态。zgc的region不会像G1那样在一开始就被划分为固定大小的region。

zgc的region核心亮点就是：**动态**。

动态表现为：

1. 动态地创建和销毁。
2. 动态地决定region的大小。它的最小单位是2MB的一个块。然后每个region的大小就是是2MB*N就是。

### （3）Compacting

每次进行GC时，都会对page进行压缩操作，所以完全避免了CMS算法中的碎片化问题。

### （4）NUMA-aware

### （2）Colored Pointers

是zgc的一个核心概念。你还可以叫它tag pointers，version pointers。

和以往的标记算法比较不同，CMS和G1会在对象的对象头进行标记，而ZGC是标记对象的指针。



<center><img src="https:////upload-images.jianshu.io/upload_images/6271376-fa4a95f418bb2f70?imageMogr2/auto-orient/strip|imageView2/2/w/1080/format/webp"/></center>



一个pointer共64bit。开始的18bit暂时没有被用到（以后有可能要用），然后是四个bit，分别表示Finalizable、Remapped、Marked1、Marked0，这是gc过程中每个对象的状态。最后42bit是用来存储对象的地址。

### （3）Using load barriers

因为在标记和移动过程中，GC线程和应用线程是并发执行的，所以存在这种情况：对象A内部的引用所指的对象B在标记或者移动状态，为了保证应用线程拿到的B对象是对的，那么在读取B的指针时会经过一个 “load barriers” 读屏障，这个屏障可以保证在执行GC时，数据读取的正确性。



# Reference

- [ZGC-掘金](https://juejin.cn/post/6844903988907737101)
- [ZGC-简书](https://www.jianshu.com/p/b31f17da63ab)