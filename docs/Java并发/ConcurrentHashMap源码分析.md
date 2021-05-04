`ConcurrentHashMap`是`J.U.C(java.util.concurrent包)`的重要成员，它是`HashMap`的一个线程安全的、支持高效并发的版本

# JDK7 ConcurrentHashMap

## 底层结构

它维护了一个 `Segment` 数组，每个 `Segment` 对应一把锁 (`Segment`继承了`ReentrantLock`) 

- 优点：如果多个线程访问不同的 `Segment`，实际是没有冲突的，这与 jdk8 中是类似的
- 缺点：`segments` 数组默认大小为16，这个容量初始化指定后就不能改变了，并且不是懒惰初始化  

无论是读操作还是写操作都能保证很高的性能：在进行读操作时(几乎)不需要加锁，而在写操作时通过锁分段技术只对所操作的段加锁而不影响客户端对其它段的访问。在理想状态下，`ConcurrentHashMap` 可以支持 16 个线程执行并发写操作（发生在不同的段上）（如果并发级别设为16），及任意数量线程的读操作

![JDK1.7的ConcurrentHashMap](https://img-blog.csdnimg.cn/img_convert/aa8338b765f04f405db0d9fb4fd710e6.png)


**在JDK7中，`ConcurrentHashMap`本质上是一个`Segment`数组，而一个`Segment实例`又包含若干个桶，每个桶中都包含一条由若干个 `HashEntry` 对象链接起来的链表**。

## 重要属性和内部类

```java
// HashEntry 用来封装具体的 Key/Value 对
static final class HashEntry<K,V> {
    final int hash;
    final K key;
    volatile V value;
    volatile HashEntry<K,V> next;

    HashEntry(int hash, K key, V value, HashEntry<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
    // ... 
}

// Segment 用来充当锁的角色，每个 Segment 对象守护整个ConcurrentHashMap的若干个桶，每个Segment对应一个小的哈希表
static final class Segment<K,V> extends ReentrantLock implements Serializable {
    // ...
    /**
     * 每个 Segment 对象用来守护它的成员对象 table 中包含的若干个桶(链表数组)
     * The per-segment table. Elements are accessed via
     * entryAt/setEntryAt providing volatile semantics.
     */
    transient volatile HashEntry<K,V>[] table;
    
    /**
     * 表示每个 Segment 对象管理的 table 数组包含的 HashEntry 对象的个数，也就是 Segment 中包含的 HashEntry 对象的总数
     * The number of elements. Accessed only either within locks
     * or among other volatile reads that maintain visibility.
     */
    transient int count;
    
    /**
     * 对count的大小造成影响的操作的次数（比如put或者remove操作）
     * The total number of mutative operations in this segment.
     * Even though this may overflows 32 bits, it provides
     * sufficient accuracy for stability checks in CHM isEmpty()
     * and size() methods.  Accessed only either within locks or
     * among other volatile reads that maintain visibility.
     */
     transient int modCount;
    
    /**
     * 阈值，段中元素的数量超过这个值就会对Segment进行扩容
     * The table is rehashed when its size exceeds this threshold.
     * (The value of this field is always <tt>(int)(capacity *
     * loadFactor)</tt>.)
     */
    transient int threshold;
    /**
     * 段的负载因子，其值等同于ConcurrentHashMap的负载因子
     * The load factor for the hash table.  Even though this value
     * is same for all segments, it is replicated to avoid needing
     * links to outer object.
     * @serial
     */
    final float loadFactor;
    // ...
}

/**
 * 一个ConcurrentHashMap实例中包含由若干个Segment实例组成的数组，而一个Segment实例又包含由若干个桶，每个桶中都包含一条由若干个HashEntry 对象链接起来的链表
 * The segments, each of which is a specialized hash table.
 */
final Segment<K,V>[] segments;

/**
 * 用于定位段，大小等于segments数组的大小减 1，是不可变的
 * Mask value for indexing into segments. The upper bits of a
 * key's hash code are used to choose the segment.
 */
final int segmentMask;

/**
 * 用于定位段，大小等于32(hash值的位数)减去对segments的大小取以2为底的对数值，是不可变的
 * Shift value for indexing within segments.
 */
final int segmentShift;
```

## 构造方法

```java
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // Find power-of-two sizes best matching arguments
    int sshift = 0;
    // ssize 必须是 2^n, 即 2, 4, 8, 16 ... 表示了 segments 数组的大小
    int ssize = 1;
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    // segmentShift 默认是 32 - 4 = 28
    this.segmentShift = 32 - sshift;
    // segmentMask 默认是 15 即 0000 0000 0000 1111
    this.segmentMask = ssize - 1;
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    // 每个段所拥有的桶的数目(2的幂次方,默认值为2)
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    while (cap < c)
        cap <<= 1;
    // create segments and segments[0]
    // segments数组并不是懒惰初始化
    Segment<K,V> s0 =
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    this.segments = ss;
}
```

- JDK7中，`ConcurrentHashMap` 没有实现懒惰初始化，空间占用不友好  
- `this.segmentShift` 和` this.segmentMask` 的作用是决定将 `key` 的 `hash` 结果匹配到哪个 `Segment  `
  - 例如，根据某一 `hash` 值求 `Segment` 位置，先将其高位向低位移动 `this.segmentShift` 位 ，结果再与 `this.segmentMask` 做位与运算，最终得到 `segments` 数组中的下标 
  - 假设`ConcurrentHashMap`一共分为$2^n$个段，每个段中有$2^m$个桶，那么段的定位方式是将`key`的`hash`值的高n位与$2^{n}-1$相与。在定位到某个段后，再将`key`的`hash`值的低m位与$2^m-1$相与，定位到具体的桶位
  - **根据`key`的`hash`值的高n位就可以确定元素到底在哪一个`Segment`中**

```java
/**
 * 构造一个具有默认初始容量(16)、默认负载因子(0.75)和默认并发级别(16)的空ConcurrentHashMap
 * Creates a new, empty map with a default initial capacity (16),
 * load factor (0.75) and concurrencyLevel (16).
 */
public ConcurrentHashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
}
```

## put()

**用分段锁机制实现多个线程间的并发写操作**，`ConcurrentHashMap`对`Segment`的`put`操作是加锁完成的，`Segment`是`ReentrantLock`的子类，因此`Segment`本身就是一种可重入的`Lock`，所以可以直接调用其继承而来的`lock()`方法和`unlock()`方法对代码进行上锁/解锁。这里的加锁操作是针对某个具体的`Segment`，**锁定的也是该`Segment`而不是整个`ConcurrentHashMap`**。因为插入键/值对操作只是在这个`Segment`包含的某个桶中完成，不需要锁定整个`ConcurrentHashMap`。因此，其他写线程对另外15个`Segment`的加锁并不会因为当前线程对这个`Segment`的加锁而阻塞。大概流程如下：

1. 根据 key 的 `hash`值定位到 `Segment` , `segments`的数组下标为：`j = (hash >>> segmentShift) & segmentMask`
2. 若 `Segment` 对象此时为 `null`，则通过 cas 的方式创建 `Segment` 对象
3. 进入 `Segment` 的 `put` 流程：
   - 尝试加锁 `tryLock() `
     - 若加锁失败，则进入 `scanAndLockForPut`流程，在多核CPU下，会最多 `tryLock` 64 次, 然后调用 `lock()`方法进入阻塞态，等待获得锁；在尝试期间, 还可以顺便看该节点在链表中有没有, 如果没有顺便创建出来
   - 加锁成功后
     - 首先遍历链表，检查该桶中是否存在相同`key`的节点，若存在则更新 `value` ，随后退出 `for` 循环
     - 否则创建新节点，插入链表头部
     - 如果 `Segment` 中元素数量 `c` 超过阈值，则在 `rehash()`中进行扩容，扩容完成后，才加入新的节点

```java
/**
 * Maps the specified key to the specified value in this table.
 * Neither the key nor the value can be null.
 *
 * @throws NullPointerException if the specified key or value is null
 */
@SuppressWarnings("unchecked")
public V put(K key, V value) {
    Segment<K,V> s;
    // key 与 value 都不能为 null 
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key);
    // 计算出 Segment 下标
    int j = (hash >>> segmentShift) & segmentMask;
    // 获得 Segment 对象, 判断是否为 null, 是则创建该 Segment
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        // 这时不能确定是否真的为 null, 因为其它线程也发现该 Segment 为 null,
        // 因此在 ensureSegment 里用 cas 方式保证该 Segment 安全性
        s = ensureSegment(j);
    // 进入 Segment 的 put 流程
    return s.put(key, hash, value, false);
}

// Segment 类中的方法
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 尝试加锁
    HashEntry<K,V> node = tryLock() ? null :
    	// 如果不成功，进入 scanAndLockForPut 流程
    	// 如果是多核 cpu 最多 tryLock 64 次, 进入 lock 流程
    	// 在尝试期间, 还可以顺便看该节点在链表中有没有, 如果没有顺便创建出来
    	scanAndLockForPut(key, hash, value);
    
    // 执行到这里 Segment 已经被成功加锁，可以安全执行
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;
        // 定位到段中的桶
        int index = (tab.length - 1) & hash;
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                K k;
                // 遍历
                // 检查该桶中是否存在相同key的节点，若存在则更新 value ，随后退出 for 循环
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            }
            else {
                // 新增
                // 1) 之前等待锁时，node 已经在 scanAndLockForPut 中被创建，next 指向链表头
                if (node != null)
                    node.setNext(first);
                else
                    // 2) 创建新 node 
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
                // 3) 扩容
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    // 新增节点 node 作为 链表头，头插
                    setEntryAt(tab, index, node);
                // 结构性修改，modCount加1
                ++modCount;
                // count值更新
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        // 解锁 
        // 同一个Segment有竞争时，加锁是在 scanAndLockForPut() 中完成的
        unlock();
    }
    return oldValue;
}

// Segment 类中的方法
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1; // negative while locating node
    while (!tryLock()) {
        HashEntry<K,V> f; // to recheck first below
        if (retries < 0) {
            if (e == null) {
                // 创建 node 
                if (node == null) // speculatively create node
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            }
            else if (key.equals(e.key))
                retries = 0;
            else
                e = e.next;
        }
        // 最多重试 64 次
        else if (++retries > MAX_SCAN_RETRIES) {
            lock();
            break;
        }
        else if ((retries & 1) == 0 &&
                 (f = entryForHash(this, hash)) != first) {
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }
    return node;
}
```

### rehash()

发生在 `put` 中，因为此时已经获得了锁，因此 `rehash` 时不需要考虑线程安全 ,**`ConcurrentHashMap`的重哈希实际上是对`ConcurrentHashMap`的某个段的重哈希，因此`ConcurrentHashMap`的每个段所包含的桶位自然也就不尽相同**

```java
// Segment 类中的方法
private void rehash(HashEntry<K,V> node) {
    /*
     * Reclassify nodes in each list to new table.  Because we
     * are using power-of-two expansion, the elements from
     * each bin must either stay at same index, or move with a
     * power of two offset. We eliminate unnecessary node
     * creation by catching cases where old nodes can be
     * reused because their next fields won't change.
     * Statistically, at the default threshold, only about
     * one-sixth of them need cloning when a table
     * doubles. The nodes they replace will be garbage
     * collectable as soon as they are no longer referenced by
     * any reader thread that may be in the midst of
     * concurrently traversing table. Entry accesses use plain
     * array indexing because they are followed by volatile
     * table write.
     */
    // 扩容前的table
    HashEntry<K,V>[] oldTable = table;
    int oldCapacity = oldTable.length;
    // 扩容为原来的两倍
    int newCapacity = oldCapacity << 1;
    // 新的阈值
    threshold = (int)(newCapacity * loadFactor);
    // 新创建一个table，其容量是原来的2倍
    HashEntry<K,V>[] newTable =
        (HashEntry<K,V>[]) new HashEntry[newCapacity];
    // 用于定位桶
    int sizeMask = newCapacity - 1;
    for (int i = 0; i < oldCapacity ; i++) {
        // 依次指向旧table中的每个桶的链表表头
        HashEntry<K,V> e = oldTable[i];
        // 旧table的该桶中链表不为空
        if (e != null) {
            // 记录下一个节点
            HashEntry<K,V> next = e.next;
            // 重哈希定位到新桶
            int idx = e.hash & sizeMask;
            //  旧table的该桶中只有一个节点
            if (next == null)   //  Single node on list
                newTable[idx] = e;
            else { // Reuse consecutive sequence at same slot
                HashEntry<K,V> lastRun = e;
                int lastIdx = idx;
                // 过一遍链表, 尽可能把 rehash 后 idx 不变的节点重用
                for (HashEntry<K,V> last = next;
                     last != null;
                     last = last.next) {
                    int k = last.hash & sizeMask;                   
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                // 直接将子链lastRun放到newTable[lastIdx]桶中
                newTable[lastIdx] = lastRun;
                // Clone remaining nodes
                // 剩余节点需要新建
                // 对该子链之前的结点，挨个遍历并把它们复制到新桶中
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }
    // 扩容完成, 才加入新的节点
    int nodeIndex = node.hash & sizeMask; // add the new node
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    // table 指向 newTable
    table = newTable;
}
```

> Because we are using power-of-two expansion, the elements from each bin must either stay at same index, or move with a power of two offset.

JDK官方的注释已经解释的很清楚了。**由于扩容是按照2的幂次方进行的，所以扩容前在同一个桶中的元素，现在要么还是在原来的序号的桶里，或者就是原来的序号再加上一个2的幂次方，就这两种选择**

## get()

**`get` 时并未加锁，用了 `UNSAFE` 方法保证了可见性**，扩容过程中，`get` 先发生就从旧表取内容，`get` 后发生就从新表取内容  

```java
public V get(Object key) {
    Segment<K,V> s; // manually integrate access methods to reduce overhead
    HashEntry<K,V>[] tab;
    int h = hash(key);
    // 定位到段， u 为 Segment 对象在数组中的偏移量
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    // s 即为 Segment
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        // 遍历链表，查找链中是否存在指定Key的键值对
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
             e != null; e = e.next) {
            K k;
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null;
}
```

## size()

- 计算元素个数前，**先不加锁计算两次，如果前后两次结果一样**，认为个数正确返回  
- 如果不一样，进行重试，**重试次数超过 3，将所有 Segment 锁住，重新计算个数返回**  

```java
/**
 * Returns the number of key-value mappings in this map.  If the
 * map contains more than <tt>Integer.MAX_VALUE</tt> elements, returns
 * <tt>Integer.MAX_VALUE</tt>.
 *
 * @return the number of key-value mappings in this map
 */
public int size() {
    // Try a few times to get accurate count. On failure due to
    // continuous async changes in table, resort to locking.
    final Segment<K,V>[] segments = this.segments;
    int size;
    boolean overflow; // true if size overflows 32 bits
    long sum;         // sum of modCounts
    long last = 0L;   // previous sum
    int retries = -1; // first iteration isn't retry
    try {
        for (;;) {
            if (retries++ == RETRIES_BEFORE_LOCK) {
                // 超过重试次数, 需要创建所有 Segment 并加锁
                for (int j = 0; j < segments.length; ++j)
                    ensureSegment(j).lock(); // force creation
            }
            sum = 0L;
            size = 0;
            overflow = false;
            for (int j = 0; j < segments.length; ++j) {
                Segment<K,V> seg = segmentAt(segments, j);
                if (seg != null) {
                    sum += seg.modCount;
                    int c = seg.count;
                    if (c < 0 || (size += c) < 0)
                        overflow = true;
                }
            }
            if (sum == last)
                break;
            last = sum;
        }
    } finally {
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                segmentAt(segments, j).unlock();
        }
    }
    return overflow ? Integer.MAX_VALUE : size;
}
```



# JDK8 ConcurrentHashMap  

## 底层结构

Java 8 中，`ConcurrentHashMap`改成了与`HashMap`一样的数据结构(数组+链表/红黑树)，使用`synchronized+CAS`来保证线程安全

![Java8 ConcurrentHashMap 存储结构（图片来自 javadoop）](https://img-blog.csdnimg.cn/img_convert/980c3cb6aed390fd3608b48301611ef0.png)

## 重要属性和内部类

```java
/**
 * hash 表，volatile 配合 CAS 来保证线程安全
 * The array of bins. Lazily initialized upon first insertion.
 * Size is always a power of two. Accessed directly by iterators.
 */
transient volatile Node<K,V>[] table;

/**
 * 扩容时的新 hash 表
 * The next table to use; non-null only while resizing.
 */
private transient volatile Node<K,V>[] nextTable;


/**
 * 默认为0
 * 当初始化时，为-1
 * 当扩容时，为 -(1 + 扩容线程数)
 * 当初始化或扩容完成后，为下一次的扩容的阈值大小
 * Table initialization and resizing control.  When negative, the
 * table is being initialized or resized: -1 for initialization,
 * else -(1 + the number of active resizing threads).  Otherwise,
 * when table is null, holds the initial table size to use upon
 * creation, or 0 for default. After initialization, holds the
 * next element count value upon which to resize the table.
 */
private transient volatile int sizeCtl;

/**
 * 整个 ConcurrentHashMap 就是一个 Node[]
 */
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;
    // ... 
}

/**
 * 扩容时如果某个 bin 迁移完毕, 用 ForwardingNode 作为旧 table bin 的头结点
 * A node inserted at head of bins during transfer operations.
 */
static final class ForwardingNode<K,V> extends Node<K,V> {}

/**
 * 作为 treebin 的头节点, 存储 root 和 first
 */
static final class TreeBin<K,V> extends Node<K,V> {}

/**
 *  作为 treebin 的节点, 存储 parent, left, right
 */
static final class TreeNode<K,V> extends Node<K,V> {}
```

## 重要方法

```java
// 获取 Node[] 中第 i 个 Node
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}

// cas 修改 Node[] 中第 i 个 Node 的值, c 为旧值, v 为新值
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
// 直接修改 Node[] 中第 i 个 Node 的值, v 为新值
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
    U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
}
```

## get()：无锁

流程如下：

1. 调用`spread()`方法保证`key.hashCode()`返回正整数
2. 如果`table`为`null`直接返回`null`
3. 如果`table`不为`null`，通过`(n-1)&h`定位到所在的桶
   - 如果头结点是要查找的`key`（先比较`hash`值，再通过`==`或`equals()`方法比较`key`值）,则直接返回
   - 如果头结点的`hash`值为负数，表示 bin 在扩容中(`ForwardingNode`的值为-1)或是 treebin (树节点) （`TreeBin`也是负数-2），这时调用`find()`方法来查找
   - 否则，就正常遍历链表，查找对应的键值对

```java
public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    	// spread 方法能确保返回结果是正数，负数有特殊用途（扩容或树节点）
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            // 如果头结点已经是要查找的 key
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            // 头结点的 hash 为负数表示该 bin 在扩容中或是 treebin, 这时调用 find 方法来查找
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            // 正常遍历链表
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```



## put()

- 数组简称 table，链表简称 bin
- `key`和`value`都不能为`null`

流程如下：

1. 调用`spread()`方法获得`key`的`hash`值
2. 进入一个死循环
   - 如果 table 为 `null`，则调用`initTable()`进行初始化（懒惰初始化），初始化 table 无需 `synchronized` 加锁，使用 cas 机制保证只会有一个线程初始化成功，初始化成功后，进入下一轮循环
   - 如果 table 不为 `null`，通过 `(n-1)&hash` 计算出桶下标，通过 cas 获取链表头节点`f`，若`f==null`，则使用 cas 创建一个节点作为链表头结点，cas 创建成功，则退出死循环，cas失败，则再次进入下一轮循环
   - 如果`f.hash==-1`说明当前`f`是`ForwardingNode`节点，表示正在扩容，则调用`helpTransfer()`帮忙扩容，扩容完成后，进入下一轮循环
   - 如果当前既不是处在扩容过程中也不是处在初始化过程中，而且出现了哈希碰撞，则对头节点加锁`synchronized(f)`，再次利用`tabAt(tab, i) == f`判断，防止被其他线程修改；根据头结点的`hash`值`fh`的正负区分当前是链表还是红黑树
     - 若为正，说明`f`是链表结构的头结点，则遍历链表，更新`value`值或者追加一个新节点在链表尾（使用变量`binCount`计算链表中节点个数）
     - 若`f`是树节点，则在树结构上遍历元素，更新或者增加节点
     - 遍历完成后，会释放头结点的锁，如果`binCount >= TREEIFY_THRESHOLD`即链表长度大于等于8（且哈希桶的数量大于64，在`treeifyBin()`中进行判断），进行链表转红黑树，然后跳出死循环
3. 增加`size`计数，在`addCount()`中判断 table 是否需要扩容(类似`LongAdder`计数器，设置多个累加单元来进行计数)

```java
public V put(K key, V value) {
    // putIfAbsent=false:表示会用新值覆盖掉旧值
    return putVal(key, value, false);
}

/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // key 和 value 都不能为 null
    if (key == null || value == null) throw new NullPointerException();
    // spread 方法能确保返回结果是正数且会综合高低位，具有更好的 hash 性
    int hash = spread(key.hashCode());
    int binCount = 0;
    // 进入死循环
    for (Node<K,V>[] tab = table;;) {
        // f 是哈希桶的头节点
        // fh 是头结点的 hash
        // i 是在 table 中的下标
        Node<K,V> f; int n, i, fh;
        
        // table 未初始化
        if (tab == null || (n = tab.length) == 0)
            // 初始化 table 使用了 cas，无需 synchronized 加锁，创建成功，则进入下一轮循环
            tab = initTable();
        // 创建链表的头结点
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 使用cas添加链表头，若添加成功，则跳出死循环，否则进入下一轮循环
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        // 帮忙扩容
        else if ((fh = f.hash) == MOVED)
            // 帮忙之后，进入下一轮循环
            tab = helpTransfer(tab, f);
        else {
            // 既不是处在扩容过程中也不是处在初始化过程中，而且出现了哈希碰撞
            V oldVal = null;
            // 锁住头结点
            synchronized (f) {
                // 再次确认头结点，防止被其他线程修改
                if (tabAt(tab, i) == f) {
                    // 链表
                    if (fh >= 0) {
                        // 统计桶中链表节点个数
                        binCount = 1;  
                        // 遍历链表
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            // 找到相同的 key
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                // 更新 value
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            // 已经是最后的节点了，新增 Node， 追加至链表尾
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    // 红黑树
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        // 在树结构上遍历元素，更新或增加节点
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                              value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
                // 释放头节点的锁
            }
            if (binCount != 0) {
                // 链表长度大于等于树化阈值（默认8），链表转红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    // 增加 size 计数
    addCount(1L, binCount);
    return null;
}
```



### initTable() : table初始化

`sizeCtl`默认为0，如果`ConcurrentHashMap`实例化时有传参数，`sizeCtl`会是一个2的幂次方的值。所以执行第一次`put`操作的线程会执行`Unsafe.compareAndSwapInt`方法修改`sizeCtl`为-1，**有且只有一个线程能够修改成功，其它线程通过`Thread.yield()`让出CPU时间片等待table初始化完成**  

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        // sizeCtl<0，意味着另外的线程执行CAS操作成功，当前线程只需要让出CPU时间片
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        // 尝试将 sizeCtl 设置为-1(表示初始化 table)
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                // 某一个线程获得锁，创建 table， 这时其他线程会在 while() 循环中 yield 直至 table 创建完成
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    // 初始化 table
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    // 下一次扩容时的阈值
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

### addCount() : table 计数与扩容

```java
// check 是 之前 binCount 的个数
// x 代表计数值 1
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    // 计数
    if (
        // 已经有了 counterCells，向 cell 累加，进入当前 if 语句块
        (as = counterCells) != null ||
        // 还没有 counterCells，向 baseCount 累加，累加失败则进入当前 if 语句块
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)
    ) {
        CounterCell a; long v; int m;
        boolean uncontended = true;
        if (
            // 还没有 counterCells
            as == null || (m = as.length - 1) < 0 ||
            // 还没有 cell
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            // cell cas 增加计数失败
            !(uncontended = U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))
        ) {
            // 创建累加单元数组和 cell，累加重试
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)
            return;
        // 获取元素个数
        s = sumCount();
    }
    // 检查是否需要扩容
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if (
                    (sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0
                )
                    break;
                // newtable 已经创建了，帮忙扩容
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            // 需要扩容，这时 newtable 未创建
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}
```

#### table扩容

当table中的元素数量达到容量阈值`sizeCtl`时，需要对table进行扩容，整个扩容分为两部分：

1. 初始化一个nextTable，大小为table的两倍
   - 通过`U.compareAndSwapInt`修改sizeCtl值，**保证只有一个线程能够初始化nextTable**，扩容后的数组长度为原来的两倍
2. 把table中的数据复制到nextTable中

### transfer()：把table中的节点移动到nextTable中

节点从table移动到nextTable，大体思想是遍历、复制的过程：

1. 首先根据运算得到需要遍历的次数`i`，然后利用`tabAt()`方法获得`i`位置的元素`f`，初始化一个`ForwardingNode`实例`fwd`
2. 如果`f==null`，则利用cas在`i`位置放入`fwd`
3. 如果`f`是链表的头结点，就构造一个反序链表，把他们分别放在nextTable的`i`和`i+n`的位置上，移动完成后，给table原位置放置`fwd`
4. 如果`f`是`TreeBin`节点，也做一个反序处理，并判断是否需要`untreeify`，把处理的结果分别放在nextTable的`i`和`i+n`的位置上，移动完成后，同样给table原位置放置`fwd`

遍历过所有的节点以后就完成了复制工作，把table指向nextTable，并更新sizeCtl为新数组大小的0.75倍，扩容完成

```java
/**
 * Moves and/or copies the nodes in each bin to new table. See
 * above for explanation.
 */
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    // 创建 nextTab
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    int nextn = nextTab.length;
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    // 节点的搬迁 
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        // 当前桶为空，则利用 cas 放置 ForwardingNode 节点
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            // 锁住头结点
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    // 链表
                    if (fh >= 0) {
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                    // 红黑树
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                            (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                            (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}
```

## size()

`size` 计算实际发生在 `put,remove` 改变集合元素的操作之中

- 没有竞争发生，向 `baseCount` 累加计数
- 有竞争发生，新建 `counterCells`，向其中的一个 `cell` 累加计数
  - `counterCells` 初始有两个 `cell`
  - 如果计数竞争比较激烈，会创建新的 `cell` 来累加计数  

```java
public int size() {
    long n = sumCount();
    return ((n < 0L) ? 0 :
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
            (int)n);
}

final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    // 将 baseCount 计数与所有 cell 计数累加
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```

## 总结

Java 8中`ConcurrentHashMap`底层数据结构采用数组(table)+链表(Node)|红黑树(TreeNode)，以下数组简称table，链表简称bin

- 初始化：使用CAS来保证并发安全，懒惰初始化table
- 树化：当 table.length < 64时，先尝试扩容，超过64时，并且 bin.length > 8 时，会将链表树化，树化过程会用 `synchronized`锁住链表头
- put：如果该 bin 尚未创建，只需要使用 cas 创建 bin；如果已经有了，锁住链表头进行后续 put 操作，新增节点添加至 bin 尾部
- get：无锁操作仅需要保证可见性，扩容过程中 get 操作拿到的是 `ForwardingNode` 它会让 get 操作在新 table 进行搜索
- 扩容：扩容是以 bin 为单位进行，需要对 bin 使用 `synchronized` 加锁，但这时其他竞争线程也不是无事可做，它们会帮助把其他 bin 进行扩容，扩容时平均只有 1/6 的节点会被复制到新 table 中
- size：元素个数保存在 baseCount 中，并发时的个数变动保存在 CounterCell[] 当中，最后统计数量时累加即可

# Reference

- [黑马程序员](https://www.bilibili.com/video/BV16J411h7Rd?p=281)

- [基于JDK 1.6 ConcurrentHashMap](https://blog.csdn.net/justloveyou_/article/details/72783008)