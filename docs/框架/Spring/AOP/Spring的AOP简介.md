<!-- GFM-TOC -->

- [什么是AOP](#什么是AOP)
- [AOP的作用及优势](#AOP的作用及优势)
- [AOP的底层实现](#AOP的底层实现)
- [AOP的动态代理技术](#AOP的动态代理技术)
  - [常用的动态代理技术](#常用的动态代理技术)
    - [JDK代理 : 基于接口的动态代理技术](#jdk代理：基于接口的动态代理技术)
    - [cglib代理：基于父类的动态代理技术](#cglib代理：基于父类的动态代理技术)
- [AOP相关概念](#AOP相关概念)
- [AOP开发明确的事项](#AOP开发明确的事项)
- [总结](#总结)
- [Reference](#Reference)

<!-- GFM-TOC -->

# 什么是AOP

**AOP** 为 **A**spect **O**riented **P**rogramming 的缩写，意思为**面向切面编程**，是通过**预编译方式和运行期动态代理**实现程序功能的统一维护的一种技术。



AOP 是 OOP 的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的**耦合度降低，提高程序的可重用性**，同时提高了开发的效率。

# AOP的作用及优势

- 作用：在程序运行期间，在不修改源码的情况下对方法进行功能增强

- 优势：减少重复代码，提高开发效率，并且便于维护

# AOP的底层实现

AOP 的底层是通过 Spring 提供的的动态代理技术实现的。<font color="red">在运行期间，Spring通过动态代理技术动态的生成代理对象，代理对象方法执行时进行增强功能的介入，在去调用目标对象的方法，从而完成功能的增强</font>

# AOP的动态代理技术

## 常用的动态代理技术

### jdk代理：基于接口的动态代理技术

<center><img src="https://i.im5i.com/2021/06/07/qYX38.png" alt="qYX38.png" border="0" /></center>

目标类接口：

```java
// 目标类接口
public interface TargetInterface {  
    public void save();
}
```

目标类：

```java
// 目标类
public class Target implements TargetInterface {
    @Override
    public void save() {
        System.out.println("save running.....");
    }
}
```



动态代理类：

```java
public class ProxyTest {
    public static void main(String[] args) {

        //目标对象
        final Target target = new Target();

        //增强对象
        final Advice advice = new Advice();

        //返回值 就是动态生成的代理对象
        TargetInterface proxy = (TargetInterface) Proxy.newProxyInstance(
                target.getClass().getClassLoader(), //目标对象类加载器
                target.getClass().getInterfaces(), //目标对象相同的接口字节码对象数组
                new InvocationHandler() {
                    //调用代理对象的任何方法  实质执行的都是invoke方法
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        advice.before(); //前置增强
                        Object invoke = method.invoke(target, args);//执行目标方法
                        advice.afterReturning(); //后置增强
                        return invoke;
                    }
                }
        );
        //调用代理对象的方法
        proxy.save();
    }
}
```

增强类：

```java
public class Advice {
    public void before(){
        System.out.println("前置增强....");
    }

    public void afterReturning(){
        System.out.println("后置增强....");
    }
}
```

`ProxyTest`测试输出：

```java
前置增强....
save running.....
后置增强....

Process finished with exit code 0
```



### cglib代理：基于父类的动态代理技术

<center><img src="https://i.im5i.com/2021/06/07/qY2kU.png" alt="qY2kU.png" border="0" /></center>

目标类：

```java
public class Target {
    public void save() {
        System.out.println("save running.....");
    }
}
```

动态代理代码：

```java
import com.itheima.proxy.jdk.TargetInterface;
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ProxyTest {

    public static void main(String[] args) {

        //目标对象
        final Target target = new Target();

        //增强对象
        final Advice advice = new Advice();

        //返回值 就是动态生成的代理对象  基于cglib
        //1、创建增强器
        Enhancer enhancer = new Enhancer();
        //2、设置父类（目标）
        enhancer.setSuperclass(Target.class);
        //3、设置回调
        enhancer.setCallback(new MethodInterceptor() {
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                advice.before(); //执行前置
                Object invoke = method.invoke(target, args);//执行目标
                advice.afterReturning(); //执行后置
                return invoke;
            }
        });
        //4、创建代理对象
        Target proxy = (Target) enhancer.create();
        proxy.save();
    }
}
```

`ProxyTest`测试输出：

```java
前置增强....
save running.....
后置增强....

Process finished with exit code 0
```

# AOP相关概念

Spring 的 AOP 实现底层就是对上面的动态代理的代码进行了封装，封装后只需要对需要关注的部分进行代码编写，并通过配置的方式完成指定目标的方法增强。

AOP常用术语如下：

- Target（目标对象）：代理的目标对象

- Proxy （代理）：一个类被 AOP 织入增强后，就产生一个结果代理类

- Joinpoint（连接点）：所谓连接点是指那些被拦截到的点。在spring中，这些点指的是方法，因为spring只支持方法类型的连接点

-  Pointcut（切入点）：所谓切入点是指要对哪些 Joinpoint 进行拦截的定义

- Advice（通知/ 增强）：所谓通知是指拦截到 Joinpoint 之后所要做的事情就是通知

- Aspect（切面）：是切入点和通知（引介）的结合

- Weaving（织入）：是指把增强应用到目标对象来创建新的代理对象的过程。spring采用动态代理织入，而AspectJ采用编译期织入和类装载期织入

# AOP开发明确的事项

1. 需要编写的内容

- 编写核心业务代码（目标类的目标方法）

- 编写切面类，切面类中有通知(增强功能方法)

- 在配置文件中，配置织入关系，即将哪些通知与哪些连接点进行结合

2. AOP技术实现的内容

- Spring 框架监控切入点方法的执行。一旦监控到切入点方法被运行，使用代理机制，动态创建目标对象的代理对象，根据通知类别，在代理对象的对应位置，将通知对应的功能织入，完成完整的代码逻辑运行。

3. AOP底层使用哪种代理方式

- 在 spring 中，框架会根据目标类是否实现了接口来决定采用哪种动态代理的方式

# 总结

- aop：面向切面编程

- aop底层实现：基于JDK的动态代理 和 基于Cglib的动态代理

- aop的重点概念：
  - Pointcut（切入点）：被增强的方法
  - Advice（通知/ 增强）：封装增强业务逻辑的方法
  - Aspect（切面）：切点+通知
  - Weaving（织入）：将切点与通知结合的过程

- 开发明确事项：
  - 谁是切点（切点表达式配置）
  - 谁是通知（切面类中的增强方法）
  - 将切点和通知进行织入配置

# Reference

- [黑马程序员SSM框架教程](https://www.bilibili.com/video/BV1WZ4y1P7Bp?p=125&spm_id_from=pageDriver)