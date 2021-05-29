<!-- GFM-TOC -->

- [核心配置文件](#核心配置文件)
  - [1. 环境配置（environments）](#环境配置（environments）)
    - [事务管理器（transactionManager）](#事务管理器（transactionManager）)
    - [数据源（dataSource）](#数据源（dataSource）)
  - [2. 属性（properties）](#属性（properties）)
  - [3. 类型别名（typeAliases）](#类型别名（typeAliases）)
  - [4. 设置（settings）](#设置（settings）)
  - [5. 映射器（mappers）](#映射器（mappers）)

- [Reference](#Reference)

<!-- GFM-TOC -->

# 核心配置文件

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。

**mybatis-config.xml**:

configuration（配置）

- [properties（属性）](https://mybatis.org/mybatis-3/zh/configuration.html#properties)
- [settings（设置）](https://mybatis.org/mybatis-3/zh/configuration.html#settings)
- [typeAliases（类型别名）](https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases)
- [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)
- [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)
- [plugins（插件）](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)
- [environments（环境配置）](https://mybatis.org/mybatis-3/zh/configuration.html#environments)
  - environment（环境变量）
    - transactionManager（事务管理器）
    - dataSource（数据源）
- [databaseIdProvider（数据库厂商标识）](https://mybatis.org/mybatis-3/zh/configuration.html#databaseIdProvider)
- [mappers（映射器）](https://mybatis.org/mybatis-3/zh/configuration.html#mappers)

## 环境配置（environments）

MyBatis 可以配置成适应多种环境，，这种机制有助于将 SQL 映射应用于多种数据库之中，例如，开发、测试和生产环境需要有不同的配置。

- **不过要记住：尽管可以配置多个环境，但每个 `SqlSessionFactory` 实例只能选择一种环境。**

所以，如果想连接多个数据库，就需要创建多个 `SqlSessionFactory` 实例，每个数据库对应一个

- **每个数据库对应一个 SqlSessionFactory 实例**

<font color="red">Mybatis 默认的事务管理器是JDBC，连接池：POOLED</font>

为了指定创建哪种环境，只要将它作为可选的参数传递给 SqlSessionFactoryBuilder 即可。可以接受环境配置的两个方法签名是：

```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment, properties);
```

如果忽略了环境参数，那么将会加载默认环境，如下所示：

```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, properties);
```

environments 元素定义了如何配置环境。

```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

注意一些关键点:

- 默认使用的环境 ID（比如：default="development"）。
- 每个 environment 元素定义的环境 ID（比如：id="development"）（保证默认的环境 ID 要匹配其中一个环境 ID）。
- 事务管理器的配置（比如：type="JDBC"）。
- 数据源的配置（比如：type="POOLED"）。

### 事务管理器（transactionManager）

在 MyBatis 中有两种类型的事务管理器（也就是 `type="[JDBC|MANAGED]"`）：

- JDBC – 这个配置直接使用了 JDBC 的提交和回滚设施，它依赖从数据源获得的连接来管理事务作用域
- MANAGED

> 如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。

### 数据源（dataSource）

有三种内建的数据源类型（也就是 `type="[UNPOOLED|POOLED|JNDI]"`）：

**UNPOOLED**– 这个数据源的实现会每次请求时打开和关闭连接。性能表现则依赖于使用的数据库，对某些数据库来说，使用连接池并不重要，这个配置就很适合这种情形。UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：

- `driver` – 这是 JDBC 驱动的 Java 类全限定名
- `url` – 这是数据库的 JDBC URL 地址。
- `username` – 登录数据库的用户名
- `password` – 登录数据库的密码
- `defaultTransactionIsolationLevel` – 默认的连接事务隔离级别
- `defaultNetworkTimeout` – 等待数据库操作完成的默认网络超时时间（单位：毫秒）

**POOLED**– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这种处理方式很流行，能使并发 Web 应用快速响应请求。

除了上述提到 UNPOOLED 下的属性外，还有更多属性用来配置 POOLED 的数据源：

- `poolMaximumActiveConnections` – 在任意时间可存在的活动（正在使用）连接数量，默认值：10
- `poolMaximumIdleConnections` – 任意时间可能存在的空闲连接数。
- `poolMaximumCheckoutTime` – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）

**JNDI** – 这个数据源实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用。这种数据源配置只需要两个属性：

- `initial_context` – 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么将会直接从 InitialContext 中寻找 data_source 属性。
- `data_source` – 这是引用数据源实例位置的上下文路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。

## 属性（properties）

可以通过properties属性来引用配置文件

这些属性可以在外部进行配置，并可以进行动态替换。既可以在典型的 Java 属性文件（db.properties）中配置这些属性，也可以在 properties 元素的子元素中设置。 

编写一个配置文件

db.properties

```
driver = com.mysql.jdbc.Driver
url = "jdbc:mysql://127.0.0.1:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=UTF-8"
username = root 
password = 123456
```

在核心配置文件mybatis-config.xml中引入

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

	<!--引入外部配置文件-->
    <properties resource="db.properties">
        <property name="username" value="root"></property>
        <property name="password" value="123456"></property>
    </properties>

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

    <!--每一个mapper.xml都需要注册-->
    <mappers>
        <mapper resource="com/dao/UserMapper.xml"/>
    </mappers>

</configuration>
```

如果一个属性在不只一个地方进行了配置，那么，MyBatis 将按照下面的顺序来加载：

- 首先读取在 properties 元素体内指定的属性。
- 然后根据 properties 元素中的 resource 属性读取类路径下属性文件，或根据 url 属性指定的路径读取属性文件，并覆盖之前读取过的同名属性。
- 最后读取作为方法参数传递的属性，并覆盖之前读取过的同名属性。

因此，**通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的则是 properties 元素中指定的属性**。

## 类型别名（typeAliases）

类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写。

类型别名可为 Java 类型设置一个缩写名字。

```xml
<typeAliases>
    <typeAlias type="com.pojo.User" alias="User"></typeAlias>
</typeAliases>
```

扫描实体类的包，默认别名就为这个类的类名首字母小写

```xml
<typeAliases>
    <package name="com.pojo"></package>
</typeAliases>
```

在实体类，比较少的时候使用第一种，实体类多使用第二种。

第一种可以自定义，第二则不行，但是 若有注解，则别名为其注解值 。

```java
@Alias("hello")
public class User {
}
```

## 设置（settings）

| 设置名             | 描述                                                         | 有效值                                                       | 默认值 |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------ |
| cacheEnabled       | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true \| false                                                | true   |
| lazyLoadingEnabled | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true \| false                                                | false  |
| logImpl            | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置 |

## 映射器（mappers）

这些配置会告诉 MyBatis 去哪里找映射文件

方式一: [推荐使用]

```xml
<mappers>
    <mapper resource="com/dao/UserMapper.xml"/>
</mappers>
```

方式二：

```xml
<mappers>
    <mapper class="com.dao.UserMapper" />
</mappers>
```

- 接口和它的Mapper必须同名
- 接口和它的Mapper必须在同一包下

方式三：

```xml
<mappers>
    <package name="com.dao" />
</mappers>
```

- 接口和它的Mapper必须同名
- 接口和它的Mapper必须在同一包下

# Reference

- [Mybatis入门](https://mybatis.org/mybatis-3/zh/getting-started.html)
- [Mybatis-Study](https://github.com/Donkequan/Mybatis-Study)
- [狂神说Java:Mybatis教程](https://www.bilibili.com/video/BV1NE411Q7Nx?p=10&spm_id_from=pageDriver)