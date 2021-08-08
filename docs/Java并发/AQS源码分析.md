# 0 概述

AQS全称是AbstractQueuedSynchronizer，**抽象的队列式同步器，定义了一套多线程访问共享资源的同步器框架**，许多同步类实现都依赖于它，如常用的ReentrantLock/Semaphore/CountDownLatch/ReentrantReadWriteLock

![image-20210424185836320](https://i.loli.net/2021/04/24/EHkXetWmOzDlTfd.png)

特点：

- 用`state`属性来表示资源的状态（分**独占模式和共享模式**），子类需要定义如何维护这个状态，控制如何获取锁和释放锁
  - 独占模式是只有一个线程能够访问资源，而共享模式可以允许多个线程访问资源
- 提供了基于FIFO的等待队列
- 支持多个条件变量

子类主要实现这样一些方法（默认抛出`UnsupportedOperationException`）:

```java
// 独占模式，尝试获取资源，成功则返回true，失败则返回false
boolean tryAcquire(int arg)
    
// 独占模式，尝试释放资源，成功则返回true，失败则返回false
boolean tryRelease(int arg)
    
// 共享模式，尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源
int tryAcquireShared(int arg)
    
// 共享模式，尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false
boolean tryReleaseShared(int arg)
    
// 该线程是否正在独占资源。只有用到condition才需要去实现它
boolean isHeldExclusively()
```

**至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了**

以`ReentrantLock`为例，`state`初始化为0，表示未锁定状态。A线程`lock()`时，会调用`tryAcquire()`独占该锁并将`state+1`。此后，其他线程再`tryAcquire()`时就会失败，直到A线程`unlock()`到`state=0`（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，**A线程自己是可以重复获取此锁的（`state`会累加），这就是可重入的概念**。但要注意，获取多少次就要释放多么次，这样才能保证`state`是能回到零态的。`ReentrantLock` 内部有公平锁和非公平锁两种实现，<span style="color:red">差别就在于新来的线程是否比已经在同步队列中的等待线程更早获得锁</span>。

和 `ReentrantLock` 实现方式类似，`Semaphore` 也是基于 AQS 的，差别在于 `ReentrantLock` 是独占锁，`Semaphore` 是共享锁。

再以`CountDownLatch`以例，任务分为N个子线程去执行，`state`也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后`countDown()`一次，state会CAS减1。等到所有子线程都执行完后(即`state=0`)，会`unpark()`主调用线程，然后主调用线程就会从`await()`函数返回，继续后余动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，它们也只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一种即可。但**AQS也支持自定义同步器同时实现独占和共享两种方式**，如`ReentrantReadWriteLock`。

# 1 重要属性和内部类

```java
/**
 * The synchronization state.
 * 代表共享资源
 */
private volatile int state;

// state的三种访问方式
protected final int getState() {
    return state;
}
protected final void setState(int newState) {
    state = newState;
}
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}

/**
 * Node是对每一个等待获取资源的线程的封装，其包含了需要同步的线程本身及其等待状态
 * 
 * 变量waitStatus表示当前Node结点的等待状态，共有5种取值CANCELLED、SIGNAL、CONDITION、PROPAGATE、0
 *
 * CANCELLED(1) : 表示当前结点已取消调度。当timeout或被中断（响应中断的情况下），会触发变更为此状态，进入该状态后的结点将不会再变化
 * 
 * SIGNAL(-1) : 表示后继结点在等待当前结点唤醒。后继结点入队时，会将前继结点的状态更新为SIGNAL
 * 
 * CONDITION(-2) : 表示结点等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将从等待队列转移到同步队列中，等待获取同步锁
 * 
 * PROPAGATE(-3) : 共享模式下，前继结点不仅会唤醒其后继结点，同时也可能会唤醒后继的后继结点
 *
 * 0 : 新结点入队时的默认状态
 * 
 * 负值表示结点处于有效等待状态，而正值表示结点已被取消。所以源码中很多地方用>0、<0来判断结点的状态是否正常
 */
static final class Node {
    /** Marker to indicate a node is waiting in shared mode */
    static final Node SHARED = new Node();
    /** Marker to indicate a node is waiting in exclusive mode */
    static final Node EXCLUSIVE = null;

    /** waitStatus value to indicate thread has cancelled */
    static final int CANCELLED =  1;
    /** waitStatus value to indicate successor's thread needs unparking */
    // 
    static final int SIGNAL    = -1;
    /** waitStatus value to indicate thread is waiting on condition */
    static final int CONDITION = -2;
    /**
         * waitStatus value to indicate the next acquireShared should
         * unconditionally propagate
         */
    static final int PROPAGATE = -3;

    /**
         * Status field, taking on only the values:
         *   SIGNAL:     The successor of this node is (or will soon be)
         *               blocked (via park), so the current node must
         *               unpark its successor when it releases or
         *               cancels. To avoid races, acquire methods must
         *               first indicate they need a signal,
         *               then retry the atomic acquire, and then,
         *               on failure, block.
         *   CANCELLED:  This node is cancelled due to timeout or interrupt.
         *               Nodes never leave this state. In particular,
         *               a thread with cancelled node never again blocks.
         *   CONDITION:  This node is currently on a condition queue.
         *               It will not be used as a sync queue node
         *               until transferred, at which time the status
         *               will be set to 0. (Use of this value here has
         *               nothing to do with the other uses of the
         *               field, but simplifies mechanics.)
         *   PROPAGATE:  A releaseShared should be propagated to other
         *               nodes. This is set (for head node only) in
         *               doReleaseShared to ensure propagation
         *               continues, even if other operations have
         *               since intervened.
         *   0:          None of the above
         *
         * The values are arranged numerically to simplify use.
         * Non-negative values mean that a node doesn't need to
         * signal. So, most code doesn't need to check for particular
         * values, just for sign.
         *
         * The field is initialized to 0 for normal sync nodes, and
         * CONDITION for condition nodes.  It is modified using CAS
         * (or when possible, unconditional volatile writes).
         */
    volatile int waitStatus;

    /**
         * Link to predecessor node that current node/thread relies on
         * for checking waitStatus. Assigned during enqueuing, and nulled
         * out (for sake of GC) only upon dequeuing.  Also, upon
         * cancellation of a predecessor, we short-circuit while
         * finding a non-cancelled one, which will always exist
         * because the head node is never cancelled: A node becomes
         * head only as a result of successful acquire. A
         * cancelled thread never succeeds in acquiring, and a thread only
         * cancels itself, not any other node.
         */
    volatile Node prev;

    /**
         * Link to the successor node that the current node/thread
         * unparks upon release. Assigned during enqueuing, adjusted
         * when bypassing cancelled predecessors, and nulled out (for
         * sake of GC) when dequeued.  The enq operation does not
         * assign next field of a predecessor until after attachment,
         * so seeing a null next field does not necessarily mean that
         * node is at end of queue. However, if a next field appears
         * to be null, we can scan prev's from the tail to
         * double-check.  The next field of cancelled nodes is set to
         * point to the node itself instead of null, to make life
         * easier for isOnSyncQueue.
         */
    volatile Node next;

    /**
         * The thread that enqueued this node.  Initialized on
         * construction and nulled out after use.
         */
    volatile Thread thread;

    /**
         * Link to next node waiting on condition, or the special
         * value SHARED.  Because condition queues are accessed only
         * when holding in exclusive mode, we just need a simple
         * linked queue to hold nodes while they are waiting on
         * conditions. They are then transferred to the queue to
         * re-acquire. And because conditions can only be exclusive,
         * we save a field by using special value to indicate shared
         * mode.
         */
    Node nextWaiter;

    /**
         * Returns true if node is waiting in shared mode.
         */
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    /**
         * Returns previous node, or throws NullPointerException if null.
         * Use when predecessor cannot be null.  The null check could
         * be elided, but is present to help the VM.
         *
         * @return the predecessor of this node
         */
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {    // Used to establish initial head or SHARED marker
    }

    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}


/*      +------+  prev +-----+       +-----+
 * head |      | <---- |     | <---- |     |  tail
 *      +------+       +-----+       +-----+
 */
/**
     * Head of the wait queue, lazily initialized.  Except for
     * initialization, it is modified only via method setHead.  Note:
     * If head exists, its waitStatus is guaranteed not to be
     * CANCELLED.
     */
private transient volatile Node head;      // CLH队列头节点

/**
     * Tail of the wait queue, lazily initialized.  Modified only via
     * method enq to add new wait node.
     */
private transient volatile Node tail;     // CLH队列尾节点
```

# 2 acquire(int)

此方法是独占模式下线程获取共享资源的顶层入口。**如果获取到资源，线程直接返回，否则进入等待队列，直到获取到资源为止，且整个过程忽略中断的影响**。这也正是`lock()`的语义，当然不仅仅只限于`lock()`。获取到资源后，线程就可以去执行其临界区代码了。下面是`acquire()`的源码：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

函数流程如下：

- `tryAcquire()`尝试直接去获取资源
  - 如果成功则直接返回（这里**体现了非公平锁，每个线程获取锁时会尝试直接抢占加塞一次，而CLH队列中可能还有别的线程在等待**）；
  - 如果失败
    - `addWaiter()`将该线程加入等待队列的尾部，并标记为独占模式；
    - `acquireQueued()`使线程阻塞在等待队列中获取资源，一直获取到资源后才返回。
      - 如果在整个等待过程中被中断过，则返回true，否则返回false
      - 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断`selfInterrupt()`，将中断补上

　　## 2.1 tryAcquire(int)

- 此方法尝试去获取独占资源。如果获取成功，则直接返回true，否则直接返回false
- **AQS只是一个框架，具体资源的获取/释放方式交由自定义同步器去实现**（通过`state`的`get/set/CAS`）

```java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

这里之所以没有定义成`abstract`，是因为独占模式下只用实现`tryAcquire-tryRelease`，而共享模式下只用实现`tryAcquireShared-tryReleaseShared`。如果都定义成`abstract`，那么每个模式也要去实现另一模式下的接口。这样可以尽量减少不必要的工作量

## 2.2 addWaiter(Node)

- 此方法用于将当前线程加入到等待队列的队尾，并返回当前线程所在的结点

```java
/**
     * Creates and enqueues node for current thread and given mode.
     *
     * @param mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared
     * @return the new node
     */
private Node addWaiter(Node mode) {
    // 以给定模式构造结点。mode有两种：EXCLUSIVE（独占）和SHARED（共享）
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    // 尝试快速方式直接放到队尾
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 上一步失败则通过enq入队
    enq(node);
    return node;
}

/**
 * 用于将node加入队尾
 * Inserts node into queue, initializing if necessary. See picture above.
 * @param node the node to insert
 * @return node's predecessor
 */
private Node enq(final Node node) {
    // CAS"自旋"，直到成功加入队尾
    for (;;) {
        Node t = tail;
        // 队列为空，创建一个空的标志结点作为head结点，并将tail也指向它
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            // 放入队尾
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

## 2.3 acquireQueued(Node, int)

- 执行此方法时，说明该线程获取资源失败，已经被放入等待队列尾部了，下一步，**进入等待状态休息，直到其他线程彻底释放资源后唤醒自己，自己再拿到资源后返回**
- 具体流程如下：
  - 结点进入队尾后，检查状态，找到安全休息点（在`shouldParkAfterFailedAcquire`方法中，设置前驱节点状态为`SIGNAL`）
  - 调用`park()`进入`waiting`状态，等待`unpark()`或`interrupt()`唤醒自己(`parkAndCheckInterrupt()`)
  - 被唤醒后，看自己是不是有资格获取资源（`p == head && tryAcquire(arg)`）。
    - 如果获取资源成功，`head`指向当前结点，并返回从入队到拿到资源的整个过程中是否被中断过
    - 如果失败，继续寻找安全休息点，调用`park()`进入`waiting`状态等待被唤醒

```java
final boolean acquireQueued(final Node node, int arg) {
    // 标记是否成功拿到资源
    boolean failed = true;
    try {
        // 标记等待过程中是否被中断过
        boolean interrupted = false;
        // "自旋"
        for (;;) {
            // 获取前驱节点
            final Node p = node.predecessor();
            // 如果前驱节点是head，那么该节点便有资格去尝试获取资源tryAcquire(arg)（可能是前一个节点释放完资源唤醒自己的，也可能被interrupt了）。
            if (p == head && tryAcquire(arg)) {
                // 拿到资源后，将head指向该结点
                // 所以head所指的标杆结点，就是当前获取到资源的那个结点或null
                setHead(node);
                // setHead中node.prev已置为null，此处再将head.next置为null，就是为了方便GC回收以前的head结点。也就意味着之前拿完资源的结点出队了
                p.next = null; // help GC
                // 成功获取资源
                failed = false;
                // 返回等待过程中是否被中断过
                return interrupted;
            }
            // 如果没有资格获取资源或获取资源失败，就通过park()进入waiting状态，直到被unpark()
            // 如果不可中断的情况下被中断了，那么会从park()中醒过来，发现拿不到资源，从而继续进入park()等待
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                // 如果等待过程中被中断过，就将interrupted标记为true
                interrupted = true;
        }
    } finally {
        // 如果等待过程中没有成功获取资源（如timeout，或者可中断的情况下被中断了），那么取消结点在队列中的等待
        if (failed)
            cancelAcquire(node);
    }
}

/**
     * Sets head of queue to be node, thus dequeuing. Called only by
     * acquire methods.  Also nulls out unused fields for sake of GC
     * and to suppress unnecessary signals and traversals.
     *
     * @param node the node
     */
private void setHead(Node node) {
    head = node;
    node.thread = null;
    node.prev = null;
}
```

### 2.3.1 shouldParkAfterFailedAcquire(Node, Node)

- 此方法主要用于检查前驱节点的状态，看看自己是不是真的可以调用`park()`进入`waiting`状态了

```java
/**
     * Checks and updates status for a node that failed to acquire.
     * Returns true if thread should block. This is the main signal
     * control in all acquire loops.  Requires that pred == node.prev.
     *
     * @param pred node's predecessor holding status
     * @param node the node
     * @return {@code true} if thread should block
     */
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    // 拿到前驱节点的状态
    int ws = pred.waitStatus;
    // 如果前驱节点已经是SIGNAL状态了，则直接返回
    if (ws == Node.SIGNAL)
        /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
        return true;
    // 如果前驱节点为CANCELLED即取消调度，那就一直往前找，直到找到最近一个正常等待的状态，并排在它的后边
    if (ws > 0) {
        /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 把前驱的状态设置成SIGNAL，告诉它拿完号后通知自己一下(有可能失败，前驱节点说不定刚刚释放完)
        /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

### 2.3.2 parkAndCheckInterrupt()

- 此方法就是在当前线程找好安全休息点后，真正进入等待状态
- `park()`会让当前线程进入`waiting`状态。在此状态下，有两种途径可以唤醒该线程：1）被`unpark()`；2）被`interrupt()`
- `Thread.interrupted()`会清除当前线程的中断标记位

```java
private final boolean parkAndCheckInterrupt() {
    // 调用park()使线程进入waiting状态
    LockSupport.park(this);
    // 如果被唤醒，查看自己是不是被中断的
    return Thread.interrupted();
}
```

## 2.4 selfInterrupt()

- 如果当前线程在阻塞过程中有被中断过，它是不响应的，只是设置一下中断标记（`interrupted=true`），只有在获取到资源返回后将中断补上

```java
/**
     * Convenience method to interrupt current thread.
     */
static void selfInterrupt() {
    Thread.currentThread().interrupt();
}
```

## 2.5 小结

<center><img src="https://images2015.cnblogs.com/blog/721070/201511/721070-20151102145743461-623794326.png"/></center>

# 3 release(int)

- 此方法是独占模式下线程释放共享资源的顶层入口。**它会释放指定量的资源，如果彻底释放了（即`state=0`）**,它会唤醒等待队列里的其他线程来获取资源
- `tryRelease`方法由自定义同步器来实现，**`release()`根据`tryRelease()`的返回值来判断该线程是否已经释放掉资源**

```java
public final boolean release(int arg) {
    // 释放指定量的资源
    if (tryRelease(arg)) {
        // 找到头结点
        Node h = head;
        if (h != null && h.waitStatus != 0)
            // 唤醒等待队列里的下一个线程
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

## 3.1 tryRelease(int)

- 此方法尝试去释放指定量的资源，需要独占模式的自定义同步器去实现
- 正常来说，`tryRelease()`都会成功的，因为这是独占模式，该线程来释放资源，那么它肯定已经拿到独占资源了，**直接减掉相应量的资源即可`(state-=arg)`**，也不需要考虑线程安全的问题。但要注意它的返回值，上面已经提到了，**release()是根据tryRelease()的返回值来判断该线程是否已经完成释放掉资源了！**所以自义定同步器在实现时，**如果已经彻底释放资源(`state=0`)，要返回`true`，否则返回`false`。**

```java
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
```

## 3.2 unparkSuccessor(Node) 

- 此方法**用unpark()唤醒等待队列中最前边的那个未放弃线程s**
- 此时，再和`acquireQueued()`联系起来，s被唤醒后，进入`if (p == head && tryAcquire(arg))`的判断（即使`p!=head`也没关系，它会再进入`shouldParkAfterFailedAcquire()`寻找一个安全点。这里既然s已经是等待队列中最前边的那个未放弃线程了，那么通过`shouldParkAfterFailedAcquire()`的调整，s也必然会跑到head的next结点，下一次自旋`p==head`就成立了），然后s把自己设置成head标杆结点，表示自己已经获取到资源了，`acquire()`也返回了

```java
/**
     * Wakes up node's successor, if one exists.
     *
     * @param node the node
     */
private void unparkSuccessor(Node node) {
    /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
    // node一般为当前线程所在的结点
    int ws = node.waitStatus;
    if (ws < 0)
        // 置零当前线程所在的结点状态，允许失败
        compareAndSetWaitStatus(node, ws, 0);

    /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
    // 找到下一个需要唤醒的结点s
    Node s = node.next;
    // 如果为空或已取消
    if (s == null || s.waitStatus > 0) {
        s = null;
        // 从后向前找
        for (Node t = tail; t != null && t != node; t = t.prev)
            // <=0的结点，都是还有效的结点
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        // 唤醒后继节点
        LockSupport.unpark(s.thread);
}
```

## 3.3 小结

- `release()`是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即`state=0`）,它会唤醒等待队列里的其他线程来获取资源

# 4 acquireShared(int)

- 此方法是共享模式下线程获取共享资源的顶层入口。**它会获取指定量的资源，获取成功则直接返回，获取失败则通过`doAcquireShared()`进入等待队列，直到获取到资源为止，**整个过程忽略中断。
- 具体流程如下：
  - `tryAcquireShared()`尝试获取资源
    - 成功则直接返回
    - 失败则通过`doAcquireShared()`进入等待队列`park()`，直到被`unpark()/interrupt()`并成功获取到资源才返回。整个等待过程也是忽略中断的
  - 其实跟`acquire()`的流程大同小异，只不过多了个**自己拿到资源后，还会去唤醒后继节点的操作**（毕竟是共享模式）

```java
public final void acquireShared(int arg) {
    // tryAcquireShared返回负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

## 4.1 tryAcquireShared(int)

- `tryAcquireShared()`需要自定义同步器去实现。但是AQS已经把其返回值的语义定义好了：**负值代表获取失败；0代表获取成功，但没有剩余资源；正数表示获取成功，还有剩余资源，其他线程还可以去获取**

```java
protected int tryAcquireShared(int arg) {
    throw new UnsupportedOperationException();
}
```

## 4.2 doAcquireShared(int)

- 此方法用于将当前线程加入等待队列尾部阻塞，直到其他线程释放资源唤醒自己，**自己成功拿到相应量的资源后才返回**

```java
/**
 * Acquires in shared uninterruptible mode.
 * @param arg the acquire argument
 */
private void doAcquireShared(int arg) {
    // 加入队列尾部
    final Node node = addWaiter(Node.SHARED);
    // 是否成功获取到相应量的资源标志
    boolean failed = true;
    try {
        // 等待过程中是否被中断过的标志
        boolean interrupted = false;
        for (;;) {
            // 前驱节点
            final Node p = node.predecessor();
            // 如果是head的下一个，因为head是拿到资源的线程，此时node被唤醒，很可能是head用完资源来唤醒自己的
            if (p == head) {
                // 尝试获取相应量的资源
                int r = tryAcquireShared(arg);
                // 成功
                if (r >= 0) {
                    // 将head指向自己，还有剩余资源可以再唤醒之后的线程
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    // 如果等待过程中被打断过，此时将中断补上。
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            // 判断状态，寻找安全点，进入waiting状态，等着被unpark()或interrupt()
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

- 跟`acquireQueued()`很相似，流程并没有太大区别。只不过这里将补中断的`selfInterrupt()`放到`doAcquireShared()`里了，而独占模式是放到`acquireQueued()`之外
- 跟独占模式比，还有一点需要注意的是，**这里只有线程是`head.next`时（“老二”），才会去尝试获取资源，有剩余的话还会唤醒之后的队友**。那么问题就来了，假如老大用完后释放了5个资源，而老二需要6个，老三需要1个，老四需要2个。老大先唤醒老二，老二一看资源不够，他是把资源让给老三呢，还是不让？答案是否定的！**老二会继续`park()`等待其他线程释放资源，也不会去唤醒老三和老四**。独占模式，同一时刻只有一个线程去执行，这样做未尝不可；但共享模式下，多个线程是可以同时执行的，现在因为老二的资源需求量大，而把后面量小的老三和老四也都卡住了。当然，这并不是问题，**只是AQS保证严格按照入队顺序唤醒罢了（保证公平，但降低了并发）**

### 4.2.1 setHeadAndPropagate(Node, int)

- 此方法在`setHead()`的基础上多了一步，就是自己苏醒的同时，如果条件符合（比如**还有剩余资源**），还会去唤醒后继结点，毕竟是共享模式

```java
/**
 * Sets head of queue, and checks if successor may be waiting
 * in shared mode, if so propagating if either propagate > 0 or
 * PROPAGATE status was set.
 *
 * @param node the node
 * @param propagate the return value from a tryAcquireShared
 */
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    // head指向自己
    setHead(node);
    /*
     * Try to signal next queued node if:
     *   Propagation was indicated by caller,
     *     or was recorded (as h.waitStatus either before
     *     or after setHead) by a previous operation
     *     (note: this uses sign-check of waitStatus because
     *      PROPAGATE status may transition to SIGNAL.)
     * and
     *   The next node is waiting in shared mode,
     *     or we don't know, because it appears null
     *
     * The conservatism in both of these checks may cause
     * unnecessary wake-ups, but only when there are multiple
     * racing acquires/releases, so most need signals now or soon
     * anyway.
     */
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```

# 5 releaseShared()

- 此方法是共享模式下线程释放共享资源的顶层入口。**它会释放指定量的资源，如果成功释放且允许唤醒等待线程，它会唤醒等待队列里的其他线程来获取资源**

```java
public final boolean releaseShared(int arg) {
    // 尝试释放资源
    if (tryReleaseShared(arg)) {
        // 尝试唤醒后继结点
        doReleaseShared();
        return true;
    }
    return false;
}
```

- 跟独占模式下的`release()`相似，但有一点稍微需要注意：**独占模式下的`tryRelease()`在完全释放掉资源（`state=0`）后，才会返回`true`去唤醒其他线程，这主要是基于独占下可重入的考量**；而共享模式下的`releaseShared()`则没有这种要求，**共享模式实质就是控制一定量的线程并发执行，那么拥有资源的线程在释放掉部分资源时就可以唤醒后继等待结点**。例如，资源总量是13，A（5）和B（7）分别获取到资源并发运行，C（4）来时只剩1个资源就需要等待。A在运行过程中释放掉2个资源量，然后`tryReleaseShared(2)`返回true唤醒C，C一看只有3个仍不够继续等待；随后B又释放2个，`tryReleaseShared(2)`返回true唤醒C，C一看有5个够自己用了，然后C就可以跟A和B一起运行。而`ReentrantReadWriteLock`读锁的`tryReleaseShared()`只有在完全释放掉资源（`state=0`）才返回`true`，所以自定义同步器可以根据需要决定`tryReleaseShared()`的返回值

## 5.1 doReleaseShared()

- 此方法主要用于唤醒后继节点

```java
/**
 * Release action for shared mode -- signals successor and ensures
 * propagation. (Note: For exclusive mode, release just amounts
 * to calling unparkSuccessor of head if it needs signal.)
 */
private void doReleaseShared() {
    /*
     * Ensure that a release propagates, even if there are other
     * in-progress acquires/releases.  This proceeds in the usual
     * way of trying to unparkSuccessor of head if it needs
     * signal. But if it does not, status is set to PROPAGATE to
     * ensure that upon release, propagation continues.
     * Additionally, we must loop in case a new node is added
     * while we are doing this. Also, unlike other uses of
     * unparkSuccessor, we need to know if CAS to reset status
     * fails, if so rechecking.
     */
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h); //唤醒后继
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        // head发生变化
        if (h == head)                   // loop if head changed
            break;
    }
}
```

# 总结

<center><img src="https://img2020.cnblogs.com/blog/1322310/202003/1322310-20200322170642726-1351777316.png"/></center>

# Reference

- [Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)
- [锁原理-AQS源码分析](https://www.cnblogs.com/binarylei/p/12555166.html)

- [悲观锁、乐观锁](https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453141665&idx=3&sn=3f4a66f964e07399db89b3b7109ee227&scene=21#wechat_redirect)