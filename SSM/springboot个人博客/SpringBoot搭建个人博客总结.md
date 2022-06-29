# SpringBoot搭建个人博客总结

## 一、环境搭建

开发工具：IDEA 2021.2

JDK版本：1.8

技术栈：

+ 前端：thymeleaf模板 + html页面
+ 后端：springboot 2.3.7版本整合SSM框架，数据库使用MySQL 8.0

搭建环境

### 1.  新建Springboot工程

选择JDK版本为1.8，选择阿里云镜像仓库

![](C:\Users\86198\Desktop\JavaEE\SSM\springboot个人博客\IDEA新建Springboot项目.jpg)

进入下一步之后选择相应的依赖项：根据技术栈的不同进行选择

![](C:\Users\86198\Desktop\JavaEE\SSM\springboot个人博客\选择依赖项.jpg)

### 2. 查看项目的pom文件

最初的pom文件大致如下所示，之后在开发中需要导入其他的依赖时，再向pom文件中添加

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.Myblog.Springboot</groupId>
    <artifactId>Myblog</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>Myblog</name>
    <description>Myblog</description>

    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.7.RELEASE</version>
                <configuration>
                    <mainClass>com.Myblog.springboot.MyblogApplication</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```



### 3. 编辑一些基础的配置

使用application.yml作为springboot的配置文件

```yml
# 应用名称
spring:
  application:
    name: Myblog
# THYMELEAF (ThymeleafAutoConfiguration)
# 开启模板缓存（默认值： true ）
  thymeleaf:
    cache: true
    check-template: true
    check-template-location: true
    servlet:
      content-type: text/html
    enabled: true
    encoding: UTF-8
    prefix: classpath:/templates/
    suffix: .html
  #配置数据源，默认使用springboot自带的数据源  
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springboot?serverTimezone=GMT%2B8&characterEncoding=utf-8
    username: root
    password: 123456
  #配置时区，统一日期的格式
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
#配置一些mybatis的相关配置项
mybatis:
  #configuration充当配置文件
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    map-underscore-to-camel-case: true
  mapper-locations: classpath:mapper/*.xml
# 应用服务 WEB 访问端口
server:
  port: 8080
```



创建完成之后环境就基本搭建完毕，接着引入前端界面，开始后端开发



## 二、结果展示

普通用户的前端界面展示：

![](C:\Users\86198\Desktop\JavaEE\SSM\springboot个人博客\普通用户界面.jpg)

管理员用户，即博客的拥有者需要登录才能管理

![](C:\Users\86198\Desktop\JavaEE\SSM\springboot个人博客\管理员用户登录.jpg)

后台管理界面：

![](C:\Users\86198\Desktop\JavaEE\SSM\springboot个人博客\后台管理界面.jpg)



## 三、需求分析与数据库设计

个人博客的基本功能：

管理源用户管理自己的博客，可以发布，修改，删除自己的博客，并且可以发表评论

普通用户在首页可以浏览博客，搜索相应的博客以及以游客的身份发表评论



### 1. 数据表设计

**博客表t_blog：**

+ 存储博客的基本信息，包括发布者，发布时间，发布内容

+ 存储博客所属的类别，标签

+ 设置博客的id为主键，自增

  ![](C:\Users\86198\Desktop\JavaEE\SSM\springboot个人博客\博客表.jpg)

**标签表t_tag：**

+ 存储标签的内容，标签包括前端，后端，java等等

+ 设置id为主键，自增

  ![](C:\Users\86198\Desktop\JavaEE\SSM\springboot个人博客\标签表.jpg)

**类别表t_type：**

+ 存储类别的内容，类别包括学习方法，娱乐方法等

+ 设置id为主键，自增

  ![](C:\Users\86198\Desktop\JavaEE\SSM\springboot个人博客\类别表.jpg)

**评论表t_comment：**

+ 存储用户评论的表

+ 存储用户的基本信息，包括用户名，邮箱等，存储评论的内容，创建的时间以及是否是恢复某条评论的评论

+ 设置id为主键，自增

  ![](C:\Users\86198\Desktop\JavaEE\SSM\springboot个人博客\评论表.jpg)

**用户表t_user：**

+ 存储管理员用户的基本信息

+ 设置id为主键，自增

  ![](C:\Users\86198\Desktop\JavaEE\SSM\springboot个人博客\用户表.jpg)



**数据表的对应关系：**

+ 一篇博客可以有多个标签
+ 一篇博客对应于一个博客类别
+ 一个类别下可以有多篇博客
+ 一个标签下也能有多篇博客

### 2. 业务逻辑

用户对于博客可以进行增删改操作

可以选择博客的类型，标签

可以增加博客的类型和标签



## 四、接口开发

### 1. DAO层

Javabean：每一张数据表对应一个Javabean

![](C:\Users\86198\Desktop\JavaEE\SSM\springboot个人博客\javabean.jpg)

Blog.java



Comment.java



Tag.java



Type.java



User.java



*BlogAndTag.java* 这个没看懂是干嘛的



每个类的Mapper接口，以及xxxMapper.xml的sql映射文件，解决对数据库的基本的增删改操作

![](C:\Users\86198\Desktop\JavaEE\SSM\springboot个人博客\Mapper.jpg)

### 2. Service层

包括Service接口以及Service接口的实现类，需要根据具体的业务逻辑实现来设计，不可以只调用Mapper接口的方法

BlogService.java以及BlogServiceImpl.java

```java
public interface BlogService {
    List<Blog> getAllBlogs();

    List<Blog> getAllRecommendBlogs();

    List<Blog> getIndexBlogs();

    List<Blog> getSearchBlogs(String condition);

    List<Blog> searchBlogsByAdmin(Blog blog);

    Blog getBlogById(Long id);

    Blog getDetailBlogById(Long id);

    int addBlog(Blog blog);

    int updateBlog(Blog blog);
}
@Service
public class BlogServiceImpl implements BlogService {

    @Autowired
    BlogMapper blogMapper;

    //简单的方法可以直接调用DAO层接口的方法进行实现
    @Override
    public List<Blog> getAllBlogs() {
        return blogMapper.getAllBlogs();
    }

    @Override
    public List<Blog> getAllRecommendBlogs() {
        return blogMapper.getAllRecommendBlogs();
    }

    @Override
    public List<Blog> getIndexBlogs() {
        return blogMapper.getIndexBlogs();
    }

    @Override
    public List<Blog> getSearchBlogs(String condition) {
        return blogMapper.getSearchBlogs(condition);
    }

    @Override
    public List<Blog> searchBlogsByAdmin(Blog blog) {
        return blogMapper.searchBlogsByAdmin(blog);
    }

    @Override
    public Blog getBlogById(Long id) {
        return blogMapper.getBlogById(id);
    }

    //复杂的逻辑就需要根据具体的业务逻辑进行判断
    @Override
    public Blog getDetailBlogById(Long id) {
        //展示博客的具体内容
        Blog blog = blogMapper.getDetailBlogById(id);
        if(blog == null){
            throw new NotFoundException("博客不存在");
        }
        String content = blog.getContent();
        blog.setContent(MarkdownUtils.markdownToHtmlExtensions(content));
        return blog;
    }

    @Override
    public int addBlog(Blog blog) {
        //发布一篇博客的逻辑
        //设置一些初始信息
        blog.setCreateTime(new Date());
        blog.setUpdateTime(new Date());
        blog.setViews(0);
		//调用DAO层的接口实现添加操作
        blogMapper.addBlog(blog);
        //设置一些其他的博客属性
        Long id = blog.getId();
        List<Tag> tags = blog.getTags();
        BlogAndTag blogAndTag = null;
        for (Tag tag : tags) {
            blogAndTag = new BlogAndTag(tag.getId(), id);

            blogMapper.addBlogAndTag(blogAndTag);
        }
        return 1;
    }

    @Override
    public int updateBlog(Blog blog) {
        blog.setUpdateTime(new Date());
        //将标签的数据存到t_blogs_tag表中
        List<Tag> tags = blog.getTags();
        BlogAndTag blogAndTag = null;
        for (Tag tag : tags) {
            blogAndTag = new BlogAndTag(tag.getId(), blog.getId());
            blogMapper.addBlogAndTag(blogAndTag);
        }
        blogMapper.updateBlog(blog);
        return 1;
    }
}

```



### 3. Controller层

根据前端发送的请求，拦截并进行处理

以admin模式下的BlogController以及游客模式下的IndexController为例：

BlogController.java

```java
@Controller
@RequestMapping("/admin")
public class BlogController {
    @Autowired
    BlogService blogService;

    @Autowired
    TagService tagService;

    @Autowired
    TypeService typeService;


    //管理员模式下显示博客列表
    @GetMapping("/blogs")
    public String showBlogs(@RequestParam(value = "pagenum", defaultValue = "1") int page, Model model){
        PageHelper.startPage(page, 5);
        //查询出所有的博客信息
        List<Blog> blogs = blogService.getAllBlogs();
        PageInfo<Blog> pageInfo = new PageInfo<>(blogs);
        //将数据传送给前端并进行分页显示
        model.addAttribute("pageInfo", pageInfo);
        model.addAttribute("types", typeService.getAllTypes());
        model.addAttribute("tags", tagService.getAllTags());
        return "admin/blogs";
    }

    //点击新增按钮，转入新增博客的界面
    @GetMapping("/blogs/input")
    public String toAddBlog(Model model){
        model.addAttribute("blog", new Blog());
        model.addAttribute("types", typeService.getAllTypes());
        model.addAttribute("tags", tagService.getAllTags());
        //跳转到编辑博客的界面
        return "admin/blogs-input";
    }

    //点击编辑按钮，跳转至新增博客的页面，并显示当前博客的内容
    @GetMapping("/blogs/{id}/input")
    public String toEditPage(@PathVariable Long id , Model model){
        //点击编辑按钮需要获取待编辑博客的基本信息
        Blog blog = blogService.getBlogById(id);
        //将数据传送给前端进行显示
        model.addAttribute("blog", blog);
        model.addAttribute("types", typeService.getAllTypes());
        model.addAttribute("tags", tagService.getAllTags());
        return "admin/blogs-input";
    }

    //点击发布或者保存按钮更新博客的状态
    @PostMapping("/blogs")
    public String addBlogs(Blog blog, HttpSession session, RedirectAttributes attributes){
        //设置user属性
        blog.setUser((User) session.getAttribute("user"));
        //设置用户id
        blog.setUserId(blog.getUser().getId());
        //设置blog的type
        blog.setType(typeService.getTypeById(blog.getType().getId()));
        //设置blog中typeId属性
        blog.setTypeId(blog.getType().getId());
        //给blog中的List<Tag>赋值
        blog.setTags(tagService.getTagsByString(blog.getTagIds()));

        if(blog.getId() == null){
            blogService.addBlog(blog);
        } else {
            blogService.updateBlog(blog);
        }
        attributes.addFlashAttribute("msg", "success");
        return "redirect:/admin/blogs";
    }

    //博客的搜索功能
    @PostMapping("/blogs/search")
    public String searchBlog(@RequestParam(value = "pagenum", defaultValue = "1") int page, Model model, Blog blog){
        PageHelper.startPage(page, 5);
        List<Blog> blogs = blogService.searchBlogsByAdmin(blog);
        PageInfo<Blog> pageInfo = new PageInfo<>(blogs);
        model.addAttribute("pageInfo", pageInfo);
        model.addAttribute("message", "search successfully");
        model.addAttribute("types", typeService.getAllTypes());
        model.addAttribute("tags", tagService.getAllTags());
        return "admin/blogs";
    }

}
```



### 4. 拦截器以及异常处理

配置拦截器拦截所有请求，查看用户是否登录进行访问

```java
//自定义拦截器需要实现HandlerInterceptor接口
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if(request.getSession().getAttribute("user") == null){
            response.sendRedirect("/admin");
            return false;
        }

        return true;
    }
}
//在配置类中配置拦截器
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {   //配置拦截器
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/admin/**")
                .excludePathPatterns("/admin")
                .excludePathPatterns("/admin/login");
    }
}
```



编写一个Handler处理控制层出现的异常

```java
@ControllerAdvice
public class ControllerExceptionHandler {
    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @ExceptionHandler(Exception.class)  //表示该方法可以处理所有类型异常
    public ModelAndView exceptionHandler(HttpServletRequest request, Exception e) throws Exception {

        //日志打印异常信息
        logger.error("Request url: {}, Exception: {}", request.getRequestURI(), e);

        //不处理带有ResponseStatus注解的异常
        if (AnnotationUtils.findAnnotation(e.getClass(), ResponseStatus.class) != null) {
            throw e;
        }

        //返回异常信息到自定义error页面
        ModelAndView mv = new ModelAndView();
        mv.addObject("url", request.getRequestURI());
        mv.addObject("Exception", e);
        mv.setViewName("error/error");

        return mv;
        
    }
}
```



### 5. 配置日志

使用spring AOP进行日志功能的配置

```java
@Aspect    //定义切面
@Component
public class LogAspect {

    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Pointcut("execution(* com.blog.controller.*.*(..))")        //定义切入点表达式
    public void log(){}

    @Before("log()")    //引用切入点
    public void doBefore(JoinPoint joinPoint){

        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        String url = request.getRequestURL().toString();
        String ip = request.getRemoteAddr();
        //获得类名.方法名
        String classMethod = joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName();
        //获得方法参数
        Object[] args = joinPoint.getArgs();

        RequestLog requestLog = new RequestLog(url, ip, classMethod, args);
        //打印请求信息
        logger.info("Request: {}", requestLog);
    }

    @After("log()")
    public void doAfter(){
        logger.info("------------doAfter------------");
    }

    @AfterReturning(returning = "result", pointcut = "log()")
    public void doAfterReturn(Object result){

        //打印返回值
        logger.info("Result: {}", result);
    }

    @Data
    @AllArgsConstructor
    public class RequestLog{      //用于封装请求信息
        private String url;
        private String ip;
        private String classMethod;
        private Object[] args;
    }
}
```

