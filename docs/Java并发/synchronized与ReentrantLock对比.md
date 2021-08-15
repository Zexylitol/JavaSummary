JDK 5.0为开发人员开发高性能的并发应用程序提供了一些很有效的新选择，目前存在两种锁机制：synchronized和Lock，Lock接口及其

实现类是JDK5增加的内容

[ReentrantLock](http://www.yidianzixun.com/channel/w/reentrantlock)是JDK1.5引入的，它拥有与`synchronized`相同的并发性和内存语义，并提供了超出`synchonized`的其他高级功能(例如，中断锁等候、条件变量等)，并且使用`ReentrantLock`比`synchronized`能获得更好的可伸缩性。

# 对比

## ReentrantLock

ReentrantLock的实现基于AQS(AbstractQueuedSynchronizer)和[Lock](http://www.yidianzixun.com/channel/w/lock)Support。

**AQS主要利用硬件原语指令(CAS compare-and-swap)，来实现轻量级多线程同步机制，并且不会引起CPU上文切换和调度，同时提供内存可见性和原子化更新保证**(线程安全的三要素：原子性、可见性、有序性)。

AQS的本质上是一个同步器/阻塞锁的基础框架，其作用主要是提供加锁、释放锁，并在内部维护一个FIFO等待队列，用于存储由于锁竞争而阻塞的线程。

`ReentrantLock`具备如下特点：

- 可中断
- 可以设置超时时间
- 可以设置为公平锁，默认非公平锁
- 支持多条件变量
- 与`synchronized`一样，都支持可重入
- `ReentrantLock`需创建对象

## synchronized

`synchronized`是基于jvm底层实现的数据同步的，具体而言，JAVA对象头和monitor是synchronized实现的基础；提供了互斥的语义和可见性，属于可重入锁，当一个线程获取当前互斥锁时，其他线程只能等待或者阻塞；常用于修饰方法，也可以修饰同步代码块。

synchronized是重量级锁，在JDK1.5之前synchronized是仅有的同步锁手段，JDK1.6对synchronized进行优化，为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁，以及锁的存储结构和升级过程。

`synchronized`具备如下特点：

- 不可中断（不可中断的意思是等待获取锁的时候不可中断，拿到锁之后可中断，没获取到锁的情况下，中断操作一直不会生效）
- 非公平锁
- 可重入
- `synchronized`是关键字

**区别：**

- `synchronized`锁会自带释放锁，无须用户自己执行释放锁操作，而`ReentrantLock`需要执行lock，unlock操作。
- `ReentrantLock`支持更多锁原语操作，比如锁获取，锁中断等待等操作，这些比较高级的属性在`synchronized`上是没有的。
- <span style="color:red">在性能上，`ReentrantLock`锁在锁高竞争条件下会展现出更好的性能，相较于`synchronized`锁</span>。

但是这两种锁也有部分属性是相同的，比如说**默认都是非公平调度策略**的。换句话说，就是同样多线程执行请求锁操作，最终结果不会是FIFO顺序来获得锁的。其实在多并发竞争的条件下，公平策略未必是一定必须的，为了保持这种顺序性，系统的开销还是存在的，所以`synchronized`锁和默认的`ReentrantLock`都是unfair策略的。

# 原理与特点

***synchronized******：***synchronized是基于jvm底层实现的数据同步的，具体而言，JAVA对象头和monitor是Synchronized实现的基础；提供了互斥的语义和可见性，属于可重入锁，当一个线程获取当前互斥锁时，其他线程只能等待或者阻塞；常用于修饰方法，也可以修饰同步代码块。

synchronized是重量级锁，在JDK1.5之前synchronized是仅有的同步锁手段，JDK1.6对synchronized进行优化，为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁，以及锁的存储结构和升级过程。

***ReentrantLock*** ，ReentrantLock是基于AQS实现的互斥锁，AQS是Java并发包中众多同步组件的构建基础；支持公平锁和非公平锁，属于可重入锁，可重入的意思是一个线程获得锁之后，可以再次获得锁，不会出现阻塞自身的情况。

相比原生的Synchronized，ReentrantLock增加了一些高级的扩展功能，比如它可以实现公平锁，同时也可以绑定多个Conditon。

# 源码分析

***synchronized*** ：synchronized是一种隐式锁 ，可以保证变量的原子性，可见性和顺序性。

（1）**从字节码层面**：synchronized基于Monitor对象实现

如果修饰的是同步代码块，采用monitorenter和monitorexit实现锁的获取和释放过程，monitorenter指令插入到同步代码块的开始位置，monitorexit指令插入到同步代码块结束的位置，monitorenter, monitorexit的指令解析是通过 InterpreterRuntime.cpp中的两个方法实现；

如果修饰的是方法，采用ACC_SYNCHRONIZED标志实现锁的获取和释放过程。

（2）**synchronized的优化**

JDK1.6对synchronized做了优化，增加了偏向锁，轻量级锁，锁粗化，锁消除，适应性自旋等操作。

以synchronized修饰**同步代码**为例，JVM将字节码加载到内存以后，解释器执行**monitorenter**时会进入到InterpreterRuntime.cpp的InterpreterRuntime::monitorenter函数；然后检查标志UseBiasedLocking，判断是否启动**偏向锁**；如果支持偏向锁，则执行 ObjectSynchronizer::fast_enter的逻辑，如果不支持偏向锁，则执行 ObjectSynchronizer::slow_enter逻辑，直接进入**轻量级锁**；有多个线程竞争轻量级锁，轻量级锁需要膨胀升级为**重量级锁**，重量级锁是通过对象内部的监视器(monitor)来实现，而monitor的本质是依赖操作系统底层的MutexLock实现的。

***ReentrantLock*** ：与synchronized不同，**ReentrantLock是纯JAVA实现的，与底层JVM没有直接关联**。显式锁ReentrantLock和同步工具类的实现基础都是AQS，AQS的实现基础是CAS，通过实现一种自旋锁，循环调用CAS操作来实现加锁。

AQS是个底层框架，采用模板方法模式，它定义了通用的较为复杂的逻辑骨架，围绕state提供两种基本操作“获取”和“释放”，有条双向队列存放阻塞的等待线程，并提供一系列判断和处理方法；将这些复杂但实质通用的部分抽取出来，，使用者仅需重写一些简单的指定的方法即可。

**常用方法**：

（1）ReentrantLock 分为公平锁和非公平锁，可以通过构造方法来指定具体类型：

 `public ReentrantLock()`  *默认非公平锁*

 `public ReentrantLock(boolean fair)` *可实现公平锁，fair为true，公平锁*



（2）锁操作

 ```java
public void lock()  //lock方法中调用sync的lock方法获取锁
 ```

```java
public boolean tryLock()   //尝试获取锁，成功则直接返回true，不成功立即返回false
```

```java
public void unlock()  //调用sync的release方法释放锁
```

（3）获取Condition

```java
public Condition newCondition()  //ReentrantLock支持多个Condition
```

**使用场景**

***Synchronized：***JAVA中每个对象都可以作为锁

（1）对于同步方法，锁是当前实例对象。
（2）对于静态同步方法，锁是当前对象的Class对象
（3）对于同步方法块，锁是synchonized括号里配置的对象

***ReentrantLock***：

（1）支持公平和非公平锁，防止资源使用冲突

（2）支持中端锁，取消某些操作对资源的占用

（3）对锁的细粒度和灵活度有要求的场景

# 分析比较

相同点：

（1）都是可重入锁，同一线程可以多次获得同一个锁

（2）都保证了可见性和互斥性

不同点：

（1）ReentrantLock是API级别的，synchronized是JVM级别的；

（2）ReentrantLock是显示的获得和释放锁，synchronized隐式的获得和释放锁；

（3）ReentrantLock可以实现公平锁，而synchronized只能是非公平锁；

（4）ReentrantLock可响应中断，通过lock.lockInterruptibly()来实现这个机制；

（5）ReentrantLock通过Condition可以绑定多个条件，可用来实现分组唤醒需要唤醒的线程。

# 使用场景

`synchronized`：

在资源竞争不是很激烈的情况下，偶尔会有同步的情形下，`synchronized`是很合适的。原因在于，编译程序通常会尽可能的进行优化，另外`synchronized`可读性非常好，不管用没用过5.0多线程包的程序员都能理解。

`ReentrantLock`：

可以等同于`synchronized`使用，但是比`synchronized`有更强的功能、可以提供更灵活的锁机制、同时减少死锁的发生概率。

**既然ReentrantLock在高竞争条件下拥有着更好的性能，而且它还支持了更多的高级属性，那么这是不是就意味着我们永远就使用ReentrantLock锁而完全不考虑synchronized锁了呢？**

其实，归根结底一句话：**你真正要用到哪个锁的属性时，那就去用哪种锁**。比如说，你要使用到能支持锁中断的操作，那么这时就用ReentrantLock锁。还有一点，也不是说ReentrantLock性能会比synchronized好，就盲目的都使用ReentrantLock替换现有的synchronized锁，**只有你真正证明出在当前的场景下，synchronized关键字锁存在锁竞争性能问题，然后再去换**。但是大部分的普通情况下，synchronized和ReentrantLock锁所展现出的性能是相差无几的。

而且另外一方面，synchronized锁也有着它天然的优势，被更多的开发者所熟知和使用。而且用起来比较简单，方便，比如说它无需使用者执行释放锁操作，有的时候新手用户在使用类似ReentrantLock锁的时候，忘记了执行unlock操作，导致最后程序出现各种奇怪的问题，而且还难以定位。总而言之，ReentrantLock是一种比较“高级”的锁，比较适合“高级”地去使用。

# Reference

- [简单聊聊Synchronized和ReentrantLock锁](https://www.cnblogs.com/bianqi/p/12183632.html)
- [synchronized和ReentrantLock的区别和使用](https://mp.weixin.qq.com/s/7JnYqTCqtM7kePqrTB8ZSQ)
- [synchronized不可中断](https://cloud.tencent.com/developer/article/1622813)