
# 重量级锁

## Monitor

- **每个Java对象都可以关联一个Monitor对象**，使用synchronized给对象上锁（重量级）之后，该对象头的Mark Word中就被设置指向Monitor对象的指针
- Monitor结构如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401104225726.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)

## 加锁/解锁

- 刚开始 Monitor 中 Owner 为 null  
- 当 Thread-2 执行 `synchronized(obj)` 就会将 Monitor 的所有者 Owner 置为 Thread-2，Monitor中只能有一个 Owner  
- 在 Thread-2 上锁的过程中，如果 Thread-3，Thread-4，Thread-5 也来执行 `synchronized(obj)`，就会进入EntryList BLOCKED  
- Thread-2执行完同步代码块的内容，然后唤醒EntryList中等待的线程来竞争锁，**竞争时是非公平的**
- WaitSet中的Thread-0、Thread-1是之前获得过锁，但条件不满足进入WAITING状态的线程。其过程如下：
  - Owner 线程发现条件不满足，调用 `wait` 方法，即可进入 WaitSet 变为 WAITING 状态  (BLOCKED 和 WAITING 的线程都处于阻塞状态，不占用 CPU 时间片  )
  - BLOCKED 线程会在 Owner 线程释放锁时唤醒  
  - WAITING 线程会在 Owner 线程调用 `notify` 或 `notifyAll` 时唤醒，但唤醒后并不意味者立刻获得锁，仍需进入EntryList 重新竞争  

# 轻量级锁

## 使用场景
如果一个对象虽然有多线程要加锁，**但加锁的时间是错开的（也就是没有竞争）**，那么可以使用轻量级锁来优化

## 加锁流程

- 每个线程的栈帧中会有一个**锁记录(Lock Record)对象**，用来保存锁定对象的Mark Word

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401104456337.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)

- 当一个线程尝试获取锁时，让锁记录中Object Reference指向锁对象，并尝试使用CAS来替换锁对象的Mark Word，将Mark Word的值存入锁记录

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401104540904.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


  - 如果CAS替换成功，对象头中会存储`锁记录地址和状态00`，表示由该线程给对象加锁

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021040110461594.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


  - 如果CAS失败，有两种情况：

    - 如果已经有其他线程持有了该Object的轻量级锁，这时表明有竞争，那么就进入**锁膨胀过程**
    - 如果当前线程自己执行了锁重入，那么就再添加一个锁记录，作为重入计数

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401104708650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


## 解锁流程

- 当退出 synchronized 代码块（解锁时），如果有取值为null的锁记录，表示有重入，这时会重置锁记录，表示重入计数减一

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401104742648.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


- 当退出 synchronized 代码块（解锁时）锁记录的值不为 null，则使用CAS将Mark Word恢复给对象头

  - 成功，则解锁成功
  - 失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁解锁流程

# 锁膨胀

## 场景

- **如果在尝试加轻量级锁的过程中，CAS操作失败，这时一种情况就是有其他线程为此对象加上了轻量级锁（有竞争），这时需要进行锁膨胀，将轻量级锁升级为重量级锁**

## 锁膨胀过程

- 当Thread-1进行轻量级加锁时，Thread-0已经对该对象加了轻量级锁

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401104848995.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


- 这时Thread-1加轻量级锁失败，进入锁膨胀流程

  - 即为Object申请Monitor锁，让Object的Mark Word指向重量级锁地址
  - 然后Thread-1自己进入Monitor的EntryList BLOCKED

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401104941495.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)

- 当Thread-0退出同步块解锁时，使用CAS将Mark Word的值恢复给对象头，失败。这时会进入重量级锁解锁流程，即按照Monitor地址找到Monitor对象，设置为Owner为null，唤醒EntryList中BLOCKED线程

# 自旋优化

## 场景

- 重量级锁竞争的时候，还可以使用**自旋**(多核CPU下才有意义)来进行优化，如果**当前线程自旋成功**（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程就可以**避免阻塞**

## 自旋重试成功的情况

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401105054195.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


## 自旋重试失败的情况

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401105109419.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


## 备注

- 自旋会占用CPU时间，单核CPU自旋就是浪费，多核CPU自旋才能发挥优势
- **在 Java 6 之后自旋锁是自适应的**，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会高，就多自旋几次；反之，就少自旋甚至不自旋，总之，比较智能  
- Java 7 之后不能控制是否开启自旋功能  

# 偏向锁

## 场景

- 轻量级锁在没有竞争时（就自己这个线程），每次重入仍然需要执行CAS操作

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401105204249.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


- Java 6中引入了偏向锁来做进一步优化：只有第一次使用CAS将线程ID设置到对象的Mark Word，之后重入的时候发现这个线程ID是自己的就表示没有竞争，不用重新CAS。以后只要不发生竞争，这个对象就归该线程所有

![在这里插入图片描述](https://img-blog.csdnimg.cn/202104011052314.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


## 偏向状态

- [Java对象头](https://blog.csdn.net/xylitolz/article/details/115160661)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021040110533655.png#pic_center)


- 一个对象创建时：
  - 如果开启了**偏向锁(默认开启)**，那么对象创建后，markword值为0x05即最后3位为101，这时它的thread、epoch、age都为0
  - **偏向锁是默认是延迟的，不会在程序启动时立即生效**，如果想避免延迟，可以加 VM 参数 `-XX:BiasedLockingStartupDelay=0` 来禁用延迟  
  - 如果没有开启偏向锁，那么对象创建后，markword 值为 0x01 即最后 3 位为 001，这时它的 hashcode、age 都为 0，**第一次用到 hashcode 时才会赋值**  

- **处于偏向锁的对象解锁后，线程id仍存储于对象头中**

## 撤销 - 调用对象 hashCode  

- 调用了对象的 hashCode，但偏向锁的对象 MarkWord 中存储的是线程 id，**如果调用 hashCode 会导致偏向锁被撤销**  
  - 轻量级锁会在锁记录中记录 hashCode  
  - 重量级锁会在 Monitor 中记录 hashCode

## 撤销 - 其它线程使用对象

- 当有其它线程使用偏向锁对象时，会将偏向锁升级为轻量级锁  （**多个线程访问同一个对象导致偏向锁被撤销**）

## 撤销 - 调用 wait/notify  

- 升级为重量级锁

## 批量重偏向

- 如果对象虽然被多个线程访问，但没有竞争，这时偏向了线程 T1 的对象仍有机会重新偏向 T2，重偏向会重置对象的 Thread ID
  **当撤销偏向锁阈值超过 20 次后**（刚开始会将偏向锁撤销升级为轻量级锁，第20次时，重新偏向），jvm 会这样觉得，我是不是偏向错了呢，**于是会在给这些对象加锁时重新偏向至加锁线程**  

## 批量撤销

- **当撤销偏向锁阈值超过 40 次后，jvm 会这样觉得，自己确实偏向错了，根本就不该偏向**。于是整个类的所有对象都会变为不可偏向的，新建的对象也是不可偏向的  

# 锁消除

- 在动态编译同步块的时候，**JIT编译器可以借助逃逸分析来判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程**。如果没有，**那么JIT编译器在编译这个同步块的时候就会取消对这部分代码的同步。这样就能大大提高并发性和性能**。这个取消同步的过程就**叫同步省略，也叫锁消除**



# Reference

- [黑马程序员全面深入学习java并发编程](https://www.bilibili.com/video/BV16J411h7Rd?from=search&seid=13009128250474576151)