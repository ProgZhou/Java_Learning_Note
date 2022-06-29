# SSM整合总结

### 一、整合概述

#### 1. SSM

+ **Spring**
+ **SpringMVC**
+ **Mybatis**

#### 2. 页面功能点

+ 分页显示数据
+ 数据校验
+ ajax请求
+ Rest风格URI

#### 3. 技术点

+ 基础框架---SSM
+ 数据库---SqlServer
+ 前端框架---bootStrap
+ 项目管理---Maven
+ 逆向工程---Mybatis Generator

### 二、环境搭建

#### 1. 创建一个maven工程

<img src="C:\Users\86198\Desktop\JavaEE\SSM\image\MavenCreate.jpg" style="zoom:80%;" />

#### 2. 导入相应的jar包

编写pom.xml管理相应的jar包

需要的基础jar包：

+ **spring** : 5.2.6.RELEASE
+ **springMVC** : 5.2.6.RELEASE
+ **mybatis** :  2.0.6
+ 数据库连接池druid : 1.1.10
+ mssql数据库驱动 : 9.2.1.jre11
+ junit测试单元 : 4.13.2
+ servlet-API : 3.1.0

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- $Id: pom.xml 642118 2008-03-28 08:04:16Z reinhard $ -->
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <packaging>war</packaging>

  <name>JavaSSMUnit</name>
  <groupId>com.JavaSSM</groupId>
  <artifactId>JavaSSMUnit</artifactId>
  <version>1.0-SNAPSHOT</version>

  <build>
    <plugins>
      <plugin>
        <groupId>org.mortbay.jetty</groupId>
        <artifactId>maven-jetty-plugin</artifactId>
        <version>6.1.7</version>
        <configuration>
          <connectors>
            <connector implementation="org.mortbay.jetty.nio.SelectChannelConnector">
              <port>8888</port>
              <maxIdleTime>30000</maxIdleTime>
            </connector>
          </connectors>
          <webAppSourceDirectory>${project.build.directory}/${pom.artifactId}-${pom.version}</webAppSourceDirectory>
          <contextPath>/</contextPath>
        </configuration>
      </plugin>
    </plugins>
  </build>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <spring.version>5.2.6.RELEASE</spring.version>
    <slf4j.version>1.6.6</slf4j.version>
    <log4j.version>1.2.12</log4j.version>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>compile</scope>
    </dependency>

    <!--SqlServer数据库驱动-->
    <dependency>
      <groupId>com.microsoft.sqlserver</groupId>
      <artifactId>mssql-jdbc</artifactId>
      <version>9.2.1.jre11</version>
    </dependency>


    <!--Druid数据库连接池-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.10</version>
    </dependency>

    <!-- ServletAPI -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>

    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.8.7</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>${spring.version}</version>
    </dependency>


    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <!--Spring的单元测试模块-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.thymeleaf</groupId>
      <artifactId>thymeleaf-spring5</artifactId>
      <version>3.0.12.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>2.0.6</version>
    </dependency>

    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.9</version>
    </dependency>


    <!-- 日志 -->
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.2.3</version>
    </dependency>
    <!--Mybatis逆向工程-->
    <dependency>
      <groupId>org.mybatis.generator</groupId>
      <artifactId>mybatis-generator-core</artifactId>
      <version>1.3.7</version>
    </dependency>

    <!--引入分页插件-->
    <dependency>
      <groupId>com.github.pagehelper</groupId>
      <artifactId>pagehelper</artifactId>
      <version>5.0.0</version>
    </dependency>

    <!--引入Json转换工具-->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.12.1</version>
    </dependency>
  </dependencies>
</project>
```

#### 3. 引入相应的前端框架

+ bootstrap : bootstrap-3.4.1-dist
+ jQuery : jquery-1.12.4.min.js

#### 4. 编写SSM整合的配置文件

##### 4.1 web.xml

+ 启动SpringIOC容器
+ 配置监听器
+ 配置过滤器
+ 配置前端控制器DispatchServlet

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4"
         xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
  <!--1. 启动spring容器-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <!--资源均放在resources文件夹下-->  
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <!--首先配置过滤器处理字符乱码问题-->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
      <param-name>forceResponseEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <!--配置HiddenHttpMethodFilter
      处理一些特殊的请求方式：put和delete
      将请求方式为post的请求，根据相应的条件转换为put或者delete请求
  -->
  <filter>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <!--配置前端控制器-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springMVC.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

##### 4.2 springMVC.xml

+ 配置视图解析器
+ 处理静态资源
+ 开启注解驱动

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <!--开启组件扫描-->
    <context:component-scan base-package="com.JavaSSMUnit.Controller"/>
    <!-- 配置Thymeleaf视图解析器 -->
    <!--通过bean配置创建一个java类-->
    <bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
        <!--设置当前视图解析器的优先级-->
        <property name="order" value="1"/>
        <!--设置编码-->
        <property name="characterEncoding" value="UTF-8"/>
        <!--当前使用的模板-->
        <property name="templateEngine">
            <!--内部bean-->
            <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
                <property name="templateResolver">
                    <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                        <!--后续可以根据相应的要求做修改-->
                        <!-- 视图前缀 -->
                        <property name="prefix" value="/WEB-INF/templates/"/>
                        <!-- 视图后缀 -->
                        <property name="suffix" value=".html"/>
                        <!--当前模板模型-->
                        <property name="templateMode" value="HTML5"/>
                        <!--当前页面编码-->
                        <property name="characterEncoding" value="UTF-8" />
                    </bean>
                </property>
            </bean>
        </property>
    </bean>
    <!--处理静态资源-->
    <mvc:default-servlet-handler />
    <mvc:annotation-driven />
</beans>
```

##### 4.3 mybatis-config.xml

+ mybatis全局配置文件
+ 配置分页处理器

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!--开启驼峰命名法则-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor"/>
    </plugins>
</configuration>
```

##### 4.4 applicationContext.xml

+ Spring配置文件
+ 开启注解扫描
+ 配置数据源
+ 整合Mybatis
+ 事务控制

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.0.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-2.0.xsd
                           http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

  <context:component-scan base-package="com.JavaSSMUnit">
    <!--不扫描springMVC下的控制层-->
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
  </context:component-scan>
  <!--引入外部的文件-->
  <context:property-placeholder location="classpath:jdbc.properties"/>

  <!--
        Spring配置文件的核心点
          数据源
          与Mybatis的整合
          事务控制
  -->
  <!--=============================配置数据库连接池的数据源==============================-->
  <bean id="poolDataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
  </bean>
<!--=================================================================================-->
<!--================================整合Mybatis=======================================-->
  <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!--指定mybatis全局配置文件的位置-->
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
    <!--配置数据源-->
    <property name="dataSource" ref="poolDataSource" />
    <!--指定Mybatis，sql映射文件-->
    <property name="mapperLocations" value="classpath:mapper/*.xml"/>
  </bean>
  <!--配置扫描器，将mybatis接口的实现加入到ioc容器中-->
  <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!--扫描所有的DAO接口的实现，加入到IOC容器中-->
    <property name="basePackage" value="com.JavaSSMUnit.DAO"/>
  </bean>

  <!--配置一个可以批量执行的sqlSession-->
  <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"/>
    <!--批量-->
    <constructor-arg name="executorType" value="BATCH"/>
  </bean>
<!--=================================================================================-->
<!--=================================事务控制=========================================-->
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!--控制住数据源-->
    <property name="dataSource" ref="poolDataSource"/>
  </bean>
  <!--开启基于注解的事务或者使用xml配置形式的事务(必要主要的都是使用xml配置式)-->
  <aop:config>
    <!--切入点表达式，扫描Service包下的所有方法-->
    <aop:pointcut id="txPoint" expression="execution(* com.JavaSSMUnit.Service..*(..))"/>
    <!--配置事务增强-->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPoint"/>
  </aop:config>

  <!--配置事务增强，事务如何切入-->
  <tx:advice id="txAdvice">
    <tx:attributes>
      <!--表示所有方法都是事务方法-->
      <tx:method name="*"/>
    </tx:attributes>
  </tx:advice>
<!--=================================================================================-->
</beans>
```

#### 5. 编写相应的JavaBean和Mapper

+ 使用Mybatis逆向工程自动生成相应的bean和mapper

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--配置数据库连接信息-->
        <jdbcConnection driverClass="com.microsoft.sqlserver.jdbc.SQLServerDriver"
                        connectionURL="jdbc:sqlserver://localhost:1433;DatabaseName=ssm"
                        userId="sa"
                        password="abc">
        </jdbcConnection>

        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
        <!--指定javaBean生成的位置-->
        <javaModelGenerator targetPackage="com.JavaSSMUnit.Bean" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!--指定sql映射文件生成的位置-->
        <sqlMapGenerator targetPackage="mapper"  targetProject=".\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!--指定DAO接口生成的位置-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.JavaSSMUnit.DAO"  targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!--指定每个表的生成策略-->
        <table tableName="emp_table" domainObjectName="Employee">

        </table>
        <table tableName="dept_table" domainObjectName="Department">
        </table>
    </context>
</generatorConfiguration>
```

+ 测试mbg.xml，生成bean

```java
public class MBGTest {
    public static void main(String[] args) throws Exception {
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        File configFile = new File("D:\\Java_idea\\JavaSSMUnit\\src\\main\\resources\\mbg.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
    }
}
```

+ 生成的Mapper

  ```java
  //...ByExample---按条件查询/修改/删除
  //...ByPrimaryKey---按主键查询/修改/删除
  public interface EmployeeMapper {
      long countByExample(EmployeeExample example);
      int deleteByExample(EmployeeExample example);
      int deleteByPrimaryKey(Integer id);
      int insert(Employee record);
      int insertSelective(Employee record);
      List<Employee> selectByExample(EmployeeExample example);
      Employee selectByPrimaryKey(Integer id);
      //在逆向工程的基础上自增两个查询方法，在查询员工信息的同时查询所对应的部门信息
      List<Employee> selectByExampleWithDepartment(EmployeeExample example);
      Employee selectByPrimaryKeyWithDepartment(Integer id);
      
      int updateByExampleSelective(@Param("record") Employee record, @Param("example") EmployeeExample example);
      int updateByExample(@Param("record") Employee record, @Param("example") EmployeeExample example);
      int updateByPrimaryKeySelective(Employee record);
      int updateByPrimaryKey(Employee record);
  }
  
  public interface DepartmentMapper {
      long countByExample(DepartmentExample example);
      int deleteByExample(DepartmentExample example);
      int deleteByPrimaryKey(Integer deptId);
      int insert(Department record);
      int insertSelective(Department record);
      List<Department> selectByExample(DepartmentExample example);
      Department selectByPrimaryKey(Integer deptId);
      int updateByExampleSelective(@Param("record") Department record, @Param("example") DepartmentExample example);
      int updateByExample(@Param("record") Department record, @Param("example") DepartmentExample example);
      int updateByPrimaryKeySelective(Department record);
      int updateByPrimaryKey(Department record);
  }
  ```
  
  

#### 6. 基本项目文件结构

<img src="C:\Users\86198\Desktop\JavaEE\SSM\image\projectStruct.jpg" style="zoom:80%;" />

### 三、查询逻辑

#### 1. 访问json.html页面

+ 使用bootstrap搭建前端页面
+ json.html页面直接发送ajax请求，进行员工数据的分页查询
+ 服务器后台处理请求，将数据以json字符串的形式返回给客户端
+ 客户端使用js对json进行解析，显示数据信息

##### 1.1 搭建前端页面

+ 前端页面使用thymeleaf语法
+ 使用bootstrap框架搭建基本页面

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>员工列表</title>
    <script type="text/javascript" th:src="@{/static/js/jquery-1.12.4.min.js}"></script>
    <link rel="stylesheet" th:href="@{/static/bootstrap-3.4.1-dist/css/bootstrap.min.css}"/>
    <script th:src="@{/static/bootstrap-3.4.1-dist/js/bootstrap.min.js}"></script>
</head>
<body>
<!--搭建显示页面-->
<div class="container">
    <!--标题-->
    <div class="row">
        <div class="col-md-12">
            <h1>SSM-CRUD</h1>
        </div>
    </div>
    <!--按钮-->
    <div class="row">
        <div class="col-md-4 col-md-offset-8">
            <button class="btn btn-primary" id="emp_add_model_btn">新增</button>
            <button class="btn btn-danger" id="emp_delete_all">删除</button>
        </div>
    </div>
    <!--表格数据-->
    <div class="row">
        <div class="col-md-12">
            <table class="table table-hover" id="emps_table">
                <thead>
                    <tr>
                        <th><input type="checkbox" id="choose_all"/></th>
                        <th>#</th>
                        <th>empName</th>
                        <th>gender</th>
                        <th>email</th>
                        <th>deptName</th>
                        <th>操作</th>
                    </tr>
                </thead>
                <tbody>

                </tbody>
            </table>
        </div>
    </div>
    <!--显示分页信息-->
    <div class="row">
        <!--分页文字信息-->
        <div class="col-md-6" id="page_info_area">
        </div>
        <!--分页条信息-->
        <div class="col-md-6" id="page_nav_area">
        </div>
    </div>
</div>
</body>
</html>
```

##### 1.2 发送ajax请求

+ 在html页面中发送ajax请求，解析返回的json数据串，在页面上显示员工数据

```javascript
<!--使用Ajax发送请求-->
<script type="text/javascript" th:inline="javascript">   //thymeleaf语法需要加上th:inline="javascript"属性
    // var basePath = [[${#httpServletRequest.getScheme() + "://"
    // + #httpServletRequest.getServerName() + ":"
    // + #httpServletRequest.getServerPort() + #httpServletRequest.getContextPath()}]];
    //1. 当页面加载完成之后，直接发送一个Ajax请求，获取分页数据
    $(function () {
       to_page(1);
    });
	//发送跳转到指定页面的请求
    function to_page(pn) {
        $.ajax({
            //thymeleaf视图会自动将这个url解析为http://localhost:8080/ssm/empJson
            url: [[@{/empsJson}]],    
            data: "pn=" + pn,
            type: "GET",
            success:function(result) {
            //console.log(result);
            //1. 解析并显示员工数据
            build_emps_table(result);
            //2. 解析并显示分页信息
            build_page_info(result);
            //3. 解析显示分页条数据
            build_page_nav(result);
        }
    });
    }
</script>
```

#### 2. 创建控制器

##### 2.1 创建跳转页面的处理请求

```java
//跳转的请求处理，专门放在IndexController类中
@Controller
public class IndexController {
    @RequestMapping("/")
    public String index(){
        return "json";
    }
}
```

##### 2.2 EmployeeController

+ 导入分页处理插件

```xml
<!--引入分页插件-->
<dependency>
	<groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.0.0</version>
</dependency>
```

+ 专门处理员工CRUD请求的控制类

```java
//专门处理员工的CRUD请求
@Controller
public class EmployeeController {
    @Autowired
    EmployeeService employeeService;
    //返回Json数据，方便Ajax进行处理
    @RequestMapping("/empsJson")
    @ResponseBody
    public Message getEmpsWithJson(@RequestParam(value = "pn", defaultValue = "1") Integer pn){
        //在查询之前只需要调用如下方法，传入页码以及每页的大小
        PageHelper.startPage(pn, 5);
        //startPage后面紧跟着的一个查询方法就是分页查询
        List<Employee> employees = employeeService.getAll();
        //使用PageInfo包装查询后的结果，只需要将PageInfo交给页面，封装了详细的信息，包括查询出来的信息
        PageInfo<Employee> pageInfo = new PageInfo<Employee>(employees, 5);
        return Message.success().add("pageInfo", pageInfo);
    }
}
```

##### 2.3 创建通用返回类

+ 导入相关json包

```xml
<!--引入Json转换工具-->
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.12.1</version>
</dependency>
```

+ 创建通用返回类，封装需要返回给客户端的响应数据

```java
/*
* 通用的返回类
* 封装数据，对象以Json形式返回
* */
public class Message {
    //状态码，100表示处理成功，200表示处理失败
    private int code;
    //提示信息
    private String message;
    //用户要返回给浏览器的信息
    private Map<String, Object> extend = new HashMap<>();

    public static Message success(){
        Message result = new Message();
        result.setCode(100);
        result.setMessage("success!");
        return result;
    }
    public static Message failed(){
        Message result = new Message();
        result.setCode(200);
        result.setMessage("failed!");
        return result;
    }
    public Message add(String key, Object value){
        this.getExtend().put(key, value);
        return this;
    }
    public int getCode() {
        return code;
    }
    public void setCode(int code) {
        this.code = code;
    }
    public String getMessage() {
        return message;
    }
    public void setMessage(String message) {
        this.message = message;
    }
    public Map<String, Object> getExtend() {
        return extend;
    }
    public void setExtend(Map<String, Object> extend) {
        this.extend = extend;
    }
}
```

#### 3. 浏览器解析json数据

```javascript
//解析并显示员工数据
    function build_emps_table(result) {
        $("#emps_table tbody").empty();
        var emps = result.extend.pageInfo.list;
        $.each(emps, function (index, item){
            //构建显示员工信息表格
            var checkBoxTD = $("<td><input type='checkbox' class='check_item'/></td>")
            var empIdTD = $("<td></td>").append(item.id);
            var empNameTD = $("<td></td>").append(item.empName);
            var genderTD = $("<td></td>").append(item.gender == 'M' ? "男" : "女");
            var emailTD = $("<td></td>").append(item.email);
            var deptName = $("<td></td>").append(item.dept.deptName);
            var editButton = $("<button></button>").addClass("btn btn-primary btn-sm edit_btn")
                .append($("<span></span>").addClass("glyphicon glyphicon-pencil")).append("编辑");
            editButton.attr("edit_id", item.id);
            var deleteButton = $("<button></button>").addClass("btn btn-danger btn-sm delete_btn")
                .append($("<span></span>").addClass("glyphicon glyphicon-trash")).append("删除");
            deleteButton.attr("del_id", item.id);
            var btnTD = $("<td></td>").append(editButton).append(" ").append(deleteButton);
           $("<tr></tr>").append(checkBoxTD).append(empIdTD).append(empNameTD).append(genderTD)
               .append(emailTD).append(deptName).append(btnTD)
               .appendTo("#emps_table tbody");
        });
    }
    //解析显示分页信息
    function build_page_info(result){
        $("#page_info_area").empty();
            $("#page_info_area").append("当前第" + result.extend.pageInfo.pageNum +"页，总共"+
            result.extend.pageInfo.pages +"页，共"+ result.extend.pageInfo.total +"项");
            totalRecord = result.extend.pageInfo.total;
            currentPage = result.extend.pageInfo.pageNum;
    }

    //解析显示分页条
    function build_page_nav(result) {
        $("#page_nav_area").empty();
        var ul = $("<ul></ul>").addClass("pagination");

        var firstPageLi = $("<li></li>").append($("<a></a>").append("首页").attr("href", "#"));
        var preLi = $("<li></li>").append($("<a></a>").append("&laquo;"));
        var nextLi = $("<li></li>").append($("<a></a>").append("&raquo;"));
        var lastPageLi = $("<li></li>").append($("<a></a>").append("末页").attr("href", "#"));
        //当页面在首页或者末页时，禁用向前，向后的点击翻页事件
        if(result.extend.pageInfo.hasPreviousPage == false){
            firstPageLi.addClass("disabled");
            preLi.addClass("disabled");
        }
        if(result.extend.pageInfo.hasNextPage == false){
            lastPageLi.addClass("disabled");
            nextLi.addClass("disabled");
        }
        //为元素添加点击翻页事件
        firstPageLi.click(function () {
           to_page(1);
        });
        preLi.click(function () {
           to_page(result.extend.pageInfo.pageNum - 1);
        });
        nextLi.click(function () {
            to_page(result.extend.pageInfo.pageNum + 1);
        });
        lastPageLi.click(function () {
           to_page(result.extend.pageInfo.pages);
        });
        //添加首页和前一页的显示
        ul.append(firstPageLi).append(preLi)
        //添加每一页的显示
        $.each(result.extend.pageInfo.navigatepageNums, function (index, item) {
           var numLi = $("<li></li>").append($("<a></a>").append(item));
           if(result.extend.pageInfo.pageNum == item){
               numLi.addClass("active");
           }
           numLi.click(function(){
               to_page(item);
           });
           ul.append(numLi);
        });
        //添加下一页和末页的提示
        ul.append(nextLi).append(lastPageLi);
        var navEle = $("<nav></nav>").append(ul);
        navEle.appendTo("#page_nav_area")
    }
```

#### 4. 页面效果

![](C:\Users\86198\Desktop\JavaEE\SSM\image\indexResult.jpg)

### 四、新增逻辑

+ 在页面上点击"新增"按钮
+ 弹出一个新增信息的对话框
+ 在数据库中查询所有的部门信息，显示在对话框中的部门下拉列表中
+ 用户输入数据，进行校验
  + jQuery校验，数据合法性校验
  + ajax用户名重复校验
+ 点击对话框上的"save"保存数据

#### 1. 弹出对话框

##### 1.1 创建对话框

```html
<!-- 员工添加 -->
<div class="modal fade" id="empAddModel" tabindex="-1" role="dialog" aria-labelledby="myModalLabel">
    <div class="modal-dialog" role="document">
        <div class="modal-content">
            <div class="modal-header">
                <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
                <h4 class="modal-title" id="myModalLabel">员工添加</h4>
            </div>
            <div class="modal-body">
                <form class="form-horizontal">
                    <div class="form-group">
                        <label for="empName_add_input" class="col-sm-2 control-label">Name</label>
                        <div class="col-sm-10">
                            <input type="text" name="empName" class="form-control" id="empName_add_input" placeholder="empName">
                            <span class="help-block"></span>
                        </div>
                    </div>
                    <div class="form-group">
                        <label for="email_add_input" class="col-sm-2 control-label">email</label>
                        <div class="col-sm-10">
                            <input type="text" name="email" class="form-control" id="email_add_input" placeholder="email@qq.com">
                            <span class="help-block"></span>
                        </div>
                    </div>
                    <div class="form-group">
                        <label class="col-sm-2 control-label">gender</label>
                        <div class="col-sm-10">
                            <label class="radio-inline">
                                <input type="radio" name="gender" id="gender1_add_input" value="M"> 男
                            </label>
                            <label class="radio-inline">
                                <input type="radio" name="gender" id="gender2_add_input" value="F"> 女
                            </label>
                        </div>
                    </div>

                    <div class="form-group">
                        <label class="col-sm-2 control-label">department
                        </label>
                        <div class="col-sm-10">
                            <select class="form-control" name="department" id="dept_add_select">
                            </select>
                        </div>
                    </div>
                </form>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default btn-sm" data-dismiss="modal">Close</button>
                <button type="button" class="btn btn-primary btn-sm" id="emp_save_btn">Save changes</button>
            </div>
        </div>
    </div>
</div>
```

##### 1.2 给新增按钮绑定单级事件

```javascript
//绑定模态框的单击事件，点击新增按钮，弹出模态框
$("#emp_add_model_btn").click(function () {
    //每次单击事件需要重置表单
    $("#empAddModel form")[0].reset();
    //发送ajax请求，查出部门信息，显示在下拉列表中
    getDepts("#empAddModel select");
    //弹出模态框
    $("#empAddModel").modal({
       backdrop:"static"
   })
});
```

#### 2. 部门信息请求发送和处理

##### 2.1 发送请求

+ 在点击新增按钮的同时发送一个查询部门信息的请求
+ 解析服务器端发送的json数据，并显示

```javascript
function getDepts(ele) {
    $(ele).empty();
    $.ajax({
        url:[[@{/depts}]],
        type:"GET",
        success:function (result) {
            //console.log(result);
            //显示部门信息在下拉列表中
            $.each(result.extend.depts, function () {
                var optionEle = $("<option></option>").append(this.deptName).attr("value", this.deptId);
                optionEle.appendTo(ele);
            });
        }
    });
}
```

##### 2.2 服务器端处理请求

+ 查询部门信息并返回json数据串

```java
//处理和部门有关的请求
@Controller
public class DepartmentController {
    @Autowired
    DepartmentService departmentService;
    //返回所有的部门信息
    @RequestMapping("/depts")
    @ResponseBody
    public Message getDepartments(){
        List<Department> departments = departmentService.getDepts();
        return Message.success().add("depts", departments);
    }
}
```

##### 2.3 页面显示效果

![](C:\Users\86198\Desktop\JavaEE\SSM\image\insertResult.jpg)

#### 3. 保存员工信息

##### 3.1 前端数据校验

+ 使用jQuery校验用户名和邮箱格式是否合法

```javascript
//抽取非法数据的处理方式
    function show_validate_msg(ele, status, msg) {

        $(ele).parent().remove("has-success has-error");
        $(ele).next("span").text("");
        if(status == "success"){
            $(ele).parent().addClass("has-success");
            $(ele).next("span").text(msg);
        } else if(status == "error"){
            $(ele).parent().addClass("has-error");
            $(ele).next("span").text(msg);
        }
    }
//检验表单数据
	function validate_add_form(){
        //1. 拿到要校验的数据，使用正则表达式进行校验
        var empName = $("#empName_add_input").val();
        var regName = /(^[a-zA-Z0-9_-]{3,16}$)|(^[\u2E80-\u9FFF]{2,5})/;
        if(!regName.test(empName)){
            //alert("员工姓名需要是3-16位英文或者2-5位中文！");
            show_validate_msg("#empName_add_input", "error", "员工姓名需要是3-16位英文或者2-5位中文！");
            return false;
        } else{
            show_validate_msg("#empName_add_input", "success", "");
        }
        //2. 校验邮箱信息
        var email = $("#email_add_input").val();
        var regEmail = /^([a-z0-9_\.-]+)@([\da-z\.-]+)\.([a-z\.]{2,6})$/;
        if(!regEmail.test(email)){
            //alert("邮箱格式不正确");
           show_validate_msg("#email_add_input", "error", "邮箱格式不正确");
            return false;
        } else{
            show_validate_msg("#email_add_input", "success", "");
        }
        return true;
    }
```

##### 3.2 检验用户名是否重复

+ 在点击保存按钮时，发送ajax请求检查员工姓名是否有重复

```javascript
$("#empName_add_input").change(function () {
   //发送ajax请求，校验用户名是否可用
    var empName = this.value;
    $.ajax({
        url: [[@{/checkEmp}]],
        data: "empName=" + empName,
        type: "POST",
        success: function (result) {
            if(result.code == 100){
                show_validate_msg("#empName_add_input", "success", "");
                //给保存按钮自定义一个属性，判断姓名在数据库中是否存在
                $("#emp_save_btn").attr("ajax-va", "success");
            } else{
                show_validate_msg("#empName_add_input", "error", "不可用");
                $("#emp_save_btn").attr("ajax-va", "error");
            }
        }
    })
});
```

+ 服务器端创建控制器方法，查询员工数据检验是否重复

```java
//查询员工是否重复
@RequestMapping("/checkEmp")
@ResponseBody
public Message checkEmployee(@RequestParam("empName") String empName){
    boolean b = employeeService.checkEmp(empName);
    if(b){
        return Message.success();
    } else{
        return Message.failed();
    }
}

//EmployeeService
//查询员工信息是否重复
    //返回true表示记录可用，返回false表示当前不可用
    public boolean checkEmp(String empName){
        //条件查询，创建条件，传入员工姓名
        EmployeeExample example = new EmployeeExample();
        EmployeeExample.Criteria criteria = example.createCriteria();
        criteria.andEmpNameEqualTo(empName);
        //如果查询到信息，返回查询到的数目
        long count = employeeMapper.countByExample(example);
        return count == 0;
    }
```

##### 3.3 新增员工

+ 校验全部成功，发送ajax请求，新增员工信息

```javascript
//绑定保存按钮的单击事件
$("#emp_save_btn").click(function () {
    //1. 先对提交给服务器的数据进行校验
    if(!validate_add_form()){
        return false;
    }
    //判断之前的ajax校验用户名是否成功
    if($(this).attr("ajax-va") == "error") {
        return false;
    }
    //2. 发送ajax请求保存员工
    $.ajax({
        url:[[@{/emp}]],
        type: "POST",
        data: $("#empAddModel form").serialize(),
        success: function (result) {
            //alert(result.message);
            //员工保存成功
            //1. 关闭模态框
            $("#empAddModel").modal('hide');
            //2. 来到最后一页，显示刚才的数据
            to_page(totalRecord);
        }
    });
});
```

+ 服务器端创建控制器方法，处理请求

```java
//新增员工信息
@RequestMapping(value = "/emp", method = RequestMethod.POST)
@ResponseBody
public Message saveEmployee(Employee employee){
    employeeService.saveEmployee(employee);
    return Message.success();
}

//EmployeeService
//员工保存方法
    public void saveEmployee(Employee employee){
        employeeMapper.insertSelective(employee);
    }
```

### 五、修改逻辑

#### 1. 弹出对话框

##### 1.1 创建修改对话框

```html
<!-- 员工修改 -->
<div class="modal fade" id="empUpdateModel" tabindex="-1" role="dialog" aria-labelledby="myModalLabel">
    <div class="modal-dialog" role="document">
        <div class="modal-content">
            <div class="modal-header">
                <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
                <h4 class="modal-title">员工修改</h4>
            </div>
            <div class="modal-body">
                <form class="form-horizontal">
                    <div class="form-group">
                        <label for="empName_add_input" class="col-sm-2 control-label">Name</label>
                        <div class="col-sm-10">
                            <p class="form-control-static" id="static_update_empName"></p>
                        </div>
                    </div>
                    <div class="form-group">
                        <label for="email_add_input" class="col-sm-2 control-label">email</label>
                        <div class="col-sm-10">
                            <input type="text" name="email" class="form-control" id="email_update_input" placeholder="email@qq.com">
                            <span class="help-block"></span>
                        </div>
                    </div>
                    <div class="form-group">
                        <label class="col-sm-2 control-label">gender</label>
                        <div class="col-sm-10">
                            <label class="radio-inline">
                                <input type="radio" name="gender" id="gender1_update_input" value="M"> 男
                            </label>
                            <label class="radio-inline">
                                <input type="radio" name="gender" id="gender2_update_input" value="F"> 女
                            </label>
                        </div>
                    </div>

                    <div class="form-group">
                        <label class="col-sm-2 control-label">department
                        </label>
                        <div class="col-sm-10">
                            <select class="form-control" name="department" id="dept_update_select">

                            </select>
                        </div>
                    </div>
                </form>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default btn-sm" data-dismiss="modal">Close</button>
                <button type="button" class="btn btn-primary btn-sm" id="emp_update_btn">Save changes</button>
            </div>
        </div>
    </div>
</div>
```

##### 1.2 给表格中的每个编辑按钮添加单击事件

```javascript
//在之前创建表格时：var editButton = $("<button></button>").addClass("btn btn-primary btn-sm edit_btn")
//                .append($("<span></span>").addClass("glyphicon glyphicon-pencil")).append("编辑");
//绑定编辑按钮的单击事件，显示编辑的模态框
$(document).on("click", ".edit_btn", function () {
    //1. 查出部门信息，并显示部门列表
    getDepts("#empUpdateModel select");
    //2. 查出员工信息，显示员工信息
    getEmp($(this).attr("edit_id"));
    //getDepts("#empUpdateModel select");
    //3. 把员工的id传递给模态框的更新按钮
    $("#emp_update_btn").attr("edit_id", $(this).attr("edit_id"));
    $("#empUpdateModel").modal({
        backdrop: "static"
    });
});
```

##### 1.3 页面显示效果

+ 部门显示与之前新增逻辑中的做法相同
+ 校验做法也相同

![](C:\Users\86198\Desktop\JavaEE\SSM\image\updateResult.jpg)

#### 2. 保存修改的员工信息

+ 创建控制器方法，处理修改员工信息的请求
+ 使用Restful风格的URI，修改请求使用put请求方式

```java
	@RequestMapping(value = "/emp/{id}", method = RequestMethod.PUT)
    @ResponseBody
    public Message updateEmployee(Employee employee){
//        System.out.println(employee);
        employeeService.updateEmployee(employee);
        return Message.success();
    }

//EmployeeService
//更新员工信息
    public void updateEmployee(Employee employee){
        employeeMapper.updateByPrimaryKeySelective(employee);
    }
```

+ 页面点击保存按钮，发送ajax请求

```javascript
//点击编辑按钮，更新员工信息
$("#emp_update_btn").click(function () {
   //验证邮箱格式是否合法
    var email = $("#email_update_input").val();
    var regEmail = /^([a-z0-9_\.-]+)@([\da-z\.-]+)\.([a-z\.]{2,6})$/;
    if(!regEmail.test(email)){
        //alert("邮箱格式不正确");
        show_validate_msg("#email_update_input", "error", "邮箱格式不正确");
        return false;
    } else{
        show_validate_msg("#email_update_input", "success", "");
    }
    //发送ajax请求
    $.ajax({
        url: [[@{/emp/}]] + $(this).attr("edit_id"),
        type: "POST",
        //这里需要注意，请求会被过滤器处理，封装为put请求发送给服务器端
        data: $("#empUpdateModel form").serialize() + "&_method=put",
        success: function (result) {
            alert(result.message);
           //1.关闭对话框
            $("#empUpdateModel").modal("hide");
           //2. 回到本页面
            to_page(currentPage);
        }
    });
});
```

### 六、删除逻辑

+ 单个删除：
  + 点击表格中每行后面的删除按钮，弹出询问框，询问是否删除
  + 点击确定，发送请求，后端服务器将数据删除
  + 处理完成后返回当前页面
+ 批量删除：
  + 选中每行数据前的复选框，点击表头的删除按钮，弹出询问框
  + 点击确定，服务器后台批量删除数据
  + 处理完成后返回当前页面

#### 1. 单个删除按钮单击事件

+ 单击删除按钮弹出询问框
+ 选择确定，发送ajax请求

```javascript
//绑定删除按钮的单击事件
$(document).on("click", ".delete_btn", function () {
   //1. 弹出是否确认删除对话框
    var empName = $(this).parents("tr").find("td:eq(2)").text();
    if(confirm("确认删除 " + empName + "吗?")){
        //确认，发送ajax请求
        $.ajax({
            url: [[@{/emp/}]] + $(this).attr("del_id"),
            type: "POST",
            data: "&_method=delete",
            success: function (result) {
                //alert("success");
                to_page(currentPage);
            }
        });
    }
});
```

#### 2. 批量删除按钮单级事件

+ 复选框的单击事件，选中/不选中，全选/单选

```javascript
//完场全选和全不选
$("#choose_all").click(function () {
   //使用prop获取dom原生属性的值
    $(".check_item").prop("checked", $(this).prop("checked"));
});

$(document).on("click", ".check_item", function () {
    //判断当前选择的元素是否有5个
    if($(".check_item:checked").length == $(".check_item").length){
        $("#choose_all").prop("checked", true);
    } else{
        $("#choose_all").prop("checked", false);
    }
});
```

+ 点击删除按钮，发送ajax请求，批量删除

```javascript
//点击全部删除，批量删除员工信息
$("#emp_delete_all").click(function () {
    var empNames = "";
    var del_ids = "";
    $.each($(".check_item:checked"), function () {
       empNames += $(this).parents("tr").find("td:eq(2)").text() + " ";
       del_ids += $(this).parents("tr").find("td:eq(1)").text() + "-";
    });
    del_ids = del_ids.substring(0, del_ids.length - 1);
    if(confirm("确认删除 " + empNames + "吗?")){
        $.ajax({
            url: [[@{/emp/}]] + del_ids,
            type: "POST",
            data: "&_method=delete",
            success:function (result) {
                alert(result.message)
                to_page(currentPage);
            }
        })
    }
});
```

#### 3. 服务器端处理请求

+ 将单个删除请求和批量删除请求统一
+ 规定批量删除传入员工id以字符串形式传入："1-2-3..."

```java
//删除员工
//改造方法，使得方法能够执行单个删除和批量删除
//批量删除所携带的ids：1-2-3...
@RequestMapping(value = "/emp/{ids}", method = RequestMethod.DELETE)
@ResponseBody
public Message deleteEmployee(@PathVariable("ids") String ids){
    if(ids.contains("-")){
        //如果是批量删除的请求
        String[] strings = ids.split("-");
        List<Integer> list = new ArrayList<>();
        //组装id的集合
        for (String id: strings) {
            list.add(Integer.valueOf(id));
        }
        employeeService.deleteEmployees(list);
    } else{
        //如果是单个删除的请求
        employeeService.deleteEmployee(Integer.valueOf(ids));
    }
    return Message.success();
}

//EmployeeService
	//批量删除
    public void deleteEmployees(List<Integer> ids){
        EmployeeExample employeeExample = new EmployeeExample();
        EmployeeExample.Criteria criteria = employeeExample.createCriteria();
        criteria.andIdIn(ids);
        employeeMapper.deleteByExample(employeeExample);
    }

	//单个删除
	public void deleteEmployee(Integer id){
        employeeMapper.deleteByPrimaryKey(id);
    }
```
