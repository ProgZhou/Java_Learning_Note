## Java Web

### 一、Java web的基本概念

#### 1. 什么是Java Web

+ java web是指所有通过Java编写可以通过浏览器访问的程序的总称
+ JavaWeb是基于**请求和响应**来开发的

#### 2. 什么是请求

+ 请求是指客户端向服务器发送数据，叫请求 **Request**

+ 请求的Http请求头

  ![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\Post请求的请求头.jpg)

#### 3. 什么是响应

+ 响应是指服务器给客户端回传数据，叫响应 **Response**

+ 响应的Http格式

  ![响应的Http协议格式](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\响应的Http协议格式.jpg)

#### 4. 请求和响应的关系

+ 请求和响应是**成对出现的**，有请求就有响应

#### 5. Web资源的分类

+ **静态资源**：html，css，js，txt...
+ **动态资源**：jsp页面，servlet程序

#### 6. 常用服务器

+ **Tomcat**: Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用服务器
+ **Jboss**: 是一个基于J2EE的开放源代码的应用服务器。 JBoss代码遵循LGPL许可，可以在任何商业应用中免费使用。
+ **GlassFish**: 是一款强健的商业兼容应用服务器，达到产品级质量，可免费用于开发、部署和重新分发。

#### 7. Web中的路径

+ 相对路径：
  + *.*      表示当前目录
  + ..     表示上一级目录
+ 绝对路径：http://ip:port/工程名/资源路径

#### 8. Web中'/'的不同含义

+ 在web中 */* 是一种绝对路径
  + / 如果被浏览器解析，得到的地址是：http://ip:port
  + / 如果被服务器解析，得到的地址是：http://ip:port/工程路径

#### 9. JavaEE三层架构

+ Web层：

  + 获取请求参数，封装为Bean对象
  + 调用Service层的函数处理业务
  + 将响应数据返回客户端(请求转发和请求重定向)

+ Service层：

  + 处理业务逻辑
  + 调用持久层将数据保存到数据库

+ DAO持久层：

  + 只负责对数据库进行操作，进行CRUD操作

+ 示意图：

  <img src="C:\Users\86198\Desktop\JavaEE\JavaWeb\image\JavaEE三层架构.jpg" alt="JavaEE三层架构" style="zoom:80%;" />

### 二、Servlet

#### 1. 什么是servlet

+ Servlet本质上是一个Java程序，一般由一个或者多个Servlet程序组成
+ Servlet程序在Servlet容器中运行

#### 2. Java中的Servlet API

+ **javax.servlet**: 包含定义Servlet和Servlet容器之间协议的类和接口 
+ **javax.servlet.http**: 包含定义HTTP Servlet和Servlet容器之间协议的类和接口
+ **javax.servlet.annotation**: 包含标注Servlet，filter和listener的注解
+ **javax.servlet.descriptor**: 包含提供以编程方式访问Web应用程序配置信息的类型

#### 3. Servlet继承体系

+ **主接口**: javax.servlet.Servlet，负责定义Servlet程序的访问规范

+ **实现接口的类**: javax.servlet.GenericServlet，实现了Servlet接口，并且做了许多空实现(即什么都不做，只返回数据)，含有一个ServletConfig类的引用，并对ServletConfig的使用做了一些方法

+ **继承于GenericServlet**: javax.servlet.HttpServlet，实现了service()方法，并且实现了请求的分类处理

+ 一般用户自定义的类都继承于HttpServlet类

  ```java
  public class HttpServletTest extends HttpServlet {
      //在get请求时调用
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //super.doGet(req, resp);
          System.out.println("HttpServlet doGet().");
      }
      //在post请求时调用
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //super.doPost(req, resp);
          System.out.println("HttpServlet doPost().");
      }
  }
  ```

+ 通过继承HttpServlet实现Servlet程序的步骤

  + 1). 编写一个类去继承HttpServlet类
  + 2). 根据业务的需要重写doGet()和doPost()方法处理两种不同的请求(get请求和post请求)
  + 3). 在web.xml中配置servlet程序的访问地址

+ 继承体系概念图：

  ![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\servlet继承体系.jpg)

#### 4. ServletConfig类

+ ServletConfig类是servlet程序的**配置信息类**

+ ServleConfig类的作用: 

  + 1). 可以获取servlet程序的别名，即servlet-name标签的值
  + 2). 获取初始化参数init
  + 3). 获取ServletContext对象

+ ServletConfig对象都是由**Tomcat负责创建**供用户使用的

+ ServletConfig对象是**每个servlet程序创建时，就创建一个ServletConfig对象**

+ ServletConfig类使用测试：

  

#### 5. ServletContext类

+ ServletContext是一个**接口**，标识servlet程序的上下文对象

+ 一个web工程**只有一个**ServletContext对象实例

+ ServletContext是一个域对象

  域对象是一个可以像Map一样存取数据的对象，域是指存取数据的操作范围

  |        | 存数据         | 取数据         | 删除数据          |
  | ------ | -------------- | -------------- | ----------------- |
  | Map    | put()          | get()          | remove()          |
  | 域对象 | setAttribute() | getAttribute() | removeAttribute() |

+ 作用：

  + 1). 获取web.xml中配置的上下文参数context-param
  + 2). 获取当前的工程路径
  + 3). 获取工程部署后在服务器磁盘上的绝对路径
  + 4). 像Map一样存储数据

+ ServletContext使用测试：

  ```Java
  public class ServletContextTest extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
  //        1). 获取web.xml中配置的上下文参数context-param
          ServletContext servletContext = getServletConfig().getServletContext();
          String username = servletContext.getInitParameter("username");
          System.out.println("username: " + username);
          String url = servletContext.getInitParameter("url");
          System.out.println("url: " + url);
  //        2). 获取当前的工程路径
          System.out.println("current url: " + servletContext.getContextPath());
  //        3). 获取工程部署后在服务器磁盘上的绝对路径
  //      / 被解析为http://ip:port/工程名/
          System.out.println("Absolutely url: " + servletContext.getRealPath("/"));
  //        4). 像Map一样存储数据
          //只要创建了数据，整个web工程都能够访问到包括自己
          //servletContext.getAttribute("password"); 第一次启动服务器时，这里的值是null但当这个程序再次访问时，这里的值就变为abc
          servletContext.setAttribute("driverClass", "com.microsoft.sqlserver.jdbc.SQLServerDriver");
          servletContext.setAttribute("password", "abc");
          System.out.println(servletContext.getAttribute("driverClass"));
          System.out.println(servletContext.getAttribute("password"));
      }
  
      @Override
      protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
  
      }
  }
  ```


#### 6. HttpServletRequest类

+ 作用：每次只要有请求进入Tomcat服务器，Tomcat就会把请求过来的http协议信息解析好封装到Request对象中，然后传递到service方法(主要是doGet()和doPost()方法)；用户可以通过HttpServletRequest对象，获取所有请求的信息

+ 常用方法：

  + 1. getRequestURI(): 获取请求的资源路径

  *   2. getRequestURL(): 获取请求的统一资源定位符
  *   3. getRemoteHost(): 获取客户端的ip地址
  *   4. getHeader(): 获取请求头
  *   5. getParameter(): 获取请求的参数
  *   6. getParameterValues(): 获取请求的参数（多个值时使用）
  *   7. getMethod(): 获取请求的方式(Get或者Post)
  *   8. setAttribute(key, value): 设置数据与
  *   9. getAttribute(key): 获取数据域
  *   10. getRequestDispatcher(): 获取请求转发对象

+ 测试代码：

  ```java
  public class HttpServletRequestTest extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //    1. getRequestURI(): 获取请求的资源路径
          System.out.println("URI -> " + req.getRequestURI());
          //    2. getRequestURL(): 获取请求的统一资源定位符
          System.out.println("URL -> " + req.getRequestURL());
          //    3. getRemoteHost(): 获取客户端的ip地址
          /*
          * 在IDEA中使用localhost访问时，得到的客户端IP地址是 127.0.0.1 / 0.0.0.0.0.0.0.1(ipv6)
          * 如果使用真实的ip地址访问，得到的客户端地址就是真实的ip地址
          * */
          System.out.println("Client IP -> " + req.getRemoteHost());
          //    4. getHeader(): 获取请求头
          System.out.println("Header -> " + req.getHeader("User-Agent"));
          //            7. getMethod(): 获取请求的方式(Get或者Post)
          System.out.println("Method -> " + req.getMethod());
      }
  
  }
  ```

+ 利用HttpServletRequest实现请求转发的过程：

  ![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\请求转发.jpg)

  服务器请求转发：

  ![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\04.服务器内部转发.png)

+ 请求转发的特点：

  + 浏览器地址栏没有变化
  + 只是一次请求
  + 两个程序共享Request域中的数据
  + 可以转发到WEB-INF下(使用浏览器不能直接访问WEB-INF文件夹下的文件)
  + 不能访问工程以外的资源

+ 请求转发测试代码：

  ```java
  //servlet1.java
  public class Servlet1 extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
  
          //获取请求参数
          String username = req.getParameter("username");
          System.out.println("Parameter in Servlet1: " + username);
  
          //给材料“盖章”并传递到servlet2中去查看，表示数据已经在servlet1中处理过了
          req.setAttribute("key", "Servlet1");
  
          //下一步该去哪
          /*
          * 请求转发必须要以 / 打头，‘/’表示地址为http://ip:port/工程名/ ，映射到IDEA代码的web目录
          * */
          //如果要访问WEB-INF下的文件：req.getRequestDispatcher("/WEB-INF/文件名");
          //RequestDispatcher requestDispatcher = req.getRequestDispatcher("/servlet2");
          RequestDispatcher requestDispatcher = req.getRequestDispatcher("/WEB-INF/a.html");
          //定向至servlet2
          requestDispatcher.forward(req, resp);
  
      }
  }
  
  //servlet2.java
  public class Servlet2 extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //获取请求参数,查看
          String username = req.getParameter("username");
          System.out.println("Servlet2: Parameter in Servlet1: " + username);
  
          //查看是否经过servlet1的处理
          Object key = req.getAttribute("key");
          System.out.println("if parameter in Servlet1: " + key);
  
          //servlet2处理自己的业务
          System.out.println("Doing...");
      }
  }
  ```
  
+ 请求转发的特点：一次请求响应的过程，对于客户端而言，内部经过了多少次转发，客户端是不知道的；浏览器的地址栏不发生变化

#### 7. HttpServletResponse类

+ 作用：

  + 1. HttpServletResponse类和HttpServletRequest类一样，每次请求进入，Tomcat服务器都会创建一个Response对象传递给service去使用
  + 2. HttpServletResponse表示请求过来的信息，HttpServletResponse表示所有响应的信息客户如果需要设置返回给客户端的信息，都可以通过HttpServletResponse对象进行设置

+ 两个输出流：

  + 字节流：getOutputStream()  常用于下载（传递二进制数据）
  + 字符流：getWriter()                常用于回传字符串
  + 注意：两个流只能同时使用一个，使用了字符流就不能使用字节流，反之亦然

+ 测试代码：

  ```java
  public class HttpServletResponseTest extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //设置服务器字符集
          resp.setCharacterEncoding("UTF-8");
          //通过响应头，设置浏览器也使用UTF-8字符集
          //resp.setHeader("Content-Type", "text/html; charset=UTF-8");
          //可以直接调用setContentType()方法，这个方法会同时设置服务器和客户端都是用UTF-8字符集，同时设置请求的响应头
          //这个方法必须要在获取流对象之前调用
          resp.setContentType("text/html; charset=UTF-8");
          PrintWriter writer = resp.getWriter();
          writer.println("response content");
          //显示中文可能出现乱码，需要适当调整字符集
          writer.println("急急急急急急");
      }
  }
  ```

+ HttpServletResponse的请求重定向

  ![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\请求重定向.JPG)

  服务器重定向：

  ![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\05.客户端重定向.png)

+ 请求重定向测试代码：

  ```java
  //Response1.java
  public class Response1 extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          System.out.println("Response1");
          //设置响应状态码，表示重定向
          resp.setStatus(302);
          //设置响应头，说明新的地址在哪里
          resp.setHeader("Location", "http://localhost:8080/JavaWeb/response2");
      }
  }
  
  //Response2.java
  public class Response2 extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          resp.getWriter().println("response2'result");
      }
  }
  ```
  
+ 重定向总结：两次请求响应的过程，客户端肯定知道有URL的变化；浏览器的地址栏发生变化

#### 8. xml文件的配置

+ 对于每一个Servlet程序都需要在Web项目中的web.xml中配置访问地址，这样在使用Tomcat服务器访问时才能访问到

  ```xml
  
  <!-- 配置上下文参数，属于整个web工程-->
      <context-param>
          <param-name>username</param-name>
          <param-value>sa</param-value>
      </context-param>
      <context-param>
          <param-name>url</param-name>
          <param-value>jdbc:sqlserver//localhost:1433;DatabaseName=NetSchool_Data</param-value>
      </context-param>
  <!-- servlet标签给Tomcat服务器配置servlet程序-->
  <servlet>
      <!-- servlet name标签给servlet程序起一个别名，一般是类名-->
      <servlet-name>HelloServlet</servlet-name>
      <!-- servlet class是servlet程序的全类名-->
      <servlet-class>com.JavaWeb.ServletTest.HelloServlet</servlet-class>
      
      <!-- 配置初始化参数，可以配置多个-->
          <init-param>
              <!-- 参数名-->
              <param-name>username</param-name>
              <!-- 参数值-->
              <param-value>root</param-value>
          </init-param>
  
          <init-param>
              <!-- 参数名-->
              <param-name>password</param-name>
              <!-- 参数值-->
              <param-value>123456</param-value>
          </init-param>
  </servlet>
  <!-- servlet mapping给servlet程序配置一个访问地址-->
  <servlet-mapping>
      <!-- 这个servlet name标签告诉服务器，当前配置的地址给哪个servlet程序使用-->
      <!-- 这个值一定是和上面servlet name标签中的值相同-->
      <servlet-name>HelloServlet</servlet-name>
      <!-- 配置访问地址
      /hello 标识地址为http://ip:port/工程路径/hello
      -->
      <url-pattern>/hello</url-pattern>
  </servlet-mapping>
  ```

+ 根据xml文件解析Servlet程序被访问的过程：

  ![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\servlet访问过程.jpg)

#### 9. Servlet保存作用域

原始情况下，保存作用域我们可以认为有四个：page(页面级别，现在几乎不用)，request(一次请求响应范围)，session(一次会话范围)，application(一次应用范围内有效)

+ request保存作用域: 一次请求响应范围内有效

  ```java
  @WebServlet("/demo01")
  public class Demo01Servlet extends HttpServlet{
  	@Override 
  	protected void service(HttpServletRequest request, HttpServletResponse response){
  	request.setAttribute("name", "lili");
  	request.sendRedirect("demo02");
  	}
  }
  @WebServlet("/demo02")
  public class Demo02Servlet extends HttpServlet{
  	@Override 
  	protected void service(HttpServletRequest request, HttpServletResponse response){
  	Object obj = request.getAttribute("name");
      //如果是重定向，打印null，如果是请求转发，打印lili
  	System.out.println(obj); 
  	
  	}
  }
  
  ```

  ![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\01.Request保存作用域.png)

+ session保存作用域: 一次会话范围内有效

  ```java
  @WebServlet("/demo01")
  public class Demo01Servlet extends HttpServlet{
  	@Override 
  	protected void service(HttpServletRequest request, HttpServletResponse response){
          request.getSession().setAttribute("name", "lili");
          request.sendRedirect("demo02");
  	}
  }
  @WebServlet("/demo02")
  public class Demo02Servlet extends HttpServlet{
  	@Override 
  	protected void service(HttpServletRequest request, HttpServletResponse response){
          HttpSession session = request.getSession();
  	Object obj = session.getAttribute("name");
          //如果是同一个客户端的多次访问，则打印lili，如果是不同客户端的访问则打印null
  	System.out.println(obj); 
  	
  	}
  }
  ```

  ![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\02.Session保存作用域.png)

+ application保存作用域: 一次应用范围内有效

  只要有一个请求往application域中放置了数据，所有请求都能够访问到

  ```java
  @WebServlet("/demo01")
  public class Demo01Servlet extends HttpServlet{
  	@Override 
  	protected void service(HttpServletRequest request, HttpServletResponse response){
          //ServletContext: Servlet上下文
  	ServletContext application = request.getServletContext();
          application.setAttribute("name", "lili");
  	request.sendRedirect("demo02");
  	}
  }
  @WebServlet("/demo02")
  public class Demo02Servlet extends HttpServlet{
  	@Override 
  	protected void service(HttpServletRequest request, HttpServletResponse response){
  	Object obj = request.getServletContext().getAttribute("name");
          //只要没有重启Tomcat，每一个客户端访问都可以打印lili
  	System.out.println(obj); 
  	
  	}
  }
  ```

  ![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\03.Application保存作用域.png)

#### 10. Servlet生命周期

Servlet的生命周期对应Servlet从创建到销毁的过程，分别对应了Servlet中的三个方法：init()，service()，destroy()

默认情况下：

+ 第一次接收请求时，这个Servlet会进行实例化(调用构造器方法)，初始化(init()方法)，然后进行服务(service()方法)
+ 从第二次请求开始，每一次都是服务
+ 当容器关闭时，其中所有的Servlet都会被销毁(destroy()方法)

Tomcat对Servlet的处理：

+ Servlet实例Tomcat只会创建一个，所有的请求都是这个实例去响应
+ 默认情况下，第一次请求时，tomcat才去实例化，初始化，然后再服务，这样可以提高系统的启动速度，但第一次请求的耗时较长
+ 结论：如果需要提高系统的启动速度，可以使用默认情况；如果需要提高响应速度，应该设置Servlet的初始化时机

Servlet的初始化时机：

+ 默认是第一次接收请求时，实例化，初始化
+ 我们可以通过\<load-on-startup>来设置servlet启动的先后顺序,数字越小，启动越靠前，最小值0

Servlet在容器中是单例的，线程不安全的

+ 单例：所有请求都是同一个实例去响应

+ 线程不安全：一个线程需要根据这个实例中的某个成员变量值去做逻辑判断。但是在中间某个时机，另一个线程改变了这个成员变量的值，从而导致第一个线程的执行路径发生了变化

+ 尽量的不要在servlet中定义成员变量。如果不得不定义成员变量，那么不要去：

  ​	①不要去修改成员变量的值 

  ​	②不要去根据成员变量的值做一些逻辑判断

### 三、Web架构

#### 1. Session

Session的引入：

Http协议是无状态的，服务器无法判断这两次请求是同一个客户端发过来的，还是不同的客户端发送过来的请求

无状态带来的问题：比如第一次请求是添加商品到购物车，第二次请求是结账，如果这两次请求服务器无法区分是否是同一个用户的，那么就会导致混乱

通过会话技术解决无状态带来的问题 ----- Session

![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\02.会话跟踪技术(1).png)

##### 1.1 什么是Session

*       ​	 1). Session是一个接口(HttpSession)
*       ​     2). Session是一个会话，用来维护客户端和服务器之间关联的一种技术
*       ​     3). 每个客户端都有自己的Session会话
*       ​     4). Session中 经常用来保存用户登录之后的信息

##### 1.2 创建和获取Session

+ 1). req.getSession()

   第一次调用是创建Session会话，之后调用都是获取前面创建好的Session会话对象

+ 2). isNew(): 判断Session是否是刚创建的Session

  ​		  true：表示是刚创建的Session

  ​	     false：表示不是刚创建的Session

+ 3). 每个Session都有一个唯一的ID

  ​	使用getID()得到Session会话的id值

##### 1.3 Session生命周期的控制

+ 1). void setMaxinactiveInterval(int interval) 设置Session的超时时间，超过指定时长，Session就会被销毁

+ 2). int getMaxinactiveInterval(int interval) 获取Session的超时时间 以秒为单位

+ 3).Session默认的超时时长为1800s，由服务器配置可以在web.xml中修改当前工程下的Session的默认超时时长，以分钟为单位

  ```xml
  <session-config>
       <session-timeout>数值</session-timeout>
  </session-config>
  ```

+ 4). 修改个别的Session超时时长:  调用setMaxinactiveInterval(int interval)

+ 5). Session超时是指客户端两次请求时间间隔的最大时长，如果在超时时长没有结束之前客户端再次发送请求，那么当前Session的超时时长会被重制为初始值

+ 6). 设置当前Session立即销毁     void invalidate()

+ Session的生命周期

  ![Session的生命周期](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\Session的生命周期.jpg)

+ Seesion操作的测试代码

  ```java
  public class SessionTest extends BaseServlet{
  	
      //设置Session的超时时长
      protected void updateTimeout(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //设置当前Session 3秒后超时
          req.getSession().setMaxInactiveInterval(3);
          resp.getWriter().write("当前Session已设置为3秒后超时");
      }
  	//服务器默认的Session存活时长
      protected void defaultLife(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          int time = req.getSession().getMaxInactiveInterval();
          resp.getWriter().write("Session的默认超时时长为 " + time + "秒<br/>");
      }
  	//设置Session的值
      protected void setData(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //往Session数据域中保存数据
          req.getSession().setAttribute("key1", "value1");
          resp.getWriter().write("已经向Session中保存信息");
      }
      //获取Session的值
      protected void getData(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //获取Session域中的数据
          Object key1 = req.getSession().getAttribute("key1");
          resp.getWriter().write("从Session中获取的数据是 " + key1.toString());
      }
  	//创建一个Session会话
      protected void createSession(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //创建和获取Session对象
          HttpSession session = req.getSession();
          //判断当前Session是否是新创建的
          boolean flag = session.isNew();
          //获取Session会话的唯一标识
          String id = session.getId();
  
          resp.getWriter().write("得到的Session是 " + id + "<br/>");
          resp.getWriter().write("这个Session是否是新创建的 " + flag + "<br/>");
  
      }
  }
  ```

##### 1.4 Session与浏览器的关联

+ 当浏览器在没有任何cookie的情况下向服务器发送请求，服务器会创建一个Session会话并保存至服务器内存

+ 服务器每次创建一个Session对象时都会创建一个Cookie对象，这个cookie对象key的值是JSESSIONID，值为创建的Session的ID，并通过响应返回给浏览器端

+ 浏览器收到之后，立即创建一个Cookie对象与之对应，之后的每次访问服务器都会将这个Cookie一并发送给服务器

+ 服务器接收每次请求之后都会使用req.getSession()，利用浏览器的Cookie的value值中的SessionID找到第一次创建的Session

+ 所以，当把key值为JSESSIONID的Cookie对象删除之后，当浏览器发送请求的时候，由于没有Cookie携带的SessionID，服务器找不到原来的Session，不管原来的Session有没有超时都会新建一个Session会话

+ Session底层是通过Cookie实现的

+ Session与浏览器关联的示意图

  ![Session与浏览器的关联](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\Session与浏览器的关联.jpg)

+ 会话跟踪技术总结：

  客户端第一次发送请求给服务器，服务器获取Session，获取不到，则创建新的session，然后响应给客户端

  下次客户端给服务器发送请求时，会把session ID带给服务器，服务器获取到session，就能判断这次请求和上次请求是同一个请求，从而和其他请求区分开

+ session的常用API：

  request.getSession() -> 获取当前的会话，没有则创建一个新的会话

  request.getSession(false) -> 获取当前会话，没有则返回null，不会创建新的

  session.getId() -> 获取sessionID

  session.isNew() -> 判断当前session是否是最新的

+ Session保存作用域

  session保存作用域是和具体的某一个session对应的

  常用的API：

  session.setAttribute(K, V);

  session.getAttribute(K);

  ![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\03.session保存作用域.png)

#### 2. Cookie

##### 2.1 什么是Cookie

+ 1). Cookie实际上是一小段的文本信息(key-value格式)，是**服务器通知客户端保存键值对的一种技术**
+ 2). 客户端有了Cookie后，每次请求都发送给服务器(客户端浏览器会把Cookie保存起来，当浏览器再请求该网站时，浏览器把请求的网址连同该Cookie一同提交给服务器)
+ 3). 每个Cookie大小不能超过4KB

##### 2.2 Cookie的创建

+ 1). 创建Cookie对象，这时Cookie是保存在服务端的

  Cookie cookie = new Cookie(key, value);

+ 2). 通知客户端保存Cookie

  response.addCookie(cookie);

+ 3). Cookie创建的过程

  ![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\cookie的创建.jpg)
  
  ```java
  //1. 创建Cookie对象
  Cookie cookie = new Cookie("key1", "value1");
  //2. 将这个Cookie对象保存到浏览器端
  response.addCookie(cookie);
  ```
  

##### 2.3 Cookie的获取

+ 1). req.getCookies(); 返回Cookie对象的数组，获取所有的Cookie
+ 2). 获取指定的Cookie，首先获取Cookie数组，遍历数组逐个比较key值

##### 2.4 Cookie值的修改

+ 1). 方案一：

  + i. 先创建一个与要修改的Cookie同名的Cookie对象
  + ii. 在创建的构造器中，赋予Cookie新的值
  + iii. 传回给客户端，resp.addCookie(新创建的Cookie对象)

+ 2). 方案二

  + i. 先查找到需要修改的Cookie对象

  + ii. 调用setValue()方法赋予新的Cookie值

  + iii. 调用resp.addCookie()通知客户端

##### 2.5 Cookie的生命周期

+ 调用方法：setMaxAge(int time)
  *           time是正数：表示Cookie将在经过time秒之后过期
  *           time是负数：表示浏览器关闭，Cookie就会被删除(创建Cookie时，这个值默认是-1)
  *           time=0：表示立即删除Cookie

##### 2.6 Cookie的有效路径

+ Path属性的作用：有效过滤哪些Cookie可以发送给服务器，哪些不发
  *           CookieA    path = /工程路径
  *           CookieB    path = /工程路径/abc
  *           当请求路径为http://localhost:8080/工程路径/a.html时
  *           CookieA会被发送给服务器, CookieB不会被发送
  *           当请求路径为http://localhost:8080/工程路径/abc/a.html
  *           CookieA和CookieB都会被发送给服务器
+ 小路径下可以发送大路径下的Cookie，大路径下不能发送小路径下的Cookie

##### 2.7 Cookie的测试代码

```java
public class CookieTest extends BaseServlet {
	//cookie的有效路径
    protected void getPath(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie cookie = new Cookie("path", "pathTest");
        //req.getContextPath() 方法获取当前的工程路径
        cookie.setPath(req.getContextPath() + "/abc");
        resp.addCookie(cookie);
        resp.getWriter().write("创建了一个带有路径的cookie");
    }
	//cookie的默认生命周期
    protected void defaultLife(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie cookie = new Cookie("default", "defaultTime");
        cookie.setMaxAge(-1);  //设置Cookie的存活时间
        resp.addCookie(cookie);
    }
	//更新cookie的值
    protected void updateCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
//        方案一：
//           i. 先创建一个与要修改的Cookie同名的Cookie对象
//          ii. 在创建的构造器中，赋予Cookie新的值
        Cookie cookie = new Cookie("key1", "update1");
//         iii. 传回给客户端，resp.addCookie(新创建的Cookie对象)
        resp.addCookie(cookie);
        resp.getWriter().write("Cookie已修改");
    }
	//获取cookie的值
    protected void getCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie[] cookies = req.getCookies();
        for (Cookie cookie : cookies) {
            //getName()方法返回cookie的key; getValue()方法返回cookie的value
            resp.getWriter().write("Cookie[" + cookie.getName() + "=" + cookie.getValue() + "]<br/>");
        }
//        Cookie myCookie = null;
//        //如果想要特定的Cookie，则需要遍历cookie数组
//        for (Cookie cookie : cookies) {
//            //比较key的值
//            if("key1".equals(cookie.getName())){
//                myCookie = cookie;
//                break;
//            }
//        }
//        if(myCookie != null){
//            具体操作...
//        }
    }
	//创建cookie
    protected void createCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1. 创建Cookie对象
        Cookie cookie = new Cookie("key1", "value1");
        //2. 通知客户端保存Cookie(如果没有通知客户端，客户端就不会知道创建了cookie)
        resp.addCookie(cookie);

        Cookie cookie1 = new Cookie("key2", "value2");
        resp.addCookie(cookie1);
        resp.getWriter().write("Cookie创建成功");
    }
}
```

#### 3. Filter过滤器

##### 3.1 什么是Filter

+ Filter是JavaWeb的三大组件之一，是JavaEE的一个接口

##### 3.2 Filter的作用

+ Filter的作用主要是拦截请求和过滤响应
+ 拦截请求常用的应用场景：
  + 权限检查
  + 日记操作
  + 事务管理等

##### 3.3 Filter的使用步骤

+ 1). 编写一个实现Filter类的接口，注意是javax.servlet包下的Filter

+ 2). 实现主要的拦截方法：doFilter()

+ 3). 在web.xml中配置Filter的拦截路径

  ```xml
  <!--配置Filter-->
      <filter>
          <filter-name>FilterTest</filter-name>
          <filter-class>com.JavaWeb.Filter.FilterTest</filter-class>
      </filter>
  <!--    配置Filter的拦截路径-->
      <filter-mapping>
  <!--        表示当前的拦截路径给哪个Filter使用-->
          <filter-name>FilterTest</filter-name>
  <!--        配置拦截路径
              /：http://localhost:8080/工程路径/
              这里需要配置映射到IDEA web目录下的admin下的全部文件
  -->
          <url-pattern>/admin/*</url-pattern>
      </filter-mapping>
  ```

##### 3.4 Filter的生命周期

+ 1). 构造器方法
+ 2). init初始化方法
  + 第1). 2)步在web工程启动的时候执行，即Filter已经创建
+ 3). doFilter过滤方法
  + 每次拦截请求就会执行
+ 4). destroy销毁方法
  *           停止当前web工程时就会执行，也会销毁Filter过滤器

##### 3.5 FilterConfig类

+ FilterConfig类是Filter的配置文件类，类似于ServletConfig类与Servlet类的关系
+ 作用：获取Filter过滤器的配置内容
  + a. 获取Filter的名称 <filter-name>
  + b. 获取在Filter中配置的<init-param>中初始化的参数
  + c. 获取ServletContext对象

##### 3.6 FilterChain类

+ 1). doFilter()方法：

  + a. 如果有下一个过滤器，则执行下一个Filter过滤器
  + b. 如果没有下一个Filter，则执行目标资源

+ 2). 当有多个Filter时，执行顺序由xml配置文件决定，谁先配置就谁先执行

+ 3). 如果没有filterChain.doFilter()代码，那么就不会跳转到目标资源的页面或者执行之后的Filter代码，所以一定要加上

+ 4). 多个Filter过滤器执行的特点

  *           a. 所有的Filter和目标资源默认都执行在同一线程中
  *           b. 多个Filter共同执行时，它们共享同一个Request对象，因为它们都共同处理浏览器的同一个请求

+ 多个Filter执行过程概述图

  ![Filter执行过程](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\Filter执行过程.jpg)

##### 3.7 Filter拦截路径的匹配

+ 1). 精确匹配：

  ```xml
  <url-pattern>/target.jsp</url-pattern> <!--定位http://ip:port/工程名    必须定位至指定路径-->
  ```

*       2). 目录匹配 
        
        ```xml
        <url-pattern>/admin/*</url-pattern> <!-- 定位http://ip:port/工程名  必须定位到指定目录，可访问目录下的所有资源-->
        ```
        
*       3). 后缀名匹配
        
        ```xml
         <url-pattern>*.html</url-pattern> <!--表示请求地址以.html结尾才会拦截信息  文件类型可以随意指定-->
        ```

##### 3.8 Filter测试代码

```java
public class FilterTest implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    /**专门用于拦截请求，权限检查，过滤器始终在跳转页面之前执行过滤
     * @param servletRequest
     * @param servletResponse
     * @param filterChain
     * @throws IOException
     * @throws ServletException
     */
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        //使用Session来保存用户信息
        HttpSession session = httpServletRequest.getSession();
        Object user = session.getAttribute("user");
        //基本的登录检查
        if(user == null){
            //如果没有登录，则需要跳转到登录界面
            servletRequest.getRequestDispatcher("/login.jsp").forward(servletRequest, servletResponse);
        } else{
            //让程序继续往下访问用户的目标资源
            filterChain.doFilter(servletRequest, servletResponse);
        }
    }

    @Override
    public void destroy() {

    }
}

```

#### 4. JSON

##### 4.1 什么是JSON

+ JSON是一种轻量级的数据交换格式，易于人的阅读和编写，同时也易于机器的解析和生成；JSON采用完全独立于语言的文本格式，而且很多语言都提供了对JSON的支持
+ 数据交换指的是客户端和服务器之间业务数据的传递格式

##### 4.2 JSON的使用

+ JSON的定义：json由键值对组成，并且由大括号包围，每个键用“”包围，键值对用:分隔
+ JSON的访问：
  + json本身是一个对象
  + json中的key我们可以理解为是对象中的一个属性
  + json中key访问就跟访问对象的属性一样：jsonName.key
+ json的两个常用方法:json有两种存在形式，一种是对象一种是字符串，这两种存在形式可以相互转换
  + JSON.stringify()：把json对象转换成json字符串
  + JSON.parse()：把json字符串转换为json对象
  + 注意：当需要操作json中的数据时，使用json对象的形式；当要在客户端和服务器进行数据交换时，使用json的字符串形式

#### 5. AJAX

##### 5.1 什么是AJAX

+ AJAX是*A*synchronous *J*avaScript *A*nd *X*ML，即异步JavaScript和xml，是一种创建交互式网页应用的网页开发技术
+ AJAX是一种浏览器通过异步发起请求，局部更新页面的技术

##### 5.2 AJAX示例

+ 使用javascript语言发起AJAX请求，访问服务器中的AjaxServlet

  ```html
  <html>
      <head>
          <title>Ajax Test</title>
          <script type="text/javascript">
              //使用javascript语言发起ajax请求，访问服务器
              function ajaxRequest(){
                  //1. 首先创建XMLHttpRequest
                  
                  //2. 调用open方法设置请求参数
                  
                  //3. 调用send方法发送请求
                  
                  //4. 在send方法前绑定onreadystatechange事件，处理请求完成后的操作
              }
          </script>
      </head>
      <body>
          <button onclick="ajaxRequest">ajax request</button>
          <div id="div01"></div>
      </body>
  </html>
  ```

  ```java
  public class AjaxServlet extends BaseServlet{
      protected void javaScriptAjax(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          System.out.println("Ajax Request");
      }
  }
  ```


#### 6. Listener

##### 6.1 常用的Listener 

**ServletContextListener**

监听ServletContext对象的创建和销毁的过程

**HttpSessionListener**

监听session创建和销毁的过程

**ServletRequestListener**

监听ServletRequest的创建和销毁过程

Listener的使用：

```java
@WebListener
public class MyServletContextListener implements ServletContextListener{
    //当监听到ServletContext初始化时执行
	@Override
	public void contextInitialized(ServletContextEvent servletContextEvent){
		System.out.println("监听ServletContext到初始化");
	}
}

//在Listener中初始化ioc容器
@WebListener
public class MyServletContextListener implements ServletContextListener{
	@Override
	public void contextInitialized(ServletContextEvent servletContextEvent){
		BeanFactory beanFactory = new ClassPathXmlApplicationContext();
        ServletContext application = servletContextEvent.getServletContext();
        application.setAttribute("beanFactory", beanFactory);
        
	}
}
```



##### 6.2 其他Listener

**ServletContextAttributeListener**

监听ServletContext的保存作用域的改动

**HttpSessionAttributeListener**

监听Session的保存作用域的改动

**ServletRequestAttributeListener**

监听ServletRequest的保存作用域的改动



#### 7. MVC模型

MVC：Model（模型）、View（视图）、Controller（控制器）

+ 视图层：用于做数据展示以及和用户交互的一个界面

+ 控制层：能够接受客户端的请求，具体的业务功能还是需要借助于模型组件来完成

+ 模型层：模型分为很多种，有比较简单的pojo，有业务模型组件，有数据访问组件

  1). pojo：值对象（Javabean）

  2). DAO：数据访问对象（数据访问层）

  3). BO：业务对象（Service层）

  区分业务对象和数据访问对象：
        1） DAO中的方法都是单精度方法或者称之为细粒度方法。什么叫单精度？一个方法只考虑一个操作，比如添加，那就是insert操作、查询那就是select操作....
        2） BO中的方法属于业务方法，也实际的业务是比较复杂的，因此业务方法的粒度是比较粗的
            注册这个功能属于业务功能，也就是说注册这个方法属于业务方法。
            那么这个业务方法中包含了多个DAO方法。也就是说注册这个业务功能需要通过多个DAO方法的组合调用，从而完成注册功能的开发。
  注册：

  检查用户名是否已经被注册 - DAO中的select操作
  用户记录 - DAO中的insert操作

  向用户积分表新增一条记录（新用户默认初始化积分100分） - DAO中的insert操作
  条记录（某某某新用户注册了，需要根据通讯录信息向他的联系人推送消息） - DAO中的insert操作

  向系统日志表新增一条记录（某用户在某IP在某年某月某日某时某分某秒某毫秒注册） - DAO中的insert操作

  在库存系统中添加业务层组件

#### 8. 事物管理

涉及到的组件：
     OpenSessionInViewFilter
     TransactionManager
     ThreadLocal
     ConnUtil
     BaseDAO

事物管理的原理图：

![](C:\Users\86198\Desktop\JavaEE\JavaWeb\image\05.编程式事务管理03.png)

