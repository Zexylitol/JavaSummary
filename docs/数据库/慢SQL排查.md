# 前言

高CPU消耗、高执行时间、高IO消耗以及高影响行数的SQL语句都有可能是慢SQL

# MySQL慢查询排查

## 慢查询日志

<span style="color:red">MySQL提供一种慢查询日志，用于记录MySQL中响应时间超过阈值的SQL语句，具体指运行时间超过`long_query_time`值得SQL，则会被记录到慢查询日志中</span>。

`long_query_time`的默认值为10，意思是运行10秒以上的语句。

由慢查询日志来查看哪些SQL超出了最大忍耐时间值，结合`explain`进行全面分析

**慢查询日志默认是关闭的；建议：开发调优时打开而最终部署时关闭**。

检查是否开启了慢查询日志：

```mysql
# 默认情况下slow_query_log 的值为OFF,表示慢查询日志是禁用的
show variables like '%slow_query_log%';
```

临时开启：

```mysql
set global_slow_query_log=1; # 内存开启
```

永久开启：

```shell
vi /etc/my.cnf

[mysqld]
show_query_log=1
show_query_log_file=/var/lib/mysql/localhost-slow.log
```

查看慢查询设定阈值：

```mysql
# 单位秒
show variables like '%long_query_time%';
```

临时设置慢查询阈值：

```mysql
show global long_query_time=5; # 设置完毕后，重新登录起效，不需要重启服务
```

永久设置阈值：

```shell
vi /etc/my.cnf

[mysqld]
long_query_time=3
```

慢查询的SQL被记录在了日志中，因此可以通过日志查看具体的慢SQL

```shell
cat /var/lib/mysql/localhost-slow.log
```

## 日志分析工具 mysqldumpslow

通过mysqldumpslow工具查看慢SQL，可以通过一些过滤条件快速找出需要定位的慢SQL

查看mysqldumpslow的帮助信息：

```shell
> mysqldumpslow --help
Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]

Parse and summarize the MySQL slow query log. Options are

  --verbose    verbose
  --debug      debug
  --help       write this text to standard output

  -v           verbose
  -d           debug
  -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default
                al: average lock time
                ar: average rows sent
                at: average query time
                 c: count
                 l: lock time
                 r: rows sent
                 t: query time  
  -r           reverse the sort order (largest last instead of first)
  -t NUM       just show the top n queries
  -a           don't abstract all numbers to N and strings to 'S'
  -n NUM       abstract numbers with at least n digits within names
  -g PATTERN   grep: only consider stmts that include this string 
  -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
               default is '*', i.e. match all
  -i NAME      name of server instance (if using mysql.server startup script)
  -l           don't subtract lock time from total time 
```

| 参数 | 描述                                     | 备注 |
| :--- | :--------------------------------------- | :--- |
| -s   | 表示按照何种方式排序                     |      |
| c    | 访问次数                                 |      |
| l    | 锁定时间                                 |      |
| r    | 返回记录                                 |      |
| t    | 查询时间                                 |      |
| al   | 平均锁定时间                             |      |
| ar   | 平均返回记录数                           |      |
| at   | 平均查询时间                             |      |
| -t   | 即为返回前面多少条的数据                 |      |
| -g   | 后边搭配一个正则匹配模式，大小写不敏感的 |      |

获取返回记录集最多的3个sql:

```shell
mysqldumpslow -s r -t 3 /var/log/mysql/mysql-slow.log
```

获取访问次数最多的3个sql:

```shell
mysqldumpslow -s c -t 3 /var/log/mysql/mysql-slow.log
```

按照时间排序，前10条包含left join查询语句的sql:

```shell
mysqldumpslow -s t -t -g "letf join" /var/log/mysql/mysql-slow.log
```

建议在使用这些命令时结合 | 和 more 使用，否则有可能出现爆屏情况：

```shell
mysqldumpslow -s r -t 10 /var/lib/mysql/mysql-slow.log | more
```

# show processlist

### 是什么

用于查询 mysql 进程列表，可以杀掉故障进程

### 怎么用

<center><img src="https://ss.im5i.com/2021/08/27/fryXq.png" alt="fryXq.png" border="0" /></center>

杀掉进程：

```mysql
kill Id;
```

# show profile

利用`show profile`可以查看SQL的执行周期！

## 开启profile

查看profile是否开启：

```mysql
show variables like '%profiling%';
```

开启profile：

```mysql
set profiling=1;
```

## 使用 profile

执行 `show profiles;`命令，可以查看最近的几次查询

```mysql
mysql> show profiles;
+----------+------------+-------------------------+
| Query_ID | Duration   | Query                   |
+----------+------------+-------------------------+
|        1 | 0.01291125 | show databases          |
|        2 | 0.02513475 | select miaosha          |
|        3 | 0.01309675 | SELECT DATABASE()       |
|        4 | 0.00038025 | show databases          |
|        5 | 0.00013275 | show tables             |
|        6 | 0.00960125 | show tables             |
|        7 | 0.03376150 | select * from item      |
|        8 | 0.00024525 | select * from promo     |
|        9 | 0.00016525 | select * from user_info |
|       10 | 0.00009600 | show profils            |
+----------+------------+-------------------------+
10 rows in set, 1 warning (0.03 sec)
```

根据Query_ID可以进一步执行`show profile cpu,block io for query Query_ID`来查看SQL的具体执行步骤：

```mysql
mysql> show profile cpu,block io for query 7;
+----------------------+----------+----------+------------+--------------+---------------+
| Status               | Duration | CPU_user | CPU_system | Block_ops_in | Block_ops_out |
+----------------------+----------+----------+------------+--------------+---------------+
| starting             | 0.033184 | 0.000291 |   0.032780 |         6584 |             0 |
| checking permissions | 0.000013 | 0.000000 |   0.000008 |            0 |             0 |
| Opening tables       | 0.000012 | 0.000001 |   0.000011 |            0 |             0 |
| init                 | 0.000015 | 0.000000 |   0.000014 |            0 |             0 |
| System lock          | 0.000058 | 0.000004 |   0.000123 |            0 |             0 |
| optimizing           | 0.000073 | 0.000001 |   0.000004 |            0 |             0 |
| statistics           | 0.000009 | 0.000000 |   0.000009 |            0 |             0 |
| preparing            | 0.000007 | 0.000000 |   0.000007 |            0 |             0 |
| executing            | 0.000002 | 0.000000 |   0.000002 |            0 |             0 |
| Sending data         | 0.000268 | 0.000009 |   0.000260 |            0 |             0 |
| end                  | 0.000005 | 0.000000 |   0.000004 |            0 |             0 |
| query end            | 0.000005 | 0.000000 |   0.000004 |            0 |             0 |
| closing tables       | 0.000006 | 0.000000 |   0.000005 |            0 |             0 |
| freeing items        | 0.000090 | 0.000003 |   0.000089 |            0 |             0 |
| cleaning up          | 0.000016 | 0.000001 |   0.000014 |            0 |             0 |
+----------------------+----------+----------+------------+--------------+---------------+
15 rows in set, 1 warning (0.03 sec)

```

# Reference

- [mysql常见的优化方法及慢查询sql排查](https://blog.csdn.net/zhangshuaijun123/article/details/105953379)

- [尚硅谷MySQL数据库高级](https://www.bilibili.com/video/BV1KW411u7vy)