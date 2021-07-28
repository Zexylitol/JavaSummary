# 背景

Redis的数据已经设置了TTL，不是过期就已经删除了吗？为什么还存在所谓的淘汰策略呢？这个需要从Redis的过期策略聊起

## 过期策略

### 定期删除

redis 会将每个设置了过期时间的 key 放入到一个独立的字典中，以后会定期遍历这个字典来删除到期的 key。

Redis 默认会每秒进行十次过期扫描（100ms一次），**过期扫描不会遍历过期字典中所有的 key，而是采用了一种简单的贪心策略**。

1. 从过期字典中随机 20 个 key；

2. 删除这 20 个 key 中已经过期的 key；

3. 如果过期的 key 比率超过 1/4，那就重复步骤 1；

<span style="color:blue">Redis默认是每隔 100ms 就随机抽取一些设置了过期时间的key，检查其是否过期，如果过期就删除</span>。注意这里是随机抽取的。为什么要随机呢？假如 Redis 存了几十万个 key ，每隔100ms就遍历所有的设置过期时间的 key 的话，就会给 CPU 带来很大的负载

### 惰性删除

<span style="color:red">所谓惰性策略就是在客户端访问这个key的时候，redis对key的过期时间进行检查，如果过期了就立即删除，不会返回任何数据</span>。

定期删除可能会导致很多过期key到了时间并没有被删除掉。所以就有了惰性删除。假如过期 key靠定期删除没有被删除掉，还停留在内存里，除非系统去查一下那个 key，才会被redis给删除掉。这就是所谓的惰性删除，即当你主动去查过期的key时，如果发现key过期了,就立即进行删除，不返回任何数据。

<span style="color:blue">总结：定期删除是集中处理，惰性删除是零散处理</span>

## 为什么需要淘汰策略

<span style="color:green">因为不管是定期删除还是惰性删除都不是一种完全精准的删除，还是会存在key没有被删除掉的场景，所以就需要内存淘汰策略进行补充</span>。

# 内存淘汰策略

<center><img src="https://ss.im5i.com/2021/07/28/GrS9Q.png" alt="GrS9Q.png" border="0" /></center>

|    淘汰策略     | 说明                                                         |
| :-------------: | :----------------------------------------------------------- |
|   noeviction    | 添加数据时，如果redis判断该操作会导致占用内存大小超过内存限制，就返回error，然后啥也不干 |
|   allkeys-lru   | 添加数据时，如果redis判断该操作会导致占用内存大小超过内存限制，就会扫描所有的key，淘汰一些最近未使用的key |
|  volatile-lru   | 添加数据时，如果redis判断该操作会导致占用内存大小超过内存限制，扫描那些设置了过期时间的key，淘汰一些最近未使用的key |
| allkeys-random  | 添加数据时，如果redis判断该操作会导致占用内存大小超过内存限制，就会扫描所有的key，随机淘汰一些key |
| volatile-random | 添加数据时，如果redis判断该操作会导致占用内存大小超过内存限制，扫描那些设置了过期时间的key，随机淘汰一些key |
|  volatile-ttl   | 添加数据时，如果redis判断该操作会导致占用内存大小超过内存限制，扫描那些设置了过期时间的key，淘汰一些即将过期的key |
|  volatile-lfu   | 添加数据时，如果redis判断该操作会导致占用内存大小超过内存限制，就会淘汰一些设置了过期时间的，并且最近最少使用的key |
|   allkeys-lfu   | 添加数据时，如果redis判断该操作会导致占用内存大小超过内存限制，就会扫描所有的key，淘汰一些最近最少使用的key |

# 内存淘汰过程

> 1、A client runs a new command, resulting in more data added.
>
> 2、Redis checks the memory usage, and if it is greater than the maxmemory limit , it evicts keys according to the policy.
>
> 3、A new command is executed, and so forth.

1、客户端执行一个新指令，添加数据

2、Redis检查内存使用量，如果大于maxmemory限制，就通过淘汰策略清理内存

> Setting maxmemory to zero results into no memory limits. This is the default behavior for 64 bit systems, while 32 bit systems use an implicit memory limit of 3GB.
>
> 在64位的系统中，Redis是没有对内存容量做限制的，但是在32位系统中默认内存容量的默认值是3GB

3、执行新命令，重复上述过程

# Redis的LRU实现

<center><img src="https://ss.im5i.com/2021/07/28/Grga3.png" alt="Grga3.png" border="0" /></center>

在Redis中，LRU算法被做了简化，以减轻数据淘汰对缓存性能的影响。具体来说，Redis默认会记录每个数据的最近一次访问的时间戳 ( 由键值对数据结构RedisObject中的Iru字段记录)。然后，Redis在决定淘汰的数据时，第一次会随机选出N个数据，把它们作为一个候选集合。接下来，Redis 比较这N个数据的Iru段，把Iru段值最小的数据从缓存中淘汰出去。

当需要再次淘汰数据时，Redis 需要挑选数据进入第一次淘汰时创建的候选集合。这儿的挑选标准是：能进入候选集合的数据的Iru字段值必须小于候选集合中最小的Iru值。当有新数据进入候选数据集后，如果候选数据集中的数据个数达到了maxmemory-samples，Redis 就把候选数据集中Iru字段值最小的数据淘汰出去。这样一来，<span style="color:red">Redis缓存不用为所有的数据维护一个大链表，也不用在每次数据访问时都移动链表，提升了缓存的性能</span>。

Redis中的LRU与常规的LRU实现并不相同，常规LRU会准确的淘汰掉队头的元素，但是Redis的LRU并不维护队列，只是根据配置的策略要么从所有的key中随机选择N个（N可以配置）要么从所有的设置了过期时间的key中选出N个键，然后再从这N个键中选出最久没有使用的一个key进行淘汰。

# 使用建议

- 优先使用allkeys-lru策略。这样，可以充分利用LRU这一经典缓存算法的优势，把最近最常访问的数据留在缓存中，提升应用的访问性能。如果业务数据中有明显的冷热数据区分，建议使用allkeys-lru策略

- 如果业务应用中的数据访问频率相差不大，没有明显的冷热数据区分，建议使用allkeys-random策略，随机选择淘汰的数据就行

- 如果业务中有置顶的需求，比如置顶新闻、置顶视频，那么，可以使用volatile-lru策略，同时不给这些置顶数据设置过期时间。这样一来，这些需要置顶的数据一不会被删除，其他数据会在过期时根据LRU规则进行筛选

# Reference

- [彻底弄懂Redis的内存淘汰策略](https://zhuanlan.zhihu.com/p/105587132)
- [Redis内存淘汰策略详解](https://blog.csdn.net/henry_yang2018/article/details/107916516)

- [Redis核心技术与实战](https://time.geekbang.org/column/intro/100056701)