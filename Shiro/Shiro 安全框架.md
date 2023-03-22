# Shiro 安全框架

## 一、概述

### 1.1 什么是Shiro

Apache Shiro是一个功能强大且易于使用的**Java安全权限框架**。Shiro可以完成：认证、授权、加密、会话管理、与Web集成、缓存等功能。借助Shiro可以**快速轻松**地保护任何应用程序（从最小的移动应用程序到最大的Web和企业应用程序）

> **Apache Shiro™** is a powerful and `easy-to-use` Java security framework that performs authentication, authorization, cryptography, and session management. With Shiro’s easy-to-understand API, you can quickly and easily secure any application – from the smallest mobile applications to the largest web and enterprise applications.

### 1.2 Shiro的优点

+ **易于使用**：使用Shiro构建系统安全框架非常简单，就算第一次接触也可以快速掌握
+ **全面**：Shiro包含系统安全框架需要的功能，满足安全需求的“一站式服务”
+ **灵活**：Shiro可以在任何应用程序环境中工作，它可以在Web、EJB和Ioc环境中工作，但不需要依赖它们。Shiro也没有强制要求任何规范，甚至没有很多的依赖项
+ **强力支持Web**：Shiro具有出色的Web应用程序支持，可以基于应用程序URL和Web协议（比如REST）创建灵活的安全策略，同时还提供一组JSP库来控制页面输出
+ **兼容性强**：Shiro的设计模式使其易于与其他框架和应用集成，Shiro与Spring、Grails、Wicket等框架无缝集成
+ **社区支持**：Shiro是Apache软件基金会的一个开源项目，有完备的社区支持，文档支持

### 1.3 Shiro与Spring Security的对比

（1）Spring Security基于Spring开发，项目若使用Spring作为基础，配合Spring Security做权限控制更加方便，而Shiro需要和Spring进行一个整合

（2）Spring Security的功能更加丰富

（3）Spring Security的社区资源相比Shiro更加丰富

（4）Shiro的配置和使用比较简单，Spring Security上手更加复杂

（5）Shiro依赖性低，不需要任何框架和容器，可以独立运行；Spring Security依赖于Spring容器

（6）Shiro不仅可以使用在Web中，它可以工作在任何应用环境中

### 1.4 基本功能

<img src="Shiro 安全框架.assets/image-20230202091110257.png" alt="image-20230202091110257" style="zoom:80%;" />

### 1.5 Shiro运行流程及基本原理

从应用程序角度看Shiro的运行流程：





## 二、Shiro的基本使用

### 2.1 环境准备

由于Shiro可以独立于其他应用环境，所以可以直接创建maven工程，并引入依赖

```xml
<dependencies>
    <!--shiro的核心包-->
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-core</artifactId>
         <version>1.9.0</version>
    </dependency>

    <!--记录日志-->
    <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>1.2</version>
    </dependency>
</dependencies>
```

### 2.2 创建INI文件

Shiro获取权限相关信息可以通过数据库查询，也可以通过ini配置文件获取

<img src="Shiro 安全框架.assets/image-20230202093609335.png" alt="image-20230202093609335" style="zoom:80%;" />

### 2.3 登录认证

**（1）登录认证的概念**

+ 身份验证：一般需要提供身份ID等一些表示信息来表明登陆这的身份，比如：用户名 / 密码来证明
+ 在shiro中，用户需要提供`principals(身份)`和`credentails(证明)`给shiro，从而来验证用户的身份
+ `principals`：身份，即主体的标识属性，可以是任何属性，比如用户名、邮箱等，唯一即可。一个主体可以有多个`principals`，但只有一个`primary principals`，一般是用户名 / 邮箱 / 手机号
+ `credentials`：证明 / 凭证，即只有主体知道的安全值，如密码 / 数字证书等
+ 最常见的`principals`和`credentials`的组合就是用户名和密码 

**（2）登录认证流程**

+ 收集用户身份 / 凭证，即查询用户的用户名和密码
+ 调用`Subject.login`进行登录；如果失败将抛出异常提示用户
+ Realm类用于保存用户的凭证，可以创建自定义的Realm类

<img src="Shiro 安全框架.assets/image-20230202100334340.png" alt="image-20230202100334340" style="zoom:80%;" />

**（3）Shiro登录认证实例**

最基本的登录认证：

```java
public class ShiroLoginDemo {

    public static void main(String[] args) {
        //1. 初始化获取SecurityManager，在初始化的同时会获取用户在系统中存储的用户名和密码
        IniSecurityManagerFactory factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager securityManager = factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);
        //2. 获取Subject对象
        Subject subject = SecurityUtils.getSubject();
        //3. 创建token对象（即用户在界面输入的用户名和密码）
        AuthenticationToken token = new UsernamePasswordToken("zhangsan", "z123456"); //如果是web应用，可以接收从前端传来的数据
        //4. 调用subject的login方法进行登录
        try {
            //根据相应的错误可以抛出相应的异常，根据异常为用户提供错误信息
            subject.login(token);
            System.out.println("登录成功！");
        } catch (UnknownAccountException e) {
            System.out.println("用户名不存在");
        } catch (IncorrectCredentialsException e) {
            System.out.println("密码错误");
        } catch (AuthenticationException e) {
            System.out.println("其他错误");
        }
    }
}
```

### 2.4 角色授权

**（1）授权的概念**

+ 授权：也叫访问控制，即在应用中**控制谁访问哪些资源**（页面、编辑数据等），在授权中需了解的几个关键对象：主体（subject）、资源（resource）、权限（permission）、角色（role）
+ 主体（Subject）：访问应用的用户，在Shiro中使用Subject代表用户，用户只有授权后才允许访问相应的资源
+ 资源（Resource）：在**应用中用户可以访问的URL**，比如web页面、查看/编辑某些数据、访问某个业务方法、打印文本等等都是资源。用户只有被授权之后才能访问
+ 权限（Permission）：安全策略中的原子授权单位，通过权限我们可以**表示在应用中用户有没有操作某个资源的权力**。
+ Shiro支持粗粒度权限（比如某一模块的所有权限）和细粒度权限（某一个操作的权限）
+ 角色（Role）：**权限的集合**，一般情况下会赋予用户角色而不是权限，即这样用户可以拥有一组权限，赋予权限时比较方便，典型的如：项目经理、技术总监、CTO、开发工程师等，不同角色拥有一组不同的权限

**（2）授权方式**

+ 编程式：通过`if-else`结构判断用户权限

  ```java
  if(subject.hasRole("admin")) {
      //如果用户是admin
  } else {
      //如果不是
  }
  ```

+ 注解式：通过在执行的Java方法上放置相应的注解，如果没有权限将抛出相应的异常

  ```java
  @RequireRoles("admin")
  public void crud() {
      
  }
  ```

**（3）授权流程**

+ 在用户登录成功之后，调用`subject.isPermitted()`方法鉴别用户是否有某项操作的权限（`subject.hasRole()方法鉴别用户是否是某个角色`），将鉴权的流程委托给SecurityManager
+ SecurityManager中的Authorizer是真正的授权者
+ 在进行授权之前，其会调用相应的Realm获取用户相应的角色/权限用于匹配传入的角色/权限
+ Authorizer会判断Realm角色/权限是否和需要的角色相匹配，如果有多个Realm，会委托给ModularRealmAuthorizer进行循环判断
+ 进行授权操作的前提：用户必须通过登录认证

<img src="Shiro 安全框架.assets/image-20230202105816256.png" alt="image-20230202105816256" style="zoom:80%;" />

**（4）授权实例**

在shiro.ini配置文件中加上用户的角色以及角色对应的权限，直接在密码后面添加即可

```ini
[users]
zhangsan=z123456,admin,teacher
lisi=l123456,student

[roles] #角色对用的权限
admin=user:insert,user:select,user:update  
```

在之前登录成功之后即可进行权限验证

```java
public class ShiroLoginDemo {

    public static void main(String[] args) {
        //1. 初始化获取SecurityManager，在初始化的同时会获取用户在系统中存储的用户名和密码
        IniSecurityManagerFactory factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager securityManager = factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);
        //2. 获取Subject对象
        Subject subject = SecurityUtils.getSubject();
        //3. 创建token对象（即用户在界面输入的用户名和密码）
        AuthenticationToken token = new UsernamePasswordToken("zhangsan", "z123456"); //如果是web应用，可以接收从前端传来的数据
        //4. 调用subject的login方法进行登录
        try {
            //根据相应的错误可以抛出相应的异常，根据异常为用户提供错误信息
            subject.login(token);
            System.out.println("登录成功！");
            //5. 登录成功之后即可判断用户是否有相关的角色
            boolean isAdmin = subject.hasRole("admin");
            System.out.println("zhangsan是否有admin角色: " + isAdmin);

            //6. 判断是否有相应的权限
            boolean canSelected = subject.isPermitted("user:select");
            boolean canDeleted = subject.isPermitted("user:delete");
            System.out.println("zhangsan是否有user:select权限: " + canSelected);
            System.out.println("zhangsan是否有user:delete权限: " + canDeleted);
            //也可以直接使用check方法
            subject.checkPermission("user:update");  //checkPermission没有返回值，如果用户没有相应的权限会抛出异常
            System.out.println("zhangsan拥有user:update的权限");
        } catch (UnknownAccountException e) {
            System.out.println("用户名不存在");
        } catch (IncorrectCredentialsException e) {
            System.out.println("密码错误");
        } catch (AuthenticationException e) {
            System.out.println("其他错误");
        }
    }
}

输出：
登录成功！
zhangsan是否有admin角色: true
zhangsan是否有user:select权限: true
zhangsan是否有user:delete权限: false
zhangsan拥有user:update的权限
```

### 2.5 加密

在实际的系统开发中，一些敏感信息需要进行加密，比如用户的密码。Shiro内嵌很多常用的加密算法，比如MD5加密

```java
public class ShiroMD5 {
    public static void main(String[] args) {
        String password = "z123456";
        //使用Shiro内部封装的MD5加密
        Md5Hash security = new Md5Hash(password);
        System.out.println("加密之后的密文: " + security);
        Md5Hash md5Hash = new Md5Hash(password, "abc");
        System.out.println("带盐值的加密: " + md5Hash);
    }
}
```

> **Shiro自带的登录认证并不带有加密的流程**

### 2.6 Shiro自定义登录、鉴权

如果想要实现加密认证的流程需要进行自定义登录认证

 ```java
 /** 自定义登录认证流程*/
 public class MyRealm extends AuthenticatingRealm {
 
     //自定义登录认证方法，如果想要自定义的Realm生效，需要在ini文件或者Springboot的配置文件中进行配置
     @Override //该方法只是获取进行对比的信息（即存储在数据库中的用户信息）
     protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
         //1. 获取用户输入的身份信息和凭证信息（即用户名和密码）
         String username = authenticationToken.getPrincipal().toString();
 
         //2. 从数据库中根据用户名查询密码
         UserService userService = new UserService();
         String password = userService.getPasswordByUsername(username);
         if(password == null || "".equals(password)) {
             throw new UnknownAccountException("账户不存在！");
         }
         System.out.println("用户输入的用户名为：" + username);
         //3. 创建封装校验的逻辑对象
         AuthenticationInfo info = new SimpleAuthenticationInfo(
             authenticationToken.getPrincipal(),   //用户输入的身份
             password,    //数据库中保存的用户的密码
                 ByteSource.Util.bytes("abc"),  //加密所需要的盐值
                 this.getName()   //当前使用的Realm的名称
         );
         return info;
     }
 }
 
 /** 模拟数据库操作*/
 public class UserService {
 
     //模拟从数据库中根据用户名查询密码
     public String getPasswordByUsername(String username) {
         return "a2d3938871213dc7a0be0d92db1153b2";
     }
 }
 ```

对ini文件进行配置

```ini
[main]
md5CredentialsMatcher=org.apache.shiro.authc.credential.Md5CredentialsMatcher
md5CredentialsMatcher.hashIterations=1

myrealm=com.java.shiro.demo.MyRealm
myrealm.credentialsMatcher=$md5CredentialsMatcher
securityManager.realms=$myrealm
```

**如果还想要自定义鉴权，也需要自定义Realm**

```java
public class MyRealm extends AuthorizingRealm {

    //自定义登录认证方法，如果想要自定义的Realm生效，需要在ini文件或者Springboot的配置文件中进行配置
    @Override //该方法只是获取进行对比的信息（即存储在数据库中的用户信息）
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //1. 获取用户输入的身份信息和凭证信息（即用户名和密码）
        String username = authenticationToken.getPrincipal().toString();

        //2. 从数据库中根据用户名查询密码
        UserService userService = new UserService();
        String password = userService.getPasswordByUsername(username);
        if(password == null || "".equals(password)) {
            throw new UnknownAccountException("账户不存在！");
        }
        System.out.println("用户输入的用户名为：" + username);
        //3. 创建封装校验的逻辑对象
        AuthenticationInfo info = new SimpleAuthenticationInfo(
            authenticationToken.getPrincipal(),   //用户输入的身份
            password,    //数据库中保存的用户的密码
                ByteSource.Util.bytes("abc"),  //加密所需要的盐值
                this.getName()   //当前Realm的名称，直接调用getName()方法即可
        );
        return info;
    }

	
    //自定义鉴权方法，主要是从数据库中获取用户的角色和权限
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        //1. 获取用户名
        String username = principals.getPrimaryPrincipal().toString();
        //2. 根据用户名从数据库中获取用户所具有的角色和权限
        UserService userService = new UserService();
        List<String> roles = userService.getRoleByUsername(username);
        List<String> permissions = userService.getPermissionByRole(username);
        //3. 构建资源校验对象
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        info.addRoles(roles);
        info.addStringPermissions(permissions);;
        return info;
    }
}

public class UserService {

    //模拟从数据库中根据用户名查询密码
    public String getPasswordByUsername(String username) {
        return "a2d3938871213dc7a0be0d92db1153b2";
    }

    //模拟从数据库中根据用户名查询用户的权限
    public List<String> getRoleByUsername(String username) {
        List<String> roleList = new ArrayList<>();
        roleList.add("admin");
        roleList.add("teacher");
        return roleList;
    }

    //模拟从数据库中根据角色名查询角色对应的权限
    public List<String> getPermissionByRole(String roleName) {
        List<String> permissionList = new ArrayList<>();
        permissionList.add("user:select");
        permissionList.add("user:update");
        permissionList.add("user:insert");
        return permissionList;
    }

}
```

## 三、Shiro与Springboot的整合

### 3.1 框架整合

以最基本的登录认证为例

（1）导入依赖

```xml
<dependency>
	<groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-web-starter</artifactId>
	<version>1.9.0</version>
</dependency>
```

（2）根据要求编写自定义Realm实现自定义登录认证

```java
@Component
public class MyRealm extends AuthorizingRealm {
    
    @Autowired
    private UserService userService;

    @Override //该方法只是获取进行对比的信息（即存储在数据库中的用户信息）
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //1. 获取用户输入的身份信息和凭证信息（即用户名和密码）
        String username = authenticationToken.getPrincipal().toString();

        //2. 从数据库中根据用户名查询用户信息
        User user = userService.getUserByUsername(username);
        if(user == null) {
            throw new UnknownAccountException("账户不存在！");
        }
        System.out.println("用户输入的用户名为：" + username);
        //3. 创建封装校验的逻辑对象
        AuthenticationInfo info = new SimpleAuthenticationInfo(
            authenticationToken.getPrincipal(),   //用户输入的身份
            user.getPassword(),    //数据库中保存的用户的密码
            ByteSource.Util.bytes("abc"),  //加密所需要的盐值，如果在数据库中保存了盐值，则可从数据库中获取
            this.getName()   //当前Realm的名称，直接调用getName()方法即可
        );
        return info;
    }

	
    //自定义鉴权方法，主要是从数据库中获取用户的角色和权限
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
    }
}
```

（3）编写ShiroConfig配置类

```java
@Configuration
public class ShiroConfig {
    
    @Autowired
    private MyRealm myRealm;
    
    //添加默认的web SecurityManager
    @Bean
    public DefaultWebSecurityManager defaultWebSecurityManager() {
        
        //1. 创建DefaultWebSecurityManager对象
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        //2. 创建加密对象（MD5加密）---  这部分可以在自定义Realm时添加进去
        HashedCrendenitalsMatcher matcher = new HashedCrendenitalsMatcher();
        //2.1 设置加密算法
        matcher.setHashAlgorithmName("md5");
        //2.2 设置加密迭代次数
        matcher.setHashIterations(1);
        //3. 将加密对象存储到MyRealm中
        myRealm.setCredentialsMatcher(matcher);
        //4. 将MyRealm存储到DefaultWebSecurityManager对象中
        defaultWebSecurityManager.setRealm(myRealm);
        return defaultWebSecurityManager;
    }
    
    //添加过滤器
    @Bean
    public DefaultShiroFilterChainDefinition shiroFilterChainDefinition() {
        DefaultShiroFilterChainDefinition definition = new DefaultShiroFilterChainDefinition();
        //设置哪些资源需要登录认证之后才能访问
        definition.addPathDefinition("/**", "authc");  //"authc"即代表需要登录认证才能访问
        //设置哪些资源不需要登录即可访问
        definition.addPathDefinition("/login", "anon");  //"anon"即代表不需要登录即可访问
        return definition;
    }
    
}
```

