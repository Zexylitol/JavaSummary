
# 1. 运行时数据区内部结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210313200031309.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)

# 2. PC Register

## 2.1 简介

- JVM中的程序计数器(Program Counter Register)并非广义上所指的物理寄存器，<font color="red">而是对物理PC寄存器的一种抽象模拟</font>，或许将其翻译为PC计数器(或指令计数器)会更加贴切。
- 它是一块很小的内存空间，几乎可以忽略不计，也是运行速度最快的存储区域
- 在JVM规范中，**PC寄存器是线程私有的，每个线程都有自己的程序计数器，生命周期与线程的生命周期保持一致**
- 程序计数器会存储线程当前正在执行的方法的JVM指令地址；如果是在执行native方法，则是未指定值(undefined)
- 它是程序控制流的指示器
- 字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令
- 它是唯一一个在JVM规范中没有规定任何OutOfMemoryError情况的区域(<font color="red">PC寄存器即没有OOM也没有GC</font>)

## 2.2 作用

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031320013077.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5bGl0b2x6,size_16,color_FFFFFF,t_70#pic_center)


- <font color="red">**PC寄存器用来存储指向下一条指令的地址，也即将要执行的指令代码；由执行引擎读取下一条指令**。</font>



## 2.3 使用PC寄存器存储字节码指令地址有什么用？

- 因为CPU需要不停的切换各个线程，切换回来以后，字节码解释器就得知道接着从哪开始继续执行



## 2.4 PC寄存器为什么会被设定为线程私有？

- 多线程环境下，CPU会不停的做任务切换，这样必然导致线程经常中断或恢复，**为了能够准确记录各个线程正在执行的当前字节码指令地址，最好的办法自然是为每一个线程都分配一个PC寄存器**，这样一来各个线程之间便可以独立计算，从而不会出现相互干扰的情况



# 3. HotSpot VM整体结构

![image-20210304202518367](https://img-blog.csdnimg.cn/img_convert/e18e23df6ac587d1b087a7a5182ea6a3.png)

# Reference

- [尚硅谷详解JVM虚拟机](https://www.bilibili.com/video/BV1PJ411n7xZ?from=search&seid=17650210674210078083)