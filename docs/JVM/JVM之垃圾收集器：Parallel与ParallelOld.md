
# 简介

|          | Parallel Scavenge                                            | Parallel Old                                                 |
| :------: | :----------------------------------------------------------- | :----------------------------------------------------------- |
|   简介   | 和ParNew收集器不同，Parallel Scavenge收集器的目标是达到一个<font color="red">可控制的吞吐量(Throughput)</font>，<br/>它也被称为<font color="red">吞吐量优先的垃圾收集器</font><br/>自适应调节策略也是Parallel Scavenge与ParNew一个重要区别 | Parallel收集器在JDK1.6时提供了用于执行老年代垃圾收集的Parallel Old收集器，用来替代老年代的Serial Old收集器 |
| 回收方式 | <font color="blue">复制算法、并行回收和"Stop-the-world"机制</font> | <font color="blue">标记-压缩算法、并行回收和"Stop-the-world"机制</font> |
| 回收区域 | 新生代                                                       | 老年代                                                       |

- **高吞吐量**：（意味着单位时间内，STW的时间最短）可以高效率的利用CPU时间，尽快完成程序的运算任务，<font color="red">主要适合在后台运算而不需要太多交互的任务。因此，常见在服务器环境中使用</font>，例如，那些执行批量处理、订单处理、工资支付、科学计算的应用程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021040621232967.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


- **在Java 8中，默认采用Parallel Scavenge + Parallel Old**

# 设置

**`-XX:+UseParallelGC`**

Enables the use of the parallel scavenge garbage collector (also known as the throughput collector) to improve the performance of your application by leveraging multiple processors.

By default, this option is disabled and the collector is chosen automatically based on the configuration of the machine and type of the JVM. If it’s enabled, then the `-XX:+UseParallelOldGC` option is automatically enabled, unless you explicitly disable it.

**`-XX:+UseParallelOldGC`**

Enables the use of the parallel garbage collector for full GCs. By default, this option is disabled. Enabling it automatically enables the `-XX:+UseParallelGC` option.

# Reference

- [尚硅谷JVM详解](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=11705427724146495636)