<center><img src="https://ss.im5i.com/2021/08/12/VqtRw.png" alt="VqtRw.png" border="0" /></center>

# ReentrantLock

`state`初始化为0，表示未锁定状态。A线程`lock()`时，会调用`tryAcquire()`独占该锁并将`state+1`。此后，其他线程再`tryAcquire()`时就会失败，直到A线程`unlock()`到`state=0`（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，**A线程自己是可以重复获取此锁的（`state`会累加），这就是可重入的概念**。但要注意，**获取多少次就要释放多么次**，这样才能保证`state`是能回到零态的。`ReentrantLock` 内部有公平锁和非公平锁两种实现，<span style="color:red">差别就在于新来的线程是否比已经在同步队列中的等待线程更早获得锁</span>。

# Semaphore

和 `ReentrantLock` 实现方式类似，`Semaphore` 也是基于 AQS 的，差别在于 `ReentrantLock` 是独占锁，`Semaphore` 是<span style="color:red">共享锁</span>

`Semaphore`（信号量），**用来限制能同时访问共享资源的线程上限**

## 基本使用

```java
public class semaphoreTest {
    public static void main(String[] args) {
        // 1. 创建 semaphore 对象
        Semaphore semaphore = new Semaphore(3);
        // 2. 10个线程同时运行
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
               // 3. 获取许可
                try {
                    semaphore.acquire();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                try {
                    System.out.println(Thread.currentThread().getName() + " running...");
                    Thread.sleep(1000);
                    System.out.println(Thread.currentThread().getName() + " end...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    // 4. 释放许可
                    semaphore.release();
                }
            }).start();
        }
    }
}
```

运行：

```
Thread-2 running...
Thread-1 running...
Thread-0 running...
Thread-2 end...
Thread-1 end...
Thread-0 end...
Thread-3 running...
Thread-4 running...
Thread-5 running...
Thread-3 end...
Thread-4 end...
Thread-5 end...
Thread-7 running...
Thread-6 running...
Thread-8 running...
Thread-8 end...
Thread-9 running...
Thread-6 end...
Thread-7 end...
Thread-9 end...

Process finished with exit code 0
```

## 应用

### 限制对共享资源的使用

`semaphore` 实现

- 使用`semaphore`限流，在访问高峰期时，让请求线程阻塞，高峰期过去再释放许可，当然**它只适合限制单机线程数量，并且仅是限制线程数，而不是限制资源数**（例如连接数，请对比Tomcat LimitLatch的实现)

## 源码分析

```java
static final class NonfairSync extends Sync {
     private static final long serialVersionUID = -2694183684443567898L;
     NonfairSync(int permits) {
         // permits 即 state
         super(permits);
     }
 
     // Semaphore 方法, 方便阅读, 放在此处
     public void acquire() throws InterruptedException {
     	sync.acquireSharedInterruptibly(1);
     }
     // AQS 继承过来的方法, 方便阅读, 放在此处
     public final void acquireSharedInterruptibly(int arg) throws InterruptedException {
         if (Thread.interrupted())
         	throw new InterruptedException();
         if (tryAcquireShared(arg) < 0)
         	doAcquireSharedInterruptibly(arg);
     }
 
     // 尝试获得共享锁
     protected int tryAcquireShared(int acquires) {
     	return nonfairTryAcquireShared(acquires);
     }
 
     // Sync 继承过来的方法, 方便阅读, 放在此处
     final int nonfairTryAcquireShared(int acquires) {
         for (;;) {
             int available = getState();
             int remaining = available - acquires; 
             if (
                 // 如果许可已经用完, 返回负数, 表示获取失败, 进入 doAcquireSharedInterruptibly
                 remaining < 0 ||
                 // 如果 cas 重试成功, 返回正数, 表示获取成功
                 compareAndSetState(available, remaining)
                  ) {
             	return remaining;
             }
         }
     }
    
     // AQS 继承过来的方法, 方便阅读, 放在此处
     private void doAcquireSharedInterruptibly(int arg) throws InterruptedException {
         final Node node = addWaiter(Node.SHARED);
         boolean failed = true;
         try {
             for (;;) {
                 final Node p = node.predecessor();
                 if (p == head) {
                     // 再次尝试获取许可
                     int r = tryAcquireShared(arg);
                     if (r >= 0) {
                         // 成功后本线程出队（AQS）, 所在 Node设置为 head
                         // 如果 head.waitStatus == Node.SIGNAL ==> 0 成功, 下一个节点 unpark
                         // 如果 head.waitStatus == 0 ==> Node.PROPAGATE 
                         // r 表示可用资源数, 为 0 则不会继续传播
                         setHeadAndPropagate(node, r);
                         p.next = null; // help GC
                         failed = false;
                         return;
                 	}
                 }
                 // 不成功, 设置上一个节点 waitStatus = Node.SIGNAL, 下轮进入 park 阻塞
                 if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                 	throw new InterruptedException();
             }
         } finally {
             if (failed)
             	cancelAcquire(node);
         }
     }
    
     // Semaphore 方法, 方便阅读, 放在此处
     public void release() {
     	sync.releaseShared(1);
     }
    
     // AQS 继承过来的方法, 方便阅读, 放在此处
     public final boolean releaseShared(int arg) {
         if (tryReleaseShared(arg)) {
             doReleaseShared();
             return true;
         }
         return false;
     }
    // Sync 继承过来的方法, 方便阅读, 放在此处
    protected final boolean tryReleaseShared(int releases) {
         for (;;) {
             int current = getState();
             int next = current + releases;
             if (next < current) // overflow
             	throw new Error("Maximum permit count exceeded");
             if (compareAndSetState(current, next))
             	return true;
         }
     }
}
```



# CountDownLatch

**用来进行线程同步协作**，等待所有线程完成倒计时。

其中构造参数用来初始化等待计数值，`await()`用来等待计数归零，`countDown()`用来让计数减一

任务分为N个子线程去执行，`state`也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后`countDown()`一次，state会CAS减1。等到所有子线程都执行完后(即`state=0`)，会`unpark()`主调用线程，然后主调用线程就会从`await()`函数返回，继续后余动作

## 基本使用

```java
public class countDownLatchTest {
    public static void main(String[] args) {
        CountDownLatch countDownLatch = new CountDownLatch(3);
        ExecutorService service = Executors.newFixedThreadPool(4);
        service.submit(() -> {
            try {
                System.out.println(Thread.currentThread().getName() + " begin...");
                Thread.sleep(1000);
                countDownLatch.countDown();
                System.out.println(Thread.currentThread().getName() + " end..." + " " + countDownLatch.getCount());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        });
        service.submit(() -> {
            try {
                System.out.println(Thread.currentThread().getName() + " begin...");
                Thread.sleep(2000);
                countDownLatch.countDown();
                System.out.println(Thread.currentThread().getName() + " end..." + " " + countDownLatch.getCount());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        });
        service.submit(() -> {
            try {
                System.out.println(Thread.currentThread().getName() + " begin...");
                Thread.sleep(3000);
                countDownLatch.countDown();
                System.out.println(Thread.currentThread().getName() + " end..." + " " + countDownLatch.getCount());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        });
        service.submit(() -> {
            try {
                System.out.println(Thread.currentThread().getName() + " waiting...");
                countDownLatch.await();
                System.out.println(Thread.currentThread().getName() + " wait end...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
}
```

运行：

```
pool-1-thread-1 begin...
pool-1-thread-2 begin...
pool-1-thread-3 begin...
pool-1-thread-4 waiting...
pool-1-thread-1 end... 2
pool-1-thread-2 end... 1
pool-1-thread-3 end... 0
pool-1-thread-4 wait end...
```

## 应用

### 同步等待多线程准备完毕

```java
public class CountDownLatchAppTest {
    public static void main(String[] args) throws InterruptedException {
        AtomicInteger num = new AtomicInteger(0);
        ExecutorService executorService = Executors.newFixedThreadPool(10, (r) -> {
            return new Thread(r, "t" + num.getAndIncrement());
        });
        CountDownLatch latch = new CountDownLatch(10);
        String[] all = new String[10];
        Random r = new Random();
        for (int j = 0; j < 10; j++) {
            int x = j;
            executorService.submit(() -> {
                for (int i = 0; i < 101; i++) {
                    try {
                        Thread.sleep(r.nextInt(1000));
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    all[x] = Thread.currentThread().getName() + "(" + (i + "%") + ")";
                    System.out.print("\r" + Arrays.toString(all));
                }
                latch.countDown();
            });
        }
        latch.await();
        System.out.println("\n游戏开始...");
        executorService.shutdown();
    }
}
```

运行：

中间输出：

```
[t0(48%), t1(39%), t2(43%), t3(43%), t4(42%), t5(38%), t6(40%), t7(44%), t8(40%), t9(45%)]
```

最后输出：

```
[t0(100%), t1(100%), t2(100%), t3(100%), t4(100%), t5(100%), t6(100%), t7(100%), t8(100%), t9(100%)]
游戏开始...
```

## CyclicBarrier

['sazklrk 'baeria]循环栅栏，**用来进行线程协作**，等待线程满足某个计数。构造时设置『计数个数』，每个线程执行到某个需要“同步"的时刻调用await()方法进行等待，当等待的线程数满足『计数个数』时，继续执行

> 注意`CyclicBarrier` 与`CountDownLatch`的主要区别在于`CyclicBarrier`是可以重用的
>
> `CyclicBarrier`可以被比喻为『人满发车』

# ReentrantReadWriteLock

一般来说，自定义同步器要么是独占方法，要么是共享方式，它们也只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一种即可。但**AQS也支持自定义同步器同时实现独占和共享两种方式**，如`ReentrantReadWriteLock`。

当读操作远远高于写操作时，这时候使用`读写锁`让`读读`可以并发，提高性能。类似于数据库中的`select ...from ... lock in share mode`：

- 读-读：可以并发
- 读-写：相互阻塞
- 写-写：相互阻塞



