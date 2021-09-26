# 1 前言

Bean生命周期描述的是Spring中**一个Bean创建过程和销毁过程**中所经历的步骤，其中Bean创建过程是重点。

Spring预留了很多的接口，程序员可以在Bean实例化前后、初始化前后写入一些逻辑对Bean进行**自定义加工**。

可以简单地将Bean的生命周期分为五个主干流程：

- 容器启动阶段（严格来讲这个不属于Bean的生命周期）
- Bean（单例非懒加载）的**实例化阶段**
- Bean的**属性注入阶段**
- Bean的**初始化阶段**
- Bean的**销毁阶段**

# 2 什么是BeanDefinition?

## 2.1 简介

BeanDefinition表示Bean定义，Spring根据BeanDefinition来创建Bean对象，BeanDefinition有很多的属性用来描述Bean，BeanDefinition是Spring中非常核心的概念。

## 2.2 BeanDefinition中重要的属性

| 属性             | 说明                                                         |
| :--------------- | :----------------------------------------------------------- |
| `beanClass`      | 表示一个bean的类型，比如:`UserService.class`、`OrderService.class`，Spring在创建Bean的过程中会根据此属性来实例化得到对象 |
| `scope`          | 表示一个bean的作用域,比如:<br/>scope等于singleton，该bean就是一个单例Bean<br/>scope等于prototype，该bean就是一个原型bean |
| `isLazy`         | 表示一个bean是不是需要懒加载<br/>原型bean的isLazy属性不起作用<br/>懒加载的单例bean，会在第一次`getBean`的时候生成该bean<br/>非懒加载的单例bean，则会在Spring启动过程中直接生成好 |
| `dependsOn`      | 表示一个bean在创建之前所依赖的其他bean，在一个bean创建之前，它所依赖的这些bean得先全部创建好 |
| `primary`        | 表示一个bean是主bean，在Spring中一个类型可以有多个bean对象，在进行依赖注入时，如果根据类型找到了多个bean，此时会判断这些bean中是否存在一个主bean，如果存在，则直接将这个bean注入给属性 |
| `initMethodName` | 表示一个bean的初始化方法，一个bean的生命周期过程中有一个步骤叫初始化，Spring会在这个步骤中去调用bean的初始化方法，初始化逻辑由程序员自己控制，表示程序员可以自定义逻辑对bean进行加工 |

> @Component、@Controller、@Service、@Repository、@Bean、\<bean/>这些都会解析为BeanDefinition 

# 3 什么是BeanFactory?

## 3.1 简介

BeanFactory是一种“Spring容器”，BeanFactory翻译过来就是Bean工厂，顾名思义，它可以用来创建Bean、获取Bean，BeanFactory是Spring中非常核心的组件。

## 3.2 BeanDefinition、BeanFactory、Bean对象

<span style="color:red">BeanFactory将利用BeanDefinition来生成Bean对象，BeanDefinition相当于BeanFactory的原材料，Bean对象就相当于BeanFactory所生产出来的产品</span>。

## 3.3 BeanFactory的核心子接口和实现类

```java
ListableBeanFactory
    
ConfigurableBeanFactory
    
AutowireCapableBeanFactory
    
AbstractBeanFactory
    
DefaultListableBeanFactory    
```

**DefaultListableBeanFactory的功能：**

支持单例Bean、支持Bean别名、支持父子BeanFactory、支持Bean类型转化、支持Bean后置处理、支持FactoryBean、支持自动装配，等等

# 4 什么是FactoryBean?

FactoryBean是Spring所提供的一种较灵活的创建Bean的方式，可以通过实现FactoryBean接口中的`getObject()`方法来返回一个对象，这个对象就是最终的Bean对象。

FactoryBean接口中的方法：

```java
Object getObject(); // 返回的是Bean对象
boolean isSingleton(); // 返回的是否是单例Bean对象
Class getObjectType(); // 返回的是Bean对象的类型
```

```java
@Component("zhouyu")
public class ZhouyuFactoryBean implements FactoryBean {
    @Override
    // Bean对象
    public 0bject get0bject() throws Exception {
    	return new User();
    }
    @Override
   	// Bean对象的类型
    public class<?> get0bjectType() {
    	return User.class;
    }
    @@0verride
    // 所定义的Bean是单例还是原型
    public boolean issingleton() {
    	return true;
    }
}
```

**FactoryBean的特殊点**

上述代码，实际上对应了两个Bean对象:

1、beanName为"zhouyu"，bean对象为`getObject`方法所返回的User对象。

2、beanName为"&zhouyu"，bean对象为ZhouyuFactoryBean类的实例对象。

FactoryBean对象本身也是一个Bean，同时它相当于一个小型工厂，可以生产出另外的Bean。

BeanFactory是一个Spring容器，是一个大型工厂，它可以生产出各种各样的Bean。

FactoryBean机制被广泛的应用在Spring内部和Spring与第三方框架或组件的整合过程中。

# 5 Bean的生命周期

1. 根据`BeanDefinition`信息，实例化对象，通过构造方法反射得到一个实例化对象

2. 根据`BeanDefinition`信息，配置Bean的所有属性（将bean的引用注入到bean对应的属性，*可能存在循环依赖问题）

3. 如果Bean实现了`BeanNameAware`接口，工厂调用Bean的`setBeanName`，参数为Bean的Id

4. 如果Bean实现`BeanFactoryAware`接口，Spring将调用`setBeanFactory()`方法，将BeanFactory容器实例传入

5. 如果Bean实现`ApplicationContextAware`接口，Spring将调用`setApplicationContext()`方法，将bean所在的应用上下文的引用传入进来

6. 如果存在类实现了`BeanPostProcessor`接口，执行这些实现类的`postProcessBeforeInitialization`方法，这相当于在Bean初始化之前插入逻辑  

7. 如果Bean实现`InitializingBean`接口， 执行`afterPropertiesSet`方法

8. 如果Bean指定了`init-method`方法，就会调用该方法。例：`<bean init-method="init"> ` 

9. 如果存在类实现了`BeanPostProcessor`接口，执行这些实现类的`postProcessAfterInitialization`方法，这相当于在Bean初始化之后插入逻辑 
10. 这个阶段Bean已经可以使用了，scope为singleton的Bean会被缓存在IOC容器中  
11. 如果Bean实现了`DisposableBean`接口， 在容器销毁的时候执行`destroy`方法
12. 如果配置了`destory-method`方法，就调用该方法。例：`<bean destroy-method="customerDestroy">` 

<center><img src="https://ss.im5i.com/2021/09/25/lJtF3.jpg" alt="lJtF3.jpg" border="0" /></center>

# 6 验证

通过一个实际的例子验证上图

1. 首先定义一个业务Bean，实现了诸多扩展接口

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;

public class Car implements BeanFactoryAware, BeanNameAware,
        InitializingBean, DisposableBean {
    private String carName;
    private BeanFactory beanFactory;
    private String beanName;

    public Car() {
        System.out.println("Car-bean的构造函数");
    }
    public void setCarName(String carName) {
        System.out.println("Car-bean属性注入");
        this.carName = carName;
    }

    /**
     * BeanFactoryAware接口方法
     * @param beanFactory
     * @throws BeansException
     */
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("BeanFactoryAware#setBeanFactory()");
        this.beanFactory = beanFactory;
    }

    /**
     * BeanNameAware接口方法
     * @param s
     */
    @Override
    public void setBeanName(String s) {
        System.out.println("BeanNameAware#setBeanName()");
        this.beanName = s;
    }

    /**
     * DisposableBean接口方法
     * @throws Exception
     */
    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean#destroy()");
    }

    /**
     * InitializingBean接口方法
     * @throws Exception
     */
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean#afterPropertiesSet()");
    }

    public void carInit() {
        System.out.println("<bean>的init-method属性指定的初始化方法");
    }

    public void carDestory() {
        System.out.println("<bean>的destroy-method属性指定的初始化方法");
    }
}
```



2. 自定义一个特殊的`BeanFactoryPostProcessor`类型的Bean

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;

public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    public MyBeanFactoryPostProcessor() {
        super();
        System.out.println("MyBeanFactoryPostProcessor构造函数");
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out.println("MyBeanFactoryPostProcessor#postProcessBeanFactory()");
    }
}
```



3. 自定义一个特殊的`InstantiationAwareBeanPostProcessor`类型的Bean

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.PropertyValues;
import org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessorAdapter;

import java.beans.PropertyDescriptor;

public class MyInstantiationAwareBeanPostProcessor extends InstantiationAwareBeanPostProcessorAdapter {
    public MyInstantiationAwareBeanPostProcessor() {
        super();
        System.out.println("MyInstantiationAwareBeanPostProcessor构造函数");
    }

    /**
     * 接口方法、实例化Bean之前调用
     * @param beanClass
     * @param beanName
     * @return
     * @throws BeansException
     */
    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if ("car".equals(beanName)) {
            System.out.println("MyInstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation()");
        }
        return null;
    }

    /**
     * 接口方法、实例化Bean之后调用
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if ("car".equals(beanName)) {
            System.out.println("MyInstantiationAwareBeanPostProcessor#postProcessAfterInstantiation()");
        }
        return true;
    }

    /**
     * 接口方法、设置某个属性时调用
     * @param pvs
     * @param pds
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {
        if ("car".equals(beanName)) {
            System.out.println("MyInstantiationAwareBeanPostProcessor#postProcessPropertyValues()");
        }
        return pvs;
    }
}
```



4. 自定义一个特殊的`BeanPostProcessor`类型的Bean

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class MyBeanPostProcessor implements BeanPostProcessor {
    public MyBeanPostProcessor() {
        super();
        System.out.println("MyBeanPostProcessor构造函数");
    }

    @Override
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
        //可以针对指定的Bean做一些操作
        if ("car".equals(s)) {
            System.out.println("MyBeanPostProcessor#postProcessBeforeInitialization()");
        }
        return o;
    }

    @Override
    public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        if ("car".equals(s)) {
            System.out.println("MyBeanPostProcessor#postProcessAfterInitialization()");
        }
        return o;
    }
}
```

4. 将上述1个业务Bean和3个特殊的Bean配置到xml中

```java
    <bean id="beanPostProcessor" class="IoC.beanLifeCycle.MyBeanPostProcessor"/>
    <bean id = "instantiationAwareBeanPostProcessor" class="IoC.beanLifeCycle.MyInstantiationAwareBeanPostProcessor"/>
    <bean id="beanFactoryPostProcessor" class="IoC.beanLifeCycle.MyBeanFactoryPostProcessor"/>
    <bean id="car" class="IoC.beanLifeCycle.Car" init-method="carInit" destroy-method="carDestory"
          scope="singleton" />
```

5. 下面来见证下整个流程

```java
public class Main {
    public static void main(String[] args) {
        String xmlPath = "IoC/beanLifeCycle/applicationContext.xml";
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(xmlPath);
        Car car = context.getBean("car", Car.class);
        System.out.println("car: " + car);
        context.destroy();
    }
}
```

6. 控制台输出如下

```java
MyBeanFactoryPostProcessor构造函数
MyBeanFactoryPostProcessor#postProcessBeanFactory()
MyBeanPostProcessor构造函数
MyInstantiationAwareBeanPostProcessor构造函数
MyInstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation()
Car-bean的构造函数
MyInstantiationAwareBeanPostProcessor#postProcessAfterInstantiation()
MyInstantiationAwareBeanPostProcessor#postProcessPropertyValues()
BeanNameAware.setBeanName()
BeanFactoryAware.setBeanFactory()
MyBeanPostProcessor#postProcessBeforeInitialization()
InitializingBean.afterPropertiesSet()
<bean>的init-method属性指定的初始化方法
MyBeanPostProcessor#postProcessAfterInitialization()
car: IoC.beanLifeCycle.Car@d2cc05a
DisposableBean.destroy()
<bean>的destroy-method属性指定的初始化方法
```

