# 日志工厂

Mybatis 通过使用内置的日志工厂提供日志功能。可以通过在 MyBatis 配置文件 mybatis-config.xml 里面添加一项 setting 来选择日志实现。

```xml
<configuration>
    <settings>
        ...
        <setting name="logImpl" value="LOG4J"/>
        ...
    </settings>
</configuration>
```

| 设置名  | 描述                                                  | 可选值                                                       | 默认值 |
| :------ | :---------------------------------------------------- | :----------------------------------------------------------- | :----- |
| logImpl | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。 | SLF4J \| **LOG4J** \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| **STDOUT_LOGGING** \| NO_LOGGING | 未设置 |

## STDOUT_LOGGING

在mybatis-config中设置：

```xml
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

日志输出：

```java
Logging initialized using 'class org.apache.ibatis.logging.stdout.StdOutImpl' adapter.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
Opening JDBC Connection
Created connection 1039949752.
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@3dfc5fb8]
==>  Preparing: select * from mybatis.user where name like "%"?"%" 
==> Parameters: o(String)
<==      Total: 0
Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@3dfc5fb8]
Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@3dfc5fb8]
Returned connection 1039949752 to pool.
```

## Log4j

### 1. 添加 Log4J 的 jar 包/导入maven依赖

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

### 2. 配置Log4j

a. 在应用的类路径中创建一个名为 `log4j.properties` 的文件，文件的具体内容如下：

```xml
### set log levels ###
log4j.rootLogger = DEBUG,console,file

### 输出到控制台 ###
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold = DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern = [%c]-%m%n

### 输出到日志文件 ###
log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/hou.log
log4j.appender.file.MaxFileSize=10mb 
log4j.appender.file.Threshold=DEBUG 
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

# 日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

b. 在mybatis-config中设置：

```xml
<settings>
    <setting name="logImpl" value="LOG4J"/>
</settings>
```

c. 使用

```java
import org.junit.Test;
import org.apache.log4j.Logger;

public class UserDAOTest {

    static Logger logger = Logger.getLogger(UserDAOTest.class);

    @Test
    public void testLog4j(){
        logger.info("info:进入了testlog4j");
        logger.debug("debug:进入了testlog4j");
        logger.error("error:进入了testlog4j");
    }
}
```

输出：

```java
[com.dao.UserDAOTest]-info:进入了testlog4j
[com.dao.UserDAOTest]-debug:进入了testlog4j
[com.dao.UserDAOTest]-error:进入了testlog4j
```

# Reference

- [Mybatis-Study](https://github.com/Donkequan/Mybatis-Study)
- [mybatis – MyBatis 3 | 日志](https://mybatis.org/mybatis-3/zh/logging.html)



