# 7种经典垃圾回收器总结

截止JDK1.8，一共有7款不同的垃圾收集器，每一款不同的垃圾收集器都有不同的特点

|  垃圾收集器  |    分类    |    作用位置    |           使用算法           |     特点     |                           适用场景                           |
| :----------: | :--------: | :------------: | :--------------------------: | :----------: | :----------------------------------------------------------: |
|    Serial    |  串行运行  |     新生代     |           复制算法           | 响应速度优先 |                适用于单CPU环境下的client模式                 |
|    ParNew    |  并行运行  |     新生代     |           复制算法           | 响应速度优先 |              多CPU环境Server模式下与CMS配合使用              |
|   Parallel   |  并行运行  |     新生代     |           复制算法           |  吞吐量优先  |             适用于后台运算而不需要太多交互的场景             |
|  Serial Old  |  串行运行  |     老年代     |        标记-压缩算法         | 响应速度优先 | 适用于单CPU环境下的client模式<br/>或作为CMS出现"Concurrent Mode Failure"失败的后备预案 |
| Parallel Old |  并行运行  |     老年代     |        标记-压缩算法         |  吞吐量优先  |             适用于后台运算而不需要太多交互的场景             |
|     CMS      |  并发运行  |     老年代     |        标记-清除算法         | 响应速度优先 |                    适用于互联网或B/S业务                     |
|      G1      | 并发、并行 | 新生代、老年代 | 复制算法、<br/>标记-压缩算法 | 响应速度优先 |                        面向服务端应用                        |

GC发展阶段：

- Serial => Parallel (并行) => CMS(并发) => G1 => ZGC

# 垃圾回收器组合

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210411204851802.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


- 两个收集器间有连线，表明它们可以搭配使用：
  - Serial + Serial Old
  - ParNew + Serial Old
  - Serial + CMS
  - ParNew + CMS
  - Parallel Scavenge + Serial Old
  - Parallel Scavenge + Parallel Old
  - G1
- Serial Old 作为CMS出现"Current Mode Failure"失败的后备预案
- <font color="red">红色虚线</font>：
  - 由于维护和兼容性测试的成本，在JDK8时将Serial + CMS、ParNew + Serial Old这两个组合声明为Deprecated，并在JDK9中完全取消了这些组合的支持，即：移除
- <font color="green">绿色虚线</font>：
  - JDK14中：弃用Parallel Scavenge + Serial Old组合
- JDK14中，删除CMS垃圾回收器

# 怎么选择垃圾回收器？

- Java垃圾收集器的配置对于JVM优化来说是一个很重要的选择，选择合适的垃圾收集器可以让JVM的性能有一个很大的提升。

- 怎么选择垃圾收集器

  - 1.优先调整堆的大小让JVM自适应完成

  - 2.如果内存小于100M，使用串行收集器
  - 3.如果是单核、单机程序，并且没有停顿时间的要求，串行收集器

  - 4.如果是多CPU、需要高吞吐量、允许停顿时间超过1秒，选择并行或者JVM自己选择
  - 5.如果是多CPU、追求低停顿时间，需快速响应(比如延迟不能超过1秒，如互联网应用)，使用并发收集器
    - 官方推荐G1，性能高。**现在互联网的项目，基本都是使用G1**

- 没有最好的收集器，更没有万能的收集器；调优永远是针对特定场景、特定需求，不存在一劳永逸的收集器

# Reference

- [尚硅谷JVM详解](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=11705427724146495636)