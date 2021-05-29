<!-- GFM-TOC -->

- [Mybatis简介](#Mybatis简介)
  - [原始JDBC操作（查询数据）](#原始JDBC操作（查询数据）)
  - [原始JDBC操作（增删改）](#原始JDBC操作（增删改）)
  - [原始JDBC操作分析](#原始JDBC操作分析)
  - [Mybatis](#Mybatis)
- [Mybatis快速入门](#Mybatis快速入门)
  - [1. 搭建环境，创建数据库表](#_1-搭建环境，创建数据库表)
  - [2. 创建maven工程，导入Mybatis坐标](#_2-创建maven工程，导入Mybatis坐标)
  - [3. 在src/resources下创建mybatis-config.xml文件](#_3-在srcresources下创建mybatis-configxml文件)
  - [4. 编写Mybatis工具类](#_4-编写Mybatis工具类)
  - [5. 编写User实体类](#_5-编写User实体类)
  - [6. 编写UserDAO/UserMapper接口](#_6-编写UserDAOUserMapper接口)
  - [7. 编写UserMapper.xml](#_7-编写UserMapperxml)
  - [8. 编写测试类](#_8-编写测试类)

- [Reference](#Reference)

<!-- GFM-TOC -->

# Mybatis简介

## 原始JDBC操作（查询数据）

```java
package com.atguigu3.utils;

import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

/**
 * 操作数据库的工具类
 * @author yzze
 * @create 2021-02-21 19:30
 */
public class JDBCUtils {
    /**
     * 获取数据库的连接
     * @return
     * @throws Exception
     */
    public static Connection getConnection() throws Exception {
        // 1. 读取配置文件的4个基本信息
        InputStream inputStream = ClassLoader.getSystemClassLoader().getResourceAsStream("jdbc.properties");
        Properties pros = new Properties();
        pros.load(inputStream);

        String user = pros.getProperty("user");
        String password = pros.getProperty("password");
        String url = pros.getProperty("url");
        String driverClass = pros.getProperty("driverClass");

        //2. 加载驱动
        Class.forName(driverClass);

        //3. 获取连接
        Connection conn = DriverManager.getConnection(url, user, password);
        return conn;
    }

    /**
     * 关闭连接和Statement的操作
     * @param conn
     * @param ps
     */
    public static void closeResource(Connection conn, Statement ps) {
        //7. 资源的关闭
        try {
            if (ps!=null) {
                ps.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            if (conn!=null) {
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

```java
/**
     * 针对于不同的表的通用的查询操作，返回表中的一条记录
     * @param clazz
     * @param sql
     * @param args
     * @param <T>
     * @return
     */
public <T> T getInstance(Class<T> clazz, String sql, Object ...args) {
  Connection conn = null;
  PreparedStatement ps = null;
  ResultSet rs = null;
  try {
    conn = JDBCUtils.getConnection();
    ps = conn.prepareStatement(sql);
    for (int i = 0; i < args.length; i++) {
      ps.setObject(i+1, args[i]);
    }
    rs = ps.executeQuery();
    // 结果集的元数据
    ResultSetMetaData rsmd = rs.getMetaData();
    // 通过ResultSetMetaData获取结果集中的列数
    int columnCount = rsmd.getColumnCount();
    if (rs.next()) {
      //T t = clazz.newInstance();
      T t = clazz.getDeclaredConstructor().newInstance();
      // 处理结果集一行数据中的每一个列
      for (int i = 0; i < columnCount; i++) {
        // 获取列值
        Object columValue = rs.getObject(i + 1);
        // 获取每个列的列名: getColumnName()
        // 获取列的别名：getColumnLabel()
        String columnLabel = rsmd.getColumnLabel(i + 1);
        // 给 clazz 对象的某个属性，赋值为 columValue : 通过反射
        Field field = clazz.getDeclaredField(columnLabel);
        field.setAccessible(true);
        field.set(t, columValue);
      }
      return t;
    }
  } catch (Exception e) {
    e.printStackTrace();
  } finally {
    JDBCUtils.closeResource(conn, ps, rs);
  }
  return null;
}
```

## 原始JDBC操作（增删改）

```java
//通用的增、删、改操作
public void update(String sql,Object ... args){
    Connection conn = null;
    PreparedStatement ps = null;
    try {
        //1.获取数据库的连接
        conn = JDBCUtils.getConnection();

        //2.获取PreparedStatement的实例 (或：预编译sql语句)
        ps = conn.prepareStatement(sql);
        //3.填充占位符
        for(int i = 0;i < args.length;i++){
            ps.setObject(i + 1, args[i]);
        }

        //4.执行sql语句
        ps.execute();
    } catch (Exception e) {

        e.printStackTrace();
    }finally{
        //5.关闭资源
        JDBCUtils.closeResource(conn, ps);

    }
}
```

## 原始JDBC操作分析

- 原始jdbc开发存在的问题如下：
  - 数据库连接创建、释放频繁造成系统资源浪费从而影响系统性能
  - sql 语句在代码中硬编码，造成代码不易维护，实际应用 sql 变化的可能较大，sql 变动需要改变java代码。
  - 查询操作时，需要手动将结果集中的数据手动封装到实体中。插入操作时，需要手动将实体的数据设置到sql语句的占位符位置
- 应对上述问题给出的解决方案：
  - 使用数据库连接池初始化连接资源
  - 将sql语句抽取到xml配置文件中
  - 使用反射、内省等底层技术，自动将实体与表进行属性与字段的自动映射

## Mybatis

- MyBatis 是一款优秀的**持久层框架**，它支持自定义 SQL、存储过程以及高级映射。**MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录**。

- 它内部封装了jdbc，使开发者只需要关注sql语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程。
- mybatis通过xml或注解的方式将要执行的各种 statement配置起来，并通过java对象和statement中sql的动态参数进行映射生成最终执行的sql语句。
- mybatis框架执行sql并将结果映射为java对象并返回。采用ORM思想解决了实体和数据库映射的问题，对jdbc 进行了封装，屏蔽了jdbc api 底层访问细节，使我们不用与jdbc api 打交道，就可以完成对数据库的持久化操作。

# Mybatis快速入门

## 1. 搭建环境，创建数据库表

```mysql
create database mybatis;

use mybatis;

create table user(
	id int(20) not null primary key,
  name varchar(30) default null,
  pwd varchar(30) default null
)engine=innodb default charset=utf8;

insert into user (id, name, pwd) values
(1, 'abc', '123456'),
(2, 'cde', '234567'),
(3, 'def', '456789');
```



## 2. 创建maven工程，导入Mybatis坐标

```java
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.4.6</version>
</dependency>
```



## 3. 在src/resources下创建mybatis-config.xml文件

- XML 配置文件中包含了对 MyBatis 系统的核心设置，包括获取数据库连接实例的数据源（DataSource）以及决定事务作用域和控制方式的事务管理器（TransactionManager）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;
                useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="1997"/>
            </dataSource>
        </environment>
    </environments>
<!--    每一个mapper.xml都必须在mybatis核心配置文件中注册 -->
    <mappers>
        <mapper resource="com/dao/UserMapper.xml" />
    </mappers>
</configuration>
```

## 4. 编写Mybatis工具类

- 每个基于 MyBatis 的应用都是以一个 `SqlSessionFactory` 的实例为核心的。`SqlSessionFactory` 的实例可以通过 `SqlSessionFactoryBuilder` 获得。而 `SqlSessionFactoryBuilder` 则可以从 XML 配置文件或一个预先配置的 Configuration 实例来构建出 `SqlSessionFactory` 实例

  ```java
  package com.utils;
  
  import org.apache.ibatis.io.Resources;
  import org.apache.ibatis.session.SqlSession;
  import org.apache.ibatis.session.SqlSessionFactory;
  import org.apache.ibatis.session.SqlSessionFactoryBuilder;
  
  import java.io.IOException;
  import java.io.InputStream;
  
  /**
   * @author yangzeze
   */
  public class MybatisUtils {
      private static SqlSessionFactory sqlSessionFactory;
      static {
          // 从 XML 文件中构建 SqlSessionFactory 的实例
          String resource = "mybatis-config.xml";
          try {
              InputStream inputStream = Resources.getResourceAsStream(resource);
              sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  
      /**
       * 从 SqlSessionFactory 中获取 SqlSession
       * SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句
       * @return
       */
      public static SqlSession getSqlSession() {
          return sqlSessionFactory.openSession();
      }
  }
  ```

  

## 5. 编写User实体类

```java
package com.pojo;

/**
 * @author yangzeze
 */
public class User {
    private int id;
    private String name;
    private String pwd;
  
    public User() {
    }

    public User(int id, String name, String pwd) {
      this.id = id;
      this.name = name;
      this.pwd = pwd;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPwd() {
        return pwd;
    }

    public void setPwd(String pwd) {
        this.pwd = pwd;
    }
        @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", pwd='" + pwd + '\'' +
                '}';
    }
}
```

## 6. 编写UserDAO/UserMapper接口

```java
package com.dao;

import com.pojo.User;

import java.util.List;
import java.util.Map;

/**
 * @author yangzeze
 */
public interface UserDAO {
    /**
     * 查询全部用户
     * @return
     */
    List<User> getUserList();

    /**
     * 根据id查询用户
     * @param id
     * @return
     */
    User getUserById(int id);

    /**
     * 插入用户
     * @param user
     */
    void addUser(User user);

    //修改用户
    int updateUser(User user);

    /**
     * 删除用户
     * @param id
     * @return
     */
    int deleteUser(int id);

    List<User> getUserLike(String value);

    /**
     * 万能Map
     * Map传递参数，直接在sql中取出key即可
     * 对象传递参数，直接在sql中取对象的属性即可
     */
    int addUser2(Map<String, Object> map);
    User getUserById2(Map<String, Object> map);
    
    /**
     * 分页查询
     * 减少数据量
     * @param map
     * @return
     */
    List<User> getUserByLimit(Map<String, Integer> map);
}

```

## 7. 编写UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.dao.UserDAO">
    <select id="getUserList" resultType="com.pojo.User">
    select * from mybatis.user
  </select>

    <select id="getUserById" resultType="com.pojo.User" parameterType="int">
        select * from mybatis.user where id = #{id}
    </select>

    <!--对象中的属性可以直接取出来-->
    <insert id="addUser" parameterType="com.pojo.User">
        insert into mybatis.user (id, name, pwd) values (#{id}, #{name}, #{pwd});
    </insert>

    <update id="updateUser" parameterType="com.pojo.User">
        update mybatis.user set name=#{name}, pwd=#{pwd} where id =#{id};
    </update>

    <delete id="deleteUser" parameterType="int">
        delete from mybatis.user where id=#{id};
    </delete>

    <insert id="addUser2" parameterType="map">
        insert into mybatis.user (id, name, pwd) values (#{id1}, #{name1}, #{pwd1});
    </insert>

    <select id="getUserById2" parameterType="map" resultType="com.pojo.User">
        select * from mybatis.user where id = #{helloid} and name = #{name};
    </select>

    <select id="getUserLike" resultType="com.pojo.User">
        select * from mybatis.user where name like "%"#{value}"%"
    </select>
    
    <select id="getUserByLimit" parameterType="map"
            resultType="com.pojo.User">
        select * from mybatis.user limit #{startIndex},#{pageSize}
    </select>
</mapper>
```

## 8. 编写测试类

```java
package com.dao;

import com.pojo.User;
import com.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class UserDAOTest {
    @Test
    public void getUserListTest() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserDAO userDAO = sqlSession.getMapper(UserDAO.class);
        List<User> userList = userDAO.getUserList();
        for (int i = 0; i < userList.size(); i++) {
            System.out.println(userList.get(i));
        }
        sqlSession.close();
    }

    @Test
    public void getUserByIdTest() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        UserDAO mapper = sqlSession.getMapper(UserDAO.class);
        User user = mapper.getUserById(1);
        System.out.println(user);

        sqlSession.close();
    }

    //增删改需要提交事务
    @Test
    public void addUser(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        UserDAO mapper = sqlSession.getMapper(UserDAO.class);
        mapper.addUser(new User(4,"hou","123456"));

        //提交事务
        sqlSession.commit();

        sqlSession.close();
    }

    @Test
    public void updateUser(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        UserDAO mapper = sqlSession.getMapper(UserDAO.class);
        mapper.updateUser(new User(4,"hou","123"));

        //提交事务
        sqlSession.commit();

        sqlSession.close();
    }

    @Test
    public void deleteUser(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        UserDAO mapper = sqlSession.getMapper(UserDAO.class);
        mapper.deleteUser(5);

        //提交事务
        sqlSession.commit();

        sqlSession.close();
    }

    @Test
    public void addUser2(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        UserDAO mapper = sqlSession.getMapper(UserDAO.class);
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("id1",5);
        map.put("name1","dong");
        map.put("pwd1","12345");
        mapper.addUser2(map);

        //提交事务
        sqlSession.commit();

        sqlSession.close();
    }

    @Test
    public void getUserById2(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        UserDAO mapper = sqlSession.getMapper(UserDAO.class);
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("helloid",5);
        map.put("name","dong");
        User user = mapper.getUserById2(map);
        System.out.println(user);
        sqlSession.close();
    }

    @Test
    public void getUserLike(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        UserDAO mapper = sqlSession.getMapper(UserDAO.class);
        List<User> list = mapper.getUserLike("o");

        for(User user : list){
            System.out.println(user);
        }

        sqlSession.close();
    }

    @Test
    public void judge() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        SqlSession sqlSession1 = MybatisUtils.getSqlSession();
        System.out.println(sqlSession == sqlSession1);  // false
    }
    
    @Test
    public void getByLimit(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserDAO mapper = sqlSession.getMapper(UserDAO.class);
        Map<String, Integer> map = new HashMap<String, Integer>();
        map.put("startIndex", 1);
        map.put("pageSize", 2);
        List<User> userList = mapper.getUserByLimit(map);

        for(User user:userList){
            System.out.println(user);
        }

        sqlSession.close();
    }
}
```

- 注意：

  - Mapper.xml文件注册

  - ```xml
    <!--    每一个mapper.xml都必须在mybatis核心配置文件中注册 -->
    <mappers>
      <mapper resource="com/dao/UserMapper.xml" />
    </mappers>
    ```

  - maven资源过滤

  - ```xml
    <!--在build中配置resources，来防止我们资源导出失败的问题-->
    <build>
      <resources>
        <resource>
          <directory>src/main/resources</directory>
          <includes>
            <include>**/*.properties</include>
            <include>**/*.xml</include>
          </includes>
        </resource>
        <resource>
          <directory>src/main/java</directory>
          <includes>
            <include>**/*.properties</include>
            <include>**/*.xml</include>
          </includes>
        </resource>
      </resources>
    </build>
    ```

    

# Reference

- [Mybatis入门](https://mybatis.org/mybatis-3/zh/getting-started.html)
- [Mybatis-Study](https://github.com/Donkequan/Mybatis-Study)
- [狂神说Java:Mybatis教程](https://www.bilibili.com/video/BV1NE411Q7Nx?p=10&spm_id_from=pageDriver)