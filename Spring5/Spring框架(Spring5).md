## Spring框架(Spring5)

### 一、Spring的基本概念

#### 1. 什么是Spring框架

+ Spring是轻量级的开源的JavaEE框架
+ Spring是一个轻量级的控制反转(IoC)和面向切面(AOP)的**容器框架**。

#### 2. Spring框架的目的

+ 解决企业应用开发的复杂性

#### 3. Spring的核心组成部分

+ IOC：反转控制，把创建对象的过程交给Spring进行管理
+ AOP：面向切面，在不修改源代码的情况下，进行功能增强
  + AOP是通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术。
  + AOP的主要功能：日志记录，性能统计，安全控制，事务处理，异常处理等等。

#### 4. Spring的特点

+ 方便解耦，简化开发
+ AOP编程的支持
+ 方便程序测试
+ 方便继承各种优秀框架
+ 方便进行事务操作
+ 降低API的开发难度

### 二、IOC容器(Inversion of Control)

#### 1. IOC的基本概念

+ 控制反转（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。
+ 通俗的讲，就是把对象的创建和对象的调用交给Spring管理
+ 底层原理：xml解析，工厂模式，反射

#### 2. IOC过程

+ (1). xml配置文件，配置创建对象

  ```xml
  <bean id="user" class="com.FirstSpring.Example.User"></bean>  <!--class属性为类的全路径-->
  ```

+ (2). 有service类和DAO类，创建工厂类

  ```java
  public class UserFactory{
      public static UserDAO getDAO(){
          String classlValue = bean标签下class属性的值   //1. xml解析得到
          Class c = Class.forName(classValue);   //2. 通过反射创建对象
          return (UserDAO)c.getDeclaredConstructor().newInstance();
      }
  }
  ```

+ 主要目的：进一步降低代码的耦合度

+ IOC底层实现：工行模式

  ![IOC底层实现1](C:\Users\86198\Desktop\JavaEE\Spring5\image\IOC底层实现1.jpg)

  
  
  ![IOC底层实现2](C:\Users\86198\Desktop\JavaEE\Spring5\image\IOC底层实现2.jpg)
  
  反射 + 配置文件解析：
  
  解析配置文件或许类的全类名，通过反射创建对象
  
  ![](C:\Users\86198\Desktop\JavaEE\Spring5\image\IOCReflect.jpg)
  
  

#### 3. IOC接口

+ IOC思想基于IOC容器完成，IOC容器底层就是对象工厂
+ IOC容器的目的：加载xml配置文件，并创建对象
+ IOC容器的实现方式(接口)：
  + BeanFactory：IOC容器的基本实现，是Spring内部使用的接口，一般不在开发中使用
    + 特点：加载配置文件时，不会创建实例对象，只有在获取对象时，BeanFactory才创建这个对象
  + ApplicationContext：BeanFactory的子接口，提供了更多更强大的子接口，开发中经常使用
    + 特点：在加载配置文件的时候就会创建对象
+ ApplicationContext接口的主要实现类
  + FileSystemXmlApplicationContext类：在寻找xml文件时，需要提供xml文件的绝对路径
  + ClassPathXmlApplicationContext类：在寻找xml文件时，在项目目录下的src文件夹下寻找，相对路径

#### 4. IOC操作：Bean管理

##### 4.1什么是Bean管理：

+ Spring创建对象
+ Spring注入属性

+ Bean管理的实现方式：
  + 基于xml配置文件方式实现
  + 基于注解方式实现

##### 4.2 基于xml方式的Bean管理操作

+ 创建对象

  + 在Spring配置文件中，使用bean标签，在标签内添加相应的属性
  + bean标签中的属性
    + id：唯一的标识，即别名
    + class：创建对象类的全路径
  + 使用bean标签时，默认使用的是类中的无参构造方法，所以类中需要有无参构造方法

+ 注入属性(给属性赋值)

  + DI：依赖注入，是IOC的一种具体实现

    + 可以通过在类中设置set方法实现

      ```xml
      <!--1. 创建对象-->
          <bean id="book" class="com.FirstSpring.Example.Book">
              <!--2. 使用property标签注入属性， name为属性名称 value为要赋的值-->
              <property name="name" value="Java"></property>
              <property name="author" value="Mike"></property>
              
              <!--null空值设置-->
              <property name="author">
                  <null/>
              </property>
              <!--当属性值中包含特殊符号，比如‘<’时
      			1. 可以将这些符号按照html语言的格式进行转义
      			2. 使用CDATA格式转换
      		-->
              <property name="author">
                  <value> <![CDATA[<<具体值>>]]> </value>
              </property>
          </bean>
      ```

      ```java
       @Test
          public void testBook(){
              //1. 创建对象
              ApplicationContext context = new ClassPathXmlApplicationContext("firstSpring.xml");
              //2. 获取创建的对象
              Book book = context.getBean("book", Book.class);
              System.out.println(book);
          }
      //使用的类
      public class Book {
          //创建属性
          private String name;
          private String author;
          //使用set方法进行属性注入
          public void setName(String name) {
              this.name = name;
          }
      
          public void setAuthor(String author) {
              this.author = author;
          }
      
          @Override
          public String toString() {
              return "Book{" +
                      "name='" + name + '\'' +
                      ", author='" + author + '\'' +
                      '}';
          }
      }
      ```
  
    + 通过有参构造器进行设置
  
      ```xml
      <!--1. 创建对象-->
          <bean id="book" class="com.FirstSpring.Example.Book">
              <!--2. 使用constructor-arg标签注入属性(有参构造器注入)-->
              <constructor-arg name="name" value="Python"></constructor-arg>
              <constructor-arg name="author" value="Jhon"></constructor-arg>
          </bean>
      ```
  
      ```java
       @Test
          public void testBook(){
              //1. 创建对象
              ApplicationContext context = new ClassPathXmlApplicationContext("firstSpring.xml");
              //第一个参数为bean标签中id属性的值，第二个参数为要实例化的类对象
              Book book = context.getBean("book", Book.class);
              System.out.println(book);
          }
      //使用的类
      public class Book {
          //创建属性
          private String name;
          private String author;
      
          public Book() {
          }
          //有参构造器注入属性
          public Book(String name, String author) {
              this.name = name;
              this.author = author;
          }
          @Override
          public String toString() {
              return "Book{" +
                      "name='" + name + '\'' +
                      ", author='" + author + '\'' +
                      '}';
          }
      }
      ```
  
    + p名称空间注入：简化基于xml配置方式
  
  + 注入属性的几中特殊情况：
  
    + 外部bean：示例，在service层中调用DAO层的方法，需要创建一个DAO层的对象
  
      ```java
      //在Service层调用DAO层的方法进行操作
      //DAO层接口
      public interface UserDAO {
          void update();
      }
      //DAO层实现类
      public class UserDAOimpl implements UserDAO{
      
          @Override
          public void update() {
              System.out.println("DOA update executing...");
          }
      }
      //Service层实现
      public class UserService {
          //创建UserDAO类型的属性，生成set方法
          private UserDAO userDAO;
          //在配置文件中配置
          public void setUserDAO(UserDAO userDAO) {
              this.userDAO = userDAO;
          }
      
          public void add(){
              System.out.println("add executing...");
              //原始方式调用
      //        UserDAO userDAO = new UserDAOimpl();
      //        userDAO.update();
              //使用Spring管理的调用方法
              userDAO.update();
          }
      }
      ```
    
      ```xml
      <!--1. 创建service和DAO对象-->
          <bean id="userService" class="com.SpringXML.service.UserService">
          <!--注入UserDAO对象
              ref属性：创建外部UserDAO对象的id值
          -->
              <property name="userDAO" ref="userDAO"></property>
          </bean>
          <bean id="userDAO" class="com.SpringXML.dao.UserDAOimpl">
      
          </bean>
      ```
  
      
  
    + 内部bean和级联赋值
    
      ```java
      //员工和部门的关系，一个部门对应多个员工，这种一对多的关系可以使用这样的注入方式
      //部门类
      public class Department {
          private String name;
      
          public void setName(String name) {
              this.name = name;
          }
      
          @Override
          public String toString() {
              return "Department{" +
                      "name='" + name + '\'' +
                      '}';
          }
      }
      //员工类
      public class Employee {
          private String eName;
          private String sex;
          //员工属于某一个部门
          private Department department;
      
          public void seteName(String eName) {
              this.eName = eName;
          }
      
          public void setSex(String sex) {
              this.sex = sex;
          }
      
          public void setDepartment(Department department) {
              this.department = department;
          }
      
          @Override
          public String toString() {
              return "Employee{" +
                      "eName='" + eName + '\'' +
                      ", sex='" + sex + '\'' +
                      ", department=" + department.toString() +
                      '}';
          }
      }
      ```
    
      ```xml
      <!--内部bean的属性注入-->
          <bean id="employee" class="com.SpringXML.bean.Employee">
              <!--1. 设置基本属性-->
              <property name="eName" value="Mike"></property>
              <property name="sex" value="boy"></property>
              <!--2. 设置对象类型的属性，内部属性的注入-->
              <property name="department">
                  <!--内部直接使用bean标签创建对象-->
                  <bean id="department" class="com.SpringXML.bean.Department">
                      <property name="name" value="security"></property>
                  </bean>
              </property>
          </bean>
      ```
      
    + 注入集合类型属性
    
      ```java
      //使用Student类完成对一些集合类型的属性注入
      public class Student {
          //1. 数组类型的属性
          private String[] lessons;
          //2. List集合类型的属性
          private List<String> list;
          //3. map集合类型的属性
          private Map<String, Integer> map;
      
          public void setLessons(String[] lessons) {
              this.lessons = lessons;
          }
      
          public void setList(List<String> list) {
              this.list = list;
          }
      
          public void setMap(Map<String, Integer> map) {
              this.map = map;
          }
      }
      ```
    
      ```xml
      <!--修改命名空间， 增加一个util的命名空间-->
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:util="http://www.springframework.org/schema/util"  
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
              http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
      
          <!--提取List集合的属性注入，也可以提取map，set-->
      <!--    <util:list id="">-->
      <!--  普通类型使用value标签      <value></value>-->
      <!--   自定义类使用ref标签      <ref bean=""></ref>-->
      <!--    </util:list>-->
      
      
      <!--集合类型属性的注入-->
          <bean id="student" class="com.SpringXML.CollectionType.Student">
              <!--1. 数组类型的属性注入-->
              <property name="lessons">
                  <array>
                      <value>Java</value>
                      <value>DataBase</value>
                  </array>
              </property>
              <!--2. List类型集合的属性注入-->
              <property name="list">
                  <list>
                      <value>Mike</value>
                      <value>Jhonson</value>
                      <!--当List中的元素是用户自定义类时，可以使用外部bean的注入方式进行属性注入-->
                      <!--<ref bean="">--> <!--bean属性的值是外部bean标签的id属性值-->
                      <!--<ref bean="">-->
                  </list>
              </property>
              <!--当有了外部创建的公共集合，可以直接使用外部公共创建的集合进行注入
      			ref属性的值，是外部util:list标签中id属性的值
      		-->
      <!--        <property name="" ref=""></property>-->
              <!--3. Map类型集合的属性注入-->
              <property name="map">
                  <map>
                      <entry key="Java" value="15"></entry>
                      <entry key="DataBase" value="20"></entry>
                  </map>
              </property>
          </bean>
          <!--在外部创建List元素对象的实例-->
      <!--    <bean id="" class=""></bean>-->
      </beans>
      ```
    
    + xml自动装配：根据指定装配规则(属性名称或者属性类型)，Spring自动将匹配的属性值进行注入
    
      ```java
      public class Depeartment{
          public String toString(){
              return "Depeartment{}";
          }
      }
      public class Employee{
          private Depeartment depeartment;
          public void setDepeartment(Depeartment depeartment){
              this.depeartment = depeartment;
          }
          public String toString(){
              return "Employee{depeartment:" + depeartment + "}"; 
          }
      }
      ```
    
      ```xml
      <!--实现自动装配
      	使用autowire属性配置自动装配
      	autowire属性常用的两个值：
      	byName根据属性名称注入: 注入值的bean的id值和类属性名称一样   
      	byType根据属性类型注入: 
      -->
      <bean id="employee" class="com.SpringXML.bean.Employee" autowire="byName/byType">
      	
      </bean>
      <bean id="department" class="com.SpringXML.bean.Department"></bean>
      ```
    

##### 4.3 Factory Bean

+ Spring中有两种bean，一种是普通bean；还有一种是工厂bean(Factory Bean)

+ 普通bean：在配置文件中定义bean类型，就是返回类型；之前所作的案例全部是普通bean

+ 工厂bean：在配置文件中定义bean类型可以和返回类型不一样

  + 第一步，创建类，让这个类作为工厂bean，实现接口FactoryBean

  + 第二步，实现接口里的方法，在实现的方法中定义返回的bean类型

    ```java
    //工厂bean的意思就是，这个类被定义为MyBean, 但是通过配置文件生成后，返回的却是Student类型的对象
    public class MyBean implements FactoryBean<Student> {
        //定义返回bean，返回什么类型的对象通过FactoryBean<T>中的泛型确定
        @Override
        public Student getObject() throws Exception {
            Student student = new Student();
            student.setLessons(new String[]{"Java", "Python", "DataBase"});
            return student;
        }
    
        @Override
        public Class<?> getObjectType() {
            return null;
        }
    
        @Override
        public boolean isSingleton() {
            return FactoryBean.super.isSingleton();
        }
    }
    ```

  + 第三步，编写配置文件

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    	<!--xml的配置文件与一般的配置文件没有区别-->
        <bean id="mybean" class="com.SpringXML.FactoryBean.MyBean">
    	<!--要注意的是，虽然class中的路径指向的是MyBean类型，但是这个配置生成的是Student
    		这个就是FactoryBean的特点-->
        </bean>
    </beans>
    ```

##### 4.4 bean标签的作用域

+ 在Spring中，设置创建bean实例是单实例还是多实例

  + 单实例是指每次创建的都是同一个对象，即每次创建的对象地址相同，在容器创建时创建对象
  + 多实例是指每次创建的对象都不同，即每次创建的对象地址不同，在每次获取时创建对象

+ 在Spring中，默认情况下，bean是单实例对象

+ 如何设置创建的对象是单实例还是多实例

  + xml bean标签中的scope属性

  + singleton：表示是单实例对象，在加载spring配置文件时就会创建单实例对象

  + prototype：表示是多实例对象，在调用getBean()方法时，创建多实例对象

    ```xml
    <bean id="student" class="com.SpringXML.CollectionType.Student" scope="prototype"></bean>
    ```

##### 4.5 bean的生命周期

+ 通过构造器*(空参或有参)*创建bean实例

+ 为bean的属性设置值和对其他bean的引用*(调用类中的set方法)*

+ 调用bean的初始化方法*(需要进行初始化方法的配置)*

+ 获取到bean对象，并使用

+ 当容器关闭时，调用bean的销毁方法(需要进行销毁方法的配置 )

  ```java
  public class Orders {
      private String name;
  	//1. 首先执行空参构造器创建对象
      public Orders() {
          System.out.println("First step. Constructor");
      }
  	//2. 调用set方法进行属性注入
      public void setName(String name) {
          this.name = name;
          System.out.println("Second step. Set...");
      }
      //3. 初始化方法，在xml文件中需要单独配置
      public void init(){
          System.out.println("Third step. init()...");
      }
      //5. 销毁方法，在xml文件中需要单独配置
      public void destroy(){
          System.out.println("Fifth step. destroy()...");
      }
  }
   @Test
      public void testBeanLife(){
          ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beanLife.xml");
          //4. 获取创建的bean对象
          Orders orders = context.getBean("orders", Orders.class);
          System.out.println("Fourth step. Get the Object...");
          System.out.println(orders);
          context.close();
  
      }
  ```

  

  ```xml
  <!--init-method属性配置初始化方法；destroy-method配置销毁方法-->
  <bean id="orders" class="com.SpringXML.bean.Orders" init-method="init" destroy-method="destroy">
      <property name="name" value="Phone"></property>
  </bean>
  ```

  ##### 4.6 基于注解方式的bean管理

  + 注解：
    + 什么是注解：注解是代码的特殊标记，格式：@注解名称(属性名=属性值,  属性名=属性值...)
    + 注解的作用范围：注解可以作用在类上面，方法上面以及属性上面
    + 注解的目的：简化xml配置

  + 创建对象

    + @Component
    + @Service
    + @Controller
    + @Repository
    + 上述四个注解的功能都是一样的，都可以用来创建bean实例

    ```xml
    <!--第一步，开启组件扫描，使用xml配置文件进行配置-->
    <?xml version="1.0" encoding="UTF-8"?>
    <!--需要导入一个命名空间context-->
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
        <!--开启组件扫描
            如果要扫描多个包
            1. 多个包使用"，"隔开
            2. 扫描包的上层目录
        -->
        <context:component-scan base-package="com.SpringAnnotation"></context:component-scan>
        
        
        <!--几个配置细节
            use-default-filters属性，表示不适用默认的filters，使用用户配置的filters，默认是扫描包下的所有类
            context:include-filter，设置扫描哪些内容
        -->
        <context:component-scan base-package="com.SpringAnnotation" use-default-filters="false">
            <!--表示只扫描带有Component注解的类-->
            <context:include-filter type="annotation" expression="org.springframework.stereotype.Component"/>
        </context:component-scan>
        <!--
            exclude-filter: 设置哪些内容不进行扫描
    
        -->
        <context:component-scan base-package="com.SpringAnnotation">
            <!--表示不扫描带有Component注解的类-->
            <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Component"/>
        </context:component-scan>
    </beans>
    ```

    ```java
    //第二步，创建类，并在类上加上注解
    //@Component @Service @Controller @Repository任意一个
    //value可以不写，默认创建对象的名称位类的名称，将首字母小写
    @Component(value = "userService") //和<bean id="" class""></bean>写法相同，value的值位创建对象的名称
    public class UserService {
        public void add(){
            System.out.println("add()...");
        }
    }
    //测试：
        @Test
        public void test1(){
            ApplicationContext context = new ClassPathXmlApplicationContext("annotation.xml");
            //第一个参数是注解中value的值
            UserService userService = context.getBean("userService", UserService.class);
            System.out.println(userService);
            userService.add();
        }
    ```
    
  + 注入属性

    + @AutoWired: 根据属性类型进行自动注入
    + @Qualifier: 根据属性的名称进行注入，要和@AutoWired一起使用

    ```java
    //场景：在service层中调用DAO层中的函数进行操作
    //第一步创建类，并在类上加上注解
    public interface UserDAO {
        void update();
    }
    //添加创建对象的注解
    @Repository(value = "userDAO")  //当一个类型有多个实现类时，创建对象时，可以使用value属性进行区分，在注入属性时可以用到
    public class UserDAOimpl implements UserDAO{
        @Override
        public void update() {
            System.out.println("Update information...");
        }
    }
    @Service(value = "userService")
    public class UserService {
        //第二步，定义属性，并添加注解
        //注意，不需要添加set方法
        @Autowired   //根据类型进行注入，但如果一个接口有多个实现类，那么这种方法就行不通
        @Qualifier(value = "userDAO")   //根据名称进行注入，注入的对象是创建时对象的名称，即注解中value属性的值，需要配合AutoWired注解进行使用
        private UserDAO userDAO;
        public void add(){
            System.out.println("UserService add()...");
            userDAO.update();
        }
    }
    ```

    + @Resource: 可以根据类型注入，也可以根据名称注入

    ```java
    @Service(value = "userService") //和<bean id="" class""></bean>写法相同，value的值位创建对象的名称
    public class UserService {
        //Resource是java自带的注解，不是Spring中的注解
        //@Resource  //单写一个@Resource表示按类型注入
        @Resource(name = "userDAO2")  //name属性按照名称注入，名称为创建对象时对象的名称
        private UserDAO userDAO;
        public void add(){
            System.out.println("UserService add()...");
            userDAO.update();
        }
    }
    ```

    + @Value: 注入普通类型：比如String，Integer等

  + 完全注解开发：脱离xml配置文件，一般在SpringBoot中使用

    + 第一步，创建配置类，代替xml配置文件

    ```java
    @Configuration
    @ComponentScan(basePackage = {"com.SpringAnnotation"})  //x相当于之前配置文件中<context:component-scan base-package="com.SpringAnnotation"></context:component-scan>
    public class SpringConfig{}
    ```

    + 第二步，编写测试类

    ```java
     @Test
        public void test3(){
            //加载配置类
            ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
            UserService userService = context.getBean("userService", UserService.class);
            System.out.println(userService);
            userService.add();
        }
    ```

    

### 三、AOP

#### 1. AOP的基本概念

##### 1.1 面向切面编程 

+ Aspect Oriented Programming ，通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术。
+ 利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

##### 1.2 AOP功能

+ 在不改变源代码的情况下，在主干功能中添加新功能

##### 1.3 AOP示例

![AOP示例](C:\Users\86198\Desktop\JavaEE\Spring5\image\AOP示例.jpg)

#### 2. AOP底层原理

##### 2.1 动态代理

+ 动态代理的概念：动态代理就是，在程序运行期，创建目标对象的代理对象，并对目标对象中的方法进行功能性增强的一种技术。

+ 有接口的动态代理-----JDK动态代理
  + 创建接口实现类的代理对象，增强类的方法
  
  ![JDK动态代理](C:\Users\86198\Desktop\JavaEE\Spring5\image\JDK动态代理.jpg)
  
+ 无接口的动态代理-----CGLIB动态代理
  +  创建目标类子类的代理对象，增强类方法
  
  ![CGLIB动态代理](C:\Users\86198\Desktop\JavaEE\Spring5\image\CGLIB动态代理.jpg)
  
+ JDK动态代理示例：

  + 步骤一，创建接口，定义方法

    ```java
    //定义接口，定义方法
    interface UserDAO{
        int add(int a, int b);
        String update(String id);
    }
    ```

  + 步骤二，创建接口的实现类，实现方法

    ```java
    //创建接口实现类，实现方法
    class User implements UserDAO{
    
        @Override
        public int add(int a, int b) {
            return a + b;
        }
    
        @Override
        public String update(String id) {
            return id;
        }
    }
    ```

  + 使用java.lang.Reflection.Proxy类中的newProxyInstance()方法创建接口代理对象

    + 参数一: ClassLoader    类加载器
    + 参数二: Class<>[] interfaces    增强方法所在的类，这个类实现的接口，可以有多个接口
    + 参数三: InvocationHandler(接口)    实现这个接口，调用接口中的invoke方法就相当于调用了被代理类中需要增强的方法

    ```java
    class Factory {
        //obj为被代理对象
        public static Object getProxyInstance(Object obj){
            //返回一个代理类的对象
            return Proxy.newProxyInstance(obj.getClass().getClassLoader(),obj.getClass().getInterfaces(),
                    new InvocationHandler() {
                        //在这个方法中编写需要增强的逻辑
                        @Override
                        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable 					  {
                            //假设在方法之前需要做一些处理
                            System.out.println("Before execute param : " + Arrays.toString(args));
                            //被增强的方法
                            Object object = method.invoke(obj, args);
                            //假设在方法之后需要做一些处理
                            System.out.println("After executing... " + obj);
                            return object;
                        }
                    })  ;
        }
    }
    ```

  + 测试方法：

    ```java
    public static void main(String[] args) {
        UserDAO userDAO = new User();
        UserDAO instance = (UserDAO) Factory.getProxyInstance(userDAO);
        //在这里调用add或者update方法时，会自动地调用InvocationHandler中的invoke方法
        System.out.println(instance.add(1, 2));
    }
    ```

##### 2.2 AOP术语

+ 连接点：类里面哪些方法可以被增强，这些方法称为连接点
+ 切入点：实际被真正增强的方法就被称为切入点
+ 通知 / 增强：实际增加的逻辑部分称为通知(比如给登录方法增加一个权限判断的功能)
  + 前置通知：在被增强方法执行之前执行
  + 后置通知：在被增强方法返回值之后执行，如果被增强方法抛出异常，则不执行
  + 环绕通知：在被增强方法执行之前和之后都执行，如果被增强方法中有异常，则在方法之后的那一部分不执行
  + 异常通知：在被增强方法中出异常时执行
  + 最终通知：在被增强方法之后执行，不管有没有异常均执行
+ 切面：把通知应用到切入点的过程(把权限判断增加到登录方法中这个过程)

#### 3. AOP操作

##### 3.1 AspectJ

+ Spring框架一般都是基于AspectJ实现AOP操作
+ AspectJ不是Spring的组成部分，是一个独立的AOP框架，一般把AspectJ和Spring框架一起使用，进行AOP操作

##### 3.2 基于AspectJ的AOP操作

+ xml配置文件实现

+ 注解方式实现

  + 步骤一，创建类，在里面定义方法，作为被代理类

    ```java
    //1. 创建基本类，作为被代理类
    @Component(value = "user")
    //3. 使用注解方式创建两个类的对象
    public class User {
        public void add(){
            System.out.println("add()...");
        }
    }
    ```

  + 步骤二，创建代理类，在里面写增强逻辑

    ```java
    //2. 创建代理类，增强被代理中的方法
    @Component(value = "userProxy")
    @Aspect  //3. 在增强类上添加Aspect注解
    public class UserProxy {
        //再增强类里面，在通知方法上面添加通知类型注解，使用切入点表达式进行配置
        //@Before表示前置通知，在被增强方法执行之前执行此方法
        @Before(value = "execution(* com.Spring.AopAnnotation.User.add(..))")
        public void before(){
            System.out.println("Before adding..." );
        }
    	//@After表示最终通知，不管有没有异常都会被执行
        //@AfterReturning表示后置通知，有异常则不执行
        @After(value = "execution(* com.Spring.AopAnnotation.User.add(..))")
        public void after(){
            System.out.println("After adding...");
        }
        //环绕通知---在被增强方法执行之前和之后都会执行这个方法
        @Around(value = "execution(* com.Spring.AopAnnotation.User.add(..))")
        public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
            System.out.println("before around...");
            //表示被增强方法执行
            proceedingJoinPoint.proceed();
            System.out.println("after around...");
    
        }
    }
    ```
  
  + 步骤三，进行通知配置
  
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:aop="http://www.springframework.org/schema/aop"   <!--添加命名空间-->
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                               http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                               http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
        <!--1. 开启注解扫描-->
        <context:component-scan base-package="com.Spring.AopAnnotation">
        </context:component-scan>
        <!--4. 开启AspectJ生成代理对象-->
        <aop:aspectj-autoproxy>
        </aop:aspectj-autoproxy>
    
    </beans>
    ```
  
  + 步骤四，配置不同类型的通知
  
    + @Before(value = "execution(* com.Spring.AopAnnotation.User.add(..))")
    + @After(value = "execution(* com.Spring.AopAnnotation.User.add(..))")
    + @Around(value = "execution(* com.Spring.AopAnnotation.User.add(..))")
    + ...
    
  + 几个细节
  
    + 相同切入点的提取，即代理类中有许多方法对被代理类中的同一个方法进行增强，可以提取相同切入点的切入点表达式
  
      ```java
      @Pointcut(value = "切入点表达式")
      public void pointDemo(){
          
      }
      @Before(value = "pointDemo()")
      public void before(){
          ...
      }
      ```
  
    + 如果有多个增强类对同一个方法进行增强，可以设置增强类之间的优先级
  
      ```java
      //PersonProxy同样对User类中的add方法进行增强
      @Component
      @Aspect
      @Order(1)   //@Order注解为每个增强类设置优先级，数字越小，优先级越高
      public class PersonProxy{
          public void before(){
              ...
          }
      }
      ```
  
      



### 四、JDBC Template

#### 1. JDBC Template的基本概念

##### 1.1 什么是JDBC Template

+ Spring框架对JDBC进行封装，使用JDBC Template方便实现对数据库操作

##### 1.2 准备工作

+ 在Spring的配置文件中配置数据库连接池需要的一些基本信息

  ```xml
  <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
      <property name="url" value="jdbc:sqlserver://localhost:1433;DatabaseName=userSpring"></property>
      <property name="username" value="sa"></property>
      <property name="password" value="abc"></property>
      <property name="driverClassName" value="com.microsoft.sqlserver.jdbc.SQLServerDriver"></property>
  </bean>
  ```

+ 配置JDBCTemplate对象，注入DataSource属性

  ```xml
  <!--创建JDBCTemplate对象-->
  <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
      <!--注入DataSource，外部bean方式-->
      <property name="dataSource" ref="dataSource"></property>
  </bean>
  ```

+ 根据web三层架构创建一些需要的类UserService和UserDAO，并使用注解创建对象和注入属性

  ```java
  @Repository(value = "userDAO")
  public class UserDAO {
      //UserDAO中注入JdbcTemplate操作数据库
      @Autowired
      private JdbcTemplate jdbcTemplate;
  }
  @Service(value = "userService")
  public class UserService {
      //UserService中注入UserDAO
      @Autowired
      private UserDAO userDAO;
  }
  
  ```


#### 2. JDBCTemplate操作数据库

##### 2.1 添加操作

+ **(1)**. 创建数据表对应的实体类

  ```java
  public class User {
      private int userId;
      private String username;
      private String ustatus;
  
      public void setUserId(int userId) {
          this.userId = userId;
      }
  
      public void setUsername(String username) {
          this.username = username;
      }
  
      public void setUstatus(String ustatus) {
          this.ustatus = ustatus;
      }
  }
  ```

+ **(2)**. 在DAO层使用JDBCTemplate中的update方法进行添加操作

  + String sql： 添加的SQL语句
  + Object ...args：可变参数，填充占位符

  ```java
  public void add(User user){
      //1. 创建SQL语句
      String sql = "insert into user_t(userId, username, ustatus) values(?,?,?)";
      //2. 调用方法实现
      int update = jdbcTemplate.update(sql, user.getUserId(), user.getUsername(), user.getUstatus());
      System.out.println(update);
  }
  ```

+ **(3)**. 测试类

  ```java
  @Test
  public void testAdd(){
      ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
      UserService userService = context.getBean("userService", UserService.class);
      User user = new User();
      user.setUserId(1);
      user.setUsername("Mike");
      user.setUstatus("VIP");
      userService.addUser(user);
  }
  
  输出：
  12月 07, 2021 7:31:08 下午 com.alibaba.druid.support.logging.JakartaCommonsLoggingImpl info
  信息: {dataSource-1} inited
  1
  ```

##### 2.2 修改和删除操作

+ 总体操作与添加相同，只是使用了不同的SQL语句

+ **(1)**. 在DAO层使用JDBCTemplate中的update方法进行添加操作

  ```java
  public void update(User user){
      String sql = "update user_t set username=?, ustatus=? where userId=?";
      int update = jdbcTemplate.update(sql, user.getUsername(), user.getUstatus(), user.getUserId());
      System.out.println(update);
  
  }
  
  public void delete(int id){
      String sql = "delete from user_t where userId = ?";
      int update = jdbcTemplate.update(sql, id);
      System.out.println(update);
  }
  ```

+ **(2)**. 测试类

  ```java
  @Test
  public void testUpdate(){
      ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
      UserService userService = context.getBean("userService", UserService.class);
      User user = new User();
      user.setUserId(1);
      user.setUsername("Jhon");
      user.setUstatus("VIP");
      userService.updateUser(user);
  }
  
  @Test
  public void testDelete(){
      ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
      UserService userService = context.getBean("userService", UserService.class);
      userService.deleteUser(3);
  }
  ```

##### 2.3 查询操作

###### 2.3.1 查询返回特定值

+ **(1)**. 在DAO层使用JDBCTemplate中的queryForObject方法进行查询操作

  + String sql：待查询的Sql语句
  + Class<T> type：返回值的类型

  ```java
  public int queryCount(){
      //可以查询其他返回特定值的聚合函数
      String sql = "select count(*) from user_t";
      Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
      if(count != null){
          return count;
      } else{
          return -1;
      }
  }
  ```

+ **(2)**. 测试方法

  ```java
  @Test
  public void testQuery1(){
      ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
      UserService userService = context.getBean("userService", UserService.class);
      int count = userService.queryCount();
      System.out.println(count);
  }
  
  输出：
  12月 07, 2021 7:55:41 下午 com.alibaba.druid.support.logging.JakartaCommonsLoggingImpl info
  信息: {dataSource-1} inited
  3
  
  ```

###### 2.3.2 查询返回特定对象

+ **(1)**. 在DAO层使用JDBCTemplate中的queryForObject方法进行查询操作，这个方法与查找特定值的方法不同

  + String sql：sql语句
  + RowMapper<T> rowMapper：本身是一个接口
  + Object ...args：填充占位符

  ```java
  public User queryByName(String name){
      String sql = "select * from user_t where username = ?";
      //BeanPropertyRowMapper<>()是RowMapper<>的一个实现类
      User user = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(User.class), name);
      return user;
  }
  ```

+ **(2)**. 测试方法

  ```java
  @Test
  public void testQuery2(){
      ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
      UserService userService = context.getBean("userService", UserService.class);
      User mike = userService.queryByName("Mike");
      if(mike != null){
          System.out.println(mike);
      }
      
  输出：
  12月 07, 2021 8:21:13 下午 com.alibaba.druid.support.logging.JakartaCommonsLoggingImpl info
  信息: {dataSource-1} inited
  User{userId=2, username='Mike', ustatus='VIP'}
  ```

###### 2.3.3 查询返回集合

+ **(1)**. 在DAO层使用JDBCTemplate中的query方法进行查询操作，参数同queryForObject方法

  ```java
  public List<User> queryForAll(){
      String sql = "select * from user_t";
      return jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(User.class));
  }
  ```

+ **(2)**. 测试方法

  ```java
  @Test
  public void testQuery3(){
      ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
      UserService userService = context.getBean("userService", UserService.class);
      List<User> users = userService.queryForAll();
      if(users != null){
          System.out.println(users.toString());
      }
  }
  
  输出：
  [User{userId=1, username='Jhon', ustatus='VIP'}, 
   User{userId=2, username='Mike', ustatus='VIP'}, 
   User{userId=3, username='Bob', ustatus='normal'}]
  ```

##### 2.4 批量操作

###### 2.4.1 批量添加

+ **(1)**. 在DAO层中使用JDBCTemplate中的batchUpdate()方法

  + String sql：sql语句
  + List<Object []> batchArgs：添加的多条记录的数据

  ```java
  public void batchAddUser(List<Object[]> batchArgs){
      //Object[]中是每条数据的值，比如[1,"Jhon","normal"]
      String sql = "insert into user_t(userId, username, ustatus) values(?,?,?)";
      int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
      System.out.println(Arrays.toString(ints));
  }
  ```

+ **(2)**. 测试方法

  ```java
  @Test
  public void testQuery4(){
      ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
      UserService userService = context.getBean("userService", UserService.class);
      List<Object[]> batchArgs = new ArrayList<>();
      Object[] o1 = {4, "Jhon", "normal"};
      Object[] o2 = {5, "Amy", "normal"};
      Object[] o3 = {6, "Jack", "VIP"};
      batchArgs.add(o1);
      batchArgs.add(o2);
      batchArgs.add(o3);
      userService.batchAdd(batchArgs);
  
  }
  
  输出：
  12月 09, 2021 2:50:37 下午 com.alibaba.druid.support.logging.JakartaCommonsLoggingImpl info
  信息: {dataSource-1} inited
  [1, 1, 1]
  ```

### 五、事务操作

#### 1. 事务的基本概念

##### 1.1 事务的定义

+ 事务是数据库操作最基本单元，逻辑上一组操作，要么都成功，如果有一个操作失败，所有操作都失败
+ 典型的场景：银行转账

##### 1.2 事务的特性(ACID)

+ **原子性（Automicity）**
  原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。 
+ **一致性（Consistency）**
  事务必须使数据库从一个一致性状态变换到另外一个一致性状态。
+ **隔离性（Isolation）**
  事务的隔离性是指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。
+ **持久性（Durability）**
  持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响。

##### 1.3 事务环境搭建

+ 基于银行转账的应用场景

  ![事务环境搭建](C:\Users\86198\Desktop\JavaEE\Spring5\image\事务环境搭建.jpg)

+ 创建一张表，添加相关记录

+ 创建service，搭建DAO，完成对象的创建和注入

  ```java
  //为了省事，省去了实现接口的部分
  @Repository(value = "userDAO")
  public class UserDAO {
      @Autowired
      private JdbcTemplate jdbcTemplate;
  }
  
  @Service(value = "userService")
  public class UserService {
      @Autowired
      UserDAO userDAO;
  }
  ```

+ 在DAO中创建两个方法，一个支出的方法，一个收钱的方法

  ```java
  //支出的方法
  public void addMoney(){
      String sql = "update account set money=money-? where username=?";
      jdbcTemplate.update(sql, 100, "Lucy");
  }
  //收钱的方法
  public void reduceMoney(){
      String sql = "update account set money=money+? where username=?";
      jdbcTemplate.update(sql, 100, "Mary")
  }
  ```

+ 在Service中创建转账的方法模拟转账的过程

  ```java
  public void accountMoney(){
      //Lucy支出100
      userDAO.reduceMoney();
  	//添加可能的异常，如果出现异常，执行结果就会出错
      int i = 10 / 0;
      //Mary增加100
      userDAO.addMoney();
  }
  ```

+ 测试方法

  ```java
  @Test
  public void test1(){
      ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
      UserService userService = context.getBean("userService", UserService.class);
      userService.accountMoney();
  }
  ```

##### 1.4 解决转账中途异常问题

+ 通过事务解决
+ 事务操作的基本过程
  + 开启事务操作
  + 进行业务上的操作
  + 如果没有异常，提交事务
  + 如果出现异常，事务回滚

#### 2. Spring中的事务操作

##### 2.1 Spring事务管理介绍

+ 事务一般都添加在Service层
+ 操作的方式
  + 编程式事务管理
  + 声明式事务管理(一般常用这种)
    + 注解方式
    + xml方式
+ 在Spring中进行声明式事务管理，底层使用AOP原理

##### 2.2 Spring中事务操作的API

+ 事务管理器接口，针对不同框架，提供不同的实现类----PlatformTransactionManager

##### 2.3 注解声明式事务管理

+ 在Spring配置文件中配置事务管理器

  ```xml
  <!--创建事务管理器-->
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
      <!--注入数据源-->
      <property name="dataSource" ref="dataSource">
      </property>
  </bean>
  ```

+ 在Spring文件中开启事务注解

  ```xml
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xmlns:tx="http://www.springframework.org/schema/tx"     引入名称空间
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                             http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                             http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
                             http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
      
      <!--开启事务注解-->
      <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
  ```

+ 在Service类上(或者在Service类中的方法上)添加事务注解

  ```java
  @Service(value = "userService")
  @Transactional //表示这个类中的所有方法都添加事务
  public class UserService
  ```

+ 执行结果：

  ![事务操作执行结果](C:\Users\86198\Desktop\JavaEE\Spring5\image\事务操作执行结果.jpg)

##### 2.4 声明式事务管理参数配置@Transaction注解中的参数

+ propagation：事务传播行为

  + 事务传播行为：事务传播行为（propagation behavior）指的就是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行。

    例如：methodA事务方法调用methodB事务方法时，methodB是继续在调用者methodA的事务中运行呢，还是为自己开启一个新事务运行，这就是由methodB的事务传播行为决定的。

  + PROPAGATION_REQUIRED：如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务。 

    ![](C:\Users\86198\Desktop\JavaEE\Spring5\image\事务传播Required.jpg)

    ```java
    @Transactional(propagation = Propagation.REQUIRED)
    public void methodA() {
     methodB();
    // do something
    }
     
    @Transactional(propagation = Propagation.REQUIRED)
    public void methodB() {
        // do something
    }
    /*
    由于@Transactional(propagation = Propagation.REQUIRED)的约束，刚开始，方法并不会直接创建事务
    单独调用methodB方法时，因为当前上下文不存在事务，所以会开启一个新的事务。 
    调用methodA方法时，因为当前上下文不存在事务，所以会开启一个新的事务。当执行到methodB时，methodB发现当前上下文有事务，因此就加入到当前事务中来。
    */
    ```

  + PROPAGATION_REQUIRED_NEW：表示当前方法必须运行在自己的事务中，如果一个事务已经存在，则先将这个存在的事务挂起。

    ![事务传播RequiredNew](C:\Users\86198\Desktop\JavaEE\Spring5\image\事务传播RequiredNew.jpg)

    ```java
    @Transactional(propagation = Propagation.REQUIRED)
    public void methodA() {
    doSomeThingA();
    methodB();
    doSomeThingB();
    // do something else
    }
    // 事务属性为REQUIRES_NEW
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void methodB() {
        // do something
    }
    /*
    如果methodA方法在调用methodB方法后的doSomeThingB方法失败了，methodB方法所做的结果依然被提交。而除了 methodB之外的其它代码导致的结果却被回滚了
    */
    ```

  + 其他事务传播

    ![事务传播总](C:\Users\86198\Desktop\JavaEE\Spring5\image\事务传播总.jpg)

+ isolation：事务隔离级别

  + 隔离性：多事务操作之间不会产生影响

    ```java
    @Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.READ_UNCOMMITTED) //表示这个类中的所有方法都添加事务
    public class UserService
    ```

  + 其他一些隔离性设置

    ![事务隔离性](C:\Users\86198\Desktop\JavaEE\Spring5\image\事务隔离性.jpg)

+ timeout：超时时间

  + 事务需要在一定时间内提交，如果不提交进行回滚
  + 默认是-1，即不超时，设置时间时以秒为单位

+ readOnly：是否只读

  + 读：查询，写：增删改

  + 默认是false，表示可以进行CRUD操作
  + 设置成true之后表示只能进行查询

+ rollbackFor：回滚

  + 设置出现哪些异常进行数据回滚

+ noRollbackFor：不回滚

  + 设置出现哪些异常但不进行数据回滚



### 六、Spring5的新特性

