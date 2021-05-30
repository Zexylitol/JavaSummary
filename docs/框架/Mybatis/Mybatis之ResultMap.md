# ResultMap

## 简介

- `resultMap` – 描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。
- `resultMap` 元素是 MyBatis 中最重要最强大的元素。它可以让你从 90% 的 JDBC `ResultSets` 数据提取代码中解放出来，并在一些情形下允许你进行一些 JDBC 不支持的操作。实际上，在为一些比如连接的复杂语句编写映射代码的时候，一份 `resultMap` 能够代替实现同等功能的数千行代码。
- ResultMap 的设计思想是，对简单的语句做到零配置，对于复杂一点的语句，只需要描述语句之间的关系就行了。

## 多对一

a. 建表

```mysql
CREATE TABLE `teacher` (
	`id` INT(10) NOT NULL PRIMARY KEY,
	`name` VARCHAR(30) DEFAULT NULL
)ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO teacher (`id`, `name`) VALUES (1, 'li');

CREATE TABLE `student` (
	`id` INT(10) NOT NULL,
	`name` VARCHAR(30) DEFAULT NULL,
	`tid` INT(10) DEFAULT NULL,
	PRIMARY KEY (`id`),
	KEY `fktid` (`tid`),
	CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO student (`id`, `name`, `tid`) VALUES (1, 'Tom', 1);
INSERT INTO student (`id`, `name`, `tid`) VALUES (2, 'Jerry', 1);
INSERT INTO student (`id`, `name`, `tid`) VALUES (3, 'Alice', 1);
INSERT INTO student (`id`, `name`, `tid`) VALUES (4, 'Sam', 1);
INSERT INTO student (`id`, `name`, `tid`) VALUES (5, 'Merry', 1);
```

<center><img src="https://i.im5i.com/2021/05/30/CZqk3.png" alt="CZqk3.png" border="0" /></center>

b. 创建实体类

```java
package com.pojo;
import lombok.Data;

@Data
public class Student {
    private int id;
    private String name;

    //学生需要关联一个老师
    private Teacher teacher;
}
```

```java
package com.pojo;
import lombok.Data;

@Data
public class Teacher {
    private int id;
    private String name;
}
```

c. 在mybatis-config.xml中绑定接口

```xml
<!--    接口绑定 -->
<!--    接口和它的Mapper必须同名-->
<!--    接口和它的Mapper必须在同一包下-->
<mappers>
    <mapper class="com.dao.StudentMapper"></mapper>
    <mapper class="com.dao.TeacherMapper"></mapper>
</mappers>
```

d. 创建Mapper.xml文件和Mapper接口

```java
package com.dao;

import com.pojo.Student;

import java.util.List;

public interface StudentMapper {
    // 查询所有的学生信息以及对应的老师的信息
    List<Student> getStudent();

    List<Student> getStudent2();
}
```

```java
package com.dao;
import com.pojo.Teacher;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
import java.util.List;

public interface TeacherMapper {
    @Select("select * from teacher where id = #{tid}")
    Teacher getTeacher(@Param("tid") int id);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.dao.StudentMapper">

<!--    select s.id sid,s.name sname,t.name tname-->
<!--    from student s,teacher t where s.tid=t.id;-->
<!--    按照结果嵌套处理-->
    <select id="getStudent2" resultMap="StudentTeacher2">
    select s.id sid,s.name sname,t.name tname
    from student s,teacher t where s.tid=t.id;
</select>

    <resultMap id="StudentTeacher2" type="com.pojo.Student">
        <result property="id" column="sid"></result>
        <result property="name" column="sname"></result>
        <association property="teacher" javaType="com.pojo.Teacher">
            <result property="name" column="tname"></result>
        </association>

    </resultMap>


    <!--    按照查询嵌套处理 -->
    <select id="getStudent" resultMap="StudentTeacher">
      select * from student;
    </select>

    <resultMap id="StudentTeacher" type="com.pojo.Student">
        <result property="id" column="id"></result>
        <result property="name" column="name"></result>
        <!--对象使用assiociation-->
        <!--集合用collection-->
        <association property="teacher" column="tid"
                     javaType="com.pojo.Teacher"
                     select="getTeacher"></association>
    </resultMap>

    <select id="getTeacher" resultType="com.pojo.Teacher">
      select * from teacher where id = #{id};
    </select>

</mapper>
```

c. 编写测试类

```java
package com.dao;

import com.pojo.Student;
import com.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class StudentMapperTest {
    @Test
    public void getStudentTest() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
        List<Student> student = mapper.getStudent();
        for (Student stu : student) {
            System.out.println(stu);
        }
        sqlSession.close();
    }

    @Test
    public void getStudent2Test() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
        List<Student> student = mapper.getStudent2();
        for (Student stu : student) {
            System.out.println(stu);
        }
        sqlSession.close();
    }
}
```

输出：

```java
Logging initialized using 'class org.apache.ibatis.logging.stdout.StdOutImpl' adapter.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
Opening JDBC Connection
Created connection 1780132728.
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@6a1aab78]
==>  Preparing: select * from student; 
==> Parameters: 
<==    Columns: id, name, tid
<==        Row: 1, Tom, 1
====>  Preparing: select * from teacher where id = ?; 
====> Parameters: 1(Integer)
<====    Columns: id, name
<====        Row: 1, li
<====      Total: 1
<==        Row: 2, Jerry, 1
<==        Row: 3, Alice, 1
<==        Row: 4, Sam, 1
<==        Row: 5, Merry, 1
<==      Total: 5
Student(id=1, name=Tom, teacher=Teacher(id=1, name=li))
Student(id=2, name=Jerry, teacher=Teacher(id=1, name=li))
Student(id=3, name=Alice, teacher=Teacher(id=1, name=li))
Student(id=4, name=Sam, teacher=Teacher(id=1, name=li))
Student(id=5, name=Merry, teacher=Teacher(id=1, name=li))
Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@6a1aab78]
Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@6a1aab78]
Returned connection 1780132728 to pool.

Process finished with exit code 0
```

## 一对多

a. 创建实体类

```java
package com.pojo;

import lombok.Data;

@Data
public class Student {
    private int id;
    private String name;
    private int tid;
}
```

```java
package com.pojo;


import lombok.Data;
import java.util.List;

@Data
public class Teacher {
    private int id;
    private String name;
    private List<Student> studentList;
}
```

b. 创建Mapper接口和对应的Mapper.xml

```java
package com.dao;

import com.pojo.Teacher;
import org.apache.ibatis.annotations.Param;

import java.util.List;

public interface TeacherMapper {
    List<Teacher> getTeacher();

    // 获取指定老师下的所有学生以及老师的信息
    // 按照结果查询
    Teacher getTeacher(@Param("tid") int id);

    // 按照查询嵌套处理
    Teacher getTeacher2(@Param("tid") int id);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.dao.TeacherMapper">

<!--    按照结果查询-->
    <select id="getTeacher" resultMap="TeacherStudent">
        select s.id sid, s.name sname, t.name tname, t.id tid
        from student s, teacher t
        where s.tid = t.id and t.id = #{tid};
    </select>

    <resultMap id="TeacherStudent" type="com.pojo.Teacher">
        <result property="id" column="tid"></result>
        <result property="name" column="tname"></result>
        <!--集合中的泛型信息，我们用oftype获取-->
        <collection property="studentList" ofType="com.pojo.Student">
            <result property="id" column="sid"></result>
            <result property="name" column="sname"></result>
        </collection>
    </resultMap>

<!--    按照查询嵌套处理-->
    <select id="getTeacher2" resultMap="TeacherStudent2">
        select * from mybatis.teacher where id = #{tid}
    </select>

    <resultMap id="TeacherStudent2" type="com.pojo.Teacher">
        <collection property="studentList" column="id" javaType="ArrayList"
            ofType="com.pojo.Student"
            select="getStudentByTeacherId"></collection>
    </resultMap>

    <select id="getStudentByTeacherId" resultType="com.pojo.Student">
        select * from mybatis.student where tid = #{id}
    </select>

</mapper>
```

```java
package com.dao;

import com.pojo.Student;

public interface StudentMapper {
    Student getStudentByTeacherId(int id);
}
```

c. 编写测试类

```java
package com.dao;

import com.pojo.Teacher;
import com.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class TeacherMapperTest {
    @Test
    public void getTeacherTest() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
        Teacher teacher = mapper.getTeacher(1);
        System.out.println(teacher);
        sqlSession.close();
    }

    @Test
    public void getTeachar2Test() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
        Teacher teacher = mapper.getTeacher2(1);
        System.out.println(teacher);
        sqlSession.close();
    }
}
```

输出：

```java
com.intellij.rt.junit.JUnitStarter -ideVersion5 -junit4 com.dao.TeacherMapperTest,getTeachar2Test
Logging initialized using 'class org.apache.ibatis.logging.stdout.StdOutImpl' adapter.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
Opening JDBC Connection
Created connection 1032986144.
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@3d921e20]
==>  Preparing: select * from mybatis.teacher where id = ? 
==> Parameters: 1(Integer)
<==    Columns: id, name
<==        Row: 1, li
====>  Preparing: select * from mybatis.student where tid = ? 
====> Parameters: 1(Integer)
<====    Columns: id, name, tid
<====        Row: 1, Tom, 1
<====        Row: 2, Jerry, 1
<====        Row: 3, Alice, 1
<====        Row: 4, Sam, 1
<====        Row: 5, Merry, 1
<====      Total: 5
<==      Total: 1
com.pojo.Teacher@1a3869f4
Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@3d921e20]
Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@3d921e20]
Returned connection 1032986144 to pool.

Process finished with exit code 0
```

## 总结

- 关联 - association 多对一

- 集合 - collection 一对多

- javaType & ofType

1. JavaType用来指定实体中属性类型
2. ofType映射到list中的类型，泛型中的约束类型

# Reference

- [狂神说Java:Mybatis教程](https://www.bilibili.com/video/BV1NE411Q7Nx?p=20)
- [XML映射文件](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#Result_Maps)

