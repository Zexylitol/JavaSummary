# 1. 运行时数据区内部结构

![image-20210313100451279](https://img-blog.csdnimg.cn/img_convert/4c0926f71dffa52cd271f4dd59d82e1e.png)

# 2. 栈、堆、方法区的交互关系

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210314203617404.png#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210314203633672.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


# 3. 方法区(Method Area)

## 3.1 概述

- 《Java虚拟机规范》中明确说明：尽管所有的方法区在逻辑上是属于堆的一部分，但一些简单的实现可能不会选择去进行垃圾收集或者进行压缩。
  - 但对于HotSpot JVM而言，方法区还有一个别名叫做Non-Heap(非堆)，目的就是要和堆分开
  - <font color="red">所以方法区看作是一块独立于Java堆的内存空间</font>

- **方法区(Method Area) 与Java堆一 样，是各个线程共享的内存区域**

- **方法区在JVM启动的时候被创建**，并且它的实际的物理内存空间中和Java堆区一样都可以是不连续的

- 方法区的大小，跟堆空间一样，可以选择固定大小或者可扩展

- **方法区的大小决定了系统可以保存多少个类**，如果系统定义了太多的类，导致方法区溢出，虛拟机同样会抛出内存溢出错误: `java. lang . outofMemoryError:PermGen space` 或者`java.lang.OutOfMemoryError: Metaspace`
  - 方法区OOM的场景：
    - 加载大量的第三方的jar包
    - Tomcat部署的工程过多（30-50个）
    - 大量动态的生成反射类

- 关闭JVM就会释放这个区域的内存

## 3.2 HotSpot中方法区的演进

- 在Jdk7及以前，习惯上把方法区称为**永久代**，jdk8开始，使用**元空间**取代了永久代
- 《Java虚拟机规范》对如何实现方法区，不做统一的要求；可以把方法区看成所谓的接口，把永久代和元空间看成接口的不同的实现
- 元空间与永久代最大的区别在于：<font color="red">**元空间不在虚拟机设置的内存中，而是使用本地内存**</font>

### 3.2.1 jdk7

![image-20210313155747898](https://img-blog.csdnimg.cn/img_convert/3d6148d79566d24a9f4daa205282c7e9.png)

### 3.2.2 jdk8

![image-20210313155809335](https://img-blog.csdnimg.cn/img_convert/1decaa8f43853d06e9d61db16e1d4eb7.png)

## 3.3 方法区的内部结构

《深入理解Java虚拟机》中对方法区存储的内容描述如下：它用于存储已被虚拟机加载的**类型信息、常量、静态变量、即时编译器编译后的代码缓存**等。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210314203851856.png#pic_center)


|  方法区的内部结构  | 说明                                                         |
| :----------------: | :----------------------------------------------------------- |
|      类型信息      | 对每个加载的类型(类class、接口interface、枚举enum、注解annotation)，JVM必须在方法区中存储以下类型信息：<br/>1. 这个类型的完整有效名称(全名=包名.类名)<br/>2. 这个类型直接父类的完整有效名(对于interface或是java.lang.object, 都没有父类)<br/>3. 这个类型的修饰符(public, abstract， final的某个子集)<br/>4. 这个类型直接接口的一个有序列表 |
|  域（Field）信息   | JVM必须在方法区中保存类型的所有域的相关信息以及域的声明顺序<br/>域的相关信息包括：域名称、 域类型、域修饰符(public, private,protected, static, final, volatile, transient的某个子集) |
| 方法（Method）信息 | JVM必须保存所有方法的以下信息，同域信息一样包括声明顺序<br/>1. 方法名称<br/>2. 方法的返回类型(或void)<br/>3. 方法参数的数量和类型(按顺序)<br/>4. 方法的修饰符(public, private, protected, static, final,synchronized, native,abstract的-一个子集)<br/>5. 方法的字节码(bytecodes)、操作数栈、局部变量表及大小(abstract和native方法除外)<br/>6. 异常表(abstract和native方法除外)<br/>    每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引 |

### 3.3.1 全局常量 vs non-final的类变量

- 静态变量和类关联在一起，随着类的加载而加载，它们称为类数据在逻辑上的一部分，类变量被类的所有实例共享，即时，没有类实例时也可以访问它
- **被声明为final的全局常量在编译的时候就会被分配了**

### 3.3.2 为什么需要常量池

- 一个java源文件中的类、接口，编译后产生一个字节码文件，而Java中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式，可以存到常量池，这个字节码包含了指向常量池的引用，在动态链接式会把符号引用转换为直接引用

### 3.3.3 常量池中有什么？

- 数量值
- 字符串值
- 类引用
- 字段引用
- 方法引用

常量池可以看做是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等类型

### 3.3.4 运行时常量池 vs 常量池

- <font color="red">**运行时常量池( Runtime Constant Pool)是方法区的一部分**</font>

- <font color="red">**常量池表(Constant Pool Table) 是Class文件的一部分，用于存放编译期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中**</font>

- **运行时常量池，在加载类和接口到虚拟机后，就会创建对应的运行时常量池**

- **JVM为每个已加载的类型(类或接口)都维护一个常量池**。池中的数据项像数组项一样，是通过索引访问的

- 运行时常量池中包含多种不同的常量，**包括编译期就已经明确的数值字面量，也包括到运行期解析后才能够获得的方法或者字段引用。此时不再是常量池中的符号地址了，这里换为真实地址**

- 运行时常量池，相对于Class文件常量池的另一重要特征是：具备动态性

- 当创建类或接口的运行时常量池时，如果构造运行时常量池所需的内存空间超过了方法区所能提供的最大值，则JVM会拋OutOfMemoryError异常。

### 3.3.5 方法区在JDK6、JDK7、JDK8中的演进细节

|     版本     | HotSpot中方法区的变化                                        |
| :----------: | :----------------------------------------------------------- |
| jdk1.6及以前 | 有永久代（permanent generation），静态变量存放在永久代上     |
|    jdk1.7    | 有永久代，但已经逐步“去永久代”,**字符串常量池、静态变量被移除，保存在堆中** |
| jdk1.8及以后 | 无永久代，**类型信息、字段、方法、常量保存在本地内存的元空间，但字符串常量池、静态变量仍在堆中** |

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210314203937808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210314203950780.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031420400567.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


### 3.3.6 永久代为什么要被元空间替换？

- 为永久代设置空间大小很难确定（例如：在某些场景下，如果动态加载类过多，容易产生Perm区的OOM）
  - 元空间和永久代之间最大的区别在于：**元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制**
- 对永久代进行调优很困难：full GC

### 3.3.7 StringTable为什么要调整位置？

- **开发中会有大量的字符串被创建，而永久代的回收效率很低，在full GC的时候才会触发，这就导致StringTable回收效率不高，进而导致永久代内存不足；放到堆里，能及时回收内存**

### 3.3.8 方法区的垃圾回收行为

**<font color="blue">方法区的垃圾收集主要回收两部分内容：常量池中废弃的常量和不再使用的类型</font>**

- 方法区的回收效果比较难以令人满意，尤其是类型的卸载，条件相当苛刻

| 回收对象           | 说明                                                         |
| ------------------ | :----------------------------------------------------------- |
| 常量池中废弃的常量 | 1. 方法区内常量池之中主要存放的两大类常量：**字面量和符号引用**<br/>2. 字面量：如文本字符串、被声明为final的常量值等<br/>3. 符号引用：三类常量：类和接口的全限定名、字段的名称和描述符、方法的名称和描述符<br/>4. HotSpot对常量池的回收策略：**只要常量池中的常量没有被任何地方引用，就可以被回收** |
| 不再使用的类型     | 判定一个类型是否属于“不再被使用的类”的条件需要同时满足下面三个条件：<br/>1. **该类所有的实例都已经被回收**，也就是Java堆中不存在该类及其任何派生子类的实例<br/>2. **加载该类的类加载器已经被回收**，这个条件除非是经过精心设计的可替换类加载器的场景，如OSGi、JSP的重加载等，否则通常是很难达成的<br/>3. **该类对应的java.lang.Class对象没有在任何地方被引用**，无法在任何地方通过反射访问该类的方法 |



# 4. HotSpot VM整体结构





![image-20210304202518367](https://img-blog.csdnimg.cn/img_convert/e18e23df6ac587d1b087a7a5182ea6a3.png)

# Reference

- [尚硅谷详解JVM虚拟机](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=15299793702369147437)