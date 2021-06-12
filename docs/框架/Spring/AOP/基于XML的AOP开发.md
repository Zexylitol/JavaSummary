# 快速入门

a. 导入AOP相关坐标

```xml
<!--导入spring的context坐标，context依赖aop-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
<!-- aspectj的织入 -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.4</version>
</dependency>
```

b. 创建目标接口和目标类（内部有切点）

```java
public interface TargetInterface {

    public void save();

}
```

```java
public class Target implements TargetInterface {
    @Override
    public void save() {
        System.out.println("save running.....");
    }
}
```

c. 创建切面类（内部有增强方法）

```java
import org.aspectj.lang.ProceedingJoinPoint;

public class MyAspect {

    public void before(){
        System.out.println("前置增强..........");
    }

    public void afterReturning(){
        System.out.println("后置增强..........");
    }

    //Proceeding JoinPoint:  正在执行的连接点===切点
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("环绕前增强....");
        Object proceed = pjp.proceed();//切点方法
        System.out.println("环绕后增强....");
        return proceed;
    }

    public void afterThrowing(){
        System.out.println("异常抛出增强..........");
    }

    public void after(){
        System.out.println("最终增强..........");
    }

}
```

d. 将目标类和切面类的对象创建权交给 spring

```xml
<!--目标对象-->
<bean id="target" class="com.itheima.aop.Target"></bean>

<!--切面对象-->
<bean id="myAspect" class="com.itheima.aop.MyAspect"></bean>
```

e. 在 applicationContext.xml 中配置织入关系

- 导入aop命名空间
- 配置织入关系

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
">


    <!--目标对象-->
    <bean id="target" class="com.itheima.aop.Target"></bean>

    <!--切面对象-->
    <bean id="myAspect" class="com.itheima.aop.MyAspect"></bean>

    <!--配置织入：告诉spring框架 哪些方法(切点)需要进行哪些增强(前置、后置...)-->
    <aop:config>
        <!--声明切面-->
        <aop:aspect ref="myAspect">
            <!--抽取切点表达式-->
            <aop:pointcut id="myPointcut" expression="execution(* com.itheima.aop.*.*(..))"></aop:pointcut>
            <!--切面：切点+通知-->
            <!--<aop:before method="before" pointcut="execution(public void com.itheima.aop.Target.save())"/>-->
            <!--<aop:before method="before" pointcut="execution(* com.itheima.aop.*.*(..))"/>
            <aop:after-returning method="afterReturning" pointcut="execution(* com.itheima.aop.*.*(..))"/>-->
            <!--<aop:around method="around" pointcut="execution(* com.itheima.aop.*.*(..))"/>
            <aop:after-throwing method="afterThrowing" pointcut="execution(* com.itheima.aop.*.*(..))"/>
            <aop:after method="after" pointcut="execution(* com.itheima.aop.*.*(..))"/>-->
            <aop:around method="around" pointcut-ref="myPointcut"/>
            <aop:after method="after" pointcut-ref="myPointcut"/>

        </aop:aspect>
    </aop:config>

</beans>
```

f. 测试代码

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class AopTest {

    @Autowired
    private TargetInterface target;

    @Test
    public void test1(){
        target.save();
    }

}
```

e. 测试结果

```java
六月 12, 2021 5:23:41 下午 org.springframework.test.context.support.AbstractTestContextBootstrapper getDefaultTestExecutionListenerClassNames
信息: Loaded default TestExecutionListener class names from location [META-INF/spring.factories]: [org.springframework.test.context.web.ServletTestExecutionListener, org.springframework.test.context.support.DirtiesContextBeforeModesTestExecutionListener, org.springframework.test.context.support.DependencyInjectionTestExecutionListener, org.springframework.test.context.support.DirtiesContextTestExecutionListener, org.springframework.test.context.transaction.TransactionalTestExecutionListener, org.springframework.test.context.jdbc.SqlScriptsTestExecutionListener]
六月 12, 2021 5:23:41 下午 org.springframework.test.context.support.AbstractTestContextBootstrapper getTestExecutionListeners
信息: Using TestExecutionListeners: [org.springframework.test.context.support.DirtiesContextBeforeModesTestExecutionListener@7225790e, org.springframework.test.context.support.DependencyInjectionTestExecutionListener@54a097cc, org.springframework.test.context.support.DirtiesContextTestExecutionListener@36f6e879]
六月 12, 2021 5:23:41 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [applicationContext.xml]
六月 12, 2021 5:23:42 下午 org.springframework.context.support.AbstractApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.GenericApplicationContext@26be92ad: startup date [Sat Jun 12 17:23:42 GMT+08:00 2021]; root of context hierarchy
环绕前增强....
save running.....
环绕后增强....
最终增强..........
六月 12, 2021 5:23:42 下午 org.springframework.context.support.AbstractApplicationContext doClose
信息: Closing org.springframework.context.support.GenericApplicationContext@26be92ad: startup date [Sat Jun 12 17:23:42 GMT+08:00 2021]; root of context hierarchy

Process finished with exit code 0
```

# XML配置AOP详解

1. 切点表达式的写法

表达式语法：

```xml
execution([修饰符] 返回值类型 包名.类名.方法名(参数))
```

- 访问修饰符可以省略

- 返回值类型、包名、类名、方法名可以使用星号* 代表任意

- 包名与类名之间一个点 . 代表当前包下的类，两个点 .. 表示当前包及其子包下的类

- 参数列表可以使用两个点 .. 表示任意个数，任意类型的参数列表

示例：

```xml
execution(public void com.itheima.aop.Target.method()) 

execution(void com.itheima.aop.Target.*(..))

execution(* com.itheima.aop.*.*(..))

execution(* com.itheima.aop..*.*(..))

execution(* *..*.*(..))
```

2. 通知的类型

通知的配置语法：

```xml
<aop:通知类型 method=“切面类中方法名” pointcut=“切点表达式"></aop:通知类型>
```

| **名称**     | **标签**               | **说明**                                                     |
| :----------- | :--------------------- | :----------------------------------------------------------- |
| 前置通知     | \<aop:before>          | 用于配置前置通知。指定增强的方法在切入点方法之前执行         |
| 后置通知     | \<aop:after-returning> | 用于配置后置通知。指定增强的方法在切入点方法之后执行         |
| 环绕通知     | \<aop:around>          | 用于配置环绕通知。指定增强的方法在切入点方法之前和之后都执行 |
| 异常抛出通知 | \<aop:throwing>        | 用于配置异常抛出通知。指定增强的方法在出现异常时执行         |
| 最终通知     | \<aop:after>           | 用于配置最终通知。无论增强方式执行是否有异常都会执行         |

3. 切点表达式的抽取

当多个增强的切点表达式相同时，可以将切点表达式进行抽取，在增强中使用 pointcut-ref 属性代替 pointcut 属性来引用抽取后的切点表达式。

```xml
<aop:config>
   <!--引用myAspect的Bean为切面对象-->
    <aop:aspect ref="myAspect">
        <!--抽取切点表达式-->
        <aop:pointcut id="myPointcut" expression="execution(* com.itheima.aop.*.*(..))"></aop:pointcut>
        <aop:after method="before" pointcut-ref="myPointcut"/>
    </aop:aspect>
</aop:config>
```

# Reference

- [黑马程序员SSM框架教程](https://www.bilibili.com/video/BV1WZ4y1P7Bp?p=129)