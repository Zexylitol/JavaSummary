<!-- GFM-TOC -->

- [方式一：继承Thread类](#方式一：继承Thread类)
- [方式二：使用Runnable配合Thread](#方式二：使用Runnable配合Thread)
- [方式三：使用Callable和FutureTask](#方式三：使用Callable和FutureTask)
- [方式四：线程池](#方式四：线程池)
- [总结](#总结)
- [Reference](#Reference)

<!-- GFM-TOC -->

# 方式一：继承Thread类

| 操作     | 说明                                                    |
| :------- | :------------------------------------------------------ |
| 实现方式 | 继承`Thread`类并重写`run()`方法                         |
| 优缺点   | 1. 受限于Java的单继承机制<br/>2. 线程和任务合并在了一起 |

```java
public class MyThread extends Thread {
    /**
     * If this thread was constructed using a separate
     * <code>Runnable</code> run object, then that
     * <code>Runnable</code> object's <code>run</code> method is called;
     * otherwise, this method does nothing and returns.
     * <p>
     * Subclasses of <code>Thread</code> should override this method.
     *
     * @see #start()
     * @see #stop()
     * @see #Thread(ThreadGroup, Runnable, String)
     */
    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            System.out.println(Thread.currentThread() + ": " + i);
        }
    }

    public static void main(String[] args) {
        MyThread myThread0 = new MyThread();
        MyThread myThread1 = new MyThread();
        myThread0.start();
        myThread1.start();
    }
}
```

输出：

```
Thread[Thread-1,5,main]: 0
Thread[Thread-0,5,main]: 0
Thread[Thread-0,5,main]: 1
Thread[Thread-0,5,main]: 2
Thread[Thread-1,5,main]: 1
Thread[Thread-1,5,main]: 2
```

# 方式二：使用Runnable配合Thread

| 操作     | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| 实现方式 | 实现`Runnable`接口并重写`run()`方法，创建`Runnable`实现类的对象，作为`Thread`对象的参数target，该`Thread`对象才是真正的线程对象 |
| 优缺点   | 1. 不会受限于Java的单继承机制，`Runnable`让任务类脱离了`Thread`继承体系，更灵活<br/>2. 把【线程】与【任务】（要执行的代码）分开，`Thread`代表线程，`Runnable`可运行的任务（线程要执行的代码） |

```java
public class MyRunnable implements Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see Thread#run()
     */
    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            System.out.println(Thread.currentThread() + ": " + i);
        }
    }

    public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        new Thread(myRunnable, "thread1").start();
        new Thread(myRunnable, "thread2").start();
    }
}
```

输出：

```
Thread[thread2,5,main]: 0
Thread[thread1,5,main]: 0
Thread[thread2,5,main]: 1
Thread[thread1,5,main]: 1
Thread[thread2,5,main]: 2
Thread[thread1,5,main]: 2
```

# 方式三：使用Callable和FutureTask

| 操作     | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| 实现方式 | 1. 实现`Callable`接口并实现`call()`方法，创建`Callable`实现类的实例，使用`FutureTask`类来包装`Callable`对象<br/>2. 使用`FutureTask`对象作为`Thread`对象的target创建并启动新线程<br/>3. 调用`FutureTask`对象的`get()`方法来获得子线程执行结束后的返回值 |
| 优缺点   | 1. 上述两种方法都不能有返回值，且不能声明抛出异常。而`Callable`接口则实现了此两点，`Callable`接口如同`Runnable`接口的升级版，其提供的`call()`方法将作为线程的执行体，同时允许有返回值。<br/>2. 但是`Callable`对象不能直接作为`Thread`对象的target，因为`Callable`接口是 Java 5 新增的接口，不是`Runnable`接口的子接口。对于这个问题的解决方案，就引入`Future`接口，此接口可以接受`call()` 的返回值，`RunnableFuture`接口是`Future`接口和`Runnable`接口的子接口，可以作为`Thread`对象的target 。并且， `Future` 接口提供了一个实现类：`FutureTask` <br/>3. 运行`Callable`任务可以拿到一个`Future`对象，表示异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。通过`Future`对象可以了解任务执行情况，可取消任务的执行，还可获取执行结果 |

```java
public class MyCallable implements Callable<String> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    @Override
    public String call() throws Exception {
        for (int i = 0; i < 3; i++) {
            System.out.println(Thread.currentThread() + ": " + i);
        }
        return "Callable" + Thread.currentThread();
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyCallable myCallable = new MyCallable();
        FutureTask<String> stringFutureTask0 = new FutureTask<>(myCallable);
        FutureTask<String> stringFutureTask1 = new FutureTask<>(myCallable);
        new Thread(stringFutureTask0, "t0").start();
        new Thread(stringFutureTask1, "t1").start();
        System.out.println(stringFutureTask0.get());
        System.out.println(stringFutureTask1.get());
    }
}
```

输出：

```java
Thread[t0,5,main]: 0
Thread[t1,5,main]: 0
Thread[t0,5,main]: 1
Thread[t1,5,main]: 1
Thread[t0,5,main]: 2
Thread[t1,5,main]: 2
CallableThread[t0,5,main]
CallableThread[t1,5,main]

Process finished with exit code 0
```

# 方式四：线程池

线程池（Thread Pool）是一种**基于池化思想管理线程的工具，一方面避免了处理任务时创建销毁线程开销的代价**，**另一方面避免了线程数量膨胀导致的过分调度问题，保证了对内核的充分利用**

```java
public class ThreadPoolTest {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        for (int i = 0; i < 5; i++) {
            executorService.submit(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread() + " " + System.currentTimeMillis());
                }
            });
        }

        ExecutorService executorService1 = Executors.newSingleThreadExecutor();
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            executorService1.submit(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread() + " " + finalI);
                }
            });
        }

        ExecutorService executorService2 = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            executorService2.submit(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName() + finalI);
                }
            });
        }

        executorService.shutdown();
        executorService1.shutdown();
        executorService2.shutdown();

        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(5,
                5,
                5, TimeUnit.MINUTES,
                new LinkedBlockingQueue<>());
        for (int i = 0; i < 10; i++) {
            threadPoolExecutor.submit(() -> {
                System.out.println("Custom " + Thread.currentThread().getName() + ":" + Thread.currentThread().getId());
            });
        }

        threadPoolExecutor.shutdown();
    }
}
```



# 总结

相比继承， 接口实现可以更加灵活，不会受限于Java的单继承机制。并且通过实现接口的方式可以共享资源，适合多线程处理同一资源的情况

# Reference

- [java创建线程的三种方式及其对比](https://www.cnblogs.com/songshu120/p/7966314.html)
* [Java创建线程的主要方式](https://www.cnblogs.com/lingz/p/9692545.html)