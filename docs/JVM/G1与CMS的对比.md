# G1的优点

- 可以指定最大停顿时间
- 分Region的内存布局
- 按收益动态确定回收集

# 回收方式

与CMS的"标记-清除"算法不同，G1从整体来看是基于"标记-整理"算法实现的收集器，但从局部（两个Region之间）上看又是基于"标记-复制"算法实现，这两种算法都意味着G1运作期间不会产生内存空间碎片，垃圾收集完成之后能提供规整的可用内存。这种特性有利于程序长时间运行，在程序为大对象分配内存时不容易因无法找到连续内存空间而提前触发下一次收集。

# 内存占用（Footprint）

G1和CMS都采用卡表来处理跨代指针

G1的卡表实现更为复杂，而且堆中每个Region，无论扮演的是新生代还是老年代角色，都必须有一份卡表，着导致G1的记忆集（和其他内存消耗）可能会占整个堆容量20%乃至更多的内存空间

CMS的卡表只有唯一一份，而且只需要处理老年代到新生代的引用，由于新生代的对象具有朝生夕灭的不稳定性，引用变化频繁，能省下这个区域的维护开销很划算，但是当CMS发生Old GC时（所有收集器中只有CMS有针对老年代的Old GC）,要把整个新生代作为GC Roots来进行扫描。

# 执行负载（Overload）

由于G1对写屏障的复杂操作要比CMS消耗更多的运算资源

CMS的写屏障实现是直接的同步操作

G1不得不将其实现为类似于消息队列的结构，把写前屏障和写后屏障中要做的事情都放到队列里，然后再异步处理

# 小结

在小内存应用上CMS的表现大概率会优于G1

在大内存应用上G1则大多能发挥其优势，优劣势的Java堆容量平衡点通常在6GB至8GB之间