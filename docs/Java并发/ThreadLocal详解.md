`ThreadLocal`是除了加锁这种同步方式之外的一种规避多线程访问共享变量出现线程不安全的方法，在创建一个变量后，如果每个线程对其进行访问的时候访问的都是线程自己的变量这样就不会存在线程不安全问题



`ThreadLocal`是JDK包提供的，它提供线程本地变量，如果创建一个`ThreadLocal`变量，那么访问这个变量的每个线程都会有这个变量的一个副本，在实际多线程操作的时候，操作的是自己本地内存中的变量，从而规避了线程安全问题



`ThreadLocal`的作用主要是做数据隔离，填充的数据只属于当前线程，变量的数据对别的线程而言是相对隔离的，在多线程环境下，防止自己的变量被其它线程篡改。

<center><img src="https://img-blog.csdnimg.cn/20190705094129815.png?"/></center>

# 基本使用

开启两个线程，在每个线程内部设置了本地变量的值，然后调用`print`方法打印当前本地变量的值。如果在打印之后调用本地变量的`remove`方法会删除本地内存中的变量

```java
public class ThreadLocalTest {

    static ThreadLocal<String> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            // 设置线程1中本地变量的值
            threadLocal.set("localVar1");

            // 打印当前线程中本地内存中本地变量的值
            System.out.println("Thread1" + " : " + threadLocal.get());

            // 清除本地内存中的本地变量
            threadLocal.remove();

            // 打印本地变量
            System.out.println("After remove : " + threadLocal.get());
        });

        Thread t2 = new Thread(() -> {
            // 设置线程2中本地变量的值
            threadLocal.set("localVar2");

            // 打印当前线程中本地内存中本地变量的值
            System.out.println("Thread2" + " : " + threadLocal.get());

            // 清除本地内存中的本地变量
            threadLocal.remove();

            // 打印本地变量
            System.out.println("After remove : " + threadLocal.get());
        });

        t1.start();
        t1.join();
        t2.start();
    }
}
```

输出结果：

```java
Thread1 : localVar1
After remove : null
Thread2 : localVar2
After remove : null
```

主线程进来之后初始化一个可以泛型的`ThreadLocal`对象，t1线程和t2线程分别通过`set()`方法设置自己的值，之后每个线程只要在`remove()`之前去`get()`，都能拿到之前`set`的值，注意是`remove`之前

`ThreadLocal`可以实现**线程间数据隔离，所以别的线程使用get()方法是没办法拿到其他线程的值的**，但是`InheritableThreadLocal()`可以做到父子线程间的数据传递

# 底层原理

## set()

在查看`ThreadLocal`的`set()`源码之前，首先介绍`Thread`类中的两个重要属性`threadLocals`和`inheritableThreadLocals`，在默认情况下，每个线程中这两个变量都为`null`，只有当线程第一次调用`ThreadLocal`的`set`或者`get`方法的时候才会创建它们。

```java
public class Thread implements Runnable {
    // ...
    
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
    
    // ...
}
```

`set()`源码如下：

```java
// ThreadLocal
public void set(T value) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 获取当前线程的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    // 校验对象是否为空
    if (map != null) 
        // 不为空set，就直接添加本地变量，key为当前定义的ThreadLocal变量的this引用，值为添加的本地变量值
        map.set(this, value); 
    else
        // 为空创建一个map对象
        createMap(t, value); 
}
```

`set()`中的`map`其实就是当前线程中的`threadLocals`变量：

```java
ThreadLocalMap getMap(Thread t) {
	return t.threadLocals;
}

//第一次调用set方法,这个时候就需要调用createMap方法创建threadLocals
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

<span style="color:red">这里基本上可以找到`ThreadLocal`数据隔离的真相了，每个线程`Thread`都维护了自己的`threadLocals`变量，所以在每个线程创建`ThreadLocal`的时候，实际上数据是存在自己线程`Thread`的`threadLocals`变量里面的，其他线程没办法拿到，从而实现了隔离</span>。

## ThreadLocalMap

### 底层结构

<center><img src="https://i.loli.net/2021/04/25/LguwCN1YfGS8TDz.png"/></center>

`ThreadLocalMap`类似于一个`HashMap`，但是看源码可以发现，它并未实现`Map`接口，而且它的`Entry`是继承`WeakReference`（弱引用）的，也没有看到`HashMap`中的`next`，所以不存在链表

```java
public class ThreadLocal<T> { 
	// ...
    
        static class ThreadLocalMap {
			// ...
            static class Entry extends WeakReference<ThreadLocal<?>> {
                /** The value associated with this ThreadLocal. */
                Object value;

                Entry(ThreadLocal<?> k, Object v) {
                    super(k);
                    value = v;
                }
            }
            /**
             * The initial capacity -- MUST be a power of two.
             */
            private static final int INITIAL_CAPACITY = 16;

            /**
             * The table, resized as necessary.
             * table.length MUST always be a power of two.
             */
            private Entry[] table;

            /**
             * The number of entries in the table.
             */
            private int size = 0;

            /**
             * The next size value at which to resize.
             */
            private int threshold; // Default to 0
            
           /**
             * Construct a new map initially containing (firstKey, firstValue).
             * ThreadLocalMaps are constructed lazily, so we only create
             * one when we have at least one entry to put in it.
             */
            ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
                table = new Entry[INITIAL_CAPACITY];
                // 定位桶的方式和HashMap相同
                int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
                table[i] = new Entry(firstKey, firstValue);
                size = 1;
                setThreshold(INITIAL_CAPACITY);
            }
            
            // ...
    }    
    
    // ...
}
```

<span style="color:red">`ThreadLocalMap`使用了数组是因为，开发过程中一个线程可以有多个`TreadLocal`来存放不同类型的对象的，但是它们都将放到当前线程的`ThreadLocalMap`里，所以要用数组来存</span>

#### set()

先看一下`ThreadLocalMap`中的`set()`源码：

```java
// ThreadLocalMap
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    // 根据ThreadLocal对象的hash值，定位到table中的位置i，HashMap的味道
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
         e != null;
         // 位置i不为空且位置i处的entry对象的key与即将设置的key不相等，那就寻找下一个空位置，直到e为空为止
         e = tab[i = nextIndex(i, len)]) {
        
        ThreadLocal<?> k = e.get();

        // 位置i不为空且这个Entry对象的key正好是即将设置的key，那么就更新`Entry`中的`value`
        if (k == key) {
            e.value = value;
            return;
        }
        // 如果当前位置是空的，就初始化一个Entry对象放在位置i上
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    // 找到空位置，新建entry对象
    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}

/**
 * Increment i modulo len.
 */
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}
```

<center><img src="https://i.loli.net/2021/04/25/KV3hNEgTlpd89R7.png"/></center>

<span style="color:red">从源码可以看出，它是通过寻找下一个为空的位置来解决Hash冲突的</span>，当冲突率较高时，就会严重影响`set()/get()`的效率。

#### getEntry()

```java
// ThreadLocalMap
private Entry getEntry(ThreadLocal<?> key) {
    // 定位到桶
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;
    // get的时候一样是根据ThreadLocal获取到table的i值，然后查找数据拿到后会对比key是否相等  if (e != null && e.get() == key)。
    while (e != null) {
        ThreadLocal<?> k = e.get();
        // 相等就直接返回，不相等就继续查找，找到相等位置。
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            // 继续往后查找
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

## get()

`get()`流程如下：

1. 获取当前线程
2. 获取当前线程的`threadLocals`变量
3. 如果`threadLocals`变量不为`null`，就可以在`map`中查找到本地变量的值
4. 如果`threadLocals`变量为`null`，则执行`setInitialValue`方法初始化`threadLocals`变量

```java
// ThreadLocal
public T get() {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 获取当前线程的threadLocals变量
    ThreadLocalMap map = getMap(t);
    // 如果threadLocals变量不为null，就可以在map中查找到本地变量的值
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    // 执行到此处，threadLocals为null或者e为null，初始化当前线程的threadLocals变量
    // threadLocals为null,说明还没有进行过set操作
    // e为null，说明不存在key为当前ThreadLocal的Entry对象
    return setInitialValue();
}

/**
     * Variant of set() to establish initialValue. Used instead
     * of set() in case user has overridden the set() method.
     *
     * @return the initial value
     */
private T setInitialValue() {
    // value为null
    T value = initialValue();
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 获取当前线程的threadLocals变量
    ThreadLocalMap map = getMap(t);
    // 如果map不为null，就直接添加本地变量，key为当前线程，值为null
    if (map != null)
        map.set(this, value);
    else
        // 如果map为null，说明首次添加，需要首先创建出对应的map
        createMap(t, value);
    return value;
}

protected T initialValue() {
    return null;
}
```

## remove()

```java
public void remove() {
    // 获取当前线程绑定的threadLocals
    ThreadLocalMap m = getMap(Thread.currentThread());
    // //如果map不为null，就移除当前线程中指定ThreadLocal实例的本地变量
    if (m != null)
        //         private void remove(ThreadLocal<?> key) {}
        m.remove(this);
}
```

每个线程内部有一个名为`threadLocals`的成员变量，该变量的类型为`ThreadLocal.ThreadLocalMap`类型（类似于一个`HashMap`），<span style="color:red">其中的`key`为当前定义的`ThreadLocal`变量的`this`引用</span>，`value`为我们使用`set`方法设置的值。<span style="color:red">每个线程的本地变量存放在自己的本地内存变量`threadLocals`中，如果当前线程一直不消亡，那么这些本地变量就会一直存在（所以可能会导致内存溢出），因此使用完毕需要将其`remove`掉</span>。

# ThreadLocal的实例存放位置

`ThreadLocal`的实例都是位于堆上，只是通过一些技巧将可见性修改成了线程可见



# `InheritableThreadLocal`共享线程的ThreadLocal数据

使用`InheritableThreadLocal`可以实现多个线程访问`ThreadLocal`的值，在主线程中创建一个`InheritableThreadLocal`的实例，然后在子线程中得到这个`InheritableThreadLocal`实例设置的值。

```java
public class InheritableThreadLocalTest {
    public static void main(String[] args) {
        ThreadLocal threadLocal = new InheritableThreadLocal();
        threadLocal.set("Share");
        new Thread(()->{
            System.out.println(threadLocal.get());
        }, "t1").start();
    }
}
```

输出结果为：

```java
Share
```

在子线程中是能够正常获取到`threadLocal`值的，这就是<span style="color:red">父子线程数据传递</span>的问题

## 传递逻辑

传递的逻辑很简单，`Thread`类中还有一个`inheritableThreadLocals`变量

```java
// Thread
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

在`Thread`的`init()`函数中，如果线程的`inheritThreadLocals`变量为`true`，比如上面的例子，而且父线程的`inheritableThreadLocals`也存在，那么就把父线程的`inheritableThreadLocals`给当前线程的`inheritableThreadLocals`。

```java
// Thread
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
// ...
    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
        // 设置子线程中的inheritableThreadLocals为父线程的inheritableThreadLocals
        this.inheritableThreadLocals =
        ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
// ...
}

// ThreadLocal
static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
    return new ThreadLocalMap(parentMap);
}

// ThreadLocal.ThreadLocalMap
private ThreadLocalMap(ThreadLocalMap parentMap) {
    Entry[] parentTable = parentMap.table;
    int len = parentTable.length;
    setThreshold(len);
    table = new Entry[len];

    for (int j = 0; j < len; j++) {
        Entry e = parentTable[j];
        if (e != null) {
            @SuppressWarnings("unchecked")
            ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
            if (key != null) {
                // 调用重写的方法
                Object value = key.childValue(e.value);
                Entry c = new Entry(key, value);
                int h = key.threadLocalHashCode & (len - 1);
                while (table[h] != null)
                    h = nextIndex(h, len);
                table[h] = c;
                size++;
            }
        }
    }
}
```

`InheritableThreadLocal`类的源码如下，继承了`ThreadLocal`类，并重写了`childValue、getMap、createMap`三个方法，借助多态，子类在调用`set/get`方法时，内部调用的`getMap/createMap`方法，实际上是`InheritableThreadLocal`类中重写的方法

```java
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    /**
     * Computes the child's initial value for this inheritable thread-local
     * variable as a function of the parent's value at the time the child
     * thread is created.  This method is called from within the parent
     * thread before the child is started.
     * <p>
     * This method merely returns its input argument, and should be overridden
     * if a different behavior is desired.
     *
     * @param parentValue the parent thread's value
     * @return the child thread's initial value
     */
    protected T childValue(T parentValue) {
        return parentValue;
    }

    /**
     * Get the map associated with a ThreadLocal.
     *
     * @param t the current thread
     */
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

    /**
     * Create the map associated with a ThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the table.
     */
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```

# ThreadLocal存在的问题

`ThreadLocal`有可能产生**内存泄露**问题

当线程没有结束，但是`ThreadLocal`已经被回收，则可能导致线程中存在`ThreadLocalMap<null, Object>`的键值对，**造成内存泄露。**

```java
public class ThreadLocal<T> {
    // ...
    static class ThreadLocalMap {
        // ...
            static class Entry extends WeakReference<ThreadLocal<?>> {
                /** The value associated with this ThreadLocal. */
                Object value;

                Entry(ThreadLocal<?> k, Object v) {
                    // k：ThreadLocal的引用，被传递给WeakReference的构造方法
                    super(k);
                    value = v;
                }
            }
        // ...
    }
    // ...
}

//WeakReference构造方法(public class WeakReference<T> extends Reference<T> )
public WeakReference(T referent) {
    super(referent); //referent：ThreadLocal的引用
}

//Reference构造方法
Reference(T referent) {
    this(referent, null);//referent：ThreadLocal的引用
}

Reference(T referent, ReferenceQueue<? super T> queue) {
    this.referent = referent;
    this.queue = (queue == null) ? ReferenceQueue.NULL : queue;
}
```

`ThreadLocal`在保存的时候会把自己当做Key存在`ThreadLocalMap`中，正常情况应该是key和value都应该被外界强引用才对，但是现在key被设计成WeakReference弱引用了

<center><img src="https://i.loli.net/2021/04/25/U2FHmuyWVDw8ZQA.png"/></center>

> 只具有弱引用的对象拥有更短暂的生命周期，在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。
>
> 不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

这就导致了一个问题，如果创建`ThreadLocal`的线程一直持续运行，那么这个`Entry`对象中的value就有可能一直得不到回收，发生内存泄露。

就比如线程池里面的线程，线程都是复用的，那么之前的线程实例处理完之后，出于复用的目的线程依然存活，所以，`ThreadLocal`设定的value值被持有，导致内存泄露。一个线程使用完，`ThreadLocalMap`是应该要被清空的，但是现在线程被复用了。

### 解决办法

只要记得在使用的最后用`remove`把值清空就好了，这样在垃圾回收器回收的时候，会自动把它们回收掉。

```java
ThreadLocal<String> localName = new ThreadLocal();
try {
    localName.set("ThreadLocal");
    ……
} finally {
    localName.remove();
}
```



## 为什么ThreadLocalMap的key要设计成弱引用

key不设置成弱引用的话就会造成和entry中value一样内存泄漏的场景。



# 隔离作用及使用场景

Spring实现事务隔离级别的源码

Spring采用Threadlocal的方式，来保证单个线程中的数据库操作使用的是同一个数据库连接，同时，采用这种方式可以使业务层使用事务时不需要感知并管理connection对象，通过传播级别，巧妙地管理多个事务配置之间的切换，挂起和恢复。

Spring框架里面就是用的ThreadLocal来实现这种隔离，主要是在`TransactionSynchronizationManager`这个类里面，代码如下所示:

```java
private static final Log logger = LogFactory.getLog(TransactionSynchronizationManager.class);

 private static final ThreadLocal<Map<Object, Object>> resources =
   new NamedThreadLocal<>("Transactional resources");

 private static final ThreadLocal<Set<TransactionSynchronization>> synchronizations =
   new NamedThreadLocal<>("Transaction synchronizations");

 private static final ThreadLocal<String> currentTransactionName =
   new NamedThreadLocal<>("Current transaction name");

  ……
```

Spring的事务主要是ThreadLocal和AOP去做实现的

# 实际使用场景

在项目中存在一个线程经常遇到横跨若干方法调用，需要传递的对象，也就是上下文（Context），它是一种状态，经常就是是用户身份、任务信息等，就会存在过渡传参的问题。

使用到类似责任链模式，给每个方法增加一个context参数非常麻烦，而且有些时候，如果调用链有无法修改源码的第三方库，对象参数就传不进去了，所以使用到了`ThreadLocal`去做了一下改造，这样只需要在调用前在`ThreadLocal`中设置参数，其他地方`get`一下就好了。

```java
//before
  
void work(User user) {
    getInfo(user);
    checkInfo(user);
    setSomeThing(user);
    log(user);
}

//then
  
void work(User user) {
    try{
        threadLocalUser.set(user);
        // 他们内部  User u = threadLocalUser.get(); 就好了
        getInfo();
        checkInfo();
        setSomeThing();
        log();
    } finally {
        threadLocalUser.remove();
    }
}
```

很多场景的cookie，session等数据隔离都是通过ThreadLocal去做实现的

# Reference

- [Java面试必问：ThreadLocal终极篇](https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453144870&idx=1&sn=9be678c536d061e0c0d10db1be58dc07&scene=21#wechat_redirect)
- [Java中的ThreadLocal详解](https://www.cnblogs.com/fsmly/p/11020641.html)
- [ThreadLocal有什么缺陷](https://blog.csdn.net/qunqunstyle99/article/details/94717256)



