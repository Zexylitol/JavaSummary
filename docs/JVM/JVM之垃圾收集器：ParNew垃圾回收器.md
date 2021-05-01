
# 简介

|          | ParNew                                                       |
| :------: | :----------------------------------------------------------- |
|   简介   | 如果说Serial GC是年轻代中的单线程垃圾收集器，那么**ParNew收集器则是Serial收集器的多线程版本**<br/>Par是Parallel的缩写<br/>New：只能处理新生代<br/>ParNew是很多JVM运行在Server模式下新生代的默认垃圾收集器 |
| 回收方式 | <font color="blue">复制算法、并行回收和"Stop-the-world"机制</font> |
| 回收区域 | 新生代                                                       |

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406211900468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


# ParNew 与 Serial 对比

- 由于ParNew收集器是基于并行回收，那么是否可以断定ParNew收集器的回收效率在任何场景下都会比Serial收集器更高效?
  - ParNew收集器运行在多CPU的环境下，由于可以充分利用多CPU、多核心等物理硬件资源优势，可以更快速地完成垃圾收集，提升程序的吞吐量。
  - 但是在单个CPU的环境下，ParNew收集器不比Serial收集器更高效。虽然Serial收集器是基于串行回收，但是由于CPU不需要频繁地做任务切换，因此可以有效避免多线程交互过程中产生的一-些额外开销

- 除Serial外，<font color="red">**目前只有ParNew GC能与CMS收集器配合工作**</font>

# 设置

```
-XX:+UseConcMarkSweepGC
```

Enables the use of the CMS garbage collector for the old generation. CMS is an alternative to the default garbage collector (G1), which also focuses on meeting application latency requirements. By default, this option is disabled and the collector is selected automatically based on the configuration of the machine and type of the JVM. The CMS garbage collector is deprecated.

```
-XX:+UseParNewGC
```

Enables the use of parallel threads for collection in the young generation. By default, this option is disabled. It’s automatically enabled when you set the `-XX:+UseConcMarkSweepGC` option. Using the `-XX:+UseParNewGC` option without the `-XX:+UseConcMarkSweepGC` option was deprecated in JDK 8. All uses of the `-XX:+UseParNewGC` option are deprecated. Using the option without `-XX:+UseConcMarkSweepGC` isn’t possible.

- 新生代：-XX:+UseParNewGC   (JDK9中ParNewGC被标记为deprecated)
- 老年代：-XX:+UseConcMarkSweepGC (JDK9中ParNewGC被标记为deprecated)

# Reference

- [尚硅谷JVM详解](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=11705427724146495636)