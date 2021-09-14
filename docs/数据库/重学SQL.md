# 登录
```mysql
mysql 【-h主机名 -P端口号 】-u用户名 -p密码  

mysql -hlocalhost -P3306 -uroot -p1997  

mysql -uroot -p1997
```

# 重置密码

```mysql
mysqladmin -u root password root
```

# 退出
```shell
exit 

ctrl+c
```

# 查看MySQL版本
## 登录到MySQL服务端
```mysql
SELECT VERSION();
```

## 命令行
```shell
mysql --version

mysql -V
```

# 备份数据库

```mysql
mysqldump -u用户名 -p密码   --databases 数据库名 > 保存路径
```

# 常用

查看隔离级别：

```mysql
select @@tx_isolation;

show global variables like '%isolation%';
```

设置隔离级别：

```mysql
set global transaction_isolation ='read-committed';

-- 设置read uncommitted级别：
set session transaction isolation level read uncommitted;

-- 设置read committed级别：
set session transaction isolation level read committed;

-- 设置repeatable read级别：
set session transaction isolation level repeatable read;

# 设置serializable级别：
set session transaction isolation level serializable;

# session：当前会话，也就是当前连接。

# global：全局，不包含当前连接，之后新获取的连接都会生效。
```

设置事务的自动提交模式：

```mysql
SET autocommit = 0|1|ON|OFF;

BEGIN / START TRANSACTION;
# ...
commit;

# 回滚
rollback;
```

MySQL默认端口号：`3306`

```mysql
show global variables like 'port';
```

修改端口：编辑`/etc/my.cnf`文件

Redis默认端口号：`6379`

# 删除数据

常用的三种删除方式： `delete`、`truncate`、`drop`

```mysql
DELETE from TABLE_NAME where xxx;
Truncate table TABLE_NAME;
Drop table Tablename
```

## 原理

### delete

- `delete`属于数据库DML操作语言，**只删除数据不删除表的结构**，会走事务，执行时会触发trigger
- 在 InnoDB 中，`delete`其实**并不会真的把数据删除**，mysql 实际上只是给删除的数据打了个标记为已删除，因此 `delete` 删除表中的数据时，表文件在磁盘上所占空间不会变小，存储空间不会被释放，只是把删除的数据行设置为不可见。虽然未释放磁盘空间，但是下次插入数据的时候，仍然可以重用这部分空间（重用 → 覆盖）
- `delete`执行时，会先将所删除数据缓存到rollback segement中，事务`commit`之后生效
- `delete from table_name`删除表的全部数据，对于MyISAM会立刻释放磁盘空间，InnoDB 不会释放磁盘空间
- 对于`delete from table_name where xxx` 带条件的删除, 不管是InnoDB还是MyISAM都不会释放磁盘空间
- `delete`操作以后使用 `optimize table table_name `会立刻释放磁盘空间。不管是InnoDB还是MyISAM 
- `delete` 操作是**一行一行执行删除的，并且同时将该行的的删除操作日志记录在redo和undo表空间中以便进行回滚**（`rollback`）和重做操作，生成的大量日志也会占用磁盘空间

### truncate

- 属于数据库DDL定义语言，不走事务，原数据不放到 rollback segment 中，操作不触发 trigger
- 执行后立即生效，无法找回
- `truncate table table_name` 立刻释放磁盘空间 ，不管是 InnoDB和MyISAM
- `truncate`能够快速清空一个表。并且重置`auto_increment`的值
  - 对于MyISAM，`truncate`会重置`auto_increment`（自增序列）的值为1。而`delete`后表仍然保持`auto_increment`
  - 对于InnoDB，`truncate`会重置`auto_increment`的值为1。`delete`后表仍然保持`auto_increment`。但是在做`delete`整个表之后重启MySQL的话，则重启后的`auto_increment`会被置为1
    - 也就是说，InnoDB的表本身是无法持久保存`auto_increment`。`delete`表之后`auto_increment`仍然保存在内存，但是重启后就丢失了，只能从1开始。实质上重启后的auto_increment会从 `SELECT 1+MAX(ai_col) FROM t `开始

### drop

- `drop`是DDL，会隐式提交，所以，不能回滚，不会触发触发器
- `drop`语句删除表结构及所有数据，并将表所占用的空间全部释放
- `drop`语句将删除表的结构所依赖的约束，触发器，索引，依赖于该表的存储过程/函数将保留，但是变为invalid状态

## 执行速度

```
drop > truncate > delete
```

## 小结

- TRUNCATE 和DELETE只删除数据， DROP则删除整个表（结构和数据）
- delete语句为DML（data maintain Language),这个操作会被放到 rollback segment中,事务提交后才生效。如果有相应的 tigger,执行的时候将被触发。
- truncate、drop是DLL（data define language),操作立即生效，原数据不放到 rollback segment中，不能回滚
- truncate table 在功能上与不带 WHERE 子句的 DELETE 语句相同：二者均删除表中的全部行。但 TRUNCATE TABLE 比 DELETE 速度快，且使用的系统和事务日志资源少。DELETE 语句每次删除一行，并在事务日志中为所删除的每行记录一项。**TRUNCATE TABLE 通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放**
- **TRUNCATE TABLE 删除表中的所有行，但表结构及其列、约束、索引等保持不变。新行标识所用的计数值重置为该列的种子。如果想保留标识计数值，请改用 DELETE。如果要删除表定义及其数据，请使用 DROP TABLE 语句**

# MySQL的语法规范

1.不区分大小写,但建议关键字大写，表名、列名小写

2.每条命令最好用分号结尾

3.每条命令根据需要，可以进行缩进 或换行

4.注释

单行注释：#注释文字

单行注释：-- 注释文字

多行注释：/* 注释文字  */

# SQL的语言分类
1. DQL（Data Query Language）：数据查询语言

```mysql
select 
```

2. DML(Data Manipulate Language) : 数据操作语言

```mysql
insert 、update、delete
```

3. DDL（Data Define Languge）：数据定义语言

```mysql
create、drop、alter
```

4. TCL（Transaction Control Language）：事务控制语言

```mysql
commit、rollback
```

![image-20210421091818190](https://i.loli.net/2021/04/21/WlkObX2yYJ9H1EU.png)

DCL是数据控制语言（Data Control Language）的简称，它包含诸如`GRANT`之类的命令，并且主要涉及数据库系统的权限，权限和其他控件。

- `GRANT` ：允许用户访问数据库的权限
- `REVOKE`：撤消用户使用GRANT命令赋予的访问权限

# SQL 常用语句

1. 查看当前所有的数据库

```mysql
SHOW DATABASES;
```


2. 打开指定的库

```mysql
USE test;
```


3. 查看当前库的所有表

```mysql
SHOW TABLES;
```


4. 查看其它库的所有表

```mysql
SHOW TABLES FROM girls;
```


6. 查看表结构

```mysql
DESC emp;
```



# SQL 语句

## 1. DQL 语言

### 1.1 基础查询

语法：

```mysql
SELECT 【要查询的内容】
	【FROM 表名】;
```

特点：

- 通过select查询完的结果 ，是一个虚拟的表格，不是真实存在
- 【要查询的内容】可以是**常量值、表达式、字段、函数**  

示例：

```mysql
USE myemployees;
# 查询表中单个字段
	SELECT last_name FROM employees;  
# 查询表中多个字段
	SELECT last_name,salary,email FROM employees;  
# 查询表中所有字段	
	SELECT * FROM employees;  
# 查询常量值
	SELECT 100;
	SELECT 'john';
# 查询表达式
	SELECT 100%98;
# 查询函数
	SELECT version();
# 起别名
	SELECT last_name As 姓 FROM employees;
	SELECT last_name 姓,first_name 名 FROM employees;
	SELECT salary AS "out put" FROM employees;
# 去重
## 案例：查询员工表中涉及到的所有的部门编号
	SELECT department_id FROM employees;
	SELECT DISTINCT department_id FROM employees;
# +号的作用
	SELECT CONCAT('a','b','c') AS 结果;

	SELECT 
		CONCAT(last_name," ",first_name) AS 姓名
	FROM
		employees;
```

### 1.2 条件查询

条件查询：根据条件过滤原始表的数据，查询到想要的数据

语法：

```mysql
select 
		要查询的字段|表达式|常量值|函数
from 
		表
where 
		条件 ;
```

条件分类：

1. 条件表达式

```mysql
示例：salary>10000
	条件运算符：
	> < >= <= = != <>
	
SELECT last_name,salary FROM employees WHERE salary > 10000;
```

2. 逻辑表达式

```mysql
示例：salary>10000 && salary<20000

逻辑运算符：
and（&&）:两个条件如果同时成立，结果为true，否则为false
or(||)：两个条件只要有一个成立，结果为true，否则为false
not(!)：如果条件成立，则not后为false，否则为true

SELECT last_name,salary FROM employees WHERE salary > 10000 && salary < 20000;
```

3. 模糊查询

```mysql
示例：last_name like 'a%'

SELECT last_name FROM employees WHERE last_name LIKE 'a%';
SHOW VARIABLES LIKE '%char%';

# 查询员工表的job_id中包含 a和e的，并且a在e的前面
SELECT job_id
FROM employees
WHERE job_id LIKE '%a%e%';
```

### 1.3 排序查询

语法：

```mysql
select
	要查询的东西
from
	表
where 
	条件

order by 排序的字段|表达式|函数|别名 【asc|desc】
```

示例：

```mysql
# 从 employees 表中查找工资大于10000的人员姓名并按照工资排序（DESC表示降序，ASC表示升序，默认升序）
SELECT CONCAT(last_name, " ",first_name) AS 姓名, salary AS 工资
FROM employees
WHERE salary > 10000
ORDER BY salary DESC;

# 查询员工信息，要求先按工资降序，再按employee_id升序
SELECT *
FROM employees
ORDER BY salary DESC,employee_id ASC;

# 查询员工表的employee_id job_id last_name, 按department_id降序，salary升序
SELECT employee_id, job_id, last_name
FROM employees
ORDER BY department_id DESC, salary ASC;
```

### 1.4 常见函数

#### 1.4.1 单行函数

1. 字符函数

```mysql
concat 拼接
substr 截取子串
upper 转换成大写
lower 转换成小写
trim 去前后指定的空格和字符
ltrim 去左边空格
rtrim 去右边空格
replace 替换
lpad 左填充
rpad 右填充
instr 返回子串第一次出现的索引
length 获取字节个数
```

示例：

```mysql
# length 获取参数值的字节个数
SELECT LENGTH('john');

# 将姓变大写，名变小写，然后拼接
SELECT CONCAT(UPPER(last_name),LOWER(first_name))  姓名 FROM employees;

# 索引从1开始
# 截取从指定索引处后面所有字符
SELECT SUBSTR('Hello World!', 7) output;
# 截取从指定索引处指定字符长度的字符
SELECT SUBSTR('Hello World!', 7, 5) output;

# 返回子串第一次出现的索引，如果找不到返回0
SELECT INSTR('Hello World', 'World') output;

# 去前后指定的空格和字符
SELECT LENGTH(TRIM('   Hello World     ')) output;
SELECT TRIM('aa' FROM 'aaaaHelloaaWorldaaaa') output;

# lpad 用指定的字符实现左填充到指定长度
SELECT LPAD('Hello World',10,'*') AS out_put;

# rpad 用指定的字符实现右填充到指定长度
SELECT RPAD('Hello World',12,'ab') AS out_put;

# replace 将Hello替换为World
SELECT REPLACE('HelloHelloWorldHello','Hello','World') AS out_put;
```



2. 数学函数

```mysql
round 四舍五入
rand 随机数
floor 向下取整
ceil 向上取整
mod 取余
truncate 截断
```

示例：

```mysql
SELECT MOD(10,-3);
SELECT 10%3;
SELECT ROUND(1.2);
SELECT ROUND(1.567,2);        #小数点后保留两位
SELECT CEIL(1.2);
SELECT FLOOR(1.2);
SELECT RAND();
SELECT TRUNCATE(1.69999,1);  # 小数点后保留一位
```

3. 日期函数

```mysql
now 当前系统日期+时间
curdate 当前系统日期
curtime 当前系统时间
str_to_date 将字符转换成日期
date_format 将日期转换成字符
```

示例：

```mysql
# now 返回当前系统日期+时间
SELECT NOW();

# curdate 返回当前系统日期，不包含时间
SELECT CURDATE();

# curtime 返回当前时间，不包含日期
SELECT CURTIME();

# 可以获取指定的部分，年、月、日、小时、分钟、秒  
SELECT YEAR(NOW()) 年;
SELECT YEAR('1998-1-1') 年;
SELECT  YEAR(hiredate) 年 FROM employees;
SELECT MONTH(NOW()) 月;
SELECT MONTHNAME(NOW()) 月;

# str_to_date 将字符通过指定的格式转换成日期
SELECT STR_TO_DATE('1998-3-2','%Y-%c-%d') AS out_put;

# 查询入职日期为1992--4-3的员工信息
SELECT * FROM employees WHERE hiredate = '1992-4-3';
SELECT * FROM employees WHERE hiredate = STR_TO_DATE('4-3 1992','%c-%d %Y');

# date_format 将日期转换成字符
SELECT DATE_FORMAT(NOW(),'%y年%m月%d日') AS out_put;
# 查询有奖金的员工名和入职日期(xx月/xx日 xx年)
SELECT last_name,DATE_FORMAT(hiredate,'%m月/%d日 %y年') 入职日期
FROM employees
WHERE commission_pct IS NOT NULL;
```



4. 流程控制函数

```mysql
if 处理双分支
case语句 处理多分支
	情况1：处理等值判断
	情况2：处理条件判断

# switch case 
case 要判断的字段或表达式
when 常量1 then 要显示的值1或语句1;
when 常量2 then 要显示的值2或语句2;
...
else 要显示的值n或语句n;
end

# 多重 if
case 
when 条件1 then 要显示的值1或语句1
when 条件2 then 要显示的值2或语句2
...
else 要显示的值n或语句n
end
```
示例：

```mysql
SELECT IF(10<5,'大','小');
SELECT last_name,commission_pct,IF(commission_pct IS NULL,'没奖金','有奖金') 备注
FROM employees;

/*案例：查询员工的工资，要求
部门号=30，显示的工资为1.1倍
部门号=40，显示的工资为1.2倍
部门号=50，显示的工资为1.3倍
其他部门，显示的工资为原工资
*/
SELECT salary 原始工资,department_id,
CASE department_id
WHEN 30 THEN salary*1.1
WHEN 40 THEN salary*1.2
WHEN 50 THEN salary*1.3
ELSE salary
END AS 新工资
FROM employees;

/* 案例：查询员工的工资的情况
如果工资>20000,显示A级别
如果工资>15000,显示B级别
如果工资>10000，显示C级别
否则，显示D级别 
*/
SELECT salary,
CASE 
WHEN salary>20000 THEN 'A'
WHEN salary>15000 THEN 'B'
WHEN salary>10000 THEN 'C'
ELSE 'D'
END AS 工资级别
FROM employees;
```



5. 其他

```mysql
version 版本
database 当前库
user 当前连接用户
```
示例：

```mysql
SELECT VERSION();
SELECT DATABASE();
SELECT USER();
```

#### 1.4.2 分组函数

```mysql
sum 求和
max 最大值
min 最小值
avg 平均值
count 计数
```
特点：

1、以上五个分组函数都忽略`null`值，除了`count(*)`

2、`sum`和`avg`一般用于处理数值型，`max、min、count`可以处理任何数据类型

3、都可以搭配`distinct`使用，用于统计去重后的结果

4、`count`的参数可以支持：字段、\*、常量值，一般放1，建议使用 `count(*)`

效率：
MYISAM存储引擎下  ，`COUNT(*)`的效率高

INNODB存储引擎下，`COUNT(*)`和`COUNT(1)`的效率差不多，比`COUNT(字段)`要高一些

示例：

```mysql
SELECT COUNT(*) FROM employees;    #统计行数
SELECT COUNT(1) FROM employees;

# 和分组函数一同查询的字段有限制
SELECT AVG(salary),employee_id  FROM employees;
```



### 1.5 分组查询

语法：

```sql
select 查询的字段，分组函数
from 表
group by 分组的字段
```
特点：

1、可以按单个字段分组

2、和分组函数一同查询的字段最好是分组后的字段

3、分组筛选

|            |    针对的表    |       位置       |  关键字  |
| :--------: | :------------: | :--------------: | :------: |
| 分组前筛选 |     原始表     | `group by`的前面 | `where`  |
| 分组后筛选 | 分组后的结果集 | `group by`的后面 | `having` |

4、可以按多个字段分组，字段之间用逗号隔开

5、可以支持排序

6、`having`后可以支持别名

示例：

```mysql
# 查询各个管理者手下员工的最低工资，没有管理者的员工不计算在内
SELECT MIN(salary),manager_id
FROM employees
WHERE manager_id IS NOT NULL
GROUP BY manager_id

# 查询各个管理者手下员工的最低工资，其中最低工资不能低于6000，没有管理者的员工不计算在内
SELECT MIN(salary),manager_id
FROM employees
WHERE manager_id IS NOT NULL
GROUP BY manager_id
HAVING MIN(salary)>=6000;

# 查询所有部门的编号，员工数量和工资平均值,并按平均工资降序
SELECT department_id,COUNT(*),AVG(salary) a
FROM employees
GROUP BY department_id
ORDER BY a DESC;
```

**LeetCode 184. 部门工资最高的员工**

`Employee` 表包含所有员工信息，每个员工有其对应的 `Id`, `salary` 和 `department Id`。

> +----+-------+--------+--------------+
> | Id | Name  | Salary | DepartmentId |
> +----+-------+--------+--------------+
> | 1  | Joe   | 70000  | 1            |
> | 2  | Jim   | 90000  | 1            |
> | 3  | Henry | 80000  | 2            |
> | 4  | Sam   | 60000  | 2            |
> | 5  | Max   | 90000  | 1            |
> +----+-------+--------+--------------+

`Department` 表包含公司所有部门的信息。

> +----+----------+
> | Id | Name     |
> +----+----------+
> | 1  | IT       |
> | 2  | Sales    |
> +----+----------+

编写一个 SQL 查询，找出每个部门工资最高的员工。

> +------------+----------+--------+
> | Department | Employee | Salary |
> +------------+----------+--------+
> | IT         | Max      | 90000  |
> | IT         | Jim      | 90000  |
> | Sales      | Henry    | 80000  |
> +------------+----------+--------+

解释：

Max 和 Jim 在 IT 部门的工资都是最高的，Henry 在销售部的工资最高。

```mysql
SELECT
    Department.name AS 'Department',
    Employee.name AS 'Employee',
    Salary
FROM
    Employee
        JOIN
    Department ON Employee.DepartmentId = Department.Id
WHERE
    (Employee.DepartmentId , Salary) IN
    (   SELECT
            DepartmentId, MAX(Salary)
        FROM
            Employee
        GROUP BY DepartmentId
	)
;
```

**LeetCode 185. 部门工资前三高的所有员工**

```mysql
SELECT
	Department.NAME AS Department,
	e1.NAME AS Employee,
	e1.Salary AS Salary 
FROM
	Employee AS e1,Department 
WHERE
	e1.DepartmentId = Department.Id 
	AND 3 > (SELECT  count( DISTINCT e2.Salary ) 
			 FROM	Employee AS e2 
			 WHERE	e1.Salary < e2.Salary 	AND e1.DepartmentId = e2.DepartmentId 	) 
ORDER BY Department.NAME,Salary DESC;
```



### 1.6 多表连接查询

含义：又称多表查询，当查询的字段来自于多个表时，就会用到连接查询

笛卡尔乘积现象：表1有m行，表2有n行，结果=m*n行

发生原因：没有有效的连接条件

如何避免：添加有效的连接条件

<table>
    <tr>
        <td rowspan="2">按年代分类</td>
        <td colspan="2">sql92标准:仅仅支持内连接</td>
    </tr>
    <tr>
        <td colspan="2">sql99标准【推荐】：支持内连接+外连接（左外和右外）+交叉连接</td>
    </tr>
    <tr>
        <td rowspan="7">按功能分类</td>
        <td rowspan="3">内连接</td>
        <td>等值连接</td>
    </tr>
    <tr>
        <td>非等值连接</td>
    </tr>
    <tr>
        <td>自连接</td>
    </tr>
    <tr>
        <td rowspan="3">外连接</td>
        <td>左外连接</td>
    </tr>
    <tr>
        <td>右外连接</td>
    </tr>
    <tr>
        <td>全外连接</td>
    </tr>
    <tr>
        <td colspan="2">交叉连接</td>
    </tr>
</table>



#### 1.6.1 传统模式下的连接：等值连接/非等值连接

1. 等值连接的结果 = 多个表的交集
2. n表连接，至少需要n-1个连接条件
3. 多个表不分主次，没有顺序要求
4. 一般为表起别名，提高阅读性和性能

示例：

```mysql
# 等值连接

# 查询员工名和对应的部门名 
SELECT last_name,department_name
FROM employees,departments
WHERE employees.`department_id`=departments.`department_id`;

# 为表起别名
## 提高语句的简洁度 
## 区分多个重名的字段
## 注意：如果为表起了别名，则查询的字段就不能使用原来的表名去限定
# 可以加筛选
# 查询有奖金的员工名、部门名
SELECT last_name,department_name,commission_pct
FROM employees e,departments d
WHERE e.`department_id`=d.`department_id`
AND e.`commission_pct` IS NOT NULL;

# 可以加分组
# 查询每个城市的部门个数
SELECT COUNT(*) 个数,city
FROM departments d,locations l
WHERE d.`location_id`=l.`location_id`
GROUP BY city;

# 可以加排序
# 查询每个工种的工种名和员工的个数，并且按员工个数降序
SELECT job_title,COUNT(*)
FROM employees e,jobs j
WHERE e.`job_id`=j.`job_id`
GROUP BY job_title
ORDER BY COUNT(*) DESC;

# 三表连接
# 查询员工名、部门名和所在的城市
SELECT last_name,department_name,city
FROM employees e,departments d,locations l
WHERE e.`department_id`=d.`department_id`
AND d.`location_id`=l.`location_id`;

# 非等值连接

# 查询员工工资级别为A的员工
SELECT salary,grade_level
FROM employees e,job_grades g
WHERE salary BETWEEN g.`lowest_sal` AND g.`highest_sal`
AND g.`grade_level`='A';

# 自连接

# 查询 员工名和上级的名称
SELECT e.employee_id,e.last_name,m.employee_id,m.last_name
FROM employees e,employees m
WHERE e.`manager_id`=m.`employee_id`;
```

#### 1.6.2 sql99语法：通过join关键字实现连接

含义：1999年推出的sql语法
支持：

- 等值连接、非等值连接 （内连接）
- 外连接
- 交叉连接

分类：

- 内连接（★）：inner
- 外连接
  - ​	左外(★) : left 【outer】 
  - ​	右外(★)：right 【outer】
  - ​	全外：full【outer】
- 交叉连接：cross 

语法：

```mysql
select 查询列表
from 表1
【inner|left outer|right outer|cross】join 表2 on  连接条件
【inner|left outer|right outer|cross】join 表3 on  连接条件
【where 筛选条件】
【group by 分组字段】
【having 分组后的筛选条件】
【order by 排序的字段或表达式】
```

优点：语句上，连接条件和筛选条件实现了分离，简洁明了！

##### 1.6.2.1 内连接

语法：

```mysql
select 查询列表
from 表1 别名
inner join 表2 别名 on 连接条件;
```

分类：

- 等值连接
- 非等值连接
- 自连接

特点：

- 添加排序、分组、筛选
- inner可以省略
- 筛选条件放在where后面，连接条件放在on后面，提高分离性，便于阅读
- **inner join连接和sql92语法中的等值连接效果是一样的，都是查询多表的交集**

示例：

等值连接：

```mysql
# 查询员工名、部门名
SELECT last_name, department_name
FROM employees e
INNER JOIN departments d ON e.department_id=d.department_id;

# 查询名字中包含e的员工名和工种名（添加筛选）
SELECT last_name,job_title
FROM employees e
INNER JOIN jobs j ON e.`job_id`=  j.`job_id`
WHERE e.`last_name` LIKE '%e%';

# 查询部门个数>3的城市名和部门个数（添加分组+筛选）
# ①查询每个城市的部门个数
# ②在①结果上筛选满足条件的
SELECT city,COUNT(*) 部门个数
FROM departments d
INNER JOIN locations l ON d.`location_id`=l.`location_id`
GROUP BY city
HAVING COUNT(*)>3;

# 查询部门的员工个数>3的部门名和员工个数，并按个数降序（添加排序）
#①查询每个部门的员工个数
SELECT COUNT(*),department_name
FROM employees e
INNER JOIN departments d ON e.`department_id`=d.`department_id`
GROUP BY department_name
#② 在①结果上筛选员工个数>3的记录，并排序
SELECT COUNT(*) 个数,department_name
FROM employees e
INNER JOIN departments d ON e.`department_id`=d.`department_id`
GROUP BY department_name
HAVING COUNT(*)>3
ORDER BY COUNT(*) DESC;

# 查询员工名、部门名、工种名，并按部门名降序（添加三表连接）
SELECT last_name,department_name,job_title
FROM employees e
INNER JOIN departments d ON e.`department_id`=d.`department_id`
INNER JOIN jobs j ON e.`job_id` = j.`job_id`
ORDER BY department_name DESC;
```

非等值连接：

```mysql
# 查询员工的工资级别
SELECT salary,grade_level
FROM employees e
JOIN job_grades g ON e.`salary` BETWEEN g.`lowest_sal` AND g.`highest_sal`;

# 查询工资级别的个数>20的个数，并且按工资级别降序
SELECT COUNT(*),grade_level
FROM employees e
JOIN job_grades g ON e.`salary` BETWEEN g.`lowest_sal` AND g.`highest_sal`
GROUP BY grade_level
HAVING COUNT(*)>20
ORDER BY grade_level DESC;
```

自连接：

```mysql
# 查询姓名中包含字符k的员工的名字、上级的名字
SELECT e.last_name,m.last_name
FROM employees e
JOIN employees m
ON e.`manager_id`= m.`employee_id`
WHERE e.`last_name` LIKE '%k%';
```

##### 1.6.2.2  外连接

应用场景：用于查询一个表中有，另一个表没有的记录

 特点：

- 外连接的查询结果为**主表中的所有记录**

  - 如果从表中有和它匹配的，则显示匹配的值

  	- 如果从表中没有和它匹配的，则显示null
  	- 外连接查询结果 = 内连接结果 + 主表中有而从表没有的记录

- 左外连接，left join左边的是主表

- 右外连接，right join右边的是主表

- 左外和右外交换两个表的顺序，可以实现同样的效果 

- 全外连接=内连接的结果+表1中有但表2没有的+表2中有但表1没有的

![John连接总结](https://i.loli.net/2021/04/16/jIvTyVcaz2A5fLO.png)

![John连接总结1](https://i.loli.net/2021/04/16/mhw3SMpQTWH7z92.png)

示例：

```mysql
# 查询哪个部门没有员工
#左外
SELECT d.*,e.employee_id
FROM departments d                      # 主表
LEFT OUTER JOIN employees e
ON d.`department_id` = e.`department_id`
WHERE e.`employee_id` IS NULL;
# 右外 
SELECT d.*,e.employee_id
FROM employees e                        # 主表
RIGHT OUTER JOIN departments d
ON d.`department_id` = e.`department_id`
WHERE e.`employee_id` IS NULL;

# 查询哪个城市没有部门
# 右连
SELECT city,d.*
FROM departments d
RIGHT OUTER JOIN locations l 
ON d.`location_id`=l.`location_id`
WHERE  d.`department_id` IS NULL;

# 查询部门名为SAL或IT的员工信息
SELECT e.*,d.department_name,d.`department_id`
FROM departments  d
LEFT JOIN employees e
ON d.`department_id` = e.`department_id`
WHERE d.`department_name` IN('SAL','IT');
```

##### 补充：SQL语句执行顺序

SQL语句执行顺序：

```mysql
# 手写
SELECT DISTINCT
   <select_list>
FROM
   <left_table> <join_type>
JOIN <right_table> ON <join_condition>
WHERE
   <where_condition>
GROUP BY
   <group_by_list>
HAVING
   <having_condition>
ORDER BY
   <order_by_condition>
LIMIT <limit_number>

# 机读
1 FROM < left_table >
2 ON <join_condition>
3 <join_type> JOIN <right_table>
4 WHERE <where_condition>
5 GROUP BY <group_by_list>
6 HAVING <having_condition>
7 SELECT
8 DISTINCT <select_list>
9 ORDER BY <order_by_condition>
10 LIMIT <limit_number>
```

![img](https://i.loli.net/2021/04/16/kVKS9Pzw7AThtBF.jpg)

### 1.7 子查询

含义：

- **一条查询语句中又嵌套了另一条完整的select语句**，其中被嵌套的select语句，称为**子查询或内查询**。在外面的查询语句，称为主查询或外查询

特点：

- 子查询都放在**小括号**内

- 子查询可以放在from后面、select后面、where后面、having后面，但一般放在条件的右侧

- **子查询优先于主查询执行，主查询使用了子查询的执行结果**

- 子查询根据查询结果的行数不同分为以下两类：
  - 单行子查询
    - 结果集只有一行
    - 一般搭配单行操作符使用：> < = <> >= <= 
    - 非法使用子查询的情况：
      - 子查询的结果为一组值
      - 子查询的结果为空	
  - 多行子查询
    - 结果集有多行
    - 一般搭配多行操作符使用：any、all、in、not in
    - in： 属于子查询结果中的任意一个就行
    - any和all往往可以用其他查询代替

![多行子查询](https://i.loli.net/2021/04/16/eTXvWQGESCZj6uo.png)

分类：

<table>
    <tr>
        <td rowspan="6">按子查询出现的位置</td>
        <td>select后面</td>
        <td>仅仅支持标量子查询</td>
    </tr>
    <tr>
        <td>from后面</td>
        <td>支持表子查询</td>
    </tr>
    <tr>
        <td rowspan="3">where或having后面☆</td>
        <td>标量子查询（单行）√</td>
    </tr>
    <tr>
        <td>列子查询（多行）√</td>
    </tr>
    <tr>
        <td>行子查询</td>
    </tr>
    <tr>
        <td>exists后面（相关子查询）</td>
        <td>表子查询</td>
    </tr>
    <tr>
        <td rowspan="4">按结果集的行列数不同</td>
        <td>标量子查询</td>
        <td>结果集只有一行一列</td>
    </tr>
    <tr>
        <td>列子查询</td>
        <td>结果集只有一列多行</td>
    </tr>
    <tr>
        <td>行子查询</td>
        <td>结果集有一行多列</td>
    </tr>
    <tr>
        <td>表子查询</td>
        <td>结果集一般为多行多列</td>
    </tr>
</table>

#### 1.7.1 where或having后面

1、标量子查询（单行子查询）
2、列子查询（多行子查询）

3、行子查询（多列多行）

特点：
①子查询放在小括号内
②子查询一般放在条件的右侧
③标量子查询，一般搭配着单行操作符使用：< >= <= = <>

列子查询，一般搭配着多行操作符使用：in、any/some、all

④子查询的执行优先于主查询执行，主查询的条件用到了子查询的结果

示例：

##### 1.7.1.1 标量子查询(单行子查询) ☆

```mysql
# 案例1：谁的工资比 Abel 高?
#①查询Abel的工资
SELECT salary
FROM employees
WHERE last_name = 'Abel';
#②查询员工的信息，满足 salary>①结果
SELECT *
FROM employees
WHERE salary>(
	SELECT salary
	FROM employees
	WHERE last_name = 'Abel'
);

#案例2：返回job_id与141号员工相同，salary比143号员工多的员工 姓名，job_id 和工资
#①查询141号员工的job_id
SELECT job_id
FROM employees
WHERE employee_id = 141;
#②查询143号员工的salary
SELECT salary
FROM employees
WHERE employee_id = 143;
#③查询员工的姓名，job_id 和工资，要求job_id=①并且salary>②
SELECT last_name,job_id,salary
FROM employees
WHERE job_id = (
	SELECT job_id
	FROM employees
	WHERE employee_id = 141
) AND salary>(
	SELECT salary
	FROM employees
	WHERE employee_id = 143
);

#案例3：返回公司工资最少的员工的last_name,job_id和salary
#①查询公司的 最低工资
SELECT MIN(salary)
FROM employees
#②查询last_name,job_id和salary，要求salary=①
SELECT last_name,job_id,salary
FROM employees
WHERE salary=(
	SELECT MIN(salary)
	FROM employees
);

#案例4：查询最低工资大于50号部门最低工资的部门id和其最低工资
#①查询50号部门的最低工资
SELECT  MIN(salary)
FROM employees
WHERE department_id = 50
#②查询每个部门的最低工资
SELECT MIN(salary),department_id
FROM employees
GROUP BY department_id
#③ 在②基础上筛选，满足min(salary)>①
SELECT MIN(salary),department_id
FROM employees
GROUP BY department_id
HAVING MIN(salary)>(
	SELECT  MIN(salary)
	FROM employees
	WHERE department_id = 50
);
```

##### 1.7.1.2 列子查询（多行子查询）★

```mysql
#案例1：返回location_id是1400或1700的部门中的所有员工姓名
# ①查询location_id是1400或1700的部门编号
SELECT DISTINCT department_id
FROM departments
WHERE location_id IN(1400,1700)
# ②查询员工姓名，要求部门号是①列表中的某一个
SELECT last_name
FROM employees
WHERE department_id  IN(
	SELECT DISTINCT department_id
	FROM departments
	WHERE location_id IN(1400,1700)
);
# 或
SELECT last_name
FROM employees
WHERE department_id =ALL(
	SELECT DISTINCT department_id
	FROM departments
	WHERE location_id IN(1400,1700)
);

#案例2：返回其它工种中比job_id为‘IT_PROG’工种任一工资低的员工的员工号、姓名、job_id 以及salary
#①查询job_id为‘IT_PROG’部门任一工资
SELECT DISTINCT salary
FROM employees
WHERE job_id = 'IT_PROG'
#②查询员工号、姓名、job_id 以及salary，salary<(①)的任意一个
SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary<ANY(
	SELECT DISTINCT salary
	FROM employees
	WHERE job_id = 'IT_PROG'

) AND job_id<>'IT_PROG';
#或
SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary<(
	SELECT MAX(salary)                # 小于任意一个用max，小于所有用min
	                                  # 任意是 all， 任一是 any
	FROM employees
	WHERE job_id = 'IT_PROG'

) AND job_id<>'IT_PROG';

#案例3：返回其它部门中比job_id为‘IT_PROG’部门所有工资都低的员工   的员工号、姓名、job_id 以及salary
SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary<ALL(
	SELECT DISTINCT salary
	FROM employees
	WHERE job_id = 'IT_PROG'

) AND job_id<>'IT_PROG';
#或
SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary<(
	SELECT MIN( salary)
	FROM employees
	WHERE job_id = 'IT_PROG'

) AND job_id<>'IT_PROG';
```

##### 1.7.1.3 行子查询（一行多列或多行多列）

```mysql
#案例：查询员工编号最小并且工资最高的员工信息
#①查询最小的员工编号
SELECT MIN(employee_id)
FROM employees
#②查询最高工资
SELECT MAX(salary)
FROM employees
#③查询员工信息
SELECT *
FROM employees
WHERE employee_id=(
	SELECT MIN(employee_id)
	FROM employees
)AND salary=(
	SELECT MAX(salary)
	FROM employees
);
# 或
SELECT * 
FROM employees
WHERE (employee_id,salary)=(
	SELECT MIN(employee_id),MAX(salary)
	FROM employees
);
```

#### 1.7.2 select 后面

仅仅支持标量子查询(单行子查询)

示例：

```mysql
#案例：查询每个部门的员工个数
SELECT d.*,(
	SELECT COUNT(*)
	FROM employees e
	WHERE e.department_id = d.`department_id`
 ) 个数
 FROM departments d;
 
 #案例2：查询员工号=102的部门名
SELECT (
	SELECT department_name,e.department_id
	FROM departments d
	INNER JOIN employees e
	ON d.department_id=e.department_id
	WHERE e.employee_id=102
) 部门名;
```

#### 1.7.3 from后面

将子查询结果充当一张表，要求必须起别名

```mysql
#案例：查询每个部门的平均工资的工资等级
#①查询每个部门的平均工资
SELECT AVG(salary),department_id
FROM employees
GROUP BY department_id
#②连接①的结果集和job_grades表，筛选条件平均工资 between lowest_sal and highest_sal
SELECT  ag_dep.*,g.`grade_level`
FROM (
	SELECT AVG(salary) ag,department_id
	FROM employees
	GROUP BY department_id
) ag_dep
INNER JOIN job_grades g
ON ag_dep.ag BETWEEN lowest_sal AND highest_sal;
```

#### 1.7.4 exists后面（相关子查询）

语法：

```mysql
exists(完整的查询语句)
```

结果：

```mysql
1或0
```

示例：

```mysql
#案例1：查询有员工的部门名
#in
SELECT department_name
FROM departments d
WHERE d.`department_id` IN(
	SELECT department_id
	FROM employees
)
#exists
SELECT department_name
FROM departments d
WHERE EXISTS(
	SELECT *
	FROM employees e
	WHERE d.`department_id`=e.`department_id`
);
```



#### 1.7.5 练习

```mysql
# 查询和Zlotkey相同部门的员工姓名和工资
#①查询Zlotkey的部门
SELECT department_id
FROM employees
WHERE last_name = 'Zlotkey'
#②查询部门号=①的姓名和工资
SELECT last_name,salary
FROM employees
WHERE department_id = (
	SELECT department_id
	FROM employees
	WHERE last_name = 'Zlotkey'
);

# 查询工资比公司平均工资高的员工的员工号，姓名和工资。
#①查询平均工资
SELECT AVG(salary)
FROM employees
#②查询工资>①的员工号，姓名和工资。
SELECT last_name,employee_id,salary
FROM employees
WHERE salary>(
	SELECT AVG(salary)
	FROM employees
);

# 查询各部门中工资比本部门平均工资高的员工的员工号, 姓名和工资
#①查询各部门的平均工资
SELECT AVG(salary),department_id
FROM employees
GROUP BY department_id
#②连接①结果集和employees表，进行筛选
SELECT employee_id,last_name,salary,e.department_id
FROM employees e
INNER JOIN (
	SELECT AVG(salary) ag,department_id
	FROM employees
	GROUP BY department_id 
) ag_dep
ON e.department_id = ag_dep.department_id
WHERE salary>ag_dep.ag ;

# 查询和姓名中包含字母u的员工在相同部门的员工的员工号和姓名
#①查询姓名中包含字母u的员工的部门
SELECT  DISTINCT department_id
FROM employees
WHERE last_name LIKE '%u%'
#②查询部门号=①中的任意一个的员工号和姓名
SELECT last_name,employee_id
FROM employees
WHERE department_id IN (
	SELECT  DISTINCT department_id
	FROM employees
	WHERE last_name LIKE '%u%'
);

# 查询在部门的location_id为1700的部门工作的员工的员工号
#①查询location_id为1700的部门
SELECT DISTINCT department_id
FROM departments 
WHERE location_id  = 1700
#②查询部门号=①中的任意一个的员工号
SELECT employee_id
FROM employees
WHERE department_id = ANY(
	SELECT DISTINCT department_id
	FROM departments 
	WHERE location_id  = 1700

);

# 查询管理者是King的员工姓名和工资
#①查询姓名为king的员工编号
SELECT employee_id
FROM employees
WHERE last_name  = 'K_ing'
#②查询哪个员工的manager_id = ①
SELECT last_name,salary
FROM employees
WHERE manager_id IN(
	SELECT employee_id
	FROM employees
	WHERE last_name  = 'K_ing'
);

# 查询工资最高的员工的姓名，要求first_name和last_name显示为一列，列名为 姓.名
#①查询最高工资
SELECT MAX(salary)
FROM employees
#②查询工资=①的姓.名
SELECT CONCAT(first_name,last_name) "姓.名"
FROM employees
WHERE salary=(
	SELECT MAX(salary)
	FROM employees
);
```

### 1.8 分页查询

应用场景：实际的web项目中需要根据用户的需求提交对应的分页查询的sql语句（当要显示的数据，一页显示不全，需要分页提交sql请求）

语法：

```mysql
select 查询列表
from 表
【join type】 join 表2
on 连接条件
where 筛选条件
group by 分组字段
having 分组后的筛选
order by 排序的字段】
limit 【offset,】size;

# offset要显示条目的起始索引（起始索引从0开始）
# size 要显示的条目个数
```

`LIMIT`里面不能做运算：

- `limit 2,1`：跳过2条取出1条数据，即读取第3条数据

- `limit 2 offset 1`：跳过2条取出1条数据，即读取第2,3条

示例：

```mysql
#案例1：查询前五条员工信息
SELECT * FROM  employees LIMIT 0,5;
SELECT * FROM  employees LIMIT 5;
#案例2：查询第11条——第25条
SELECT * FROM  employees LIMIT 10,15;
#案例3：有奖金的员工信息，并且工资较高的前10名显示出来
SELECT 
    * 
FROM
    employees 
WHERE commission_pct IS NOT NULL 
ORDER BY salary DESC 
LIMIT 10;
```

```mysql
# 检索记录行 6-15
SELECT * FROM table LIMIT 5,10;  

# 检索记录行 96-last.
SELECT * FROM table LIMIT 95,-1; 

# 检索前 5 个记录行 
# LIMIT n 等价于 LIMIT 0,n
SELECT * FROM table LIMIT 5;     
```

子查询的分页方式：

```mysql
# 随着数据量的增加，页数会越来越多，越往后分页，LIMIT语句的偏移量就会越大，速度也会明显变慢
select * from articles where category_id=123 order by id limit 10000, 10;

# 通过子查询的方式来提高分页效率
select * from articles 
where id >= (
	select id from articles where category_id=123 order by id limit 10000,1
) 
limit 10;
```





### 1.9 联合查询

引入：union

定义：将多条查询语句的结果合并成一个结果

应用场景：要查询的结果来自于多个表，且多个表没有直接的连接关系，但查询的信息一致时（**上下连接，并集**）

语法：

```mysql
select 字段|常量|表达式|函数 【from 表】 【where 条件】 union 【all】
select 字段|常量|表达式|函数 【from 表】 【where 条件】 union 【all】
select 字段|常量|表达式|函数 【from 表】 【where 条件】 union 【all】
.....
select 字段|常量|表达式|函数 【from 表】 【where 条件】
```

特点：

1、多条查询语句的查询的**列数**必须是一致的
2、要求多条查询语句的查询的每一列的类型和顺序最好一致
3、union代表去重，union all代表不去重

示例：

```mysql
# 查询部门编号>90 或 邮箱包含a的员工信息
SELECT * FROM employees WHERE email LIKE '%a%' OR department_id>90;;

SELECT * FROM employees  WHERE email LIKE '%a%'
UNION
SELECT * FROM employees  WHERE department_id>90;
```



## 2. DML 语言

### 2.1 插入语句

#### 2.1.1 方式一

语法：

```mysql
insert into 表名(列名,...) values(值1,...);
```

特点：

1、字段类型和值类型一致或兼容，而且一一对应
2、可以为空的字段，可以不用插入值，或用null填充
3、不可以为空的字段，必须插入值
4、字段个数和值的个数必须一致
5、字段可以省略，但默认所有字段，并且顺序和表中的存储顺序一致

#### 2.1.2 方式二

语法：

```mysql
insert into 表名
set 列名=值,列名=值,...
```

#### 2.1.3 比较

1、方式一支持插入多行，方式二不支持

```mysql
INSERT INTO demo
VALUES(23,'uestc','女','1990-4-23','1898888888',NULL,2)
,(24,'nuc','女','1990-4-23','1898888888',NULL,2);

INSERT INTO demo
SELECT 23,'uestc','女','1990-4-23','1898888888',NULL,2 UNION
SELECT 24,'nuc','女','1990-4-23','1898888888',NULL,2;
```

2、方式一支持子查询，方式二不支持

```mysql
INSERT INTO demo(NAME,phone)
SELECT boyname,'1234567'
FROM demo1 WHERE id<4;

INSERT INTO demo(id,NAME,phone)
SELECT 26,'nucc','11809866';
```

### 2.2 修改语句

#### 2.2.1 修改单表的记录☆

语法：

```mysql
update 表名
set 列=新值,列=新值,...
where 筛选条件;
```

示例：

```mysql
# 案例1：修改beauty表中姓唐的女生的电话为123456789
UPDATE beauty 
SET phone='123456789' 
WHERE NAME LIKE '唐%';

# 案例2：修改boys表中id好为2的名字为张飞，魅力值 10
UPDATE boys 
SET boyname='张飞' , usercp = 10
WHERE id = 2;
```



#### 2.2.2 修改多表的记录

语法：

```mysql
# sql92语法：
update 表1 别名,表2 别名
set 列=值,...
where 连接条件
and 筛选条件;

# sql99语法：
update 表1 别名
inner|left|right join 表2 别名
on 连接条件
set 列=值,...
where 筛选条件;
```

示例：

```mysql
# 案例：修改张无忌的女朋友的手机号为119
UPDATE boys bo
INNER JOIN beauty b ON bo.`id`=b.`boyfriend_id`
SET b.`phone`='119'
WHERE bo.`boyName`='张无忌';
```



### 2.3 删除语句

#### 2.3.1 方式一：delete

语法：

```mysql
# 1、单表的删除【★】
delete from 表名 where 筛选条件

# 2、多表的删除【补充】
## sql92语法：
delete 表1的别名,表2的别名
from 表1 别名,表2 别名
where 连接条件
and 筛选条件;
## sql99语法：
delete 表1的别名,表2的别名
from 表1 别名
inner|left|right join 表2 别名 on 连接条件
where 筛选条件;
```

示例：

```mysql
#1.单表的删除 
##案例：删除手机号以9结尾的女生信息
DELETE FROM beauty WHERE phone LIKE '%9';
##案例：删除表中所有数据
DELETE FROM 表名;

#2.多表的删除
##案例：删除张无忌的女朋友的信息
DELETE b
FROM beauty b
INNER JOIN boys bo ON b.`boyfriend_id` = bo.`id`
WHERE bo.`boyName`='张无忌';
##案例：删除黄晓明的信息以及他女朋友的信息
DELETE b,bo
FROM beauty b
INNER JOIN boys bo ON b.`boyfriend_id`=bo.`id`
WHERE bo.`boyName`='黄晓明';
```



#### 2.3.2 方式二：truncate

语法：

```mysql
# 删除表中所有数据
truncate table 表名;
```

#### 2.3.3 比较

1.**delete 可以加where 条件，truncate不能加**

2.**truncate删除，效率高一丢丢**
3.假如要删除的表中有自增长列，
	如果用delete删除后，再插入数据，自增长列的值从断点开始，
	而truncate删除后，再插入数据，自增长列的值从1开始。
4.truncate删除没有返回值，delete删除有返回值

5.**truncate删除不能回滚，delete删除可以回滚.**



## 3. DDL语句

### 3.1 库的管理

创建、修改、删除

#### 3.1.1 库的创建

语法：

```mysql
create database  [if not exists]库名;
```

示例：

```mysql
#案例：创建库Books 
CREATE DATABASE IF NOT EXISTS books ;  
```

#### 3.1.2 库的修改

语法：

```mysql
# 更改库名
RENAME DATABASE 库名 TO 新库名;

# 更改库的字符集
ALTER DATABASE 库名 CHARACTER SET gbk;
```

#### 3.1.3 库的删除

语法：

```mysql
DROP DATABASE IF EXISTS books;
```

### 3.2 表的管理

创建、修改、删除

#### 3.2.1 表的创建☆

语法：

```mysql
create table 表名(
	列名 列的类型【(长度) 约束】,
	列名 列的类型【(长度) 约束】,
	列名 列的类型【(长度) 约束】,
	...
	列名 列的类型【(长度) 约束】
);
```

示例：

```mysql
# 案例：创建表Book
CREATE TABLE book(
	id INT,#编号
	bName VARCHAR(20),#图书名
	price DOUBLE,#价格
	authorId  INT,#作者编号
	publishDate DATETIME#出版日期
);
# 查看表结构
DESC book;
```

常见约束：

```mysql
NOT NULL
DEFAULT
UNIQUE
CHECK
PRIMARY KEY
FOREIGN KEY
AUTO_INCREMENT
```



#### 3.2.2 表的修改

语法：

```mysql
alter table 表名 add|drop|modify|change column 列名 【列类型 约束】;
```

示例：

```mysql
# ①修改列名
ALTER TABLE book CHANGE COLUMN publishDate pubDate DATETIME;

# ②修改列的类型或约束
ALTER TABLE book MODIFY COLUMN pubDate TIMESTAMP;
## 案例：向表emp2的id列中添加PRIMARY KEY约束（my_emp_id_pk）
ALTER TABLE emp2 MODIFY COLUMN id INT PRIMARY KEY;
ALTER TABLE emp2 ADD CONSTRAINT my_emp_id_pk PRIMARY KEY(id);
## 案例：向表emp2中添加列dept_id，并在其中定义FOREIGN KEY约束，与之相关联的列是dept2表中的id列。
ALTER TABLE emp2 ADD COLUMN dept_id INT;
ALTER TABLE emp2 ADD CONSTRAINT fk_emp2_dept2 FOREIGN KEY(dept_id) REFERENCES dept2(id);
## 			   位置			支持的约束类型				 是否可以起约束名
## 列级约束：	列的后面		语法都支持，但外键没有效果	 不可以
## 表级约束：	所有列的下面	   默认和非空不支持，其他支持	可以（主键没有效果）

# ③添加新列
ALTER TABLE book ADD COLUMN press VARCHAR(20);

# ④删除列
ALTER TABLE book DROP COLUMN press;

# ⑤修改表名
ALTER TABLE book RENAME TO books;
```

#### 3.2.3 表的删除

语法：

```mysql
DROP TABLE IF EXISTS 表名;
```

通用的写法：

```mysql
DROP DATABASE IF EXISTS 旧库名;
CREATE DATABASE 新库名;

DROP TABLE IF EXISTS 旧表名;
CREATE TABLE  表名();
```

### 3.3 表的复制

```mysql
#1.仅仅复制表的结构
CREATE TABLE copy LIKE book;

#2.复制表的结构+数据
CREATE TABLE copy2 
SELECT * FROM book;

#只复制部分数据
CREATE TABLE copy3
SELECT id,price
FROM book 
WHERE authorId=3;


#仅仅复制某些字段
CREATE TABLE copy4 
SELECT id,price
FROM book
WHERE 0;
```

### 3.4 常见约束

含义：一种限制，用于限制表中的数据，为了保证表中的数据的准确和可靠性

分类：六大约束

- `NOT NULL`：非空，用于保证该字段的值不能为空，比如姓名、学号等
- `DEFAULT`：默认，用于保证该字段有默认值，比如性别
- `PRIMARY KEY`：**主键，用于保证该字段的值具有唯一性，并且非空**， 比如学号、员工编号等
- `UNIQUE`：**唯一，用于保证该字段的值具有唯一性，可以为空**，比如座位号
- `CHECK`：检查约束【mysql中不支持】，比如年龄、性别
- `FOREIGN KEY`：外键，用于限制两个表的关系，用于保证该字段的值必须来自于主表的关联列的值
  - 在从表添加外键约束，用于引用主表中某列的值
  - 比如学生表的专业编号，员工表的部门编号，员工表的工种编号

添加约束的时机：

- 创建表时
- 修改表时

约束的添加分类：

- 列级约束：六大约束语法上都支持，但外键约束没有效果
- 表级约束：除了非空、默认，其他的都支持

主键约束和唯一约束的对比：

|      | 保证唯一性 | 是否允许为空 | 一个表是否可以一多个 | 是否允许组合 |
| :--: | :--------: | :----------: | :------------------: | :----------: |
| 主键 |     √      |      ×       |       至多1个        | √，但不推荐  |
| 唯一 |     √      |      √       |      可以有多个      | √，但不推荐  |

外键：
	1、要求在从表设置外键关系
	2、从表的外键列的类型和主表的关联列的类型要求一致或兼容，名称无要求
	3、主表的关联列必须是一个key（一般是主键或唯一）
	4、插入数据时，先插入主表，再插入从表；删除数据时，先删除从表，再删除主表



# Reference

- [零散的MySQL基础知识](https://mp.weixin.qq.com/s/wKIcd2Pq3RAeh7gB4uwinw)
- [mysql中drop、truncate和delete的区别](https://blog.csdn.net/czh500/article/details/106978028)

