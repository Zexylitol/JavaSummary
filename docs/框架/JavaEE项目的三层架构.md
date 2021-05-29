# JavaEE项目的三层架构

<img src="https://i.im5i.com/2021/05/09/C6uhW.png" alt="C6uhW.png" border="0" />

分层的目的是为了解耦。 解耦就是为了降低代码的耦合度。 方便项目后期的维护和升级。  

```java
web 层          com.atguigu.web/servlet/controller
service 层      com.atguigu.service                            Service 接口包
                com.atguigu.service.impl                       Service 接口实现类
dao 持久层       com.atguigu.dao                                Dao 接口包
                com.atguigu.dao.impl                           Dao 接口实现类
实体 bean 对象    com.atguigu.pojo/entity/domain/bean           JavaBean 类
测试包           com.atguigu.test/junit
工具类           com.atguigu.utils
```



# MVC设计模式

MVC即Model-View-Controller（模型-视图-控制器）是一种软件设计模式，最早出现在Smalltalk语言中，后被Sun公司推荐为Java EE平台的设计模式。

MVC把应用程序分成了上面3个核心模块，这3个模块又可被称为业务层-视图层-控制层。顾名思义，它们三者在应用程序中的主要作用如下：

　　**业务层**：负责实现应用程序的业务逻辑，封装有各种对数据的处理方法。它不关心它会如何被视图层显示或被控制器调用，它只接受数据并处理，然后返回一个结果。

　　**视图层**：负责应用程序对用户的显示，它从用户那里获取输入数据并通过控制层传给业务层处理，然后再通过控制层获取业务层返回的结果并显示给用户。

　　**控制层**：负责控制应用程序的流程，它接收从视图层传过来的数据，然后选择业务层中的某个业务来处理，接收业务层返回的结果并选择视图层中的某个视图来显示结果。

<img src="https://i.im5i.com/2021/05/09/C6Hnz.png" alt="C6Hnz.png" border="0" />



<img src="https://i.im5i.com/2021/05/09/C65JG.gif" alt="C65JG.gif" border="0" />



# Reference

- [MVC设计模式](https://blog.csdn.net/zhouym_/article/details/90611839)

- [MVC设计模式思想及简单实现](https://www.cnblogs.com/ysyasd/p/10771464.html)

- [MVC设计模式之Web应用剖析](https://zhuanlan.zhihu.com/p/51642345)

- [MVC设计模式简介](http://c.biancheng.net/view/4391.html)