|                             Java                             |                            数据库                            |                           LeetCode                           |                          计算机网络                          |                           操作系统                           |                           设计模式                           |                             框架                             |                             工具                             |                           系统设计                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [<img src="https://i.im5i.com/2021/05/02/wauQd.jpg" alt="wauQd.jpg" border="0" style="zoom:6%;" />](#Java) | [​<img src="https://i.im5i.com/2021/05/02/wa5y4.jpg" alt="wa5y4.jpg" border="0" style="zoom:5%;" />​](#数据库) | [<img src="https://i.im5i.com/2021/05/02/waHYW.jpg" alt="waHYW.jpg" border="0" style="zoom:7%;" />](#LeetCode) | [<img src="https://i.im5i.com/2021/05/02/waj7G.jpg" alt="waj7G.jpg" border="0" style="zoom:4%;" />](#计算机网络) | [<img src="https://i.im5i.com/2021/05/02/waWPz.png" alt="waWPz.png" border="0" style="zoom:8%;" />](#操作系统) | [<img src="https://i.im5i.com/2021/05/02/waXT5.jpg" alt="waXT5.jpg" border="0" style="zoom:5%;" />](#设计模式) | [<img src="https://i.im5i.com/2021/05/24/C2z7t.png" alt="C2z7t.png" border="0" style="zoom:20%;" />](#框架) | [<img src="https://i.im5i.com/2021/06/06/qd2C3.jpg" alt="qd2C3.jpg" border="0" style="zoom:10%;" />](#工具) | [<img src="https://ss.im5i.com/2021/08/24/fFunw.jpg" alt="fFunw.jpg" border="0" style="zoom:4%;" />](#系统设计) |

本项目基于[Docsify](https://docsify.js.org/#/)构建，可通过以下站点访问：

- GitHub Pages : https://zexylitol.github.io/JavaSummary/



> Tips : 
>
> 1. 善用Ctrl+F，搜索关键字
> 2. GitHub Flavored Markdown不支持LaTex，部分文章中的公式显示为源码，为了不影响阅读建议通过 https://zexylitol.github.io/JavaSummary/ 访问，或者安装chrome的插件[MathJax Plugin for Github](https://chrome.google.com/webstore/detail/mathjax-plugin-for-github/ioemnmodlmafdkllaclgeombjnmnbima)在个人浏览器解析LaTex公式



# Java

## Java基础

- [访问控制修饰符](docs/Java基础/访问控制修饰符.md)

- [关于 == 与 equasl()](docs/Java基础/关于==与equasl().md)
- [String、StringBuffer、StringBuilder的区别](docs/Java基础/String、StringBuffer、StringBuilder的区别.md)
- [抽象类与接口](docs/Java基础/抽象类与接口.md)
- [Java中Comparable和Comparator的区别](docs/Java基础/Java中Comparable和Comparator的区别.md)
- [Java包装类详解（一）](docs/Java基础/Java包装类详解（一）.md)
- [Java包装类详解（二）](docs/Java基础/Java包装类详解（二）.md)
- [反射机制](docs/Java基础/Java反射机制.md)
- 注解
  - [注解简介](docs/Java基础/注解简介.md)
  - [解析注解](docs/Java基础/解析注解.md)

## Java容器



## Java并发

- [Java创建线程的4种方式](docs/Java并发/Java创建线程的4种方式.md)
- [Java线程池](docs/Java并发/Java线程池.md)
- [Java线程状态转换](docs/Java并发/Java线程状态转换.md)
- [ConcurrentHashMap源码分析](docs/Java并发/ConcurrentHashMap源码分析.md)
- [AQS源码分析](docs/Java并发/AQS源码分析.md)
- [基于AQS的并发工具类](docs/Java并发/基于AQS的并发工具类.md)
- [synchronized原理](docs/Java并发/synchronized原理.md)
- [synchronized与ReentrantLock对比](docs/Java并发/synchronized与ReentrantLock对比.md)
- [happens-before](docs/Java并发/happens-before.md)
- [ThreadLocal详解](docs/Java并发/ThreadLocal详解.md)
- 并发编程应用
  - [自定义锁](docs/Java并发/并发编程应用/自定义锁.md)
  - [同步模式之保护性暂停](docs/Java并发/并发编程应用/同步模式之保护性暂停.md)
  - [异步模式之生产者消费者](docs/Java并发/并发编程应用/异步模式之生产者消费者.md)

## JVM

- [虚拟机和Java虚拟机简介](docs/JVM/虚拟机和Java虚拟机简介.md)
- [JVM之类加载子系统](docs/JVM/JVM之类加载子系统.md)
- [JVM之运行时数据区：堆](docs/JVM/JVM之运行时数据区：堆.md)
- [JVM之运行时数据区：方法区](docs/JVM/JVM之运行时数据区：方法区.md)
- [JVM之运行时数据区：程序计数器(PC寄存器)](docs/JVM/JVM之运行时数据区：程序计数器(PC寄存器).md)
- [JVM之运行时数据区：虚拟机栈](docs/JVM/JVM之运行时数据区：虚拟机栈.md)
- [JVM之运行时数据区：本地方法栈](docs/JVM/JVM之运行时数据区：本地方法栈)
- [JVM之运行时数据区总结](docs/JVM/JVM之运行时数据区总结.md)
- [对象的创建及内存布局](docs/JVM/对象的创建及内存布局.md)
- [JVM之垃圾回收：垃圾标记](docs/JVM/JVM之垃圾回收：垃圾标记.md)
- [JVM之垃圾回收：垃圾清除](docs/JVM/JVM之垃圾回收：垃圾清除.md)
- [JVM之垃圾收集器：Serial 与 Serial Old垃圾回收器](docs/JVM/JVM之垃圾收集器：Serial与SerialOld.md)
- [JVM之垃圾收集器：ParNew垃圾回收器](docs/JVM/JVM之垃圾收集器：ParNew垃圾回收器.md)
- [JVM之垃圾收集器：Parallel与Parallel Old垃圾回收器](docs/JVM/JVM之垃圾收集器：Parallel与ParallelOld.md)
- [JVM之垃圾回收器：CMS垃圾回收器](docs/JVM/JVM之垃圾回收器：CMS垃圾回收器.md)
- [JVM之垃圾收集器：G1收集器](docs/JVM/JVM之垃圾收集器：G1收集器.md)
- [JVM之垃圾回收器：7种经典垃圾回收器总结](docs/JVM/JVM之垃圾回收器：7种经典垃圾回收器总结.md)
- [JVM之垃圾收集器：革命性的ZGC](docs/JVM/JVM之垃圾收集器：革命性的ZGC.md)
- [JVM之MinorGC、MajorGC和FullGC的对比](docs/JVM/JVM之MinorGC、MajorGC和FullGC的对比.md)
- [G1与CMS的对比](docs/JVM/G1与CMS的对比.md)
- [Java中几种不同引用](docs/JVM/Java中几种不同引用.md)
- [JVM面试题](docs/JVM/JVM面试题.md)



# 数据库

- [重学SQL](docs/数据库/重学SQL.md)
- [三大范式](docs/数据库/三大范式.md)
- [JDBC核心技术](docs/数据库/JDBC核心技术.md)
- [事务](docs/数据库/事务.md)
- [分布式事务](docs/数据库/分布式事务.md)
- [索引（一）](docs/数据库/索引（一）.md)
- [索引（二）](docs/数据库/索引（二）.md)
- [连接的原理](docs/数据库/连接的原理.md)
- [MySQL锁机制](docs/数据库/MySQL锁机制.md)
- [MySQL乐观锁与悲观锁应用](docs/数据库/MySQL乐观锁与悲观锁应用.md)
- [基于数据库的分布式锁](docs/数据库/基于数据库的分布式锁.md)
- [MySQL日志文件](docs/数据库/MySQL日志文件.md)
- [读写分离](docs/数据库/读写分离.md)
- [分库分表](docs/数据库/分库分表.md)
- [慢SQL排查](docs/数据库/慢SQL排查.md)
- [慢SQL优化](docs/数据库/慢SQL优化.md)
- [explain性能分析](docs/数据库/explain性能分析.md)
- Redis
  - [Redis简介](docs/数据库/Redis/Redis简介.md)
  - [Redis中的数据结构](数据库/Redis/redis中的数据结构.md)
  - [缓存雪崩、击穿、穿透](docs/数据库/Redis/缓存雪崩、击穿、穿透.md)
  - [使用Redis实现分布式锁](docs/数据库/Redis/使用Redis实现分布式锁.md)
  - [Redis内存淘汰策略](docs/数据库/Redis/Redis内存淘汰策略.md)
  - [Redis持久化：AOF](docs/数据库/Redis/Redis持久化：AOF.md)
  - [Redis持久化：RDB](docs/数据库/Redis/Redis持久化：RDB.md)
  - [Redis主从复制](docs/数据库/Redis/Redis主从复制.md)
  - [Redis哨兵机制](docs/数据库/Redis/Redis哨兵机制.md)
  - [Redis面试题](docs/数据库/Redis/Redis面试题.md)
- [BST、AVL、红黑树、B树、B+树](docs/数据库/BST、AVL、红黑树、B树、B+树.md)
- [数据库和缓存一致性方案解析](docs/数据库/数据库和缓存一致性方案解析.md)
- [数据库面试题](docs/数据库/数据库面试题.md)

# LeetCode

- [小技巧](docs/LeetCode/小技巧.md)
- [动态规划](LeetCode/动态规划/动态规划.md)
  - 背包问题
    - [01背包问题](docs/LeetCode/动态规划/01背包问题.md)
    - [完全背包问题](docs/LeetCode/动态规划/完全背包问题.md)
    - [多重背包问题I](docs/LeetCode/动态规划/多重背包问题I.md)
    - [多重背包问题II](docs/LeetCode/动态规划/多重背包问题II.md)
    - [混合背包问题](docs/LeetCode/动态规划/混合背包问题.md)
    - [二维费用的背包问题](docs/LeetCode/动态规划/二维费用的背包问题)
    - [分组背包问题](docs/LeetCode/动态规划/分组背包问题.md)
- [二叉树](docs/LeetCode/二叉树.md)
- [链表](docs/LeetCode/链表.md)
- [二分](docs/LeetCode/二分.md)
- [数学](docs/LeetCode/数学.md)
- [栈](docs/LeetCode/栈.md)
- [队列](docs/LeetCode/队列.md)
- [双指针](docs/LeetCode/双指针.md)
- [滑动窗口](docs/LeetCode/滑动窗口.md)
- [原地哈希](docs/LeetCode/原地哈希.md)
- [前缀和](docs/LeetCode/前缀和.md)
- [回溯](docs/LeetCode/回溯.md)
- [贪心](docs/LeetCode/贪心.md)
- [BFS](docs/LeetCode/BFS.md)
- [DFS](docs/LeetCode/DFS.md)
- [排序](docs/LeetCode/排序.md)
- [模拟](docs/LeetCode/模拟.md)
- [图](docs/LeetCode/图.md)
- [字典树](docs/LeetCode/字典树.md)

# 计算机网络

- [TCP/IP与OSI参考模型](docs/计算机网络/TCP-IP与OSI参考模型.md)
- [IPv4首部](docs/计算机网络/IPv4首部)
- [Socket](docs/计算机网络/Socket)
- [TCP三次握手及四次挥手](docs/计算机网络/TCP三次握手及四次挥手.md)
- [TCP的可靠传输](docs/计算机网络/TCP的可靠传输.md)
- [HTTP协议简介](docs/计算机网络/HTTP协议简介.md)
- [浏览器中输入网址到网页展现全过程](docs/计算机网络/浏览器中输入网址到网页展现全过程.md)
- [一台Linux服务器最多能支撑多少个TCP连接？](docs/计算机网络/一台Linux服务器最多能支撑多少个TCP连接.md)

# 操作系统

- [死锁](docs/操作系统/死锁.md)
- [进程、线程、协程](docs/操作系统/进程、线程、协程.md)
- [Linux内核的IO模型详解](docs/操作系统/Linux内核的IO模型详解.md)
- [CPU缓存结构](docs/操作系统/CPU缓存结构.md)

# 设计模式

- [设计模式概述与分类](docs/设计模式/设计模式概述与分类.md)
- [UML图及类图](docs/设计模式/UML图及类图.md)
- [软件设计原则](docs/设计模式/软件设计原则.md)
- 创建型模式
  - [单例模式](docs/设计模式/单例模式.md)
  - [工厂模式](docs/设计模式/工厂模式.md)
  - [原型模式](docs/设计模式/原型模式.md)
  - [建造者模式](docs/设计模式/建造者模式.md)
  - [创建者模式对比](docs/设计模式/创建者模式对比.md)
- [结构型模式](docs/设计模式/结构型模式.md)
  - [代理模式](docs/设计模式/代理模式.md)
  - [适配器模式](docs/设计模式/适配器模式.md)
  - [装饰者模式](docs/设计模式/装饰者模式.md)
  - [桥接模式](docs/设计模式/桥接模式.md)
  - [外观模式](docs/设计模式/外观模式.md)
  - [组合模式](docs/设计模式/组合模式.md)
  - [享元模式](docs/设计模式/享元模式.md)
- [行为型模式](docs/设计模式/行为型模式/行为型模式.md)
  - [模板方法模式](docs/设计模式/行为型模式/模板方法模式.md)
  - [策略模式](docs/设计模式/行为型模式/策略模式.md)
  - [命令模式](docs/设计模式/行为型模式/命令模式.md)
  - [责任链模式](docs/设计模式/行为型模式/责任链模式.md)
  - [状态模式](docs/设计模式/行为型模式/状态模式.md)
  - [观察者模式](docs/设计模式/行为型模式/观察者模式.md)

#  框架

- [JavaEE项目的三层架构](docs/框架/JavaEE项目的三层架构.md)
- [软件架构的演进](docs/框架/软件架构的演进.md)
- [RPC由浅入深](docs/框架/RPC由浅入深/RPC由浅入深.md)
  - [Netty](docs/框架/RPC由浅入深/Netty.md)
  - [序列化和反序列化](docs/框架/RPC由浅入深/序列化和反序列化.md)
- Mybatis
  - [Mybatis入门](docs/框架/Mybatis/Mybatis入门.md)
  - [Mybatis的XML配置解析](docs/框架/Mybatis/Mybatis的XML配置解析.md)
  - [Mybatis之生命周期和作用域](docs/框架/Mybatis/Mybatis之生命周期和作用域.md)
  - [Mybatis之日志](docs/框架/Mybatis/Mybatis之日志.md)
  - [Mybatis之注解开发](docs框架/Mybatis/Mybatis之注解开发.md)
  - [Mybatis执行流程分析](docs/框架/Mybatis/Mybatis执行流程分析.md)
  - [Mybatis之ResultMap](docs/框架/Mybatis/Mybatis之ResultMap.md)
  - [Mybatis之动态SQL](docs/框架/Mybatis/Mybatis之动态SQL.md)
- Spring
  - AOP
    - [Spring的AOP简介](docs/框架/Spring/AOP/Spring的AOP简介.md)
    - [基于XML的AOP开发](docs/框架/Spring/AOP/基于XML的AOP开发.md)
    - [基于注解的AOP开发](docs/框架/Spring/AOP/基于注解的AOP开发.md)
- 微服务
  - [服务发现](docs/框架/微服务/服务发现.md)
  - [配置管理](docs/框架/微服务/配置管理.md)
  - [负载均衡](docs/框架/微服务/负载均衡.md)
- kafka
  - [kafka术语](docs/框架/kafka/kafka术语.md)

# 系统设计

- [接口幂等性讨论](docs/系统设计/接口幂等性讨论.md)
- [JavaWeb](docs/系统设计/JavaWeb.md)
- [电商系统](docs/系统设计/电商系统.md)
- [关于秒杀](docs/系统设计/关于秒杀.md)

# 工具

- [Linux命令](docs/工具/Linux命令.md)
- [Git命令](docs/工具/Git命令.md)
- [IntelliJ IDEA必备插件](docs/工具/IntelliJIDEA必备插件.md)
- [IntelliJ IDEA快捷键](docs/工具/IDEA快捷键.md)
- [Arthas](docs/工具/Arthas.md)













































