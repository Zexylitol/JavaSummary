# 简介

|          | Serial                                                       | Serial Old                                                   |
| :------: | :----------------------------------------------------------- | :----------------------------------------------------------- |
|   简介   | HotSpot中Client模式下的默认新生代垃圾收集器                  | Client模式下的默认老年代垃圾收集器                           |
| 回收方式 | <font color="blue">采用复制算法、串行回收和"Stop-the-world"机制的方式执行内存回收</font> | <font color="blue">标记-压缩算法、串行回收和"Stop-the-world"机制</font> |
| 回收区域 | 新生代                                                       | 老年代                                                       |

Serial Old在Server模式下主要有两个用途：

- 1. 与新生代的Parallel Scavenge配合使用
- 2. 作为老年代CMS收集器的后备垃圾收集方案

串行回收：同一时间段内只允许有一个CPU用于执行垃圾回收操作，此时工作线程被暂停，直至垃圾收集工作结束

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406210241706.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


- Serial是单线程的收集器，但它的“单线程”的意义并不仅仅说明它**只会使用一个CPU或一条收集线程去完成垃圾收集工作**，更重要的是在它进行垃圾收集时，**必须暂停其他所有的工作线程**，直到它收集结束（Stop The World）

# 设置

- 在HotSpot虚拟机中，使用`-XX:+UseSerialGC`参数可以指定年轻代和老年代都使用串行收集器
  - 等价于新生代使用Serial GC其老年代使用Serial Old GC

# 总结

- 在限定单个CPU的环境下，Serial收集器没有线程交互的开销，是个不错的选择，可以获得最高的单线程收集效率
- 对于交互较强的应用而言，Serial收集器是不能接受的
- 现在已经不用串行的了

# Reference

- [尚硅谷JVM详解](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=11705427724146495636)