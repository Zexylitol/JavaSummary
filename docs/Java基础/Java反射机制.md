<!-- GFM-TOC -->

- [1. Java反射机制概述](#_1-java反射机制概述)
- [★2. 获取Class实例](#_2-获取class实例)
- [3. 类的加载与ClassLoader的理解](#_3-类的加载与classloader的理解)
- [★4. 创建运行时类的对象](#_4-创建运行时类的对象)
- [5. 获取运行时类的完整结构](#_5-获取运行时类的完整结构)
  - [5.1 获取运行时类的属性](#_51-获取运行时类的属性)
  - [5.2 获取运行时类的方法](#_52-获取运行时类的方法)
  - [5.3 获取运行时类的构造器](#_53-获取运行时类的构造器)
  - [5.4 获取运行时类的父类及父类的泛型](#_54-获取运行时类的父类及父类的泛型)
  - [5.5 获取运行时类实现的接口](#_55-获取运行时类实现的接口)
  - [5.6 获取运行时类所在的包](#_56-获取运行时类所在的包)
  - [5.7 获取运行时类声明的注解](#_57-获取运行时类声明的注解)
  - [5.8 小结](#_58-小结)
- [★6. 调用运行时类的指定结构](#_6-调用运行时类的指定结构)
  - [6.1 调用运行时类中的指定属性](#_61-调用运行时类中的指定属性)
  - [6.2 调用运行时类中的指定方法](#_62-调用运行时类中的指定方法)
  - [6.3 调用运行时类中的指定构造器](#_63-调用运行时类中的指定构造器)
  - [6.4 小结](#_64-小结)
- [7. 反射的应用：动态代理](#_7-反射的应用：动态代理)
  - [7.1 静态代理举例](#_71-静态代理举例)
  - [7.2 动态代理举例](#_72-动态代理举例)
  - [7.3 动态代理与AOP](#_73-动态代理与aop)
- [Reference](#Reference)

<!-- GFM-TOC -->

# 1. Java反射机制概述

<center><img src="https://i.loli.net/2021/02/27/RKF5Jc7ZpdgXAEU.png"/></center>

<center><img src="https://i.loli.net/2021/02/27/aDbL2trOHMhmxJE.png"/></center>

<center><img src="https://i.loli.net/2021/02/27/PmVqHoTDZpEvBRL.png"/></center>

<center><img src="https://i.loli.net/2021/02/27/meG6OdEzsQ3jxfa.png"/></center>

# 2. 获取Class实例

- java.lang.Class：用来描述类的类
- E:\Java\javaWorkSpace\Reflection\src\com\atgui\java\ReflectionTest.java

> 疑问1：通过直接new的方式或反射的方式都可以调用公共的结构，开发中到底用哪个？
>
> 答：建议直接new的方式。
>
> ​		什么时候会使用反射的方式：**反射的特征---动态性**
>
> 疑问2：反射机制与面向对象中的封装性是不是矛盾的？如何看待这两个技术？
>
> 答：不矛盾。
>
> 
>
> 关于java.lang.Class类的理解：
>
> ​	<font color="red">Class的实例就对应着一个运行时类</font>
>
> ​     <center><img src="https://i.loli.net/2021/02/28/V1HdCMqwWK92QtZ.png"/></center>
>
> 3. 加载到内存中的运行时类，会缓存一定的时间。在此时间之内，可以通过不同的方式来**获取此运行时类**。
>
>    ```java
>    /**
>         * 获取Class的实例的方式
>         */
>    @Test
>    public void test3() throws Exception {
>        // 方式一:调用运行时类的属性：.class
>        Class<Person> clazz1 = Person.class;
>        System.out.println(clazz1);
>    
>        // 方式二：通过运行时类的对象，调用getClass()
>        Person p1 = new Person();
>        Class<? extends Person> clazz2 = p1.getClass();
>        System.out.println(clazz2);
>    
>        // 方式三：调用Class的静态方法：forName(String classPath)
>        Class<?> clazz3 = Class.forName("com.atgui.java.Person");
>        //        clazz3 = Class.forName("java.lang.String");
>        System.out.println(clazz3);
>    
>        System.out.println(clazz1 == clazz2);  // true  指向同一个运行时类，使用不同的方式获取运行时类
>        System.out.println(clazz1 == clazz3);  // true
>    
>        // 方式四：使用类的加载器：ClassLoader
>        ClassLoader classLoader = ReflectionTest.class.getClassLoader();
>        Class<?> clazz4 = classLoader.loadClass("com.atgui.java.Person");
>        System.out.println(clazz4);
>        System.out.println(clazz1 == clazz4);  // true
>    }
>    ```
>
>    

* 获取Class对象的方式：
	1. `Class.forName("全类名")`：将字节码文件加载进内存，返回Class对象
		* 多用于配置文件，将类名定义在配置文件中。读取文件，加载类
	2. `类名.class`：通过类名的属性class获取
		* 多用于参数的传递
	3. `对象.getClass()`：getClass()方法在Object类中定义着。
		* 多用于对象的获取字节码的方式

	* 结论：
		同一个字节码文件(*.class)在一次程序运行过程中，只会被加载一次，不论通过哪一种方式获取的Class对象都是同一个。

<center><img src="https://i.loli.net/2021/03/04/u3yhBZ7bwpFTd9L.png"/></center>

<center><img src="https://i.loli.net/2021/03/04/nzuJGeW17ALfXwd.png"/></center>

# 3. 类的加载与ClassLoader的理解

<center><img src="https://i.loli.net/2021/03/04/1YaunOPsJvdHNV6.png"/></center>

<center><img src="https://i.loli.net/2021/03/04/oLaxWi3EjO2c74h.png"/></center>

类的加载：

- 加载
- 链接
  - 验证
  - 准备
  - 解析
- 初始化

<center><img src="https://i.loli.net/2021/03/04/kCZDTLFy3jSOngN.png"/></center>

<center><img src="https://i.loli.net/2021/03/04/DBH2UPZNtCiOxu9.png"/></center>

- 一个`.class`文件对应一个类

<center><img src="https://i.loli.net/2021/03/04/29phK7JLWtjD6qr.png"/></center>

# 4. 创建运行时类的对象

- 运行时类：加载进内存中的类

```java
import org.junit.Test;

import java.util.Random;

/**
 * 通过反射创建对应的运行时类的对象
 * @create 2021-04-22 17:20
 */
public class NewInstanceTest {
    @Test
    public void test1() throws IllegalAccessException, InstantiationException {
        Class<Person> personClass = Person.class;
        // newInstance() : 调用此方法，创建对应的运行时类的对象，内部调用了运行时类的空参构造器
        // 要想此方法正常的创建运行时类的对象，要求：
        // 1. 运行时类必须提供空参的构造器
        // 2. 提供满足条件的空参构造器的权限修饰符（通常设置为public）
        // 在 javabean 中要求提供一个public的空参构造器。原因：
        // 1. 便于通过反射，创建运行时类的对象
        // 2. 便于子类继承此运行时类时，默认调用super()时，保证父类有此构造器
        Person person = personClass.newInstance();
        System.out.println(person);
    }

    /**
     * 体会反射的动态性
     * @throws Exception
     */
    @Test
    public void test2() throws Exception {
        int num = new Random().nextInt(3); // 0 1 2
        String classPath = "";
        System.out.println("num : " + num);
        switch (num) {
            case 0 :
                classPath = "java.util.Date";
                break;
            case 1:
                classPath = "java.lang.Object";
                break;
            case 2 :
                classPath = "com.atguigu.java.Person";
                break;
            default :
                break;
        }

        Object instance = getInstance(classPath);
        System.out.println(instance);
    }

    /**
     * 创建一个指定类的对象
     * @param classPath 指定类的全类名
     * @return
     * @throws Exception
     */
    public Object getInstance(String classPath) throws Exception {
        Class<?> aClass = Class.forName(classPath);
        return aClass.newInstance();
    }
}
```

# 5. 获取运行时类的完整结构

## 5.1 获取运行时类的属性

```java
@Test
public void test1() {
    Class<Person> personClass = Person.class;
    // 获取属性结构
    // getFields()：获取当前运行时类及其父类中声明为public属性
    Field[] fields = personClass.getFields();
    System.out.println("getFields: ");
    for (Field f : fields) {
        System.out.println(f);
    }

    // getDeclaredFields()：获取当前运行时类中声明的所有属性（不包括父类中声明的属性）
    Field[] declaredFields = personClass.getDeclaredFields();
    System.out.println("getDeclaredFields: ");
    for (Field f : declaredFields) {
        System.out.println(f);
    }
    // 权限修饰符 数据类型 变量名
    for (Field f : declaredFields) {
        // 1. 权限修饰符
        int modifier = f.getModifiers();
        System.out.print(Modifier.toString(modifier) + '\t');
        // 2. 数据类型
        Class<?> type = f.getType();
        System.out.print(type.getName() + '\t');
        // 3. 变量名
        String name = f.getName();
        System.out.print(name);
        System.out.println();
    }
}
```

## 5.2 获取运行时类的方法

```java
@Test
public void test1() {
    Class<Person> personClass = Person.class;

    // getMethods(): 获取当前运行时类及其所有父类中声明为public权限的方法
    Method[] methods = personClass.getMethods();
    for (Method m : methods) {
        System.out.println(m);
    }

    // getDeclaredMethods(): 获取当前运行时类中声明的所有方法（不包含父类中声明的方法）
    Method[] declaredMethods = personClass.getDeclaredMethods();
    for (Method m : declaredMethods) {
        System.out.println(m);
    }

    /**
         * @Xxx
         * 权限修饰符 返回值类型 方法名（参数类型列表） throws XxxException{}
         */
    for (Method m : declaredMethods) {
        // 1. 获取方法声明的注解
        Annotation[] annotations = m.getAnnotations();
        for (Annotation a : annotations) {
            System.out.println(a);
        }

        // 2. 权限修饰符
        System.out.print(Modifier.toString(m.getModifiers()) + '\t');

        // 3. 返回值类型
        System.out.print(m.getReturnType().getName() + '\t');

        // 4. 方法名
        System.out.print(m.getName() + '\t');

        // 5. 形参列表
        System.out.print('(');
        Class<?>[] parameterTypes = m.getParameterTypes();
        if (!(parameterTypes == null && parameterTypes.length == 0)) {
            for (int i = 0; i < parameterTypes.length; i++) {
                if (i == parameterTypes.length - 1) {
                    System.out.print(parameterTypes[i].getName() + " args_" + i);
                    break;
                }
                System.out.print(parameterTypes[i].getName() + " args_" + i + ",");
            }
        }
        System.out.print(')');

        // 6. 抛出的异常
        Class<?>[] exceptionTypes = m.getExceptionTypes();
        if (!(exceptionTypes == null && exceptionTypes.length==0)) {
            System.out.print("throws ");
            for (int i = 0; i < exceptionTypes.length; i++) {
                if (i == exceptionTypes.length-1) {
                    System.out.print(exceptionTypes[i].getName());
                    break;
                }
                System.out.print(exceptionTypes[i].getName() + ",");
            }
        }

        System.out.println();
    }
}
```

## 5.3 获取运行时类的构造器

```java
@Test
public void test1() {
    Class<Person> personClass = Person.class;
    // getConstructors(): 获取当前运行时类中声明为public的构造器
    Constructor<?>[] constructors = personClass.getConstructors();
    for (Constructor c :constructors) {
        System.out.println(c);
    }

    // getDeclaredConstructors(): 获取当前运行时类中声明的所有的构造器
    Constructor<?>[] declaredConstructors = personClass.getDeclaredConstructors();
    for (Constructor c : declaredConstructors) {
        System.out.println(c);
    }
}
```

## 5.4 获取运行时类的父类及父类的泛型

```java
@Test
public void test1() {
    Class<Person> personClass = Person.class;

    Class<? super Person> superclass = personClass.getSuperclass();
    System.out.println(superclass);

    // 带泛型的父类
    Type genericSuperclass = personClass.getGenericSuperclass();
    System.out.println(genericSuperclass);

    // 带泛型的父类的泛型
    ParameterizedType parameterizedType = (ParameterizedType) genericSuperclass;
    Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
    for (Type t : actualTypeArguments) {
        System.out.println(t.getTypeName());
    }
}
```

## 5.5 获取运行时类实现的接口

```java
@Test
public void test1() {
    Class<Person> personClass = Person.class;

    Class<?>[] interfaces = personClass.getInterfaces();
    for (Class c : interfaces) {
        System.out.println(c);
    }

    // 获取运行时类的父类实现的接口
    Class<?>[] interfaces1 = personClass.getSuperclass().getInterfaces();
    for (Class c : interfaces1) {
        System.out.println(c);
    }
}
```

## 5.6 获取运行时类所在的包

```java
@Test
public void test1() {
    Class<Person> personClass = Person.class;
    Package aPackage = personClass.getPackage();
    System.out.println(aPackage);
}
```

## 5.7 获取运行时类声明的注解

```java
@Test
public void test1() {
    Class<Person> personClass = Person.class;

    Annotation[] annotations = personClass.getAnnotations();
    for (Annotation a : annotations) {
        System.out.println(a);
    }
}
```

## 5.8 小结

* Class对象功能：
	* 获取功能：
		1. 获取成员变量们
			* Field[] getFields() ：获取所有public修饰的成员变量
			* Field getField(String name)   获取指定名称的 public修饰的成员变量

			* Field[] getDeclaredFields()  获取所有的成员变量，不考虑修饰符
			* Field getDeclaredField(String name)  
		2. 获取构造方法们
			* Constructor<?>[] getConstructors()  
			* Constructor<T> getConstructor(类<?>... parameterTypes)  

			* Constructor<T> getDeclaredConstructor(类<?>... parameterTypes)  
			* Constructor<?>[] getDeclaredConstructors()  
		3. 获取成员方法们：
			* Method[] getMethods()  
			* Method getMethod(String name, 类<?>... parameterTypes)  

			* Method[] getDeclaredMethods()  
			* Method getDeclaredMethod(String name, 类<?>... parameterTypes)  

		4. 获取全类名	
			* String getName()  

# 6. 调用运行时类的指定结构

- 属性、方法、构造器

## 6.1 调用运行时类中的指定属性

```java
@Test
public void test1() throws Exception {
    Class<Person> personClass = Person.class;

    // 创建运行时类的对象
    Person person = personClass.newInstance();

    // 获取指定的属性：要求运行时类中属性声明为public
    // 通常不采用此方式
    // public int id;
    Field id = personClass.getField("id");

    /**
         * 设置当前属性的值
         * set()：参数1：指明设置哪个对象的属性
         *        参数2：将此属性设置为多少
         */
    id.set(person, 1001);

    /**
         * 获取当前属性的值
         * get(): 参数1：获取哪个对象的当前属性值
         */
    int pId = (int)id.get(person);
    System.out.println("pId: " + pId);


    /**
     * 操作运行时类中的指定的属性（☆推荐☆）
     */
    //1. getDeclaredField(String name): 获取运行时类中指定变量名的属性
    // private String name;
    Field name = personClass.getDeclaredField("name");
    //2. 保证当前属性是可访问的
    name.setAccessible(true);
    //3. 获取、设置指定对象的此属性值
    name.set(person, "Tom");
    System.out.println(name.get(person));
}
```

## 6.2 调用运行时类中的指定方法

```java
@Test
public void test2() throws Exception {
    Class<Person> personClass = Person.class;

    // 创建运行时类的对象
    Person person = personClass.newInstance();

    /**
         * 1. 获取指定的某个方法
         * private String show(String nation) {}
         * getDeclaredMethod(): 参数1： 指明获取的方法的名称
         *                      参数2： 指明获取的方法的形参列表
         */
    Method show = personClass.getDeclaredMethod("show", String.class);

    /**
         * 2. 保证当前方法是可访问的
         */
    show.setAccessible(true);

    /**
         * 3. 调用invoke()
         * invoke(): 参数1：方法的调用者
         *           参数2：给方法形参赋值的实参
         * invoke()的返回值即为对应类中调用的方法的返回值
         */
    String china = (String)show.invoke(person, "China");
    System.out.println(china);

    /*********************************调用静态方法**********************************/
    // private static void showDesc() {}
    Method showDesc = personClass.getDeclaredMethod("showDesc");
    showDesc.setAccessible(true);
    // 没有返回值则返回null
    Object invoke = showDesc.invoke(personClass);
    // 输出为null
    System.out.println(invoke);
    // 对于静态方法，参数1可以为null
    Object invoke1 = showDesc.invoke(null);
}
```

## 6.3 调用运行时类中的指定构造器

```java
@Test
public void test3() throws Exception {
    Class<Person> personClass = Person.class;

    // 创建运行时类的对象
    Person person = personClass.newInstance();

    // private Person(String name) {}
    /**
         * 1. 获取指定的构造器
         * getDeclaredConstructor(): 参数：指明构造器的参数列表
         */
    Constructor<Person> declaredConstructor = personClass.getDeclaredConstructor(String.class);

    /**
         * 2. 保证此构造器是可访问的
         */
    declaredConstructor.setAccessible(true);

    /**
         * 3. 调用此构造器创建运行时类的对象
         */
    Person tom = declaredConstructor.newInstance("Tom");
    System.out.println(tom);
}
```

## 6.4 小结

* Field：成员变量
	* 操作：
		1. 设置值
			* void set(Object obj, Object value)  
		2. 获取值
			* get(Object obj) 

		3. 忽略访问权限修饰符的安全检查
			* setAccessible(true):暴力反射

* Constructor:构造方法
	* 创建对象：
		* T newInstance(Object... initargs)  
* 如果使用空参数构造方法创建对象，操作可以简化：Class对象的newInstance方法

* Method：方法对象
	* 执行方法：
		* Object invoke(Object obj, Object... args)  

	* 获取方法名称：
		* String getName:获取方法名

# 7. 反射的应用：动态代理

<center><img src="https://i.loli.net/2021/04/23/sJjXZ6CcaDwSqHr.png"/></center>

<center><img src="https://i.loli.net/2021/04/25/AjPxiJMUHtvG1TZ.png"/></center>

## 7.1 静态代理举例

```java
/**
 * 静态代理举例
 * 特点：代理类和被代理类在编译期间，就确定下来了
 * @create 2021-04-25 15:58
 */

interface ClothFactory {
    void produceCloth();
}

class ProxyClothFactory implements ClothFactory {
    // 用被代理类对象进行实例化
    private ClothFactory factory;

    public ProxyClothFactory() {
    }

    public ProxyClothFactory(ClothFactory factory) {
        this.factory = factory;
    }

    @Override
    public void produceCloth() {
        System.out.println("代理工厂做一些准备工作");

        factory.produceCloth();

        System.out.println("代理工厂做一些后续的收尾工作");
    }
}

class NikeClothFactory implements ClothFactory {
    @Override
    public void produceCloth() {
        System.out.println("Nike工厂生产运动服");
    }
}

public class StaticProxyTest {
    public static void main(String[] args) {
        // 创建被代理类的对象
        NikeClothFactory nikeClothFactory = new NikeClothFactory();
        // 创建代理类的对象
        ProxyClothFactory proxyClothFactory = new ProxyClothFactory(nikeClothFactory);

        proxyClothFactory.produceCloth();
    }
}
```

## 7.2 动态代理举例

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * 动态代理举例
 * @create 2021-04-25 16:08
 */

interface Human {
    String getBelief();

    void eat(String food);
}

// 被代理类
class SuperMan implements Human {
    @Override
    public String getBelief() {
        return "I believe I can fly!";
    }

    @Override
    public void eat(String food) {
        System.out.println("I like eating " + food + " !");
    }
}

/**
 * 要想实现动态代理，需要解决的问题？
 * 问题一：如何根据加载到内存中的被代理类，动态的创建一个代理类及其对象
 * 问题二：当通过代理类的对象调用方法时，如何动态的去调用被代理类中的同名方法
 */
class ProxyFactory {
    /**
     * 调用此方法，返回一个代理类的对象（解决问题一）
     * @param obj ： 被代理类的对象
     * @return
     */
    public static Object getProxyInstance(Object obj) {
        MyInvocationHandler handler = new MyInvocationHandler();
        handler.bind(obj);
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), handler);
    }
}

class MyInvocationHandler implements InvocationHandler {

    // 需要使用被代理类的对象进行赋值
    private Object obj;

    public void bind(Object obj) {
        this.obj = obj;
    }

    /**
     * 当通过代理类的对象，调用方法a时，就会自动调用如下方法：invoke()
     * 将被代理类要执行的方法a的功能声明在invoke()中
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // method：即为代理类对象调用的方法，此方法也就作为了被代理类对象要调用的方法
        // obj: 被代理类的对象
        Object invoke = method.invoke(obj, args);
        // 上述方法的返回值就作为当前类中的invoke()的返回值
        return invoke;
    }
}

public class ProxyTest {
    public static void main(String[] args) {
        SuperMan superMan = new SuperMan();
        // proxyInstance: 代理类的对象
        Human proxyInstance = (Human) ProxyFactory.getProxyInstance(superMan);
        // 当通过代理类对象调用方法时，会自动的调用被代理类中同名的方法
        String belief = proxyInstance.getBelief();
        System.out.println(belief);
        proxyInstance.eat("Hotpot");

        /****************************************************************/
        NikeClothFactory nikeClothFactory = new NikeClothFactory();
        ClothFactory proxyClothFactory = (ClothFactory) ProxyFactory.getProxyInstance(nikeClothFactory);
        proxyClothFactory.produceCloth();
    }
}
```

## 7.3 动态代理与AOP

<center><img src="https://i.loli.net/2021/04/25/HEUCgyX6f9qL1Mb.png"/></center>

<center><img src="https://i.loli.net/2021/04/25/omswEpPKRNIcLqu.png"/></center>

<center><img src="https://i.loli.net/2021/04/25/rjLKagod46TQ5e8.png"/></center>

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * 动态代理举例
 * @create 2021-04-25 16:08
 */

interface Human {
    String getBelief();

    void eat(String food);
}

// 被代理类
class SuperMan implements Human {
    @Override
    public String getBelief() {
        return "I believe I can fly!";
    }

    @Override
    public void eat(String food) {
        System.out.println("I like eating " + food + " !");
    }
}

class HumanUtil {
    public void method1() {
        System.out.println("================通用method1================");
    }

    public void method2() {
        System.out.println("================通用method2================");
    }
}

/**
 * 要想实现动态代理，需要解决的问题？
 * 问题一：如何根据加载到内存中的被代理类，动态的创建一个代理类及其对象
 * 问题二：当通过代理类的对象调用方法时，如何动态的去调用被代理类中的同名方法
 */
class ProxyFactory {
    /**
     * 调用此方法，返回一个代理类的对象（解决问题一）
     * @param obj ： 被代理类的对象
     * @return
     */
    public static Object getProxyInstance(Object obj) {
        MyInvocationHandler handler = new MyInvocationHandler();
        handler.bind(obj);
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), handler);
    }
}

class MyInvocationHandler implements InvocationHandler {

    // 需要使用被代理类的对象进行赋值
    private Object obj;

    public void bind(Object obj) {
        this.obj = obj;
    }

    /**
     * 当通过代理类的对象，调用方法a时，就会自动调用如下方法：invoke()
     * 将被代理类要执行的方法a的功能声明在invoke()中
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        HumanUtil util = new HumanUtil();
        util.method1();

        // method：即为代理类对象调用的方法，此方法也就作为了被代理类对象要调用的方法
        // obj: 被代理类的对象
        Object invoke = method.invoke(obj, args);

        util.method2();

        // 上述方法的返回值就作为当前类中的invoke()的返回值
        return invoke;
    }
}

public class ProxyTest {
    public static void main(String[] args) {
        SuperMan superMan = new SuperMan();
        // proxyInstance: 代理类的对象
        Human proxyInstance = (Human) ProxyFactory.getProxyInstance(superMan);
        // 当通过代理类对象调用方法时，会自动的调用被代理类中同名的方法
        String belief = proxyInstance.getBelief();
        System.out.println(belief);
        proxyInstance.eat("Hotpot");

        /****************************************************************/
        NikeClothFactory nikeClothFactory = new NikeClothFactory();
        ClothFactory proxyClothFactory = (ClothFactory) ProxyFactory.getProxyInstance(nikeClothFactory);
        proxyClothFactory.produceCloth();
    }
}
```

输出：

```java
================通用method1================
================通用method2================
I believe I can fly!
================通用method1================
I like eating Hotpot !
================通用method2================
================通用method1================
Nike工厂生产运动服
================通用method2================
```

# Reference

- [尚硅谷Java教程](https://www.bilibili.com/video/BV1Kb411W75N?p=636)
- [黑马程序员JavaWeb教程](https://www.bilibili.com/video/BV1qv4y1o79t?p=6)










