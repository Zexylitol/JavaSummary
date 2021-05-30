<!-- GFM-TOC -->

- [简介](#简介)
- [动态SQL](#动态SQL)
  - [搭建环境](#搭建环境)
  - [if](#if)
  - [trim (where, set)](#trim-where-set)
    - [where](#where)
    - [set](#set)
  - [choose (when, otherwise)](#choose-when-otherwise-)
- [Reference](#Reference)

<!-- GFM-TOC -->

# 简介

- 动态sql：根据不同的条件生成不同的SQL语句

- 动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其它类似的框架，你应该能理解**根据不同条件拼接 SQL 语句有多痛苦**，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。
  - if
  - choose (when, otherwise)
  - trim (where, set)
  - foreach
- <font color="blue">动态sql本质还是SQL语句，只是可以在SQL层面去执行一个逻辑代码</font>

# 动态SQL

## 搭建环境

a. 创建数据表

```my
create table `blog`(
	`id` varchar(50) not null comment '博客id',
    `title` varchar(100) not null comment '博客标题',
    `author` varchar(30) not null comment '博客作者',
    `create_time` datetime not null comment '创建时间',
    `views` int(30) not null comment '浏览量'
	)ENGINE=InnoDB DEFAULT CHARSET=utf8
```

b. 创建实体类

```java
package com.pojo;
import lombok.Data;
import java.util.Date;

@Data
public class Blog {
    private String id;
    private String title;
    private String author;
    // 属性名和字段名不一致，在mybatis-config.xml中设置mapUnderscoreToCamelCase为True
    private Date createTime;        
    private int views;
}

```

c. 创建Mapper接口和Mapper.xml

```java
package com.dao;

import com.pojo.Blog;

public interface BlogMapper {
    // 插入数据
    int addBlog(Blog blog);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.dao.BlogMapper">
    <insert id="addBlog" parameterType="com.pojo.Blog">
        insert into mybatis.blog (id, title, author, create_time, views) values
        (#{id}, #{title}, #{author}, #{createTime}, #{views});
    </insert>
</mapper>
```

d. 在mybatis-config.xml中设置mapUnderscoreToCamelCase为True

| 设置名                   | 描述                                                         | 有效值        | 默认值 |
| :----------------------- | :----------------------------------------------------------- | :------------ | :----- |
| mapUnderscoreToCamelCase | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | true \| false | False  |

```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

e. 新增工具类

```java
package com.utils;

import org.junit.Test;
import java.util.UUID;

public class IDUtils {
    public static String getId(){
        return UUID.randomUUID().toString().replaceAll("-","");
    }

    @Test
    public void  test(){
        System.out.println(getId());
    }
}
```

f. 编写测试类，向表中添加数据

```java
package com.dao;

import com.pojo.Blog;
import com.utils.IDUtils;
import com.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.Date;

public class BlogMapperTest {
    @Test
    public void addBlogTest() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        BlogMapper blogMapper = sqlSession.getMapper(BlogMapper.class);

        Blog blog = new Blog();
        blog.setId(IDUtils.getId());
        blog.setAuthor("one");
        blog.setCreateTime(new Date());
        blog.setViews(999);
        blog.setTitle("first");

        blogMapper.addBlog(blog);

        blog.setId(IDUtils.getId());
        blog.setTitle("second");
        blogMapper.addBlog(blog);

        blog.setId(IDUtils.getId());
        blog.setTitle("third");
        blogMapper.addBlog(blog);

        blog.setId(IDUtils.getId());
        blog.setTitle("forth");
        blogMapper.addBlog(blog);

        sqlSession.commit();

        sqlSession.close();
    }
}
```

输出：

```java
com.intellij.rt.junit.JUnitStarter -ideVersion5 -junit4 com.dao.BlogMapperTest
log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
Logging initialized using 'class org.apache.ibatis.logging.stdout.StdOutImpl' adapter.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
Opening JDBC Connection
Created connection 1665404403.
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@63440df3]
==>  Preparing: insert into mybatis.blog (id, title, author, create_time, views) values (?, ?, ?, ?, ?); 
==> Parameters: 42244ca5eafd42c8ad30c4c2e256417e(String), first(String), one(String), 2021-05-30 20:39:48.37(Timestamp), 999(Integer)
<==    Updates: 1
==>  Preparing: insert into mybatis.blog (id, title, author, create_time, views) values (?, ?, ?, ?, ?); 
==> Parameters: fabffe13e1584f0795b10a82aab478e5(String), second(String), one(String), 2021-05-30 20:39:48.37(Timestamp), 999(Integer)
<==    Updates: 1
==>  Preparing: insert into mybatis.blog (id, title, author, create_time, views) values (?, ?, ?, ?, ?); 
==> Parameters: 07d0169f0fc24a3885dcef91b84d2f45(String), third(String), one(String), 2021-05-30 20:39:48.37(Timestamp), 999(Integer)
<==    Updates: 1
==>  Preparing: insert into mybatis.blog (id, title, author, create_time, views) values (?, ?, ?, ?, ?); 
==> Parameters: 5c41e8a64ff84ff49085e94a2aef6d97(String), forth(String), one(String), 2021-05-30 20:39:48.37(Timestamp), 999(Integer)
<==    Updates: 1
Committing JDBC Connection [com.mysql.jdbc.JDBC4Connection@63440df3]
Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@63440df3]
Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@63440df3]
Returned connection 1665404403 to pool.

Process finished with exit code 0
```

## if

使用动态 SQL 最常见情景是根据条件包含 where 子句的一部分

a. Mapper接口

```java
package com.dao;
import com.pojo.Blog;
import java.util.List;
import java.util.Map;

public interface BlogMapper {
    // 查询博客
    List<Blog> queryBlogIF(Map map);
}
```

b. Mapper.xml

```xml
<select id="queryBlogIF" parameterType="map" resultType="com.pojo.Blog">
    select * from mybatis.blog where 1=1
    <if test="title != null">
        and title = #{title}
    </if>
    <if test="author != author">
        and author = #{author}
    </if>
</select>
```

c. test

```java
@Test
public void queryBlogIFTest() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    BlogMapper blogMapper = sqlSession.getMapper(BlogMapper.class);
    Map map = new HashMap();

    //        map.put("title", "second");
    map.put("author", "one");

    List<Blog> list = blogMapper.queryBlogIF(map);

    for (Blog blog : list) {
        System.out.println(blog);
    }

    sqlSession.close();
}
```

## trim (where, set)

### where

- *where* 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。

```xml
<select id="queryBlogIF" parameterType="map" resultType="com.pojo.Blog">
    select * from mybatis.blog
    <where>
        <if test="title != null">
            and title = #{title}
        </if>
        <if test="author != author">
            and author = #{author}
        </if>
    </where>
</select>
```

### set

- *set* 元素可以用于动态包含需要更新的列，忽略其它不更新的列
- *set* 元素会动态地在行首插入 SET 关键字，并会删掉额外的逗号

```xml
<update id="updateBlogSet" parameterType="map">
    update mybatis.blog
    <set>
        <if test="title != null">
            title = #{title},
        </if>
        <if test="author != null">
            author = #{author}
        </if>
    </set>
    where id = #{id}
</update>
```

```java
public interface BlogMapper {
    // 更新博客
    int updateBlogSet(Map map);
}
```

```java
@Test
public void updateBlogSetTest() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    BlogMapper blogMapper = sqlSession.getMapper(BlogMapper.class);
    Map map = new HashMap();

    map.put("title", "java");
    //map.put("author", "one");
    map.put("id", "42244ca5eafd42c8ad30c4c2e256417e");

    int i = blogMapper.updateBlogSet(map);

    sqlSession.commit();

    sqlSession.close();
}
```

## choose (when, otherwise)

- choose 元素，它有点像 Java 中的 switch 语句

```java
public interface BlogMapper {
    // 更新博客
    int updateBlogChoose(Map map);
}
```

```xml
<select id="queryBlogChoose" parameterType="map" resultType="com.pojo.Blog>
    select * from mybatis.blog
    <where>
        <choose>
            <when test="title != null">
                title = #{title}
            </when>
            <when test="author != null">
                and author = #{author}
            </when>
            <otherwise>
                and views = #{views}
            </otherwise>
        </choose>
    </where>
</select>
```

```java
@Test
public void queryBlogChoose() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    BlogMapper blogMapper = sqlSession.getMapper(BlogMapper.class);
    Map map = new HashMap();

    map.put("title", "java");
    //map.put("author", "one");
    map.put("views", 999);

    blogMapper.queryBlogChoose(map);

    sqlSession.close();
}
```



# Reference

- [狂神说Java:Mybatis教程](https://www.bilibili.com/video/BV1NE411Q7Nx?p=23&spm_id_from=pageDriver)
- [动态SQL](https://mybatis.org/mybatis-3/zh/dynamic-sql.html)