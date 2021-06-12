# 1. 快速入门

a. 创建目标接口和目标类（内部有切点）

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

b. 创建切面类（内部有增强方法）

```java
public class MyAspect {

    //配置前置通知
    public void before(){
        System.out.println("前置增强..........");
    }

    public void afterReturning(){
        System.out.println("后置增强..........");
    }

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

    //定义切点表达式
    public void pointcut(){}

}
```

c. 将目标类和切面类的对象创建权交给 spring

```java
@Component("target")
public class Target implements TargetInterface {
    @Override
    public void save() {
        System.out.println("save running.....");
    }
}
```

```java
@Component("myAspect")
public class MyAspect {

    public void before(){
        System.out.println("前置增强..........");
    }

    public void afterReturning(){
        System.out.println("后置增强..........");
    }

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

    //定义切点表达式
    public void pointcut(){}

}
```

d. 在切面类中使用注解配置织入关系

```java
@Component("myAspect")
@Aspect //标注当前MyAspect是一个切面类
public class MyAspect {

    //配置前置通知
    //@Before("execution(* com.itheima.anno.*.*(..))")
    public void before(){
        System.out.println("前置增强..........");
    }

    public void afterReturning(){
        System.out.println("后置增强..........");
    }

    //Proceeding JoinPoint:  正在执行的连接点===切点
    //@Around("execution(* com.itheima.anno.*.*(..))")
    @Around("pointcut()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("环绕前增强....");
        Object proceed = pjp.proceed();//切点方法
        System.out.println("环绕后增强....");
        return proceed;
    }

    public void afterThrowing(){
        System.out.println("异常抛出增强..........");
    }

    //@After("execution(* com.itheima.anno.*.*(..))")
    @After("MyAspect.pointcut()")
    public void after(){
        System.out.println("最终增强..........");
    }

    //定义切点表达式
    @Pointcut("execution(* com.itheima.anno.*.*(..))")
    public void pointcut(){}

}
```

e. 在XML配置文件中开启组件扫描和 AOP 的自动代理

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
">

    <!--组件扫描-->
    <context:component-scan base-package="com.itheima.anno"/>

    <!--aop自动代理-->
    <aop:aspectj-autoproxy/>

</beans>
```

f. 测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext-anno.xml")
public class AnnoTest {

    @Autowired
    private TargetInterface target;

    @Test
    public void test1(){
        target.save();
    }
}
```

g. 测试结果

```java
六月 12, 2021 5:59:20 下午 org.springframework.test.context.support.AbstractTestContextBootstrapper getDefaultTestExecutionListenerClassNames
信息: Loaded default TestExecutionListener class names from location [META-INF/spring.factories]: [org.springframework.test.context.web.ServletTestExecutionListener, org.springframework.test.context.support.DirtiesContextBeforeModesTestExecutionListener, org.springframework.test.context.support.DependencyInjectionTestExecutionListener, org.springframework.test.context.support.DirtiesContextTestExecutionListener, org.springframework.test.context.transaction.TransactionalTestExecutionListener, org.springframework.test.context.jdbc.SqlScriptsTestExecutionListener]
六月 12, 2021 5:59:20 下午 org.springframework.test.context.support.AbstractTestContextBootstrapper getTestExecutionListeners
信息: Using TestExecutionListeners: [org.springframework.test.context.support.DirtiesContextBeforeModesTestExecutionListener@7225790e, org.springframework.test.context.support.DependencyInjectionTestExecutionListener@54a097cc, org.springframework.test.context.support.DirtiesContextTestExecutionListener@36f6e879]
六月 12, 2021 5:59:20 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [applicationContext-anno.xml]
六月 12, 2021 5:59:20 下午 org.springframework.context.support.AbstractApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.GenericApplicationContext@26be92ad: startup date [Sat Jun 12 17:59:20 GMT+08:00 2021]; root of context hierarchy
环绕前增强....
save running.....
环绕后增强....
最终增强..........
六月 12, 2021 5:59:21 下午 org.springframework.context.support.AbstractApplicationContext doClose
信息: Closing org.springframework.context.support.GenericApplicationContext@26be92ad: startup date [Sat Jun 12 17:59:20 GMT+08:00 2021]; root of context hierarchy

Process finished with exit code 0
```

# 2. 注解配置 AOP 详解

1. 注解通知的类型

通知的配置语法：`@通知注解(“切点表达式")`

| **名称**     | **注解**        | **说明**                                                     |
| :----------- | :-------------- | :----------------------------------------------------------- |
| 前置通知     | @Before         | 用于配置前置通知。指定增强的方法在切入点方法之前执行         |
| 后置通知     | @AfterReturning | 用于配置后置通知。指定增强的方法在切入点方法之后执行         |
| 环绕通知     | @Around         | 用于配置环绕通知。指定增强的方法在切入点方法之前和之后都执行 |
| 异常抛出通知 | @AfterThrowing  | 用于配置异常抛出通知。指定增强的方法在出现异常时执行         |
| 最终通知     | @After          | 用于配置最终通知。无论增强方式执行是否有异常都会执行         |

2. 切点表达式的抽取

同 xml 配置 aop 一样，我们可以将切点表达式抽取。抽取方式是在切面内定义方法，在该方法上使用`@Pointcut`注解定义切点表达式，然后在在增强注解中进行引用。具体如下：

```java
@Component("myAspect")
@Aspect //标注当前MyAspect是一个切面类
public class MyAspect {

    //配置前置通知
    //@Before("execution(* com.itheima.anno.*.*(..))")
    @Before("myAspect.myPoint()")
    public void before(){
        System.out.println("前置增强..........");
    }

    //定义切点表达式
    @Pointcut("execution(* com.itheima.anno.*.*(..))")
    public void pointcut(){}

}
```

# Reference

- [黑马程序员SSM框架教程](https://www.bilibili.com/video/BV1WZ4y1P7Bp?p=129)

