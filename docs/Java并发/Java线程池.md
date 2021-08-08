# 1 前言

## 1.1 线程池是什么

**线程池（Thread Pool）是一种基于池化思想管理线程的工具，经常出现在多线程服务器中，如MySQL。**

使用线程池可以带来一系列好处：

1. **降低资源消耗**：通过池化技术重复利用已创建的线程，降低线程创建和销毁造成的损耗。
2. **提高响应速度**：任务到达时，无需等待线程创建即可立即执行。
3. **提高线程的可管理性**：线程是稀缺资源，如果无限制创建，不仅会消耗系统资源，还会因为线程的不合理分布导致资源调度失衡，降低系统的稳定性。使用线程池可以进行统一的分配、调优和监控。
4. **提供更多更强大的功能**：线程池具备可拓展性，允许开发人员向其中增加更多的功能。比如延时定时线程池ScheduledThreadPoolExecutor，就允许任务延期执行或定期执行。

## 1.2 线程池解决的问题是什么

线程池解决的核心问题就是**资源管理**问题。在并发环境下，系统不能够确定在任意时刻中，有多少任务需要执行，有多少资源需要投入。这种不确定性将带来以下若干问题：

1. 频繁申请/销毁资源和调度资源，将带来额外的消耗，可能会非常巨大。
2. 对资源无限申请缺少抑制手段，易引发系统资源耗尽的风险。
3. 系统无法合理管理内部的资源分布，会降低系统的稳定性。

为解决资源分配这个问题，线程池采用了“池化”（Pooling）思想。池化，顾名思义，是为了最大化收益并最小化风险，而将资源统一在一起管理的一种思想。

> Pooling is the grouping together of resources (assets, equipment, personnel, effort, etc.) for the purposes of maximizing advantage or minimizing risk to the users. The term is used in finance, computing and equipment management.——wikipedia

“池化”思想不仅仅能应用在计算机领域，在金融、设备、人员管理、工作管理等领域也有相关的应用。

在计算机领域中的表现为：统一管理IT资源，包括服务器、存储、和网络资源等等。通过共享资源，使用户在低投入中获益。除去线程池，还有其他比较典型的几种使用策略包括：

1. **内存池(Memory Pooling)**：预先申请内存，提升申请内存速度，减少内存碎片。
2. **连接池(Connection Pooling)**：预先申请数据库连接，提升申请连接的速度，降低系统的开销。
3. **实例池(Object Pooling)**：循环使用对象，减少资源在初始化和释放时的昂贵损耗。

# 2 ThreadPoolExecutor

线程池是一种通过“池化”思想，帮助管理线程而获取并发性的工具，在Java中的体现是`ThreadPoolExecutor`类

## 2.1 总体设计

Java中的线程池核心实现类是`ThreadPoolExecutor`，JDK 1.8中`ThreadPoolExecutor`类的UML类图如下：

<center><img src="https://ss.im5i.com/2021/08/08/PT2AL.png" alt="PT2AL.png" border="0" /></center>

`ThreadPoolExecutor`实现的顶层接口是`Executor`：

- 顶层接口`Executor`提供了一种思想：**将任务提交和任务执行进行解耦**。用户无需关注如何创建线程，如何调度线程来执行任务，用户只需提供`Runnable`对象，将任务的运行逻辑提交到执行器(`Executor`)中，由`Executor`框架完成线程的调配和任务的执行部分。
- `ExecutorService`接口增加了一些能力：（1）扩充执行任务的能力，补充可以为一个或一批异步任务生成`Future`的方法；（2）提供了管控线程池的方法，比如停止线程池的运行。
- `AbstractExecutorService`则是上层的抽象类，**将执行任务的流程串联了起来**，保证下层的实现只需关注一个执行任务的方法即可。最下层的实现类`ThreadPoolExecutor`实现最复杂的运行部分，`ThreadPoolExecutor`将会一方面维护自身的生命周期，另一方面同时管理线程和任务，使两者良好的结合从而执行并行任务。

## 2.1 线程池状态

线程池运行的状态，并不是用户显式设置的，而是伴随着线程池的运行，由内部来维护。线程池内部使用一个变量维护两个值：运行状态(runState，高3位)和线程数量 (workerCount，低29位)。在具体实现中，线程池将运行状态(runState)、线程数量 (workerCount)两个关键参数的维护放在了一起：

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

`ctl`这个AtomicInteger类型，是对线程池的运行状态和线程池中有效线程的数量进行控制的一个字段， 它同时包含两部分的信息：**线程池的运行状态 (runState) 和线程池内有效线程的数量 (workerCount)**，两个变量之间互不干扰。用一个变量去存储两个值，可避免在做相关决策时，出现不一致的情况，不必为了维护两者的一致，而占用锁资源。线程池也提供了若干方法去供用户获得线程池当前的运行状态、线程个数。这里都使用的是位运算的方式，相比于基本运算，速度也会快很多。

```java
//计算当前运行状态
private static int runStateOf(int c)     { return c & ~CAPACITY; } 
//计算当前线程数量
private static int workerCountOf(int c)  { return c & CAPACITY; }  
//通过状态和线程数生成ctl
private static int ctlOf(int rs, int wc) { return rs | wc; }   
```

`ThreadPoolExecutor`的运行状态有5种，分别为：

| 运行状态       | 高3位 | 是否接收新任务 | 是否处理阻塞队列任务 | 状态描述                                                     |
| -------------- | :---: | :------------: | :------------------: | :----------------------------------------------------------- |
| **RUNNING**    |  111  |       Y        |          Y           | 能接受新提交的任务，并且也能处理阻塞队列中的任务。           |
| **SHUTDOWN**   |  000  |       N        |          N           | 关闭状态，不再接受新提交的任务，但却可以继续处理阻塞队列中已保存的任务。 |
| **STOP**       |  001  |       N        |          N           | 不能接受新任务，也不处理队列中的任务，会中断正在处理任务的线程。 |
| **TIDYING**    |  010  |       -        |          -           | 所有的任务都已终止了，workerCount (有效线程数) 为0           |
| **TERMINATED** |  011  |       -        |          -           | 在terminated()方法执行完后进入该状态                         |

线程池生命周期转换如下：

<center><img src="https://i.loli.net/2021/03/30/HzMavFOZPqNEjoe.png"/></center>

## 2.2 构造方法

```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {}
```

- `corePoolSize`：核心线程数
- `maximumPoolSize`：最大线程数=核心线程数+救急线程数
- `keepAliveTime`：生存时间，针对救急线程
- `unit`：时间单位，针对救急线程
- `workQueue`：阻塞队列
- `threadFactory`：线程工厂，为线程创建时起个好名字，将线程池中的线程与其他线程区分开来
- `handler`：拒绝策略

>1. 救急线程被使用的前提是使用的是有界队列
>2. 核心线程与救急线程都是懒惰创建，用到时才创建，节省资源
>3. 核心线程与救急线程的区别：救急线程有存活时间

## 2.3 任务调度

任务调度是线程池的主要入口，当用户提交了一个任务，接下来这个任务将如何执行都是由这个阶段决定的。了解这部分就相当于了解了线程池的核心运行机制。

首先，所有任务的调度都是由`execute`方法完成的，这部分完成的工作是：检查现在线程池的运行状态、运行线程数、运行策略，决定接下来执行的流程，是直接申请线程执行，或是缓冲到队列中执行，亦或是直接拒绝该任务。其执行过程如下：

1. 首先检测线程池运行状态，如果不是`RUNNING`，则直接拒绝，线程池要保证在`RUNNING`的状态下执行任务。
2. 如果`workerCount < corePoolSize`，则创建并启动一个线程来执行新提交的任务。
3. 如果`workerCount >= corePoolSize`，且线程池内的阻塞队列未满，则将任务添加到该阻塞队列中。
4. 如果`workerCount >= corePoolSize && workerCount < maximumPoolSize`，且线程池内的阻塞队列已满，则创建并启动一个线程来执行新提交的任务。
5. 如果`workerCount >= maximumPoolSize`，并且线程池内的阻塞队列已满, 则根据拒绝策略来处理该任务, 默认的处理方式是直接抛异常。

其执行流程如下图所示：

<center><img src="https://i.loli.net/2021/03/30/KTSnQdE2bx1geDU.png"/></center>

## 2.4 阻塞队列

线程池中是以生产者消费者模式，通过一个阻塞队列来实现的。阻塞队列缓存任务，工作线程从阻塞队列中获取任务。

| 名称                    | 描述                                                         |
| :---------------------- | :----------------------------------------------------------- |
| `ArrayBlockingQueue`    | 一个用数组实现的**有界**阻塞队列，此队列按照先进先出(FIFO)的原则对元素进行排序。支持公平锁和非公平锁。 |
| `LinkedBlockingQueue`   | 一个由链表结构组成的**有界**队列，此队列按照先进先出(FIFO)的原则对元素进行排序。此队列的默认长度为`Integer.MAX_VALUE`，所以默认创建的该队列有容量危险。<br/>如果不指定容量，默认为`Integer.MAX_VALUE`，也就是无界队列。所以为了避免队列过大造成机器负载或者内存爆满的情况出现，在使用的时候建议手动传一个队列的大小 |
| `PriorityBlockingQueue` | 一个支持线程优先级排序的**无界**队列，默认自然序进行排序，也可以自定义实现compareTo()方法来指定元素排序规则，不能保证同优先级元素的顺序。 |
| `DelayQueue`            | 一个实现PriorityBlockingQueue实现延迟获取的无界队列，在创建元素时，可以指定多久才能从队列中获取当前元素。只有延时期满后才能从队列中获取元素。 |
| `SynchronousQueue`      | **一个不存储元素的阻塞队列，每一个put操作必须等待take操作，否则不能添加元素**。支持公平锁和非公平锁。`SynchronousQueue`的一个使用场景是在线程池里。`Executors.newCachedThreadPool()`就使用了`SynchronousQueue`，这个线程池根据需要（新任务到来时）创建新的线程，如果有空闲线程则会重复使用，线程空闲了60秒后会被回收。 |
| `LinkedTransferQueue`   | 一个由链表结构组成的**无界**阻塞队列，相当于其它队列，`LinkedTransferQueue`队列多了`transfer`和`tryTransfer`方法。 |
| `LinkedBlockingDeque`   | 一个由链表结构组成的双向阻塞队列。队列头部和尾部都可以添加和移除元素，多线程并发时，可以将锁的竞争最多降到一半。 |

## 2.5 拒绝策略

拒绝策略是线程池的保护部分，线程池有一个最大的容量，当线程池的任务缓存队列已满，并且线程池中的线程数目达到`maximumPoolSize`时，就需要拒绝掉该任务，采取任务拒绝策略，保护线程池。

拒绝策略是一个接口，其设计如下：

```java
public interface RejectedExecutionHandler {
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```

用户可以通过实现这个接口去定制拒绝策略，也可以选择JDK提供的四种已有拒绝策略：

| 名称                                     | 描述                                                         |
| :--------------------------------------- | :----------------------------------------------------------- |
| `ThreadPoolExecutor.AbortPolicy`         | 丢弃任务并抛出`RejectedExecutionException`异常。 这是线程池**默认的拒绝策略**，在任务不能再提交的时候，抛出异常，及时反馈程序运行状态。如果是比较关键的业务，推荐使用此拒绝策略，这样子在系统不能承载更大的并发量的时候，能够及时的通过异常发现。 |
| `ThreadPoolExecutor.DiscardPolicy`       | **丢弃任务，但是不抛出异常**。 使用此策略，可能会使我们无法发现系统的异常状态。建议是一些无关紧要的业务采用此策略。 |
| `ThreadPoolExecutor.DiscardOldestPolicy` | **丢弃队列最前面的任务，然后重新提交被拒绝的任务**。是否要采用此种拒绝策略，还得根据实际业务是否允许丢弃老任务来认真衡量。<br/>**放弃队列中最早的任务，本任务取而代之** |
| `ThreadPoolExecutor.CallerRunsPolicy`    | **由调用线程（提交任务的线程）处理该任务**。这种情况是需要让所有任务都执行完毕，那么就适合大量计算的任务类型去执行，多线程仅仅是增大吞吐量的手段，最终必须要让每个任务都执行完毕。<br/>**让调用者运行任务** |

# 3 newFixedThreadPool

- 固定大小线程池，无救急线程

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

|      | 说明                                                         |
| :--: | :----------------------------------------------------------- |
| 特点 | 核心线程数==最大线程数，没有救急线程，因此也无需超时时间<br/>阻塞队列是无界的，可以放任意数量的任务 |
| 评价 | 适用于任务量已知，相对耗时的任务                             |

# 4 newCachedThreadPool

- 只要有任务就创建新的线程，都是救急线程

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

|      | 说明                                                         |
| :--: | :----------------------------------------------------------- |
| 特点 | 核心线程数是0，最大线程数是`Integer.MAX_VALUE`，全部都是救急线程，救急线程的空闲生存时间是60s（60s后可以回收），救急线程可以无限创建<br/>队列采用了`SynchronousQueue`，特点是没有容量，没有线程来取是放不进来的 |
| 评价 | 整个线程池表现为线程数会根据任务量不断增长，没有上限，当任务执行完毕，空闲1分钟后释放线程<br/><span style="color:red">适合任务数比较密集，但每个任务执行时间较短的情况</span> |

# 5 newSingleThreadExector

- 使用场景：任务之间串行

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

|          | 说明                                                         |
| :------: | :----------------------------------------------------------- |
| 使用场景 | 希望多个任务排队执行，线程数固定为1，任务数多于1时，会放入无界队列排队，任务执行完毕，这唯一的线程也不会被释放 |
|   区别   | 1. 自己创建一个单线程串行执行任务，如果任务执行失败而终止那么没有任何补救措施，而线程池还会新创建一个线程，保证池的正常工作，保证线程池中始终有一个可用线程<br/>2. `Executors.newSingleThreadExecutor()`线程个数始终为1，不能修改<br/> `FinalizableDelegatedExecutorService`应用的是装饰器模式，只对外暴露了`ExecutorService`接口，因此不能调用`ThreadPoolExecutor` 中特有的方法<br/>3. `Executors.newFixedThreadPool(1)`初始时为1，以后还可以修改<br/>对外暴露的是`ThreadPoolExecutor`对象，可以强转后调用`setCorePoolSize`等方法进行修改 |

# 6 提交任务

```java
//执行任务
void execute( Runnable command);
//提交任务task，用返回值 Future获得任务执行结果
<T> Future<T> submit(callable<T> task);
//提交tasks 中所有任务
<T> List<Future<T>> invokeAll(Collection<? extends callable<T>> tasks)
	throws InterruptedException;
//提交tasks中所有任务，带超时时间
<T> List<Future<T>> invokeAll(Collection<? extends callable<T>> tasks,long timeout，TimeUnit unit) throws InterruptedException;
//提交 tasks中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消
<T> T invokeAny(collection< ? extends callable<T>> tasks) throws InterruptedException，ExecutionException;
//提交 tasks中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消，带超时时间
<T> T invokeAny(collection< ? extends callable<T>> tasks,long timeout，TimeUnit unit)
throws InterruptedException，ExecutionException，TimeoutException;
```

# 7 关闭线程池

```java
// 线程池状态变为 SHUTDOWN
void shutdown();
// 线程池状态变为 STOP
List<Runnable> shutdownNow();
```

# Reference

- [Java线程池实现原理及其在美团业务中的实践](https://blog.csdn.net/MeituanTech/article/details/105283415?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162843250816780357210770%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=162843250816780357210770&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-105283415.pc_v2_rank_blog_default&utm_term=%E7%BA%BF%E7%A8%8B%E6%B1%A0&spm=1018.2226.3001.4450)
- [黑马程序员全面深入学习Java并发编程](https://www.bilibili.com/video/BV16J411h7Rd)













