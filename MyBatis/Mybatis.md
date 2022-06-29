# Mybatis

### 一、Mybatis概述

#### 1. 基本信息

+ MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。
+ MyBatis 使用简单的 XML或注解用于配置和原始映射，将接口和 Java 的POJOs（Plain Ordinary Java Objects，普通的 Java对象）映射成数据库中的记录。

#### 2. Mybatis的特点

+ 简单易学：本身就很小且简单。没有任何第三方依赖，最简单安装只要两个jar文件+配置几个sql映射文件易于学习，易于使用，通过文档和源代码，可以比较完全的掌握它的设计思路和实现。
+ 灵活：mybatis不会对应用程序或者数据库的现有设计强加任何影响。 sql写在xml里，便于统一管理和优化。通过sql语句可以满足操作数据库的所有需求。
+ 解除sql与程序代码的耦合：通过提供DAO层，将业务逻辑和数据访问逻辑分离，使系统的设计更清晰，更易维护，更易单元测试。sql和代码的分离，提高了可维护性。
+ 提供映射标签，支持对象与数据库的orm字段关系映射
+ 提供对象关系映射标签，支持对象关系组建维护
+ 提供xml标签，支持编写动态sql。

#### 3. Mybatis的功能架构

+ **API接口层**：提供给外部使用的接口API，开发人员通过这些本地API来操纵数据库。接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理。
+ **数据处理层**：负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。
+ **基础支撑层**：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。

#### 4. Mybatis基本框架结构

![Mybatis-architecture](C:\Users\86198\Desktop\JavaEE\MyBatis\image\Mybatis-architecture.jpg)

### 二、Mybatis的一个示例

#### 1. 准备工作

##### 1.1 创建表

+ 在使用的数据库中创建一张测试用表
+ 在java层面创建一个类对应于表中的字段(ORM)

```java
public class Employee {
    private Integer id;
    private String lastName;    //当属性名和字段名不一致时，需要在查询时给字段取别名
    private String email;
    private String gender;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", lastName='" + lastName + '\'' +
                ", email='" + email + '\'' +
                ", gender='" + gender + '\'' +
                '}';
    }
}
```

##### 1.2 导入jar

![JAR](C:\Users\86198\Desktop\JavaEE\MyBatis\image\JAR.jpg)

#### 2. 创建Mybatis的xml配置文件

##### 2.1 创建全局配置文件mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!--配置数据源-->
                <property name="driver" value="com.microsoft.sqlserver.jdbc.SQLServerDriver"/>
                <property name="url" value="jdbc:sqlserver://localhost:1433;DatabaseName=mybatis"/>
                <property name="username" value="sa"/>
                <property name="password" value="abc"/>
            </dataSource>
        </environment>
    </environments>
    <!--写好的sql映射文件一定要注册到全局配置文件中-->
    <mappers>
        <mapper resource="employeeMapper.xml"/>
    </mappers>
</configuration>
```

##### 2.2 创建sql映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace: 名称空间-->
<mapper namespace="com.Mybatis.EmployeeMapper">
    <!--id: 唯一标识
        resultType: 返回类型
        #{id}: 从传递过来的参数中取出id值
    -->
    <select id="selectEmployee" resultType="com.Mybatis.bean.Employee">
		/*与java对象属性名不一致的列名需要起别名*/
        select id,last_name lastName,email,gender from employee where id = #{id}
    </select>
</mapper>
```

#### 3. 编写java代码

+ 根据xml配置文件(全局配置文件)创建一个SqlSessionFactory对象

  ```java
  String resource = "mybatis-config.xml";
  InputStream inputStream = Resources.getResourceAsStream(resource);
  SqlSessionFactory sqlSessionFactory =
          new SqlSessionFactoryBuilder().build(inputStream);
  ```

+ 获取sqlSession实例，能直接执行已经映射的sql语句

  ```java
  SqlSession sqlSession = sqlSessionFactory.openSession();
  ```

+ 调用sqlSession的相应方法，执行sql语句

  ```java
  //statement: sql语句的唯一标识符
  //parameter: 执行sql语句要用的参数
  Employee selectOne = sqlSession.selectOne("com.Mybatis.EmployeeMapper.selectEmployee", 1);
  ```

+ 关闭sqlSession

  ```java
  sqlSession.close();
  ```

+ 完整的测试代码

  ```java
  @Test
  public void testMybatis1() throws IOException {
      //1. 根据xml配置文件(全局配置文件)创建一个SqlSessionFactory对象
      //2. 获取sqlSession实例，能直接执行已经映射的sql语句
      //3. 需要一个sql映射文件，配置了每一个sql，以及sql的封装规则等
      //4. 将sql映射文件，注册道全局配置文件中
      String resource = "mybatis-config.xml";
      InputStream inputStream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory =
              new SqlSessionFactoryBuilder().build(inputStream);
      SqlSession sqlSession = sqlSessionFactory.openSession();
      //statement: sql语句的唯一标识符
      //parameter: 执行sql语句要用的参数
      Employee selectOne = sqlSession.selectOne("com.Mybatis.EmployeeMapper.selectEmployee", 1);
      System.out.println(selectOne);
      sqlSession.close();
  }
  ```

+ 测试结果

  ![result1](C:\Users\86198\Desktop\JavaEE\MyBatis\image\result1.jpg)

### 三、Mybatis配置文件的设置

#### 1. \<configuration> 标签

+ 标志一个mybatis全局配置文件的开始和结尾，类似于html文件中的\<html>标签

  ```xml
  <configuration>
  	<!--书写其他属性-->
  </configuration>
  ```

#### 2. \<properties>标签

+ Mybatis的properties标签来引入外部properties配置文件的内容

+ resource属性：引入类路径下的资源，一般在src文件夹下

+ url属性：引入网络路径或者磁盘路径下的资源

  ```xml
  <properties resource="dbsource.properties"></properties>
  ```

  ```properties
  #配置文件一般配置的是一些常用信息
  jdbc.driver=com.microsoft.sqlserver.jdbc.SQLServerDriver
  jdbc.url=jdbc:sqlserver://localhost:1433;DatabaseName=mybatis
  jdbc.username=sa
  jdbc.password=abc
  ```

#### 3. \<settings>标签

+ settings标签包含一些重要的设置项，内部含有多个setting标签

+ setting标签用来设置某一个具体的设置项的值

+ name属性：设置项的名称

+ value属性：设置项的值

  ```xml
  <settings>
      <!--设置自动转换驼峰命名法则
  		还有一些其他设置项可以在官方文档中查看
  	-->
      <setting name="mapUnderscoreToCamelCase" value="true"/>
  </settings>
  ```

#### 4. \<environments>标签

+ 使用environments标签可以配置多个mybatis环境，default标签用来指定使用某种特定的环境

+ environment标签用来设置某一个具体环境的信息，id属性表示当前设置环境的名称

+ 每一个environment标签下都需要含有transactionManager和dataSource两个标签

+ transactionManager:事务管理器

  + type属性：事务管理器的类型，有两种取值，JDBC和MANAGED

+ dataSource：数据源

  + type属性：数据源类型，有三种取值POOLED(使用数据库连接池)/UNPOOLED/JNDI

  ```xml
  <environments default="development">
      <!--test环境-->
      <environment id="test">
          <transactionManager type="JDBC"></transactionManager>
          <dataSource type="POOLED">
              <!--可以从之前在properties标签中的配置文件获取-->
              <property name="driver" value="${jdbc.driver}"/>
              <property name="url" value="${jdbc.url}"/>
              <property name="username" value="${jdbc.username}"/>
              <property name="password" value="${jdbc.password}"/>
          </dataSource>
      </environment>
      <!--development环境-->
      <environment id="development">
          <transactionManager type="JDBC"/>
          <dataSource type="POOLED">
              <!--配置数据源-->
              <property name="driver" value="${jdbc.driver}"/>
              <property name="url" value="${jdbc.url}"/>
              <property name="username" value="${jdbc.username}"/>
              <property name="password" value="${jdbc.password}"/>
          </dataSource>
      </environment>
  </environments>
  ```

#### 5. \<databaseIdProvider>标签

+ databaseIdProvider标签用来支持多数据库厂商

+ type标签："DB_VENDOR"，得到数据库厂商标识(JDBC驱动自带)，mybatis就能根据数据库厂商标识来执行不同的sql

  ```xml
  <databaseIdProvider type="DB_VENDOR">
      <!--为不同的数据库厂商起别名-->
      <property name="MySQL" value="mysql"/>
      <property name="SQL Server" value="sqlserver"/>
  </databaseIdProvider>
  ```

#### 6. \<mappers>标签

+ mappers标签将sql映射注册到全局配置文件中

+ 由多个mapper标签组成，每一个mapper标签注册一个sql映射文件

+ 可以使用配置文件设置sql映射，由mapper的resource和url属性注册

  + resource属性：引用类路径下的sql映射文件，一般在src文件夹下
  +  url属性：引用网络下或者磁盘路径下的sql映射文件

+ 可以使用注解的方式设置sql映射，使用class属性注册到全局配置文件中

  +  class属性：引用接口
  + 必须由sql映射文件，文件名必须和待注册的接口名同名，并且和接口放在统一文件夹下
  + 没有sql映射文件，所有的sql都是利用注解写在接口上

  ```xml
  <mappers>
      <mapper resource="employeeMapper.xml"/>
      <!--注册接口时，如果没有sql映射文件，使用注解方式，在接口的方法上创建注解-->
      <mapper class="com.Mybatis.Interfaces.EmployeeMapperAnnotation" />
      <!--package标签：批量注册-->
      <package name="com.Mybatis.Interfaces"/>
  </mappers>
  ```

  + 接口：

  ```java
  import com.Mybatis.bean.Employee;
  import org.apache.ibatis.annotations.Select;
  
  public interface EmployeeMapperAnnotation {
      //注解方式，还有@Insert, @Update等注解
      @Select("select * from employee where id=#{id}")
      public Employee getEmployeeById(Integer id);
  }
  ```

### 四、Mybatis SQL映射文件

#### 1. 参数处理

+ 参数处理指的是，在sql映射文件中，对于每一个sql语句进行绑定时，都需要按照要求或多或少传入相应的参数，例如：

  + select * from table where id = #{传入参数}
  + update table set key = #{value} where id = #{参数}
  + insert into table(key1, key2, key3) values(#{value1}, #{value2}, #{value3})
  + ......

+ 当传入单个参数时，获取参数的方式为：#{参数名}

  传入单个参数时，Mybatis不会对参数进行特殊化处理，参数名可以任意

  ```xml
  <!--对应的接口方法：public Employee getEmployeeById(Integer id);-->
  <select id="getEmployeeById" resultType="com.Mybatis.bean.Employee">
          select * from employee where id = #{id}    /*可以取其他的参数名，比如#{aaa}等*/
  </select>
  ```

+ 当传入多个参数时，获取参数的方法为：#{param1}

  传入多个参数时,Mybatis会对参数进行处理，这些参数会被Mybatis封装成一个Map对象

  key: param1, param2, param3, ... paramN

  vaule: 参数值1， 参数值2...

  ```xml
  <!--对应的接口方法：public Employee getEmployeeByLastName(Integer id, String lastName);
      -->
      <select id="getEmployeeByLastName" resultType="com.Mybatis.bean.Employee">
          select * from employee where id = #{param1} and last_name = #{param2}
      </select>
  ```

  还可以使用命名参数，明确地指出封装参数时，map的key是什么，需要使用@Param注解标识传入的参数

  ```xml
  <!--对应的接口方法：
  	public Employee getEmployeeByLastName(@Param("id") Integer id, @Param("lastName") String lastName);
  -->
  <select id="getEmployeeByLastName" resultType="com.Mybatis.bean.Employee">
      select * from employee where id = #{id} and last_name = #{lastName}
  </select>
  ```

+ 当传入的参数都属于业务模型的属性时，可以直接将创建的业务逻辑对象当作参数传入

  传入一个java对象时，获取参数的方式是：#{属性名}

  ```xml
  <!--对应的接口方法：
  	public void insertEmployee(Employee employee);-->
  <insert id="insertEmployee" parameterType="com.Mybatis.bean.Employee" useGeneratedKeys="true" keyProperty="id">
          insert into employee(last_name, email, gender) values(#{lastName}, #{email}, #{gender})
      </insert>
  ```

  测试方法：

  ```java
  @Test
  public void testMybatis6() throws IOException {
      String resource = "mybatis-config.xml";
          InputStream stream = Resources.getResourceAsStream(resource);
          SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(stream);
          SqlSession session = sqlSessionFactory.openSession();
          EmployeeMapper mapper = session.getMapper(EmployeeMapper.class);
          Employee employee = new Employee("Jerry", "jerry123@qq.com", "1");
          mapper.insertEmployee(employee);
          //如果在sql映射文件中没有在insert标签中加useGeneratedKeys="true"和keyProperty属性，那么新插入的employee就不能获取到
          //当前的自增的主键id值
          System.out.println(employee.getId());
          //手动提交
          session.commit();
          session.close();
  }
  ```

+ Mybatis会将多个参数封装成Map对象，因此，传参时，可以直接传入Map对象

  当传入的参数是Map对象时，获取参数的方式是：#{key}

  ```xml
  <!--对应的接口方法：public Employee getEmployeeByMap(Map<String, Object> map);-->
      <select id="getEmployeeByMap" resultType="com.Mybatis.bean.Employee">
          select * from employee where id = #{id} and last_name = #{lastName}
      </select>
  ```

  测试方法：

  ```java
  @Test
  public void testMybatis6() throws IOException {
      String resource = "mybatis-config.xml";
      InputStream stream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(stream);
      SqlSession session = sqlSessionFactory.openSession();
      EmployeeMapper mapper = session.getMapper(EmployeeMapper.class);
      Map<String, Object> map = new HashMap<>();
      //向map中添加参数
      map.put("id", 1);
      map.put("lastName", "Tomcat");
      Employee employee = mapper.getEmployeeByMap(map);
      System.out.println(employee);
      //手动提交
      session.commit();
      session.close();
  }
  ```

+ 如果多个参数不是业务模型中的数据(即不对应java类中的属性)，但是需要经常使用，可以编写一个TO(Transfer Object)数据传输对象

+ 另外，需要特别注意的是，如果传入的参数是Collection类型或者是数组，也会进行特殊处理，会把传入的集合类型或者数组类型封装到map中

  + 如果是Collection类型的参数，在map中的key是collection，特别的如果是List类型，key是list

  + 如果是数组类型的参数，在map中的key是array

  + 举例：public Employee getEmployeeById(List<Integer> ids);

    取值：取出第一个id值，#{list.get(0)}

+ Mybatis处理参数的源码

  执行过程以public Employee getEmployeeByLastName(Integer id, String lastName)方法为例

  names是一个map: {0 = id, 1 = lastName}

  names的获取过程：首先获取每个标注@Param注解的参数的值，id，lastName，赋值给name；然后每次解析一个参数给names，key为参数的索引，value为参数的值

  如果没有@Param注解的参数，value值会是当前参数的索引

  ```java
  //某个类的构造器，主要关注names的创建过程
  public ParamNameResolver(Configuration config, Method method) {
    this.useActualParamName = config.isUseActualParamName();
    final Class<?>[] paramTypes = method.getParameterTypes();
    final Annotation[][] paramAnnotations = method.getParameterAnnotations();
    final SortedMap<Integer, String> map = new TreeMap<>();
    int paramCount = paramAnnotations.length;
    // get names from @Param annotations
    for (int paramIndex = 0; paramIndex < paramCount; paramIndex++) {
      if (isSpecialParameter(paramTypes[paramIndex])) {
        // skip special parameters
        continue;
      }
      String name = null;
      //如果传入的参数有@Param注解
      for (Annotation annotation : paramAnnotations[paramIndex]) {
        if (annotation instanceof Param) {
          hasParamAnnotation = true;
          //获取@Param注解中属性的值
          name = ((Param) annotation).value();
          break;
        }
      }
      if (name == null) {
        // @Param was not specified.
        if (useActualParamName) {
          name = getActualParamName(method, paramIndex);
        }
        //如果是没有注解的参数，那么就取参数的索引
        if (name == null) {
          // use the parameter index as the name ("0", "1", ...)
          // gcode issue #71
          name = String.valueOf(map.size());
        }
      }
     	//将name作为值存入map中，键为参数的索引
      map.put(paramIndex, name);
    }
    //此例中的names为{0="id", 1="lastName"}
    names = Collections.unmodifiableSortedMap(map);
  }
  ```

  Mybatis处理参数的过程：

  ```java
  //args = {1, "Tomcat"}
  public Object getNamedParams(Object[] args) {
    final int paramCount = names.size();
    //如果参数为空，则直接返回null
    if (args == null || paramCount == 0) {
      return null;
      //如果只有一个参数，并且没有@Param注解，即只传入了一个参数  
    } else if (!hasParamAnnotation && paramCount == 1) {
      //获取names的第一个key
      Object value = args[names.firstKey()];
      return wrapToMapIfCollection(value, useActualParamName ? names.get(0) : null);
      //多个参数，或者有@Param注解
    } else {
      final Map<String, Object> param = new ParamMap<>();
      int i = 0;
      //遍历names保存参数，并且封装到map中  {0=id, 1=lastName}
      for (Map.Entry<Integer, String> entry : names.entrySet()) {
        //names集合中的value值当作key，names集合的key又作为取值的参考
        //{id:args[0]=1,lastName:args[1]=Tomcat}
        //如果参数没有被@Param所标注，那么就是{"0"=args[0], "1"=args[1]}，所以使用参数的索引也能取得值
        param.put(entry.getValue(), args[entry.getKey()]);
        //默认的键值对，这就是因为如果没有加@Param注解，默认的键的值为paramX
        final String genericParamName = GENERIC_NAME_PREFIX + (i + 1);
        // ensure not to overwrite parameter named with @Param
        if (!names.containsValue(genericParamName)) {
          param.put(genericParamName, args[entry.getKey()]);
        }
        i++;
      }
      return param;
    }
  }
  ```

#### 2. 相关标签

##### 2.1 \<mapper>标签

+ 用法：绑定一个java接口，在标签对之间书写sql语句

  ```xml
  <mapper namespace="com.Mybatis.Interfaces.EmployeeMapperPlus">
  </mapper>
  ```

+ namespace: 名称空间，指定为接口的全类名，绑定接口，接口中有相应的方法

##### 2.2 \<select>标签

+ 用法：编写sql查询语句

  ```xml
  <select id="getEmployeeById" resultType="com.Mybatis.bean.Employee">
  /*与java对象属性名不一致的列名需要起别名*/
          select * from employee where id = #{id}
  </select>
  ```

+ 标签中的属性：

  id: 唯一标识，使用接口之后，换成接口中的方法
  resultType: 返回类型

  resultMap:自定义结果集的映射规则

  #{id}: 从传递过来的参数中取出id值
  databaseId: 标识不同的数据库厂商

  注意：resultType和resultMap只能二选一，不能同时出现

+ 除了\<select>标签之外，相似的标签还有\<insert>, \<update>, \<delete>都是用来编写sql语句

##### 2.3 \<resultMap>标签

+ 用法：自定义某个javaBean的封装规则

  ```xml
  <resultMap id="MyEmp" type="com.Mybatis.bean.Employee">
          <!--指定主键类的封装规则
              column：指定哪一列
              property：指定JavaBean中的某个属性
          -->
          <id column="id" property="id"/>
          <!--定义普通类的封装规则-->
          <result column="last_name" property="lastName"/>
          <!--其他不指定的列会自动封装-->
  </resultMap>
  ```

+ type：自定义的Java类型，全类名
  id：唯一id，引用这个resultMap，一般在\<select>等标签中的resultType属性中使用

+ 使用resultType进行联合查询(连接查询)

  场景描述：在查询员工信息的同时，查询出员工所在的部门信息，每一个员工都有与之对应的部门信息

  方法：public Employee getEmployeeAndDepartment(Integer id);

  sql映射文件：

  ```xml
  <!--联合查询：可以使用级联属性的方式封装结果集(属性的属性)
  	property中department是Employee中的Department属性
  -->
  <resultMap id="MyEmpPlus" type="com.Mybatis.bean.Employee">
      <id column="id" property="id"/>
      <result column="last_name" property="lastName"/>
      <result column="did" property="department.id"/>
      <result column="dept_name" property="department.deptName"/>
  </resultMap>
  <select id="getEmployeeAndDepartment" resultMap="MyEmpPlus">
          select e.id id, e.last_name last_name, e.gender gender, e.department department,
          d.id did, d.dept_name dept_name from employee e, department d
          where e.department = d.id and e.id = #{id}
  </select>
  ```

  测试方法：

  ```java
  //测试联合查询
  @Test
  public void testMybatis2() throws IOException {
      String resource = "mybatis-config.xml";
      InputStream stream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(stream);
      SqlSession session = sqlSessionFactory.openSession();
      EmployeeMapperPlus mapper = session.getMapper(EmployeeMapperPlus.class);
      Employee employee = mapper.getEmployeeAndDepartment(1);
      System.out.println(employee);
      //手动提交
      session.commit();
      session.close();
  }
  ```

  结果：Employee{id=1, lastName='Tomcat', email='null', gender='0', department=Department{id=1, deptName='开发部'}}

##### 2.4 \<association>标签

+ 用法：写在resultMap标签中，指定联合的javaBean对象

  ```xml
  <resultMap id="MyEmpPlusAss" type="com.Mybatis.bean.Employee">
      <id column="id" property="id"/>
      <result column="last_name" property="lastName"/>
      <association property="department" javaType="com.Mybatis.bean.Department">
          <id column="did" property="id"/>
          <result column="dept_name" property="deptName"/>
      </association>
  </resultMap>
  ```

+ property属性：指定哪个属性是联合对象
  javaType属性：指定这个属性对象的类型[不能省略]

+ 还可以使用association标签实现分步查询：
          (1). 按照员工ID查询员工信息
          (2). 根据上面的结果中员工的dept_id去查询部门信息
          (3). 部门信息设置到员工信息中

  辅助的DepartmentMapper接口：

  ```java
  public interface DepartmentMapper {
      //根据部门id查询部门信息
      public Department getDeptById(Integer id);
  }
  ```

  DepartmentMapper的sql映射文件：

  ```xml
  <mapper>
      <select id="getDeptById" resultType="com.Mybatis.bean.Department">
          select id, dept_name deptName from department where id = #{id}
      </select>
  </mapper>
  ```

  EmployeeMapper的sql映射文件，需要引用DepartmentMapper中的查询方法：

  ```xml
  <resultMap id="EmpStep" type="com.Mybatis.bean.Employee">
      <id column="id" property="id"/>
      <result column="last_name" property="lastName"/>
      <!--select：表明当前属性是调用select指定的方法查出的结果，为某个sql映射文件中namespace的值加上具体select标签中id属性值
          column：指定将哪一列的值传给select指定的方法-->
      <association property="department" select="com.Mybatis.Interfaces.DepartmentMapper.getDeptById"
          column="department">
      </association>
  </resultMap>
  <select id="getEmployeeByIdStep" resultMap="EmpStep">
      select * from  employee where id = #{id}
  </select>
  ```

  测试方法：

  ```java
  @Test
  public void testMybatis3() throws IOException {
      String resource = "mybatis-config.xml";
      InputStream stream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(stream);
      SqlSession session = sqlSessionFactory.openSession();
      EmployeeMapperPlus mapper = session.getMapper(EmployeeMapperPlus.class);
      Employee employee = mapper.getEmployeeByIdStep(1);
      System.out.println(employee);
      System.out.println(employee.getDepartment());
      //手动提交
      session.commit();
      session.close();
  }
  ```

  结果：Employee{id=1, lastName='Tomcat', email='tom163@qq.com', gender='0', department=Department{id=1, deptName='开发部'}}

##### 2.5 \<collection>标签

+ 用法：写在resultMap中，定义集合类型的属性

  ```xml
  <resultMap id="MyDept" type="com.Mybatis.bean.Department">
      <id column="did" property="id"/>
      <result column="dept_name" property="deptName"/>
      <collection property="employees" ofType="com.Mybatis.bean.Employee">
          <!--定义集合中元素的封装规则-->
          <id column="eid" property="id"/>
          <result column="last_name" property="lastName"/>
          <result column="email" property="email"/>
          <result column="gender" property="gender"/>
      </collection>
  </resultMap>
  ```

+ ofType: 指定集合中元素的类型

+ 应用场景：一对多的情况，一个部门中有多个员工，现在希望在查询部门信息的同时，查询这个部门下的所有的员工信息

  sql映射文件：

  ```xml
  <resultMap id="MyDept" type="com.Mybatis.bean.Department">
      <id column="did" property="id"/>
      <result column="dept_name" property="deptName"/>
      <collection property="employees" ofType="com.Mybatis.bean.Employee">
          <!--定义集合中元素的封装规则-->
          <id column="eid" property="id"/>
          <result column="last_name" property="lastName"/>
          <result column="email" property="email"/>
          <result column="gender" property="gender"/>
      </collection>
  </resultMap>
  <select id="getDeptByIdPlus" resultMap="MyDept">
      select dep.id did, dep.dept_name dept_name,
             emp.id eid, emp.last_name last_name, emp.email email, emp.gender gender
      from department dep left join employee emp on dep.id = emp.department where dep.id = #{id}
  </select>
  ```

  测试方法：

  ```java
  @Test
  public void testMybatis4() throws IOException {
      String resource = "mybatis-config.xml";
      InputStream stream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(stream);
      SqlSession session = sqlSessionFactory.openSession();
      DepartmentMapper mapper = session.getMapper(DepartmentMapper.class);
      Department department = mapper.getDeptByIdPlus(1);
      System.out.println(department);
      System.out.println(department.getEmployees());
      //手动提交
      session.commit();
      session.close();
  }
  ```

  结果：Department{id=1, deptName='开发部'}
  [Employee{id=1, lastName='Tomcat', email='tom163@qq.com', gender='0', department=null}, Employee{id=4, lastName='Merry', email='merry147@qq.com', gender='1', department=null}]

+ \<collection>标签的其他用法：分步查询

  EmployeeMapper的sql映射文件：

  ```xml
  <select id="getEmpsByDeptId" resultType="com.Mybatis.bean.Employee">
      select * from employee where department = #{deptId}
  </select>
  ```

  DepartmentMapper的映射文件，需要引用EmployeeMapper中的查询方法：

  ```xml
  <!--使用collection进行分步查询-->
  <resultMap id="MyDeptStep" type="com.Mybatis.bean.Department">
      <id column="id" property="id"/>
      <result column="dept_name" property="deptName"/>
      <!--与使用association标签的分步查询类似-->
      <collection property="employees" select="com.Mybatis.Interfaces.EmployeeMapperPlus.getEmpsByDeptId"
          column="id">
      </collection>
  </resultMap>
  <select id="getDeptByIdStep" resultMap="MyDeptStep">
      select id, dept_name from department where id = #{id}
  </select>
  ```

  测试方法：

  ```java
  @Test
  public void testMybatis5() throws IOException {
      String resource = "mybatis-config.xml";
      InputStream stream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(stream);
      SqlSession session = sqlSessionFactory.openSession();
      DepartmentMapper mapper = session.getMapper(DepartmentMapper.class);
      Department department = mapper.getDeptByIdStep(1);
      System.out.println(department);
      System.out.println(department.getEmployees());
      //手动提交
      session.commit();
      session.close();
  }
  ```

  结果：Department{id=1, deptName='开发部'}
  [Employee{id=1, lastName='Tomcat', email='tom163@qq.com', gender='0', department=null}, Employee{id=4, lastName='Merry', email='merry147@qq.com', gender='1', department=null}

##### 2.6 其他标签

+ \<discriminator>标签

  鉴别器，mybatis可以使用鉴别器判断某列的值，然后根据某列的值改变封装行为，写在resultType中
  column：指定判断的列
  javaType：列值对应的Java对象
  \<case>标签：指定相应条件下的封装规则

#### 3. 动态SQL

##### 3.1 if标签

+ **作用**：判断表达式, 从参数中取值进行判断, 遇见特殊符号，应该去写转义字符

+ **应用场景**：查询员工，要求，携带了哪个字段，查询条件就带上这个字段，比如有id和lastName就使用id和lastName查询

+ 使用方法：

  ```xml
  <!--对应接口中的方法：public List<Employee> getEmployeesByCondition(Employee employee);-->
  <select id="getEmployeesByCondition" resultType="com.Mybatis.bean.Employee">
      select * from employee
      <where>
          <!--都是直接进行字符串拼接，最后拼接出sql语句-->
          <if test="id!=null">
              id = #{id}
          </if>
          <if test="lastName!=null and lastName!=''">
              and last_name like #{lastName}
          </if>
          <if test="email!=null and email.trim()!=''">
              and email = #{email}
          </if>
          <if test="gender==0 or gender==1">
              and gender = #{gender}
          </if>
      </where>
  </select>
  ```

+ 结果：第一行为发出的sql语句，当条件不同时，发出的sql语句也不同

  ![](C:\Users\86198\Desktop\JavaEE\MyBatis\image\If-config.jpg)

+ 注意事项：以上例为例，当传入的参数id == null时，后面会出现sql拼接的语法错误，通常有两种解决方案

  1. 在where后面加上1=1，在后面的所有条件中均加上and，但这样做会损害效率

     ```xml
     <select id="getEmployeesByCondition" resultType="com.Mybatis.bean.Employee">
         select * from employee where 1=1 
             <if test="id!=null">
                 and id = #{id}
             </if>
             <if test="lastName!=null and lastName!=''">
                 and last_name like #{lastName}
             </if>
             <if test="email!=null and email.trim()!=''">
                 and email = #{email}
             </if>
             <if test="gender==0 or gender==1">
                 and gender = #{gender}
             </if>
     </select>
     ```

  2. mybatis使用where标签来将所有的条件包括在内，可以去除第一个多余的and，就是示例中的解决方法

  3. if标签还能在其他标签中使用，比如update中，修改指定的属性值：

     ```xml
     <!--对应方法：public void updateEmployee(Employee employee);-->
     <update id="updateEmployee">
         update employee
         <set>
             <if test="lastName!=null">
                 last_name = #{lastName},
             </if>
             <if test="email!=null">
                 email = #{email},
             </if>
             <if test="gender!=null">
                 gender = #{gender}
             </if>
         </set>
         where id = #{id}
     </update>
     ```

##### 3.2 trim标签

+ **作用**：自定义字符串截取，并不常用

+ 标签属性：

  + prefix：前缀，给拼接后的字符串加上一个前缀
  + refixOverrides：前缀覆盖，去掉整个字符串前面多余的字符
  + suffix：给拼串后的字符串加上后缀
  + suffixOverrides：去掉字符串后面多余的字符

+ **应用场景**：当id = #{id} and这种拼接方法，使用where标签不能去除，需要使用trim标签

+ 示例：注意and的放置位置与上例的区别，结果与上例相同

  ```xml
  <select id="getEmployeesByConditionTrim" resultType="com.Mybatis.bean.Employee">
      select * from employee
      <trim prefix="where" prefixOverrides="and">
          <if test="id!=null">
              id = #{id} and
          </if>
          <if test="lastName!=null and lastName!=''">
              last_name like #{lastName} and
          </if>
          <if test="email!=null and email.trim()!=''">
              email = #{email} and
          </if>
          <if test="gender==0 or gender==1">
              gender = #{gender}
          </if>
      </trim>
  </select>
  ```

##### 3.3 choose标签

+ **作用**：分支选择，包含**\<when>**和**\<otherwise>**标签

+ **应用场景**：如果带了id字段就用id查，如果带了lastName字段，就用lastName字段查，只会进入其中一个分支

+ 使用方式：

  ```xml
  <!--对应方法：public List<Employee> getEmployeesByConditionChoose(Employee employee);-->
  <select id="getEmployeesByConditionChoose" resultType="com.Mybatis.bean.Employee">
      select * from employee
      <where>
          <choose>
              <when test="id!=null">
                  id = #{id}
              </when>
              <when test="lastName!=null">
                  last_name like #{lastName}
              </when>
              <when test="email!=null">
                  email = #{email}
              </when>
              <otherwise>
                  gender = 0
              </otherwise>
          </choose>
      </where>
  </select>
  ```

+ 结果：第一条为发送的sql语句

  ![](C:\Users\86198\Desktop\JavaEE\MyBatis\image\choose-config.jpg)

##### 3.4 foreach标签

+ **作用**：遍历集合

+  **应用场景**：查询id在一个范围内的员工信息，也能用于批量保存

+ 标签属性：

  + collection：指定要遍历的集合，值为list和map，如果需要指定自己传入的参数，则需要在传入的参数前添加@Param注解
  +  item：将当前遍历出的元素赋值给指定的变量
  + separator：每个元素之间的分隔符
  + open：遍历出所有结果拼接一个开始的字符串
    #{item属性值}取出变量的值，也就是当前所遍历的元素
  + close：遍历出所有结果后，拼接一个字符串
  + index：索引，遍历list时是索引
                   遍历map时，index表示的是map的key，item是map的值

+ 使用方法一：查询id在一个范围内的员工信息

  ```xml
  <!--对应方法：public List<Employee> getEmployeesByConditionForeach(List<Integer> list);-->
  <select id="getEmployeesByConditionForeach" resultType="com.Mybatis.bean.Employee">
      select * from employee where id in
          <foreach collection="list" item="item_id" separator="," open="(" close=")">
              #{item_id}
          </foreach>
  </select>
  ```

  使用方法二：批量添加

  ```xml
  <!--对应方法：public void addEmployees(@Param("emps") List<Employee> employees);-->
  <insert id="addEmployees">
      insert into employee(last_name, email, gender, department) values
      <foreach collection="emps" item="emp" separator=",">
          (#{emp.lastName}, #{emp.email}, #{emp.gender}, #{emp.department.id})
      </foreach>
  </insert>
  ```

+ 结果（查询结果）：

  ![](C:\Users\86198\Desktop\JavaEE\MyBatis\image\foreach-config.jpg)

##### 3.5 其他标签

+ **sql标签**：抽取可重用的sql片段，方便后面引用

  使用include标签，引用外部定义的sql，include还可以自定义一些property属性，sql标签就能使用include中定义的属性，使用${}取值，标签内部也能使用if标签进行判断

  使用方法：

  ```xml
  <sql id="insertColumn">
      		<!--根据不同的数据库，抽取不同的sql部分-->
  	  		<if test="_databaseId=='oracle'">
  	  			employee_id,last_name,email
  	  		</if>
  	  		<if test="_databaseId=='mysql'">
  	  			last_name,email,gender,d_id
  	  		</if>
  </sql>
  
  <insert id="addEmps">
  	 	insert into tbl_employee(
      		<!--在其他标签中使用include标签引用sql标签中的内容-->
  	 		<include refid="insertColumn"></include>
  	 	) 
  		values
  		<foreach collection="emps" item="emp" separator=",">
  			(#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
  		</foreach>
  	 </insert>
  ```

+ Mybatis的两个内置参数：

  + **\_parameter**: 代表整个参数
                如果是单个参数\_parameter就是这个参数
                如果是多个参数参数会被封装成一个map，\_parameter就是代表这个map
  + **_databaseId**: 如果配置了databaseIdProvider标签，那么这个参数就代表当前数据库的别名

  使用方法：

  ```xml
  <!--  public Employee getEmpByIdStep(Integer id);-->
  	 <select id="getEmpByIdStep" resultMap="MyEmpByStep">
  	 	select * from tbl_employee where id=#{id}
           <!--将传入的整个参数作为判断依据，_parameter就代表整个传入的参数-->
  	 	<if test="_parameter!=null">
  	 		and 1=1
  	 	</if>
  	 </select>
  ```

### 五、数据库缓存

#### 1. 基本概述：

+ Mybatis总共管理数据库的两级缓存：

  + 一级缓存：SqlSession级别的缓存，每个SqlSession对象都独立拥有自己的一级缓存斌且不能被关闭，是一个map
    本地缓存，与数据库同一次会话期间查询到的数据会放到一级缓存中，以后如果需要获取相同的数据，直接从缓存中获取，没必要再去查询数据库

  + 二级缓存：全局缓存，基于namespace级别的缓存，一个namespace对应一个二级缓存

  + 缓存示例：测试代码

    ```java
    @Test
    public void testCache() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream stream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(stream);
        SqlSession session = sqlSessionFactory.openSession();
        EmployeeMapper mapper = session.getMapper(EmployeeMapper.class);
        Employee empl01 = mapper.getEmployeeById(1);
        System.out.println(empl01);
        //测试一级缓存
        Employee empl02 = mapper.getEmployeeById(1);
        System.out.println(empl02);
        //判断两次查询获得的结果是否是同一个对象，如果是同一个对象就表明第二次查询时并没有发送sql语句
        System.out.println(empl01 == empl02);
    
    }
    ```

    结果：从结果可以看出，两次查询只发送了一条sql语句

    ![](C:\Users\86198\Desktop\JavaEE\MyBatis\image\cache-config.jpg)

+ 一级缓存失效的四种情况：

  + sqlSession不同
  + sqlSession相同，但是查询条件不同
  + sqlSession相同，但两次查询之间执行了增删改操作
  + sqlSession相同，手动清除一级缓存

  示例：

  ```java
  @Test
      public void testCache2() throws IOException {
          String resource = "mybatis-config.xml";
          InputStream stream = Resources.getResourceAsStream(resource);
          SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(stream);
          SqlSession session = sqlSessionFactory.openSession();
          EmployeeMapper mapper = session.getMapper(EmployeeMapper.class);
          //1. 测试不同的sqlSession
  //        Employee empl01 = mapper.getEmployeeById(1);
  //        System.out.println(empl01);
  //        SqlSession session2 = sqlSessionFactory.openSession();
  //        mapper = session2.getMapper(EmployeeMapper.class);
  //        Employee empl02 = mapper.getEmployeeById(1);
  //        System.out.println(empl02);
  //        System.out.println(empl01 == empl02);
  
          //2. 两次查询之间有增删改操作
          Employee emp01 = mapper.getEmployeeById(1);
          Employee newEmp = new Employee("Jerry", "jerry188@qq.com", "1");
          newEmp.setId(3);
          mapper.updateEmployee(newEmp);
          Employee emp02 = mapper.getEmployeeById(1);
          System.out.println(emp01 == emp02);
      }
  ```

  结果一：不同sqlSession，可以看到mybatis发出了两条sql语句

  ![](C:\Users\86198\Desktop\JavaEE\MyBatis\image\cacheInvalidSqlsession.jpg)

  结果二：两次查询之间有增删改操作，可以看到sql发出了三条sql语句，其中两次查询语句是相同的，但查出来的结果对象却是封装在不同的类对象中

  ![](C:\Users\86198\Desktop\JavaEE\MyBatis\image\cacheInvalidUpdate.jpg)

+ 二级缓存的工作机制：

  + 一个会话，查询一条数据，这个数据就会被放在当前会话的一级缓存中
  + 如果当前会话关闭，会话中一级缓存的数据会被保存到二级缓存中，新的会话查询信息可以查询二级缓存
  + 不同namespace查出的数据会放在自己对应的缓存中
    效果：数据会从二级缓存中取出，查出的数据都会被默认先放在一级缓存中，只有会话关闭以后，一级缓存中的数据才会转移到二级缓存中
  + 值得注意的是，如果需要使用二级缓存，那么相对应的javaBean需要能够序列化，即实现Serializable接口

  示例：

  ```java
  @Test
  public void testSecondLevelCache() throws IOException{
      String resource = "mybatis-config.xml";
      InputStream stream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(stream);
      SqlSession openSession = sqlSessionFactory.openSession();
      SqlSession openSession2 = sqlSessionFactory.openSession();
          EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
          EmployeeMapper mapper2 = openSession2.getMapper(EmployeeMapper.class);
  
          Employee emp01 = mapper.getEmployeeById(1);
          System.out.println(emp01);
          openSession.close();
  
          //第二次查询是从二级缓存中拿到的数据，并没有发送新的sql
          //mapper2.addEmp(new Employee(null, "aaa", "nnn", "0"));
          Employee emp02 = mapper2.getEmployeeById(1);
          System.out.println(emp02);
          openSession2.close();
  }
  ```

  结果：可以看到只发送了一条sql语句，并且二级缓存的命中率为0.5，即表明第二次访问二级缓存时命中

  ![](C:\Users\86198\Desktop\JavaEE\MyBatis\image\secondLevelCache.jpg)

+ 二级缓存的使用：

  1. 开启全局二级缓存配置
  2. 在sql映射文件中配置使用二级缓存
  3. POJO需要实现序列化接口

+ 整体cache系统的原理图：

  ![](C:\Users\86198\Desktop\JavaEE\MyBatis\image\Cache.jpg)

#### 2. 和缓存相关的一些设置

1. cacheEnabled：是否关闭二级缓存
2. 每一个select标签都有useCache属性，默认都为true
              false：关闭二级缓存，一级缓存仍然使用
3. 每个增删改标签都有flushCache属性，默认为true，每一次执行增删改操作都会清空缓存，一、二级缓存都会被清空
              查询标签中也有flushCache属性，默认为false
4. sqlSession.clearCache()，清楚当前会话的一级缓存，对二级缓存没有影响
5. localCacheScope：本地缓存作用域，一级缓存设置
              SESSION：当前会话中的所有数据保存在会话缓存中
              STATEMENT：可以禁用一级缓存，一般不使用

#### 3. 映射文件中的\<cache>标签

+ \<cache>标签用来在映射文件中配置二级缓存的使用
+ 标签属性：
  + eviction：缓存的回收策略，发生容量冲突时，替换缓存块，默认为LRU
  + flushInterval：缓存刷新时间，缓存多长时间清空一次，默认不清空
  + readOnly：缓存是否只读，默认是false
  + size：缓存中存放多少元素
  + type：指定自定义缓存的全类名；常用的有ehcache
    			实现Cache接口即可；
