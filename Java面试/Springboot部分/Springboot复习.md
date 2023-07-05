# Springboot复习

### 日志系统

> 规范：项目开发中不要编写`System.out.println()`，应该用日志记录信息

| 日志接口（类型）                         | 日志实现                          |
| ---------------------------------------- | --------------------------------- |
| `JCL (Jakarta Commons Logging)`          | `Log4j`和`JUL(java.util.logging)` |
| `SLF4j (Simple Logging Facade for java)` | `Log4j`和`Logback`                |
| `jboss-logging`                          |                                   |

+ Spring使用`commons-logging`作为内部日志，但底层日志实现是开放的，可对接其他日志框架
+ 支持`jul`，`log4j2`，`logback`，Springboot提供了默认的控制台输出配置，也可以配置输出为文件
+ Springboot默认使用`logback`
+ 虽然日志框架很多，但大多数情况下，Springboot的默认日志配置就能完成绝大部分工作

Springboot默认输出的日志格式：

```
2023-05-11 09:54:44.680  INFO 442361 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data Redis repositories in DEFAULT mode.
```

+ 日期和时间：毫秒级的精度  yyyy-MM-dd hh:mm:ss.SSS
+ 日志级别：`ERROR, WARN, INFO, DEBUG, TRACE`
+ 进程id，也可以通过命令行命令`jps`查看
+ ---：消息分隔符
+ [线程名]：运行的线程
+ Logger名：通常是产生日志的类名
+ 消息：日志记录的内容

> 日志的格式可以通过`logging.pattern.console(修改控制台日志输出格式)`和`logging.pattern.file(修改输出至文件的日志格式)`修改

**日志级别**

日志级别由低到高为：`ALL,TRACE,DEBUG,INFO,WARN,ERROR,FATAL,OFF `

+ `ALL`：打印所有日志
+ `TRACE`：追踪框架详细日志流程，一般不使用
+ `DEBUG`：开发调试细节日志
+ `INFO`：关键，感兴趣信息日志
+ `WARN`：警告但不是错误的信息日志，比如：版本过时
+ `ERROR`：业务错误日志，比如出现各种异常
+ `FATAL`：致命错误日志，比如JVM崩溃
+ `OFF`：关闭所有日志记录

> 日志级别在设置时，会打印当前级别以及以后级别的日志，比如当日志级别为`INFO`时，会同时打印`INFO,WARN,ERROR,FATAL`这些日志信息，但不会打印`TRACE,DEBUG`的日志信息
>
> Springboot的日志级别默认为`INFO`,可以通过`logging.level`调整，可以通过指定包名来调整某个包的日志级别
>
> Springboot提供日志分组的功能，可以通过`logging.group.组名`来指定组内的包，比如：
>
> ```yaml
> logging:
>   group:
>     abc:  com.progZhou.controller, com.progZhou.service, com.progZhou,mapper
> #指定controller、service、mapper为一个组，组名为abc
> ```

**日志文件**

可以通过Springboot的配置将日志输出至文件

```yaml
logging:
  file:
    name: /usr/local/app/testwork/xxx.log    ##指定输出日志的文件名，可以加上路径，如果不加，则默认生成至项目同文件目录下
```

**日志文件的归档与切割**

归档：比如每天的日志单独存到一个文档中

切割：指定日志文件的最大尺寸，比如10MB，如果超过最大尺寸，切割为另一个文件

归档和切割的配置：

```yaml
logging:
  logback:
    rollingpolicy:
      file-name-pattern:${LOG_FILE}.%d{yyyy-MM-dd}.%i.log         ##指定日志归档的命名格式
      max-file-size:1MB             ##指定单个日志文件的大小
```

### Web开发

Springboot的web开发能力由SpringMVC提供

1、首先创建项目，导入Web开发场景

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-start-web</artifactId>
</dependency>
```

2、Web场景下的默认效果：

+ 包含了`ContentNegotiatingViewResolver`和`BeanNameViewResolver`组件，方便视图解析
+ 默认的静态资源处理机制：静态资源放在static目录下即可直接访问
+ 自动注册了`Converter`，`GenericConverter`，`Formatter`组件，适配常见的数据类型转换和格式化需求
+ 支持`HttpMessageConverters`，可以方便返回json数据
+ 注册`MessageCodesResolver`方便国际化及错误消息处理
+ 支持静态`index.html`
+ 自动使用`ConfigurableWebBindingInitializer`实现消息处理、数据绑定、类型转换等功能

> 如果想要保持springboot mvc的默认配置，并且自定义更多的mvc配置，比如interceptors，formatters，view controllers等，可以使用`@Configuration`注解创建一个配置类，这个配置类需要继承`WebMvcConfigurer`接口，并且**不能标注`@EnableWebMvc`注解**

3、Web开发中的错误处理机制

Springboot错误处理的自动配置都在`ErrorMvcAutoConfiguration`中，两大核心机制：

+ Springboot会自适应处理错误，响应页面（浏览器）或JSON数据（移动端）
+ SpringMVC的错误处理机制依然保留，MVC处理不了才会交给boot进行处理

> MVC的错误处理方式
>
> ```java
> @Controller
> public class WebController {
>     
>     @GetMapping("/h")
>     public String getList(String a, String b) {
>         return "list";
>     }
>     
>     @ResponseBody
>     @ExceptionHandler(Exception.class)   //@ExceptionHandler能标识Controller中的一个方法来处理业务错误，默认只能处理当前类的错误
>     public String handleException(Exception e) {
>         return "出错了, 原因：" + e.getMessage();
>     }
>     
> }
> 
> //还能使用@ControllerAdvice（类注解）处理项目下所有Controller中出现的错误
> ```

**错误处理的最佳实战**

前后端分离开发模式：

+ 后台发生的所有错误，使用`@ControllerAdvice + @ExceptionHandler`进行统一异常处理

```java
@ControllerAdvice
public class ControllerExceptionHandler {
    
    @ExceptionHandler(xxxException.class)
    public CommonResult<String> xxxExceptionHandler(xxxException e) {
        return xxx;
    }
    
}
```

