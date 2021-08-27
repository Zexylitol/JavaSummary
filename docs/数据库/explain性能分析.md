# explain性能分析

使用`EXPLAIN`关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的，分析你的查询语句或是表结构的性能瓶颈。

用法：

```mysql 
explain SQL语句
```

Explain执行后返回的信息：

```mysql
mysql> explain select * from item where id = 5;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | item  | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

```

<center><img src="https://ss.im5i.com/2021/08/26/ffykL.png" alt="ffykL.png" border="0" /></center>

### id

`select`查询的序列号，包含一组数字，表示查询中执行`select`子句或操作表的顺序

- id相同，执行顺序由上至下
- id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
- id有相同也有不同
  - id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行

<span style="color:red">关注点：id号每个号码，表示一趟独立的查询。一个sql的查询趟数越少越好。</span>

### select_type

select_type代表查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询。

|     select_type      | 含义                                                         |
| :------------------: | :----------------------------------------------------------- |
|        SIMPLE        | 简单的 `select` 查询，查询中不包含子查询或者UNION（单表查询）<br/>`explain select * from t1;` |
|       PRIMARY        | 查询中若包含任何复杂的子部分，最外层查询则被标记为Primary<br/>`explain select * from (select t1.id from t1) a;` |
|       DERIVED        | 在FROM列表中包含的子查询被标记为DERIVED(衍生)<br/>MySQL会递归执行这些子查询，把结果放在临时表里。 |
|       SUBQUERY       | 在SELECT或WHERE列表中包含了子查询<br/>`explain select t2.id from t2 where t2.id = (select t3.id from t3 where t3.id=2);` |
|  DEPEDENT SUBQUERY   | 在SELECT或WHERE列表中包含了子查询,子查询基于外层<br/>`explain select t2.id from t2 where t2.id in (select t3.id from t3 where t3.content='abc');` |
| UNCACHEABLE SUBQUERY | 无法使用缓存的子查询<br/>当使用了`@@`来引用系统变量的时候，不会使用缓存<br/>`explain select * from t3 where id=(select id from t2 where t2.id=@@sort_buffer_size);` |
|        UNION         | 若第二个SELECT出现在UNION之后，则被标记为UNION;<br/>若UNION包含在FROM子句的子查询中，外层SELECT将被标记为:DERIVED<br/>`explain select t2.id,t2.content from t2 UNION ALL select t3.id, t3.content from t3;` |
|     UNION RESULT     | 从UNION表获取结果的SELECT                                    |

### table

这个数据是基于哪张表的。

### type

type是查询的访问类型。是较为重要的一个指标，结果值从最好到最坏依次是：

```
system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
```

<span style="color:red">一般来说，得保证查询至少达到range级别，最好能达到ref</span>。

#### system

表只有一行记录（等于系统表)，这是const类型的特列，平时不会出现，这个也可以忽略不计

#### const

**表示通过索引一次就找到了**，const用于比较**primary key或者unique索引**。

因为只匹配一行数据，所以很快如将主键置于where列表中，MySQL就能将该查询转换为一个常量。

#### eq_ref

**唯一性索引扫描**，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描。

#### ref

**非唯一性索引扫描，返回匹配某个单独值的所有行**

本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以它应该属于查找和扫描的混合体

#### range

**只检索给定范围的行，使用一个索引来选择行**。

key列显示使用了哪个索引一般就是在你的`where`语句中出现了`between、<、>、in`等的查询这种范围扫描索引扫描，比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引

#### index

出现index是sql使用了索引但是没有通过索引进行过滤，<span style="color:red">一般是使用了覆盖索引或者是利用索引进行了排序分组</span>。

<center><img src="https://ss.im5i.com/2021/08/26/fftA3.png" alt="fftA3.png" border="0" /></center>

#### all

Full Table Scan，将遍历全表以找到匹配的行

#### index_merge

在查询过程中需要多个索引组合使用，通常出现在有 or 的关键字的sql中。

#### ref_or_null

对于某个字段既需要关联条件，也需要null值得情况下。查询优化器会选择用ref_or_null连接查询。

```mysql
explain select * from t2.content is null or t2.content='abc';
```

#### index_subquery

利用索引来关联子查询，不再全表扫描。

#### unique_subquery

该联接类型类似于index_subquery。子查询中的唯一索引。

### possible_keys

显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，<span style="color:red">但不一定被查询实际使用</span>。

### key

**实际使用的索引**。如果为NULL，则没有使用索引。

### key_len

**表示索引中使用的字节数**，可通过该列计算查询中使用的索引的长度。

key_len字段能够帮你检查是否充分的利用上了索引。

ken_len越长，说明索引使用的越充分。

**如何计算：**
①先看索引上字段的类型+长度比如`int=4; varchar(20)=20; char(20)=20`

②如果是`varchar`或者`char`这种字符串字段，视字符集要乘不同的值，比如`utf-8` 要乘3，`GBK`要乘⒉

③`varchar`这种动态字符串要加2个字节

④允许为空的字段要加1个字节

第一组：

```mysql
explain select * from emp where emp.age=30 and emp.name like 'ab%';
```

key_len = age 的字节长度 + name的字节长度 = 4 + 1 +(20*3+2) = 5 + 62 = 67

第二组：

```mysql
explain select * from emp where emp.age=30 and emp.name like '%abc%';
```

 key_len = age的字节长度 = 4 + 1 = 5

| 列类型                         | KEY_LEN                  | 备注                                                     |
| :----------------------------- | :----------------------- | :------------------------------------------------------- |
| id int                         | key_len = 4 + 1 = 5      | 允许为null，加1byte                                      |
| id int not null                | key_len = 4              | 不允许为null                                             |
| user char(30) utf8             | key_len = 30 * 3 + 1     | 允许为null                                               |
| user varchar(30) not null utf8 | key_len = 30 * 3 + 2     | 动态列类型，加2bytes                                     |
| user varchar(30) utf8          | key_len = 30 * 3 + 2 + 1 | 动态列类型，加2bytes；允许null，再加1bytes               |
| detail text(10) utf8           | key_len = 30 * 3 + 2 + 1 | TEXT列截取部分，被视为动态列类型，加2bytes，且允许为null |

### ref

显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值。

### rows

rows列显示MySQL认为它执行查询时必须检查的行数。越少越好!

### Extra

其他的额外重要的信息。

**Using filesort：**

- 说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成的排序操作称为“文件排序”
- 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度。

**Using temporary：**

- 使了用临时表保存中间结果,MySQL在对查询结果排序时使用临时表。常见于排序order by和分组查询group
  by

**Using index：**

- Using index代表表示相应的select 操作中使用了覆盖索引(Covering Index)，避免访问了表的数据行，效率不错!如果同时出现 using where，表明索引被用来执行索引键值的查找;如果没有同时出现 using where，表明索引只是用来读取数据而非利用索引执行查找。
- 利用索引进行了排序或分组。

**Using where：**

- 表明使用了where过滤

**Using join buffer：**

- 使用了连接缓存

**impossible where：**

- where子句的值总是false，不能用来获取任何元组。

**select tables optimized away：**

- 在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。

# Reference

- [尚硅谷MySQL数据库高级](https://www.bilibili.com/video/BV1KW411u7vy)





