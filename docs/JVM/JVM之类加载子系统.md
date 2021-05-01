@[TOC](文章目录)


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">



# 1. 类加载器子系统的作用

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305213219862.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


- 类加载器子系统负责从文件系统或者网络中加载Class文件，class文件在文件开头有特定的文件标识(cafe)

- ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定

- 加载的类信息存放于一块称为方法区的内存空间。除了类的信息外，方法区中还会存放运行时常量池信息，可能还包括字符串字面量和数字常量(这部分常量信息是Class文件中常量池部分的内存映射)

# 2. 类加载过程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305213235935.png#pic_center)


<table>
    <tr>
    	<th>类的加载过程</th>
        <th colspan="2">作用</th>
    </tr>
    <tr>
        <td>加载(Loading)</td>
        <td colspan="2" align="left">1. 将class文件字节码内容加载到内存中<br/>2. 并将这些静态数据转换成方法区的运行时数据结构<br/>3. <font color="red">在堆中生成一个代表这个类的java.lang.Class对象</font>,作为方法区这个类的各种数据的访问入口</td>        
    </tr>
    <tr>
        <td rowspan="3">链接(Linking)</td>
        <td>验证(Verify)</td>
        <td align="left">1. 确保加载的类信息符合JVM规范，例如以cafe开头，保证被加载类的正确性，不会危害虚拟机自身安全<br/>2. 主要包括四种验证：文件格式验证，元数据验证，字节码验证，符号引用验证</td>
    </tr>
    <tr>
        <td>准备(Prepare)</td>
        <td align="left">1. 正式为类变量(static)分配内存<font color="red">并设置类变量的默认初始值</font>，这些内存都将在方法区中进行分配<br/>2. 补充：<font color="red">这里不包含用final修饰的static，因为final在编译的时候就会分配了，准备阶段会显式初始化</font><br/>3. 补充：<font color="red">这里不会为实例变量分配初始化</font>，类变量会分配在方法区而实例变量是会随着对象一起分配到Java堆中</td>
    </tr>
    <tr>
    	<td>解析(Resolve)</td>
        <td align="left">1. 将常量池内的符号引用转换为直接引用的过程<br/>2. 符号引用就是一组符号来描述所引用的目标<br/>3. 直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄</td>
    </tr>
    <tr>
    	<td>初始化(Initialization)</td>
        <td colspan="2" align="left">1. <font color="red">初始化阶段就是执行类构造器方法 &lt;clinit&gt;()的过程，类构造器方法是由javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来的，构造器方法中指令按语句在源文件中出现的顺序执行</font><br/>2. <font color="red"> &lt;clinit&gt;()不同于类的构造器</font>，类构造器是构造类信息的，不是构造该类对象的构造器<br/>3. 若该类具有父类，JVM会保证子类的 &lt;clinit&gt;()执行前，父类的 &lt;clinit&gt;()已经执行完毕<br/>4. 虚拟机会保证一个类的 &lt;clinit&gt;()方法在多线程环境中被正确加锁和同步</td>
    </tr>

# 3. 类加载器的分类

- <font color="red">类加载器的作用是用来把类(class)装载进内存</font>，并将这些静态数据<font color="blue">转换成方法区的运行时数据结构</font>，然后在堆中生成一个代表这个类的java.lang.Class对象，作为方法区中类数据的访问入口
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305213300457.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


<table>
    <tr>
    	<td rowspan="3">虚拟机自带的加载器</td>
        <td>启动类加载器(引导类加载器，BootStrap ClassLoader)</td>
        <td align="left">1. 使用C/C++语言实现，嵌套在JVM内部<br/>2. <font color="red">负责加载Java的核心类库</font>(JAVA_HOME/jre/lib/rt.jar、resources.jar或sun.boot.class.path路径下的内容)<br/>3. 并不继承自java.lang.ClassLoader，没有父加载器，该加载器无法直接获取</td>
    </tr>
    <tr>
    	<td>扩展类加载器(Extension ClassLoader)</td>
        <td align="left">1. Java语言编写<br/>2. 派生于ClassLoader类<br/>3. 父类加载器为启动类加载器<br/>4. 负责jre/lib/ext目录下的jar包或java.ext.dirs指定目录下的jar包下加载类库</td>
    </tr>
    <tr>
        <td>应用程序类加载器(系统加载器，AppClassLoader)</td>
        <td align="left">1. Java语言编写<br/>2. 派生于ClassLoader类<br/>3. 父类加载器为扩展类加载器<br/>4. 负责加载环境变量classpath或系统属性java.class.path指定路径下的类库<br/>5. <font color="red">是程序中默认的类加载器</font>，一般来说，Java应用的类都是由它来完成加载</td>
    </tr>
    <tr>
        <td>用户自定义类加载器</td>
        <td colspan="2" align="left">为什么要自定义类加载器？<br/>1. 隔离加载类<br/>2. 修改类加载的方式<br/>3. 扩展加载源<br/>4. 防止源码泄露</td>
    </tr>
</table>

# 4. 双亲委派机制

## 4.1 简介

Java虚拟机对class文件采用**按需加载**的方式，当需要使用该类时才会将它的class文件加载到内存生成class对象，而加载某个类的class文件时，Java虚拟机采用的是**双亲委派模式**，即把请求交由父类处理。

## 4.2 工作原理

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305213323447.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


1. 如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行
2. 如果父类加载器还有其父类加载器，则进一步向上委托，依次递归，请求最终将达到顶层的启动类加载器
3. 如果父类加载器可以完成类加载任务，就成功返回，若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式

## 4.3 双亲委派机制的优势

1. 避免类的重复加载
2. 保护程序安全，防止核心API被随意篡改
	例如自定义类java.lang.Xylitol
	

	```java
	package java.lang;
	public class Xylitol {
	    public static void main(String[] args) {
	        Xylitol xylitol = new Xylitol();
	    }
	}
	```
	执行后输出：
	

	```java
	java.lang.SecurityException: Prohibited package name: java.lang
		at java.lang.ClassLoader.preDefineClass(ClassLoader.java:662)
		at java.lang.ClassLoader.defineClass(ClassLoader.java:761)
		at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
		at java.net.URLClassLoader.defineClass(URLClassLoader.java:468)
		at java.net.URLClassLoader.access$100(URLClassLoader.java:74)
		at java.net.URLClassLoader$1.run(URLClassLoader.java:369)
		at java.net.URLClassLoader$1.run(URLClassLoader.java:363)
		at java.security.AccessController.doPrivileged(Native Method)
		at java.net.URLClassLoader.findClass(URLClassLoader.java:362)
		at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
		at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
		at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
			at sun.launcher.LauncherHelper.checkAndLoadMain(LauncherHelper.java:495)
	Error: A JNI error has occurred, please check your installation and try again
	Exception in thread "main" 
	```
	
## 4.4 沙箱安全机制
- **保证程序安全**
- **保护Java原生的JDK代码**

**Java安全模型的核心就是Java沙箱（sandbox）**。沙箱是一个限制程序运行的环境。

沙箱机制就是将Java代码**限定在虚拟机（JVM）特定的运行范围中，并且严格限制代码对本地系统资源访问（CPU、内存、文件系统、网络）**。通过这样的措施来保证对代码的有限隔离，防止对本地系统造成破坏。

所有的Java程序运行都可以指定沙箱，可以定制安全策略。

例如：自定义如下java.lang.String类，但是在加载自定义String类的时候会率先使用引导类加载器加载，而引导类加载器在加载的过程中会先加载jdk自带的文件(rt.jar包中java\lang\String.class)，报错信息说没有main方法，就是因为加载的是rt.jar包中的String类。这样可以保证对java核心源代码的保护，这就是**沙箱安全机制**。

```java
package java.lang;
public class String {
    public static void main(String[] args) {
        String str = new String();
    }
}
```
输出：

```java
错误: 在类 java.lang.String 中找不到 main 方法, 请将 main 方法定义为:
   public static void main(String[] args)
否则 JavaFX 应用程序类必须扩展javafx.application.Application
```

# 5. HotSpot VM整体结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305213344181.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


# Reference
- [尚硅谷详解JVM虚拟机](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=17650210674210078083)