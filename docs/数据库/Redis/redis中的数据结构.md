# 1. Redis数据存储格式

- Redis自身是一个Map，其中所有的数据都是采用<span style="color:blue">key : value</span>的形式存储

- <span style="color:red">数据类型</span>指的是存储的数据的类型，也就是 value 部分的类型，key 部分永远都是字符串

<center><img src="https://ss.im5i.com/2021/07/19/gxjPG.png" alt="gxjPG.png" border="0" /></center>

# 2. 常用类型

|                | String                                                       | List                                                         | Set                                                          | SortedSet                                                    | Hash                                                         |
| :------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Java中的对应类 | Object类（任何对象都会序列化成String来存储）                 | java.util.List接口的实现类java.util.LinkedList               | java.util.Set接口                                            | java.util.SortedSet接口                                      | java.util.HashMap                                            |
| 简介           | String类型是Redis中最为基础的数据存储类型，是二进制安全的字符串，该类型可以接受任何格式的数据，如JPEG图像数据或Json对象描述信息等。在Redis中String类型的Value最多可以容纳的数据长度是512M。对于其他的几种数据结构，只有String类型的命令在写入key的时候可以带有默认的过期时间。对于其他的数据结构，key默认是不过期的，如果需要设置过期时间，必须显示调用expire函数设置过期时间 | List类型是**按照插入顺序排序的字符串链表。和数据结构中的普通链表一样**，可以在其头部(left)和尾部(right)添加新的元素。在插入时，如果该键并不存在，Redis将为该键创建一个新的链表。与此相反，如果链表中所有的元素均被移除，那么该键也将会被从数据库中删除。从元素插入和删除的效率视角来看，如果是在链表的两头插入或删除元素，这将会是非常高效的操作，即使链表中已经存储了百万条记录，该操作也可以在常量时间内完成。然而需要说明的是，如果元素插入或删除操作是作用于链表中间，那将会是非常低效的。 | **没有排序的字符串集合**，和List类型一样，也可以在该类型的数据值上执行添加、删除或判断某一元素是否存在等操作。和List类型不同的是，**Set集合中不允许出现重复的元素**，如果多次添加相同元素，Set中将仅保留该元素的一份拷贝 | SortedSet和Set类型极为相似，它们都是**字符串的集合**，都不允许重复的成员出现在一个Set中。主要差别是**SortedSet中的每一个成员都会有一个分数(score)与之关联**，Redis正是通过分数来为集合中的成员进行从小到大的排序。需要额外指出的是，尽管SortedSet中的成员必须是唯一的，但是**分数(score)却是可以重复的**。在**SortedSet中添加、删除或更新一个成员都是非常快速的操作，其时间复杂度为O(logn)**。由于SortedSet中的成员在集合中的位置是有序的，因此，即便是访问位于集合中部的成员也仍然是非常高效的。 | 相当于Java中的HaspMap，非常适合于存储值对象的信息，比如User对象含有userName、Password和Age等属性，可以使用hash来存储User，每个field对应一个属性，好处是可以做到部分更新、获取。如果Hash中包含很少的字段，那么该类型的数据也将仅占用很少的磁盘空间 |
| 使用场景       | 1. 借助incr/decr用于控制数据库表主键id，为数据库表主键提供生成策略，保障数据库表的主键唯一性 2. 控制数据的生命周期，通过数据是否失效控制业务行为 | 新闻网站，可以把每个新提交的的链接添加到一个list博客引擎实现中，可为每篇博客设置一个list，在该list中推入进博客评论 | 跟踪一些唯一性数据，比如访问某一博客的唯一IP地址信息，仅需在每次访问该博客时将访问者的IP存入Redis中，Set数据类型会自动保证IP地址的唯一性 | 可以用于一个大型在线游戏的积分排行榜。每当玩家的分数发生变化时，可以执行ZADD命令更新玩家的分数，此后再通过ZRANGE命令获取积分TOP TEN的用户信息。当然也可以利用ZRANK命令通过username来获取玩家的排行信息。最后将组合使用ZRANGE和ZRANK命令快速的获取和某个玩家积分相近的其他用户的信息。SortedSet类型还可用于构建索引数据。建立一个SortedSet中元素个数不要超过1W | 1. 对于海量数据的情况，可以自己对数据进行分桶，然后使用Hash结构来存储。2. 将对象存储为Hash结构而不是String，可以每次只更新、获取Hash中的一个field，这样可以提高效率 3. 购物车设计 |

HyperLogLog

HyperLogLog类型用来进行基数统计。利用HyperLogLog，用户可以使用少量固定大小的内存，来统计集合中唯一元素的数量（每个HyperLogLog占用12KB内存，可以计算接近264个不同元素的基数）

利用HyperLogLog得到的基数统计结果，不是精确值，而是一个带有0.81%标准差（standard error）的近似值。所以，HyperLogLog适用于一些对于统计结果精确度要求不是特别高的场景。

使用场景：

1.可以用于统计一个网站的UV。利用HyperLogLog来统计访问一个网站的不同ip的个数

# 3. 常用类型存储格式

## 3.1 string

<center><img src="https://ss.im5i.com/2021/07/19/gxTjz.png" alt="gxTjz.png" border="0" /></center>

## 3.2 hash

<center><img src="https://ss.im5i.com/2021/07/19/gxWe5.png" alt="gxWe5.png" border="0" /></center>

## 3.3 list

底层采用双向链表

<center><img src="https://ss.im5i.com/2021/07/19/gx266.png" alt="gx266.png" border="0" /></center>

## 3.4 set

与hash存储结构完全相同，仅存储键，不存储值（nil），并且键是不允许重复的

<center><img src="https://ss.im5i.com/2021/07/19/gx9U8.png" alt="gx9U8.png" border="0" /></center>

## 3.5 sorted_set类型

在set的存储结构基础上添加可排序字段

<center><img src="https://ss.im5i.com/2021/07/19/gxRiU.png" alt="gxRiU.png" border="0" /></center>











 