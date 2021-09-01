# JavaWeb

JavaWeb是指，所有通过Java 语言编写可以通过浏览器访问的程序的总称，叫JavaWeb。

JavaWeb是基于请求和响应来开发的。

# Web资源分类

web资源按实现的技术和呈现的效果的不同，又分为静态资源和动态资源两种。

**静态资源**：html、css、js、txt、 mp4视频、jpg图片

**动态资源**：jsp页面、Servlet程序

# 常用的web服务器

- **Tomcat：由Apache组织提供的一种web服务器，提供对jsp和Servlet的支持**。它是一种轻量级的javaweb容器（服务器），也是当前应用最广的JavaWeb服务器（免费）。

- Jboss：是一个遵从JavaEE规范的、开放源代码的、纯Java的EJB服务器，它支持所有的JavaE规范（免费）。

- GlssFish：由 Oracle公司开发的一款JavaWeb服务器，是一款强健的商业服务器，达到产品级质量（应用很少)。

- Resin：是 cAuCHO公司的产品，是一个非常流行的服务器，对servlet和JSP提供了良好的支持，
  性能也比较优良，resin自身采用JAVA语言开发（收费，应用比较多)。

- WebLogic：是oracle 公司的产品，是目前应用最广泛的web服务器，支持JavaEE规范，
  而且不断的完善以适应新的开发要求，适合大型项目(收费，用的不多，适合大公司）。

# Tomcat

## 安装

找到需要用的Tomcat版本对应的zip压缩包，解压到需要安装的目录即可。

## 目录介绍

| 目录    | 说明                                                         |
| :------ | :----------------------------------------------------------- |
| bin     | 存放Tomcat服务器的可执行程序                                 |
| conf    | 存放Tocmat服务器的配置文件                                   |
| lib     | 存放Tomcat服务器的jar包                                      |
| logs    | 存放Tomcat服务器运行时输出的日记信息                         |
| temp    | 存放Tomcdat运行时产生的临时数据                              |
| webapps | 存放部署的web工程                                            |
| work    | Tomcat工作时的目录，用来存放Tomcat运行时jsp翻译为Servlet的源码，和Session钝化的目录 |

## 启动Tomcat服务器

> 前提：配置JAVA_HOME环境变量

双击bin目录下的startup.bat文件，在浏览器输入以下地址测试是否启动成功：

- http://localhost:8080
- http://127.0.0.1:8080
- http://真实ip:8080 

## 关闭Tomcat服务器

双击bin目录下的shutdown.bat文件或关闭Tomcat服务器窗口

## 修改Tomcat端口号

Tomcat默认的端口号是8080，找到Tomcat的conf目录下的server.xml文件：

```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

找到Connector标签，修改port属性，端口范围：1-65535

修改完端口号，一定要重启Tomcat服务器才能生效

## 如何部署web工程到Tomcat

**第一种部署方法**

只需要把web工程的目录拷贝到Tomcat的webapps目录下即可。

eg: 

1. 在webapps下新建一个book文件夹（代表一个工程）
2. 拷贝web工程到book文件夹内
3. 访问路径：`http://ip:port/工程名/.../文件名`

> web工程目录介绍
>
> | 一级 |  二级   |  三级   | 说明                                                         |
> | :--: | :-----: | :-----: | :----------------------------------------------------------- |
> | src  |         |         | Java源代码                                                   |
> | web  |         |         | web工程资源文件，eg:html页面、css文件、js文件等              |
> |      | WEB-INF |         | 受服务器保护的目录，浏览器无法直接访问此目录的内容           |
> |      |         |   lib   | 存放第三方的jar包                                            |
> |      |         | web.xml | 整个web工程的配置部署描述文件<br/>可以在这里配置很多web工程的组件，比如：<br/>Servlet程序<br/>Filter过滤器<br/>Listener监听器<br/>...等 |

## Tomcat如何处理HTTP请求

<center><img src="https://ss.im5i.com/2021/09/01/ftL5Z.png" alt="ftL5Z.png" border="0" /></center>

假设来自客户的请求为：
http://localhost:8080/wsota/wsota_index.jsp
1)  请求被发送到本机端口8080，被在那里侦听的Coyote HTTP/1.1 Connector获得
2)  Connector把该请求交给它所在的Service的Engine来处理，并等待来自Engine的回应
3)  Engine获得请求localhost/wsota/wsota_index.jsp，匹配它所拥有的所有虚拟主机Host
4)  Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机）
5)  localhost Host获得请求/wsota/wsota_index.jsp，匹配它所拥有的所有Context
6)  Host匹配到路径为/wsota的Context（如果匹配不到就把该请求交给路径名为””的Context去处理）
7)  path=”/wsota”的Context获得请求/wsota_index.jsp，在它的mapping table中寻找对应的servlet
8)  Context匹配到URL PATTERN为*.jsp的servlet，对应于JspServlet类
9)  构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet或doPost方法
10) Context把执行完了之后的HttpServletResponse对象返回给Host
11) Host把HttpServletResponse对象返回给Engine
12) Engine把HttpServletResponse对象返回给Connector
13) Connector把HttpServletResponse对象返回给客户browser

# Servlet技术

## 什么是Servlet

1、Servlet是JavaEE规范之一。规范就是接口

2、Servlet是JavaWeb三大组件之一。三大组件分别是： Servlet程序、Filter过滤器、Listener监听器.

3、<span style="color:red">Servlet 是运行在服务器上的一个java小程序，它可以接收客户端发送过来的请求，并响应数据给客户端</span>

## 手动实现Servlet程序

1、编写一个类去实现Servlet接口，实现service方法，处理请求，并响应数据

```java
public class HelloServlet implements Servlet {

    public HelloServlet() {
        System.out.println("1 构造器方法");
    }
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("2 init初始化方法");
        // 1. 可以获取servlet程序的别名servlet-name的值
        System.out.println("HelloServlet程序的别名是：" + servletConfig.getServletName());
        // 2. 获取初始化参数init-param
        System.out.println("初始化参数username的值是：" + servletConfig.getInitParameter("username"));
        System.out.println("初始化参数url的值是：" + servletConfig.getInitParameter("url"));
        // 3. 获取ServletContext
        System.out.println(servletConfig.getServletContext());
    }
    @Override
    public ServletConfig getServletConfig() {
        return null;
    }
     /**
     * 专门用来处理请求和响应的
     * @param servletRequest
     * @param servletResponse
     * @throws ServletException
     * @throws IOException
     */
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("3 service === Hello Servlet 被访问了");
        // 类型转换 它有method()方法
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;

        // 获取请求的方式
        String method = httpServletRequest.getMethod();
        if ("GET".equals(method)) {
            doGet();
        } else if ("POST".equals(method)) {
            doPost();
        }
    }

    /**
     * 做get请求的操作
     */
    public void doGet() {
        System.out.println("get请求");
    }

    /**
     * 做post请求的操作
     */
    public void doPost() {
        System.out.println("post请求");
    }
    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {
        System.out.println("4 destroy 销毁方法");
    }
}
```



2、到web.xml中去配置servlet程序的访问地址

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!-- 上下文参数(属于整个web工程) -->
    <context-param>
        <param-name>username</param-name>
        <param-value>context</param-value>
    </context-param>
    <context-param>
        <param-name>password</param-name>
        <param-value>root</param-value>
    </context-param>

    <!-- servlet标签给tomcat配置servlet程序 -->
    <servlet>
        <!-- servlet程序起一个别名 （一般是类名） -->
        <servlet-name>HelloServlet</servlet-name>
        <!-- servlet程序的全类名 -->
        <servlet-class>com.atguigu.servlet.HelloServlet</servlet-class>

        <!-- 初始化参数 -->
        <init-param>
            <!-- 参数名 -->
            <param-name>username</param-name>
            <!-- 参数值 -->
            <param-value>root</param-value>
        </init-param>
        <init-param>
            <!-- 参数名 -->
            <param-name>url</param-name>
            <!-- 参数值 -->
            <param-value>jdbc:mysql://localhost:3306/test</param-value>
        </init-param>
    </servlet>

    <!-- servlet-mapping标签给servlet程序配置访问地址 -->
    <servlet-mapping>
        <!-- 告诉服务器，我当前配置的地址给哪个servlet程序使用 -->
        <servlet-name>HelloServlet</servlet-name>
        <!-- url-pattern标签配置访问地址 <br/>
            / 斜杠在服务器解析的时候，表示地址为：http://ip:port/工程路径   <br/>
            /hello 表示地址为：http://ip:port/工程路径/hello              <br/>
        -->
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
</web-app>
```

<center><img src="https://ss.im5i.com/2021/09/01/ft0q1.png" alt="ft0q1.png" border="0" /></center>

## Servlet的生命周期

1、执行Servlet构造器方法

2、执行init初始化方法

>  第一、二步，是在第一次访问的时候创建Servlet程序会调用

3、执行service方法

>  第三步，每次访问都会调用。

4、执行destroy销毁方法

>  第四步，在web工程停止的时候调用。

## 通过继承HttpServlet实现Servlet程序

一般在实际项目开发中，都是使用继承`HttpServlet`类的方式去实现`Servlet`程序

1、编写一个类去继承`HttpServlet`类，根据业务需要重写`doGet`或`doPost`方法

```java
public class HelloServlet2 extends HttpServlet {

    @Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
        System.out.println("重写了init初始化方法");
    }

    /**
     * 在GET请求的时候调用
     * @param req
     * @param resp
     * @throws ServletException
     * @throws IOException
     */
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("HelloServlet2 的 doGet方法");

        ServletConfig servletConfig = getServletConfig();
        System.out.println(servletConfig);
        // 2. 获取初始化参数init-param
        System.out.println("初始化参数username的值是：" + servletConfig.getInitParameter("username")); // null
        System.out.println("初始化参数url的值是：" + servletConfig.getInitParameter("url")); // null
    }

    /**
     * 在 POST 请求的时候调用
     * @param req
     * @param resp
     * @throws ServletException
     * @throws IOException
     */
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("HelloServlet2 的 doPost方法");
    }
}
```

2、到`web.xml`中的配置`Servlet`程序的访问地址

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!-- 上下文参数(属于整个web工程) -->
    <context-param>
        <param-name>username</param-name>
        <param-value>context</param-value>
    </context-param>
    <context-param>
        <param-name>password</param-name>
        <param-value>root</param-value>
    </context-param>

	<!-- servlet标签给tomcat配置servlet程序 -->
    <servlet>
        <!-- servlet程序起一个别名 （一般是类名） -->
        <servlet-name>HelloServlet2</servlet-name>
        <servlet-class>com.atguigu.servlet.HelloServlet2</servlet-class>

        <!-- 初始化参数 -->
        <init-param>
            <!-- 参数名 -->
            <param-name>username</param-name>
            <!-- 参数值 -->
            <param-value>root2</param-value>
        </init-param>
        <init-param>
            <!-- 参数名 -->
            <param-name>url</param-name>
            <!-- 参数值 -->
            <param-value>jdbc:mysql://localhost:3306/test2</param-value>
        </init-param>
    </servlet>
    <!-- servlet-mapping标签给servlet程序配置访问地址 -->
    <servlet-mapping>
        <!-- 告诉服务器，我当前配置的地址给哪个servlet程序使用 -->
        <servlet-name>HelloServlet2</servlet-name>
        <!-- url-pattern标签配置访问地址 <br/>
            / 斜杠在服务器解析的时候，表示地址为：http://ip:port/工程路径   <br/>
            /hello 表示地址为：http://ip:port/工程路径/hello              <br/>
        -->
        <url-pattern>/hello2</url-pattern>
    </servlet-mapping>
</web-app>
```

## ServletConfig类

Servlet程序的配置信息类

<span style="color:red">Servlet程序和ServletConfig对象都是由Tomcat负责创建，开发者负责使用</span>

Servlet程序默认是第一次访问的时候创建，ServletConfig 是每个Servlet程序创建时，就创建一个对应的ServletConfig对象

ServletConfig类的三大作用：

1、可以获取Servlet程序的别名servlet-name的值

2、获取初始化参数init-param

3、获取ServletContext对象

## ServletContext类

**什么是ServletContext对象：**

1、ServletContext 是一个接口，它表示Servlet上下文对象
2、一个web工程，只有一个ServletContext对象实例
3、ServletContext 对象是一个域对象
4、ServletContext是在web工程部署启动的时候创建，在web工程停止的时候销毁

**什么是域对象：**

域对象，是可以像Map一样存取数据的对象，叫域对象。

这里的域指的是存取数据的操作范围，整个web工程。

|        |     存数据     |     取数据     |     删除数据      |
| :----: | :------------: | :------------: | :---------------: |
|  Map   |     put()      |     get()      |     remove()      |
| 域对象 | setAttribute() | getAttribute() | removeAttribute() |

**ServletContext类的四个作用：**

1、获取web.xmI中配置的上下文参数context-param
2、获取当前的工程路径，格式:/工程路径
3、获取工程部署后在服务器硬盘上的绝对路径
4、像Map一样存取数据

## HttpServletRequest类

每次只要有请求进入Tomcat服务器，Tomcat服务器就会把请求过来的HTTP协议信息解析好封装到`Request`对象中。然后传递到`service`方法(`doGet`和`doPost`)中给开发者使用。开发者可以通过`HttpServletRequest`对象，获取到所有请求的信息。

| `HttpServletRequest` 类的常用方法 | 说明                               |
| :-------------------------------- | :--------------------------------- |
| `getRequestURI()`                 | 获取请求的资源路径                 |
| `getRequestURL()`                 | 获取请求的统一资源定位符(绝对路径) |
| `getRemoteHost()`                 | 获取客户端的ip地址                 |
| `getHeader()`                     | 获取请求头                         |
| `getParameter()`                  | 获取请求的参数                     |
| `getParameterValues()`            | 获取请求的参数(多个值的时候使用)   |
| `getMethod()`                     | 获取请求的方式GET或POST            |
| `setAttribute(key, value)`        | 设置域数据                         |
| `getAttribute(key)`               | 获取域数据                         |
| `getRequestDispatcher()`          | 获取请求转发对象                   |

## HttpServletResponse类

`HttpServletResponse`类和`HttpServletRequest`类一样。 每次请求进来，Tomcat 服务器都会创建一个`Response` 对象传递给`Servlet`程序去使用。`HttpServletRequest` 表示请求过来的信息，`HttpServletResponse` 表示所有响应的信息。

如果需要设置返回给客户端的信息，都可以通过`HttpServletResponse`对象来进行设置

# Nginx

## 概述

Nginx是一个<span style="color:red">高性能的HTTP和反向代理服务器</span>，特点是占有内存少，并发能力强

## Nginx作为web服务器

Nginx可以作为<span style="color:red">静态页面的web服务器</span>，同时还支持CGI协议的动态语言，比如perl、php等。但是不支持java。**Java程序只能通过与tomcat配合完成**。

## 反向代理

只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，再返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，**暴露的是代理服务器地址，隐藏了真实服务器IP地址**。

<center><img src="https://ss.im5i.com/2021/09/01/f5shq.png" alt="f5shq.png" border="0" /></center>

## 负载均衡

客户端发送多个请求到服务器，服务器处理请求，有一些可能要与数据库进行交互，服务器处理完毕后，再将结果返回给客户端。

这种架构模式对于早期的系统相对单一、并发请求相对较少的情况下是比较适合的，成本也低。但是随着信息数量的不断增长，访问量和数据量的飞速增长，以及系统业务的复杂度增加，这种架构会造成服务器响应客户端的请求日益缓慢，并发量特别大的时候，还容易造成服务器直接崩溃。很明显这是由于**服务器性能的瓶颈**造成的问题。

首先想到的可能是**升级服务器的配置**，比如提高CPU执行频率，加大内存等提高机器的物理性能来解决此问题，但是随着**摩尔定律**的日益失效，**硬件的性能提升**已经不能满足日益提升的需求了。最明显的一个例子，天猫双十一当天，某个热销商品的瞬时访问量是极其庞大的，那么类似上面的系统架构，将机器都增加到现有的顶级物理配置，都是不能够满足需求的。那么怎么办呢?

上面的分析我们去掉了**增加服务器物理配置**来解决问题的办法，也就是说纵向解决问题的办法行不通了，那么横向增加服务器的数量呢?这时候**集群**的概念产生了，单个服务器解决不了，<span style="color:red">增加服务器的数量</span>，然后将请求分发到各个服务器上，<span style="color:red">将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将流量均匀分发到不同的服务器上</span>，也就是所说的**负载均衡**

### Nginx分配服务器策略

**轮询（默认）**

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除

**weight**

weight代表权重默认为1，权重越高被分配的请求越多

```nginx.conf
http {
......
	upstream myserver {
		server 192.168.5.21 weight=10;
		server 192.168.5.22 weight=10;
	}
......
	server {
		location / {
			proxy_pass http://myserver;
			proxy_connect_timeout 10;
			...
		}
		...
	}
}
```



**ip_hash**

每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session问题

```nginx.conf
upstream myserver {
	ip_hash;
    server 192.168.5.21:80;
    server 192.168.5.22:80;
}
```



**fair(第三方)**

按后端服务器的响应时间来分配请求，响应时间短的优先分配

```nginx.conf
upstream myserver {
    server 192.168.5.21:80;
    server 192.168.5.22:80;
    fair;
}
```



## 正向代理

Nginx不仅可以做反向代理，实现负载均衡。还能用作正向代理来进行上网等功能。

**正向代理**：如果把局域网外的Internet 想象成一个巨大的资源库，则局域网中的客户端要访问Internet，则需要通过**代理服务器**来访问，这种代理服务就称为正向代理。

<center><img src="https://ss.im5i.com/2021/09/01/f5EBm.png" alt="f5EBm.png" border="0" /></center>

## 动静分离

**为了加快网站的解析速度**，可以把**动态页面（jsp、servlet）**和**静态页面（html、css、图片等）**由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。

<center><img src="https://ss.im5i.com/2021/09/01/f5bMQ.png" alt="f5bMQ.png" border="0" /></center>

Nginx动静分离简单来说就是把**动态跟静态请求分开**，不能理解成只是单纯的把动态页面和静态页面物理分离。严格意义上说应该是动态请求跟静态请求分开，**可以理解成使用Nginx处理静态页面，Tomcat处理动态页面**。动静分离从目前实现角度来讲大致分为两种，**一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案；另外一种方法就是动态跟静态文件混合在一起发布，通过nginx来分开**。

通过location指定不同的后缀名实现不同的请求转发。通过expires参数设置，可以使浏览器缓存过期时间，减少与服务器之间的请求和流量。具体Expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。**此种方法非常适合不经常变动的资源**。(如果经常更新的文件，不建议使用Expires来缓存)

# Reference

- [http请求从浏览器到进入tomcat服务器的全过程](https://blog.csdn.net/qq_33915826/article/details/81436261)





