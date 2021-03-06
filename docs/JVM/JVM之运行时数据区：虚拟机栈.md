# 1. 运行时数据区内部结构

![image-20210313100451279](https://img-blog.csdnimg.cn/img_convert/4c0926f71dffa52cd271f4dd59d82e1e.png)

# 2. Java虚拟机栈

## 2.1 虚拟机栈出现背景

- 由于跨平台性的设计，Java的指令都是根据栈来设计的(**使用零地址指令，又叫堆栈运算指令，必须依赖于栈，操作数都来自栈**)；不同平台CPU架构不同，所以不能设计为基于寄存器的
- **优点**：跨平台，指令集小，编译器容易实现，缺点是性能下降，实现同样的功能需要更多的指令

## 2.2 内存中的堆与栈

- <font color="red">**栈是运行时的单位，而堆是存储的单位**</font>
- 栈解决程序的运行问题，即如何处理数据
- 堆解决的是数据存储问题，即数据怎么放、放在哪儿

## 2.3 简介

- Java虚拟机栈(Java Virtual Machine Stack)，早期也叫Java栈，**是线程私有的**
- 每个线程启动后，JVM就会为其分配一块栈内存，**每个栈由一个个栈帧(Stack Frame)组成，对应着每次方法调用时所占用的内存**；每个线程只能有一个**活动栈帧**，对应着当前正在执行的那个方法
- **生命周期与线程一致**
- 栈是一种快速有效的分配存储方式，访问速度仅次于程序计数器
- JVM对Java栈的操作只有两个：
  - 每个方法执行时：进栈（入栈、压栈）
  - 方法执行结束后：出栈
- **Java虚拟机栈不存在垃圾回收问题**

## 2.4 作用

- 主管Java程序的运行，它保存方法的局部变量(8种基本数据类型、对象的引用地址)、部分结果，并参与方法的调用与返回

## 2.5 栈中可能出现的异常

- **Java虚拟机规范允许Java栈的大小是动态的或者是固定不变的**
  - **如果采用固定大小的Java虚拟机栈**，那每一个线程的Java虚拟机栈容量可以在线程创建的时候独立选定。**如果线程请求分配的栈容量超过Java虚拟机栈允许的最大容量，Java虚拟机将会抛出一个StackOverflowError异常**
  - **如果Java虚拟机栈可以动态扩展**，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的虚拟机栈，**那Java虚拟机将会抛出一个OutOfMemoryError 异常**

- -Xss：设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度，eg:-Xss10m
- <font color="red">HotSpot虚拟机不支持动态扩展</font>

## 2.6 栈帧的内部结构

### 2.6.1 简介
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210313201056616.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210313201124987.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)

<center>动态链接</center>

|                         栈帧中存储着                         | 说明                                                         |
| :----------------------------------------------------------: | :----------------------------------------------------------- |
|               局部变量表<br>(Local Variables)                | 1. 也称为局部变量数组或本地变量表<br/>2. **定义为一个数字数组**，主要用于存储**方法参数和定义在方法体内的局部变量**，包括各类基本数据类型、对象引用(Reference)，以及returnAddress类型<br/>3. **是线程的私有数据**，**不存在数据安全问题**，局部变量是否线程安全要具体问题具体分析<br/>4. **局部变量表所需的容量大小是在编译期确定下来的**，并保存在Code属性的maximum local variables数据项中，**在方法运行期间不会改变局部变量表的大小**<br/>5. 对一个函数而言，它的参数和局部变量越多，使得局部变量表膨胀，它的栈帧就越大，进而函数调用就会占用更多的栈空间，导致其嵌套调用次数就会减少(**方法嵌套调用的次数由栈的大小决定**)<br/>6. 局部变量表中的变量只有在当前方法调用中有效，**当方法调用结束后，随着方法栈帧的销毁而销毁**<br/>7. <font color="blue">局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收</font> |
|                 操作数栈<br/>(Operand Stack)                 | 1. 操作数栈主要**用于保存计算过程的中间结果，同时作为计算过程中变量临时的存储空间**<br/>2. 拥有一个明确的栈深度用于存储数值，其所需的最大深度在编译期就定义好了，保存在方法的Code属性中<br/>3. 32bit的数据类型占用一个栈单位深度，64bit的类型占用两个栈单位深度<br/>4. 只能通过标准的入栈(push)和出栈(pop)操作来完成一次数据访问<br/>5. **如果被调用的方法带有返回值的话，其返回值将会被压入当前栈帧的操作数栈中**，并更新寄存器中下一条需要执行的字节码指令<br/>6. <font color="red">我们说Java虚拟机的解释引擎是基于栈的执行引擎，其中的栈指的就是操作数栈</font><br/>7. **栈顶缓存技术(ToS,Top-of-Stack Cashing)**：将栈顶元素全部缓存在物理CPU的寄存器中，以此降低对内存的读/写次数，提升执行引擎的执行效率 |
| 动态链接<br>(Dynamic Linking)<br>(或指向运行时常量池的方法引用) | 1. **每一个栈帧内部都包含一个指向运行时常量池中该栈帧所属方法的引用**，包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接<br/>2. **在Java源文件被编译成字节码文件时，所有的变量和方法引用都作为符号引用(Symbolic Reference) 保存在class文件的常量池里**。比如:描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示的，那么**动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用**<br/>3. **常量池的作用**：就是为了提供一些符号和常量，便于指令的识别 |
| 方法返回地址<br/>(Return Address)<br/>(或方法正常退出或者异常退出的定义) | 1. **存放调用该方法的PC寄存器的值**<br/>2. 一个方法的结束，有两种方式：1）正常执行完成；2）出现未处理的异常，非正常退出<br/>3. 无论通过哪种方式退出，在方法退出后都返回到该方法被调用的位置。方法正常退出时，**调用者的PC计数器的值作为返回地址，即调用该方法的指令的下一条指令的地址**。而**通过异常退出的，返回地址是要通过异常表来确定**，栈帧中一般不会保存这部分信息<br/>4. 本质上，方法的退出就是当前栈帧出栈的过程<br/>5. <font color="red">正常完成出口和异常完成出口的区别在于：通过异常完成退出的不会给它的上层调用者产生任何的返回值</font> |
|                         一些附加信息                         | eg：对程序调试提供支持的信息                                 |



### 2.6.2 方法中定义的局部变量是否线程安全？

- 不确定，需具体问题具体分析
- 局部变量是线程安全的
- 但局部变量引用的对象则未必
  - 如果该对象没有逃离方法的作用范围，它是线程安全的
  - 如果该对象逃离方法的作用范围，需要考虑线程安全

# HotSpot VM整体结构

![image-20210304202518367](https://img-blog.csdnimg.cn/img_convert/e18e23df6ac587d1b087a7a5182ea6a3.png)



# Reference

- [尚硅谷详解JVM虚拟机](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=17650210674210078083)