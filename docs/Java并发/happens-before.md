- 在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系

- 与程序员密切相关的 happens-before 规则如下：

  - 程序顺序规则：一个线程中的每个操作，happens-before 于该线程中的任意后续操作
  - 监视器锁规则：对一个锁的解锁，happens-before 于随后对这个锁的加锁
  - volatile 变量规则：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读
  - 传递性：如果 A  happens-before B，且 B happens-before C，那么 A happens-before C

- **happens-before是可见性与有序性的一套规则总结**，规定了对共享变量的写操作对其他线程的读操作可见

- 抛开以下happens-before规则，JMM并不能保证一个线程对共享变量的写，对其它线程对该共享变量的读可见

  - 线程解锁 m 之前对变量的写，对于接下来对 m 加锁的其它线程对该变量的读可见  

    ```java
    static int x;
    static Object m = new Object();
    
    new Thread(()->{
        synchronized(m) {
        	x = 10;
    	}
    },"t1").start();
    
    new Thread(()->{
        synchronized(m) {
        	System.out.println(x);
        }
    },"t2").start();
    ```

  - 线程对 volatile 变量的写，对接下来其它线程对该变量的读可见  

    ```java
    volatile static int x;
    
    new Thread(()->{
    	x = 10;
    },"t1").start();
    
    new Thread(()->{
    	System.out.println(x);
    },"t2").start();
    ```

  - 线程 start 前对变量的写，对该线程开始后对该变量的读可见  

    ```java
    static int x;
    x = 10;
    
    new Thread(()->{
    	System.out.println(x);
    },"t2").start();
    ```

  - 线程结束前对变量的写，对其它线程得知它结束后的读可见（比如其它线程调用 `t1.isAlive() `或 `t1.join()`等待它结束）  

    ```java
    static int x;
    
    Thread t1 = new Thread(()->{
    	x = 10;
    },"t1");
    t1.start();
    
    t1.join();
    System.out.println(x);
    ```

  - 线程 t1 打断 t2（interrupt）前对变量的写，对于其他线程得知 t2 被打断后对变量的读可见（通过`t2.interrupted` 或 `t2.isInterrupted`）  

    ```java
    static int x;
    public static void main(String[] args) {
        Thread t2 = new Thread(()->{
            while(true) {
                if(Thread.currentThread().isInterrupted()) {
                    System.out.println(x);
                    break;
                }
            }
        },"t2");    
        t2.start();
        
        new Thread(()->{
            sleep(1);
            x = 10;
            t2.interrupt();
        },"t1").start();
        
        while(!t2.isInterrupted()) {
        	Thread.yield();
        }
        System.out.println(x);
    }
    ```

  - 对变量默认值（0，false，null）的写，对其它线程对该变量的读可见  

  - 具有传递性，如果 `x hb-> y` 并且 `y hb-> z` 那么有 `x hb-> z` ，配合 volatile 的防指令重排，有下面的例子  

    ```java
    volatile static int x;
    static int y;
    
    new Thread(()->{
        y = 10;
        x = 20;
    },"t1").start();
    
    new Thread(()->{
        // x=20 对 t2 可见, 同时 y=10 也对 t2 可见
        System.out.println(x);
    },"t2").start();
    ```

  - > 变量都是指成员变量或静态成员变量