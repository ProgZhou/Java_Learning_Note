# Spring部分

## 一、ApplicationContext的refresh流程

### 1. prepareRefresh

**要点**

+ 这一步创建和准备了Environment对象
+ 理解Environment对象的作用
  + 为后续spring程序运行时，提供键值信息
  + systemProperties：java虚拟机中的键值信息
  + systemEnvironment：操作系统提供的键值对
  + PropertySource：自定义键值信息（springboot的application.yml）
  + 总之Environment的作用之一就是为后续@Value值注入时提供键值

![image-20210902181639048](Spring部分.assets/image-20210902181639048.png)

+ 调试代码的步骤

测试代码：

```java
public static void main(String[] args) throws Exception {
    // 1) 获得 @Value 的值
    System.out.println("=======================> 仅获取 @Value 值");
    /*
    QualifierAnnotationAutowireCandidateResolver类的作用：拿到@Value注解中的值
    	+ getSuggestedValue()方法：获取目标上@Value注解的值（最原始的值，不进行任何解析，通俗地说就是字符串）
    	  +  new DependencyDescriptor(Bean1.class.getDeclaredField("name")：取的@Value的位置
    	  	Bean1.class.getDeclaredField表示要取类Bean1，name属性上的@Value的值
    	  + boolean required：是否是必须的
    */
    QualifierAnnotationAutowireCandidateResolver resolver = new QualifierAnnotationAutowireCandidateResolver();
    Object name = resolver.getSuggestedValue(new DependencyDescriptor(Bean1.class.getDeclaredField("name"), false));
    System.out.println(name);

    // 2) 解析 @Value 的值  --- 使用Environment的实现类
    System.out.println("=======================> 获取 @Value 值, 并解析${}");
    Object javaHome = resolver.getSuggestedValue(new DependencyDescriptor(Bean1.class.getDeclaredField("javaHome"), false));
    System.out.println(javaHome);
    /*
    通过resolvePlaceholders方法对${}中的内容进行解析
    */
    System.out.println(getEnvironment().resolvePlaceholders(javaHome.toString()));

    // 3) 解析 SpEL 表达式  -- 先解析内部的${}再解析外部的#{}
    System.out.println("=======================> 获取 @Value 值, 并解析#{}");
    Object expression = resolver.getSuggestedValue(new DependencyDescriptor(Bean1.class.getDeclaredField("expression"), false));
    System.out.println(expression);
    String v1 = getEnvironment().resolvePlaceholders(expression.toString());
    System.out.println(v1);
    System.out.println(new StandardBeanExpressionResolver().evaluate(v1, new BeanExpressionContext(new DefaultListableBeanFactory(),null)));
}

/*
Environment是一个接口，可以直接返回它的实现类
*/
private static Environment getEnvironment() throws IOException {
    StandardEnvironment env = new StandardEnvironment();
    //如果是在classpath下有自定义的properties文件，需要告诉spring这个文件的位置
    //env.getPropertySources().addLast(new ResourcePropertySource("jdbc", new ClassPathResource("jdbc.properties")));
    return env;
}

static class Bean1 {
    @Value("hello")
    private String name;

    @Value("${JAVA_HOME}")
    private String javaHome;

    @Value("#{'class version:' + '${java.class.version}'}")
    private String expression;
}
```

### 2. obtainFreshBeanFactory

**要点**

+ 这一步获取（或者创建）`BeanFactory`
+ 理解BeanFactory的作用
  + 负责bean的创建、依赖注入和初始化
  + bean的各项信息需要由BeanDefinition提供
+ 理解BeanDefinition的作用
  + 作为bean的设计蓝图，规定了bean的特征，如单例多例，依赖关系，初始销毁方法等
  + BeanDefinition的来源多种多样，可以通过xml解析获取，也可以通过配置类获取还能够通过包扫描获取
+ 所有的 BeanDefinition 会存入 BeanFactory 中的 beanDefinitionMap 集合

![image-20210902182004819](Spring部分.assets/image-20210902182004819.png)

**总结**

+ BeanFactory的作用就是负责bean的创建、依赖注入和初始化

测试代码：BeanDefinition的各种来源

```java
public static void main(String[] args) {
    //开始时，并没有任何BeanDefinition
    System.out.println("========================> 一开始");
    DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
    System.out.println(Arrays.toString(beanFactory.getBeanDefinitionNames()));

    //从XML文件中获取bean的定义
    System.out.println("========================> 1) 从 xml 获取 ");
    XmlBeanDefinitionReader reader1 = new XmlBeanDefinitionReader(beanFactory);
    reader1.loadBeanDefinitions(new ClassPathResource("bd.xml"));
    System.out.println(Arrays.toString(beanFactory.getBeanDefinitionNames()));

    System.out.println("========================> 2) 从配置类获取 ");
    //通过编程的方式将配置类加入到容器中
    beanFactory.registerBeanDefinition("config1", BeanDefinitionBuilder.genericBeanDefinition(Config1.class).getBeanDefinition());

    /*
    bean工厂的后处理器，解析配置类，对BeanFactory的功能做一些增强
    */
    ConfigurationClassPostProcessor postProcessor = new ConfigurationClassPostProcessor();
    postProcessor.postProcessBeanDefinitionRegistry(beanFactory);
    System.out.println(Arrays.toString(beanFactory.getBeanDefinitionNames()));

    System.out.println("========================> 3) 扫描获取 ");
    ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(beanFactory);
    scanner.scan("day04.refresh.sub");
    System.out.println(Arrays.toString(beanFactory.getBeanDefinitionNames()));
}

static class Bean1 {

}

static class Bean2 {

}

static class Config1 {
    @Bean
    public Bean2 bean2() {
        return new Bean2();
    }
}
```

### 3. prepareBeanFactory

**要点**

+ 完善BeanFactory，为BeanFactory的各个属性赋值
  + beanExpressionResolver 用来解析 SpEL，常见实现为 StandardBeanExpressionResolver
  + propertyEditorRegistrars 会注册类型转换器
  + registerResolvableDependency 来注册 beanFactory 以及 ApplicationContext，让它们也能用于依赖注入
  + beanPostProcessors 是 bean 后处理器集合，会工作在 bean 的生命周期各个阶段，此处会添加两个（ApplicationContextAwareProcessor和ApplicationListenerDetector）

![image-20210902182541925](Spring部分.assets/image-20210902182541925.png)



+ 了解谁来执行类型转换
+ 了解特殊bean注入
+ 两个内置的BeanPostProcessor的作用

### 4. postProcessBeanFactory

**要点**

+ 这一步是空实现，留给子类扩展

  ```java
  protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
  }
  ```

+ 模板方法的设计模式，提供父类方法，子类可扩展自己的方法

### 5. invokeBeanFactoryPostProcessors

**要点**

+ 理解beanFactory后处理器的作用
  + 充当beanFactory的扩展点，可以用来补充或修改BeanDefinition
+ 掌握常见的beanFactory后处理器
  + ConfigurationClassPostProcessor  –  解析 @Configuration、@Bean、@Import、@PropertySource 等
  + MapperScannerConfigurer – 补充 Mapper 接口对应的 BeanDefinition

![image-20210902183232114](Spring部分.assets/image-20210902183232114.png)

### 6. registerBeanPostProcessors

**要点**

这一步是继续从beanFactory中找出bean后置处理器，添加至beanPostProcessor集合中

+ bean后置处理器的作用：充当bean的扩展点，可以工作在bean的实例化、依赖注入、初始化阶段，常见的有：

  + AutowiredAnnotationBeanPostProcessor 功能有：解析 @Autowired，@Value 注解
  + CommonAnnotationBeanPostProcessor 功能有：解析 @Resource，@PostConstruct，@PreDestroy
  + AnnotationAwareAspectJAutoProxyCreator 功能有：为符合切点的目标 bean 自动创建代理（AOP）

  ```java
  public static void main(String[] args) {
      GenericApplicationContext context = new GenericApplicationContext();
      DefaultListableBeanFactory beanFactory = context.getDefaultListableBeanFactory();
      //通过编程的方式创建实例对象
      beanFactory.registerBeanDefinition("bean1", BeanDefinitionBuilder.genericBeanDefinition(Bean1.class).getBeanDefinition());
      beanFactory.registerBeanDefinition("bean2", BeanDefinitionBuilder.genericBeanDefinition(Bean2.class).getBeanDefinition());
      beanFactory.registerBeanDefinition("bean3", BeanDefinitionBuilder.genericBeanDefinition(Bean3.class).getBeanDefinition());
      beanFactory.registerBeanDefinition("aspect1", BeanDefinitionBuilder.genericBeanDefinition(Aspect1.class).getBeanDefinition());
      
      //添加BeanPostProcesser
      beanFactory.registerBeanDefinition("processor1",
              BeanDefinitionBuilder.genericBeanDefinition(AutowiredAnnotationBeanPostProcessor.class).getBeanDefinition());
      beanFactory.registerBeanDefinition("processor2",
              BeanDefinitionBuilder.genericBeanDefinition(CommonAnnotationBeanPostProcessor.class).getBeanDefinition());
      beanFactory.registerBeanDefinition("processor3",
              BeanDefinitionBuilder.genericBeanDefinition(AnnotationAwareAspectJAutoProxyCreator.class).getBeanDefinition());
  
      context.refresh();
      beanFactory.getBean(Bean1.class).foo();
  }
  
  static class Bean1 {
      Bean2 bean2;
      Bean3 bean3;
  
      @Autowired
      public void setBean2(Bean2 bean2) {
          System.out.println("发生了依赖注入..." + bean2);
          this.bean2 = bean2;
      }
  
      @Resource
      public void setBean3(Bean3 bean3) {
          System.out.println("发生了依赖注入..." + bean3);
          this.bean3 = bean3;
      }
  
      public void foo() {
          System.out.println("foo");
      }
  }
  
  static class Bean2 {
  }
  
  static class Bean3 {
  }
  
  @Aspect
  static class Aspect1 {
      @Before("execution(* foo())")
      public void before() {
          System.out.println("before...");
      }
  }
  ```

**总结**

前六步都在不断地完善BeanFactory

### 7. initMessageSource

第七步又回到了ApplicationContext，为ApplicationContext添加属性赋值，这一步是为 ApplicationContext 添加 messageSource 成员，实现国际化功能

**要点**

+ MessageSource的作用：去 beanFactory 内找名为 messageSource 的 bean，如果没有，则提供空的 MessageSource 实现（即不支持国际化）
+ 国际化是Application独有的功能

![image-20210902183819984](Spring部分.assets/image-20210902183819984.png)

### 8. initApplicationContextEventMulticaster

**要点**

* 这一步为 ApplicationContext 添加事件广播器成员，即 applicationContextEventMulticaster
* 它的作用是发布事件给监听器
* 去 beanFactory 找名为 applicationEventMulticaster 的 bean 作为事件广播器，若没有，会创建默认的事件广播器
* 之后就可以调用 ApplicationContext.publishEvent(事件对象) 来发布事件

![image-20210902183943469](Spring部分.assets/image-20210902183943469.png)

### 9. onRefresh

**要点**

+ 这一步是空实现，留给子类扩展
  + 常见的使用是SpringBoot中的子类在这里准备了WebServer（内嵌的Tomcat）
+ 模板方法的设计模式

### 10. registerListeners

**要点**

+ 这一步从多种途径找到事件监听器，并添加至applicationEventMulticaster
+ 事件监听器的作用就是用来接收事件广播器发布的事件，一般的来源有：
  + 事先编程添加的
  + 来自容器中的bean
  + 来自于@EventListener的解析
+ 要实现事件监听器，需要实现ApplicationListener接口，重写其中的onApplicationEvent(E e)方法

![image-20210902183943469](Spring部分.assets/image-20210902183943469-16558142044861.png)



### 11. finishBeanFactoryInitialization

**要点**

+ 这一步会将beanFactory的成员补充完整，并初始化所有非延迟单例bean
+ 了解conversionService的作用：是一套转换机制，作为PropertyEditor的补充
+ embaddedValueResolvers：内嵌值解析器，用来解析@Value中的${}表达式，底层也是调用了Environment的功能
+ singletonObjects：单例池，缓存所有的单例对象
  + 对象的创建都分为三个阶段，每一个阶段都有不同的bean后处理器参与进来，扩展功能

![image-20210902184641623](Spring部分.assets/image-20210902184641623.png)

### 12. finishRefresh

**要点**

+ 这一步会将ApplicationContext添加至lifecycleProcessor 成员，用来控制容器内需要生命周期管理的 bean

* 准备好生命周期管理器，就可以实现
  * 调用 context 的 start，即可触发所有实现 LifeCycle 接口 bean 的 start
  * 调用 context 的 stop，即可触发所有实现 LifeCycle 接口 bean 的 stop
* 发布 ContextRefreshed 事件，整个 refresh 执行完成

![image-20210902185052433](Spring部分.assets/image-20210902185052433.png)

### 总结

refresh是AbstractApplicationContext中的一个方法，负责初始化ApplicationContext容器

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      prepareBeanFactory(beanFactory);

      try {
         // Allows post-processing of the bean factory in context subclasses.
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         initMessageSource();

         // Initialize event multicaster for this context.
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         onRefresh();

         // Check for listener beans and register them.
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         finishRefresh();
      }

      catch (BeansException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("Exception encountered during context initialization - " +
                  "cancelling refresh attempt: " + ex);
         }

         // Destroy already created singletons to avoid dangling resources.
         destroyBeans();

         // Reset 'active' flag.
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }

      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
```

大致可以分为四个大步骤

+ 1准备环境
+ 2 3 4 5 6准备BeanFactory
+ 7 8 9 10 12准备ApplicationContext
+ 11初始化BeanFactory中的非延迟单例bean



## 二、Spring bean的生命周期

### 阶段划分的依据：doGetBean方法

```java
//AbstractBeanFactory类中的方法
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
      @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

}
```

### 阶段1：处理名称，检查缓存

+  这一步首先会处理bean的别名，讲别名解析为实际名称

  ```java
  //对应于代码中的这一句
  final String beanName = transformedBeanName(name);
  
  //最终这个方法会调用自己类中的成员方法：
  public String canonicalName(String name) {
  		String canonicalName = name;
  		// Handle aliasing...
  		String resolvedName;
  		do {
              //aliasMap是存放别名的map，本质上是ConcurrentHashMap
  			resolvedName = this.aliasMap.get(canonicalName);
  			if (resolvedName != null) {
  				canonicalName = resolvedName;
  			}
  		}
  		while (resolvedName != null);
  		return canonicalName;
  	}
  ```

+ 对FactoryBean也会有特殊的处理，如果以"&"开头，就表示要获取FactoryBean本身，否则表示要获取其产品

+ 对单例对象会去检查spring的一级、二级、三级缓存

  + 一级缓存：singletonObjects，存放已经完成创建的单例成品对象
  + 二级缓存：earlySingletonObjects，存放单例工厂的产品对象
  + 三级缓存： singletonFactories，存放单例工厂对象

  ```java
  //对应于这段代码
  // Eagerly check singleton cache for manually registered singletons.
  Object sharedInstance = getSingleton(beanName);
  
  //这个方法调用DefaultSingletonBeanRegistry类中的：
  protected Object getSingleton(String beanName, boolean allowEarlyReference) {
      //会先查找一级缓存中是否有这个单例对象
  		Object singletonObject = this.singletonObjects.get(beanName);
      //下面这段是解决循环依赖的代码，之后会细说
  		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
  			synchronized (this.singletonObjects) {
  				singletonObject = this.earlySingletonObjects.get(beanName);
  				if (singletonObject == null && allowEarlyReference) {
  					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
  					if (singletonFactory != null) {
  						singletonObject = singletonFactory.getObject();
  						this.earlySingletonObjects.put(beanName, singletonObject);
  						this.singletonFactories.remove(beanName);
  					}
  				}
  			}
  		}
  		return singletonObject;
  	}
  ```

### 阶段2、处理父容器

* 如果当前容器根据名字找不到这个 bean，此时若父容器存在，则执行父容器的 getBean 流程
* 父子容器的 bean 名称可以重复



### 阶段3、处理dependsOn

* 如果当前 bean 有通过 dependsOn 指定了非显式依赖的 bean，这一步会提前创建这些 dependsOn 的 bean 
* 所谓非显式依赖，就是指两个 bean 之间不存在直接依赖关系，但需要控制它们的创建先后顺序

### 阶段4、按照scope创建bean

scope即为bean的作用域，常见的有singleton和prototype

+ singleton：从单例池（一级缓存）中获取bean对象，如果没有，则创建之后再放入这个单例池中
  + 单例池是在refresh的第十一个步骤就被创建出来了，顺带把其中部分单例对象也创建出来放到单例池了
  + 单例对象的销毁是随着容器的销毁而销毁
  + 所有bean的生命周期的起点都是doGetBean方法，虽然单例池在refresh阶段被创建出来，底层仍然调用了getBean方法
+ prototype：多例bean不受spring容器的管理，每次在调用getBean方法时，都会创建一个对象，需要用户手动调用方法进行销毁，生命周期由用户决定

> `getBean()`方法是顶层接口BeanFactory中定义的方法，而AbstractBeanFactory中扩展了这个方法，最终调用其内部方法`doGetBean()`来创建对象，所以，一切bean生命周期的起点都是getBean方法

### **阶段5、创建bean**

#### 5.1 创建bean实例

有关键的两个创建方法：

+ **AutowiredAnnotationBeanPostProcessor**：
  + ① 优先选择带  @Autowired  注解的构造
  + ② 若有唯一的带参构造，也会入选
+ **采用默认的构造方法**：如果上面的后处理器和 BeanDefiniation 都没找到构造，采用默认构造，即使是私有的（会采用暴力反射的方法）

#### 5.2 依赖注入

通过各种beanPostProcessor去寻找需要注入的属性，常见的注入方式有：

+ AutowiredAnnotationBeanPostProcessor：**识别**   @Autowired  及 @Value  标注的成员，封装为  InjectionMetadata 进行依赖注入，最终进行依赖注入的是InjectionMetadata，这个beanPostProcessor只负责识别
+ CommonAnnotationBeanPostProcessor：**识别**   @Resource  标注的成员，封装为  InjectionMetadata 进行依赖注入
+ AUTOWIRE_BY_NAME：根据成员名字找 bean 对象，修改 mbd 的 propertyValues，不会考虑简单类型的成员
+ AUTOWIRE_BY_TYPE：根据成员类型执行 resolveDependency 找到依赖注入的值，修改  mbd 的 propertyValues
+ applyPropertyValues：根据 mbd 的 propertyValues 进行依赖注入（即xml中 `<property name ref|value/>`）

> 各种依赖注入的优先级：
>
> + 在xml配置中的优先级最高
> + 其次是根据名称和类型注入（就是xml配置中的byName或者byType）
> + 优先级最低的是使用注解方式进行注入的

#### 5.3 初始化阶段





### 总结

基本的生命周期：

```mermaid
graph LR;
构造实例化 --> 依赖注入
依赖注入 --> 初始化方法
初始化方法 --> bean的使用
bean的使用 --> 销毁方法
```

可以使用`BeanPostProcessor`bean后处理器对bean生命周期的各个阶段进行一定的增强

```java
@Component
@Slf4j
public class MyBeanPostProcessor implements InstantiationAwareBeanPostProcessor, DestructionAwareBeanPostProcessor {
    @Override
    public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")) {
            log.info("<<<<<<  在销毁方法执行之前执行");
        }
    }

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean"))
            log.info("<<<<<< 实例化（构造方法）之前执行, 这里返回的对象会替换掉原本的 bean");
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            log.info("<<<<<< 实例化（构造方法）之后执行, 这里如果返回 false 会跳过依赖注入阶段");
//            return false;
        }
        return true;
    }

    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean"))
            log.info("<<<<<< 依赖注入阶段执行, 如 @Autowired、@Value、@Resource");
        return pvs;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean"))
            log.info("<<<<<< 初始化方法之前执行, 这里返回的对象会替换掉原本的 bean, 如 @PostConstruct、@ConfigurationProperties");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean"))
            log.info("<<<<<< 初始化方法之后执行, 这里返回的对象会替换掉原本的 bean, 如代理增强");
        return bean;
    }
}
输出结果：
2022-07-25 15:30:09.824  INFO 6888 --- [  restartedMain] c.j.s.d.processor.MyBeanPostProcessor    : <<<<<< 实例化（构造方法）之前执行, 这里返回的对象会替换掉原本的 bean
2022-07-25 15:30:09.826  INFO 6888 --- [  restartedMain] com.java.spring.demo.life.LifeCycleBean  : bean构造方法
2022-07-25 15:30:09.829  INFO 6888 --- [  restartedMain] c.j.s.d.processor.MyBeanPostProcessor    : <<<<<< 实例化（构造方法）之后执行, 这里如果返回 false 会跳过依赖注入阶段
2022-07-25 15:30:09.829  INFO 6888 --- [  restartedMain] c.j.s.d.processor.MyBeanPostProcessor    : <<<<<< 依赖注入阶段执行, 如 @Autowired、@Value、@Resource
2022-07-25 15:30:09.830  INFO 6888 --- [  restartedMain] com.java.spring.demo.life.LifeCycleBean  : 依赖注入: C:\Program Files\Java\jdk1.8.0_74
2022-07-25 15:30:09.831  INFO 6888 --- [  restartedMain] c.j.s.d.processor.MyBeanPostProcessor    : <<<<<< 初始化方法之前执行, 这里返回的对象会替换掉原本的 bean, 如 @PostConstruct、@ConfigurationProperties
2022-07-25 15:30:09.832  INFO 6888 --- [  restartedMain] com.java.spring.demo.life.LifeCycleBean  : 初始化方法
2022-07-25 15:30:09.832  INFO 6888 --- [  restartedMain] c.j.s.d.processor.MyBeanPostProcessor    : <<<<<< 初始化方法之后执行, 这里返回的对象会替换掉原本的 bean, 如代理增强
2022-07-25 15:30:10.938  INFO 6888 --- [  restartedMain] c.j.s.d.processor.MyBeanPostProcessor    : <<<<<<  在销毁方法执行之前执行
2022-07-25 15:30:10.939  INFO 6888 --- [  restartedMain] com.java.spring.demo.life.LifeCycleBean  : 销毁方法
```



## 附录1 ApplicationContext与BeanFactory

### ApplicationContext与BeanFactory之间的关系

BeanFactory：是ApplicationContext的父接口，spring真正的核心容器，是ApplicationContext的一个成员变量

ApplicationContext中的某一部分功能是间接调用BeanFactory的功能

BeanFactory的结构：

![image-20220623204809391](Spring部分.assets/image-20220623204809391.png)

### BeanFactory的功能

在BeanFactory的接口定义中，最主要的功能就是getBean方法

实际上，BeanFactory的功能有很多，都是由其实现类提供的，包括：

+ 控制反转
+ 依赖注入
+ 以及Bean的生命周期的管理等等

一个常用的实现类DefaultListableBeanFactory

<img src="Spring部分.assets/image-20220623210101097.png" alt="image-20220623210101097" style="zoom:80%;" />

其中一个比较重要类是DefaultSingletonBeanRegistry，用于管理spring中的所有单例对象

比较重要的属性就是spring的三级缓存

```java
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);
```

### ApplicationContext的作用

ApplicationContext的作用完全取决于它所继承的接口，主要的功能有四个，分别通过继承四个接口实现

![image-20220623211826794](Spring部分.assets/image-20220623211826794.png)

+ MessageSource：国际化功能，主要提供一些语言的翻译功能，主要的方法就是`getMessage()`方法，一些信息都配置在message.properties中
+ ResourcePatternResolver：匹配资源的功能，资源是指从磁盘路径或者类路径下找到的一些文件，可以根据通配符匹配多个资源，主要的方法是`getResources()`和`getResource()`方法
+ ApplicationEventPublisher：用来发布事件对象，主要的优点就是可以实现容器内部功能的解耦，
+ EnvironmentCapable：读取一些环境信息（配置信息），主要的方法就是`getProperty()`方法

> ApplicationEventPublisher的使用：
>
> ApplicationContext可以使用`publishEvent()`方法发送事件，事件可以自己定义，但需要实现一个接口，来标记这是一个事件
>
> ```java
> public class UserRegisterEvent extends ApplicationEvent {
>     public UserRegisterEvent(Object source) {
>         super(source);
>     }
> }
> ```
>
> spring容器中任何一个对象都可以作为监听器使用，在类中可以使用某个方法来接收事件
>
> ```java
> @Component
> @Slf4j
> public class ComponentB {
>     @EventListener   //spring容器中任意一个对象都能够作为事件监听器，这个注解表示这个方法需要接收事件
>     public void receiveEvent(UserRegisterEvent event) {
>         log.debug("{}", event);
>     }
> }
> ```
>
> 这样就可以做到一些处理事件的代码解耦

### 总结

ApplicationContext与BeanFactory不仅仅是简单的接口继承关系

+ ApplicationContext组合并扩展了BeanFactory的功能（以BeanFactory作为自己的属性）
+ 事件的手法，新的代码之间解耦的方式



## 附录2 BeanFactory的实现类

 ### DefaultListableBeanFactory

是最主要的BeanFactory的实现类

```java
public class TestBeanFactory {
    public static void main(String[] args) {
        //一个最主要的beanFactory的实现类
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        //bean的定义：class，scope（作用域），初始化方法，销毁方法
        AbstractBeanDefinition beanDefinition =
                BeanDefinitionBuilder.genericBeanDefinition(Config.class).setScope("singleton").getBeanDefinition();
        //将bean注册到beanFactory中
        beanFactory.registerBeanDefinition("config", beanDefinition);

        //给beanFactory添加一些后置处理器
        AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);
        
        //获取beanFactory中的所有beanFactory的后置处理器，后置处理器的作用是补充了一些bean的定义
        Map<String, BeanFactoryPostProcessor> postProcessorMap =
                beanFactory.getBeansOfType(BeanFactoryPostProcessor.class);
        //执行每一个beanFactory的后置处理器
        postProcessorMap.values().stream().forEach(beanFactoryPostProcessor -> {
            beanFactoryPostProcessor.postProcessBeanFactory(beanFactory);
        });

        for (String name: beanFactory.getBeanDefinitionNames()) {
            System.out.println(name);
        }

        //bean后处理器，针对bean生命周期的各个阶段提供一些扩展功能，比如@Autowired，依赖注入
        Map<String, BeanPostProcessor> processorMap = beanFactory.getBeansOfType(BeanPostProcessor.class);
        processorMap.values().stream().forEach(beanFactory :: addBeanPostProcessor);

        //即使到这个地步，beanFactory仍然没有依赖注入的功能，依赖注入的功能仍然是由bean后置处理器提供的
        //bean后置处理器是针对每一个bean的生命周期进行扩展
        Bean1 bean1 = beanFactory.getBean(Bean1.class);
        System.out.println(bean1.getBean2());  

    }

    @Configuration
    static class Config {
        @Bean
        public Bean1 bean1() {
            return new Bean1();
        }

        @Bean
        public Bean2 bean2() {
            return new Bean2();
        }
    }

    static class Bean1 {
        @Autowired
        private Bean2 bean2;

        public Bean1() {
            log.debug("构造Bean1()");
        }

        public Bean2 getBean2() {
            return bean2;
        }
    }

    static class Bean2 {
        public Bean2() {
            log.debug("构造Bean2()");
        }
    }
}
```

需要注意的是，如果单纯的使用new的方法创建beanFactory，这个BeanFactory并不具有其他任何功能，因为BeanFactory只是一个基础容器，并没有其他的扩展功能，它的扩展功能都是由其后置处理器实现的，比如：

+ ConfigurationAnnotationProcessor：解析配置类的注解@Configuration
+ AutowiredAnnotationProcessor / CommonAnnotationProcessor：解析@Autowired和@Resource进行依赖注入的注解
+ ...

BeanFactory的特点：

+ 不会主动调用BeanFactory的后处理器
+ 不会主动添加Bean后处理器
+ 不会主动初始化单例对象（第一次调用方法时创建，懒加载）

### 后处理器的执行顺序

首先说明一点，使用@Autowire注解默认是按照类型注入的，如果一个接口有多个实现类，那么只使用@Autowire注解会报错，解决方法是加上@Qualifier("")使用按照名称注入，还有一种方法就是将成员名命名成和类相同的名称

```java
public interface BeanInterface{}

public class Bean1 implements BeanInterface {}
public class Bean2 implements BeanInterface {}

public class Bean3 {
    @Autowire   //使用@Autowire注解会首先按照名称去找，如果找到了就直接注入
    private BeanInterface bean1;   //这样写注入的就是Bean1
    
}
```

Bean后处理器的执行顺序是：哪个后处理器先加入到beanFactory中，哪个后处理器就先生效

但这个后处理器的执行顺序是可以被改变的



## 附录3 ApplicationContext的实现类

### ClassPathXmlApplicationContext

顾名思义是在类路径下找到xml配置文件并解析，附带了依赖注入的功能

```java
ClassPathXmlApplicationContext context =
        new ClassPathXmlApplicationContext("bean1.xml");
for (String name : context.getBeanDefinitionNames()) {   //输出容器中有多少bean
    System.out.println(name);
}

Bean2 bean2 = context.getBean(Bean2.class);
System.out.println(bean2);
System.out.println(bean2.getBean1());

输出：
bean1
bean2
com.spring.source.entity.Bean2@72e5a8e
com.spring.source.entity.Bean1@54e1c68b
```

### FileSystemXmlApplicationContext

在磁盘路径下找到xml配置文件，C://或者D://等等

```java
FileSystemXmlApplicationContext context =
        new FileSystemXmlApplicationContext("D:\\Java_idea\\JavaSpringBoot\\springboot-source\\src\\main\\resources\\bean1.xml");
for (String name : context.getBeanDefinitionNames()) {
    System.out.println(name);
}

Bean2 bean2 = context.getBean(Bean2.class);
System.out.println(bean2);
System.out.println(bean2.getBean1());

输出：
bean1
bean2
com.spring.source.entity.Bean2@72e5a8e
com.spring.source.entity.Bean1@54e1c68b
```

在这些实现类中，起始都有一个类去执行读取xml配置并转换为beanDefinition的功能，简化的实现可以看下面的代码

```java
//ClassPathXmlApplicationContext内部的实现
//ApplicationContext中都有一个beanFactory属性去完成管理bean的操作
DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
System.out.println("读取之前");
//刚开始时，beanFactory内部没有任何bean
for (String name : beanFactory.getBeanDefinitionNames()) {
    System.out.println(name);
}

//这个类主要去执行读取xml中bean的配置，将这些配置转换为BeanDefinition，并放入指定的beanFactory中
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
reader.loadBeanDefinitions(new ClassPathResource("bean1.xml"));  //FileSystemXmlApplicationContext只是在这个换成了FileSystemResource
System.out.println("读取之后");
for (String name : beanFactory.getBeanDefinitionNames()) {
    System.out.println(name);
}
```

### AnnotationConfigApplicationContext

解析配置类来获取BeanDefinition，与之前两个的区别就是会自动加入几个bean后处理器

```java
AnnotationConfigApplicationContext context =
        new AnnotationConfigApplicationContext(BeanConfig.class);

for (String name : context.getBeanDefinitionNames()) {
    System.out.println(name);
}
Bean2 bean2 = context.getBean(Bean2.class);
System.out.println(bean2);
System.out.println(bean2.getBean1());

输出：
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
beanConfig
bean1
bean2
com.spring.source.entity.Bean2@480d3575
com.spring.source.entity.Bean1@f1da57d
```

### AnnotationConfigServletWebApplicationContext

专门为Web环境准备的ApplicationContext，它的配置类中必须包含三个内容

```java
@Configuration
public class WebConfig {

    //创建Web服务器的工程
    @Bean
    public ServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();  //Springboot内嵌的Tomcat就是这样创建出来的
    }

    //前端控制器
    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }

    //将前端控制器与服务器关联
    @Bean
    public DispatcherServletRegistrationBean registrationBean(DispatcherServlet dispatcherServlet) {
        return new DispatcherServletRegistrationBean(dispatcherServlet, "/");
    }

    //处理请求的控制器方法
    @Bean("/hello")
    public Controller controller() {
        return (request, response) -> {
            response.getWriter().println("hello");
            return null;
        };
    }

}
```

```java
AnnotationConfigServletWebApplicationContext context =
        new AnnotationConfigServletWebApplicationContext(WebConfig.class);
for (String name : context.getBeanDefinitionNames()) {
    System.out.println(name);
}

输出：
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
webConfig
servletWebServerFactory
dispatcherServlet
registrationBean
/hello
```

### 设计模式 --- 模板方法

可以自己设计一个简单的BeanFactory：

```java
public class TestTemplateMethod {
    public static void main(String[] args) {
        MyBeanFactory beanFactory = new MyBeanFactory();
        beanFactory.getBean();
    }
    static class MyBeanFactory {
        //模拟一个bean的生命周期
        public Object getBean() {
            Object bean = new Object();
            System.out.println("constructor: " + bean);  //构造方法调用
            System.out.println("dependency: " + bean);  //依赖注入阶段
            System.out.println("initialization: " + bean);  //初始化阶段
            return bean;
        }
    }
}
```

但这段代码的可扩展性很差，因为如果想要对getBean方法提出一些新的要求，就需要去修改getBean的源代码，添加一些功能，这就非常麻烦，为了解决这样的问题，可以创建一个接口：

```java
interface BeanPostProcessor {
    void inject(Object bean);   //对依赖注入阶段做一些扩展
}
//修改BeanFactory的代码
static class MyBeanFactory {
        //模拟一个bean的生命周期
        public Object getBean() {
            Object bean = new Object();
            System.out.println("constructor: " + bean);  //构造方法调用
            System.out.println("dependency: " + bean);  //依赖注入阶段
            //在这个位置调用后处理器中的方法对依赖注入做一个增强
            for (BeanPostProcessor processor : processors) {
                processor.inject(bean);
            }
            System.out.println("initialization: " + bean);  //初始化阶段
            return bean;
        }

        private List<BeanPostProcessor> processors = new ArrayList<>();
        //添加后处理器
        public void addBeanPostProcessor(BeanPostProcessor processor) {
            processors.add(processor);
        }
    }
```

这样，如果向再对getBean方法提出一些新的要求，可以再接口中增加方法，并创建实现类去实现这些方法，然后添加到BeanFactory中，这样BeanFactory就有了多种多样的功能了



## 附录4 常见的Bean的后处理器

Bean后处理器的主要作用就是为bean生命周期的各个阶段提供一些扩展

**AutowiredAnnotationBeanPostProcessor**

用于解析@Autowired、@Value注解的bean后处理器

```java
public class BeanPostProcessorTest {
    public static void main(String[] args) {
        //一个比较干净的容器，没有添加一些特定的Bean后处理器
        GenericApplicationContext context = new GenericApplicationContext();

        //将三个Bean注入到容器中
        context.registerBean("bean1", Bean1.class);
        context.registerBean("bean2", Bean2.class);
        context.registerBean("bean3", Bean3.class);

        context.getDefaultListableBeanFactory().
                setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());  //暂时不管，是解析字符串用的

        //添加bean后处理器
        context.registerBean(AutowiredAnnotationBeanPostProcessor.class);  //解析@Autowired @Value注解的后处理器


        //初始化容器
        context.refresh();   //执行beanFactory后处理器，添加bean后处理器，初始化所有单例

        context.close();

    }
}
输出：在没有添加那个后处理器时，只会输出spring内部自己的日志信息
15:53:03.311 [main] INFO com.java.another.Bean1 - @Autowired 生效: com.java.another.Bean2@a1cdc6d
15:53:03.320 [main] DEBUG org.springframework.core.env.PropertySourcesPropertyResolver - Found key 'JAVA_HOME' in PropertySource 'systemEnvironment' with value of type String
15:53:03.334 [main] INFO com.java.another.Bean1 - @Value 生效: C:\Program Files\Java\jdk1.8.0_74
```

**CommonAnnotationBeanPostProcessor**

用于解析@Resource、@PostConstruct、@PreDestroy注解

```java
//添加bean后处理器
context.registerBean(AutowiredAnnotationBeanPostProcessor.class);  //解析@Autowired @Value注解的后处理器

context.registerBean(CommonAnnotationBeanPostProcessor.class);

输出：
15:58:02.089 [main] INFO com.java.another.Bean1 - @Resource 生效: com.java.another.Bean3@2118cddf   
15:58:02.106 [main] DEBUG org.springframework.core.env.PropertySourcesPropertyResolver - Found key 'JAVA_HOME' in PropertySource 'systemEnvironment' with value of type String
15:58:02.116 [main] INFO com.java.another.Bean1 - @Value 生效: C:\Program Files\Java\jdk1.8.0_74
15:58:02.116 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'bean2'
15:58:02.117 [main] INFO com.java.another.Bean1 - @Autowired 生效: com.java.another.Bean2@68c72235
15:58:02.117 [main] INFO com.java.another.Bean1 - @PostConstruct 生效
15:58:02.129 [main] DEBUG org.springframework.context.support.GenericApplicationContext - Closing org.springframework.context.support.GenericApplicationContext@5204062d, started on Mon Jul 25 15:58:01 CST 2022
15:58:02.130 [main] INFO com.java.another.Bean1 - @PreDestroy 生效
```

**ConfigurationPropertiesBindingPostProcessor**

Springboot的bean后处理器，用于将类和配置文件进行绑定，主要的作用就是解析@ConfigurationProperties注解

```java
//添加bean后处理器
context.registerBean(AutowiredAnnotationBeanPostProcessor.class);  //解析@Autowired @Value注解的后处理器

context.registerBean(CommonAnnotationBeanPostProcessor.class);   //解析@Resource、@PostConstruct、@PreDestroy注解

//Springboot的bean后处理器，用于将类与配置文件绑定@ConfigurationProperties注解
ConfigurationPropertiesBindingPostProcessor.register(context.getDefaultListableBeanFactory());

System.out.println(context.getBean("bean4"));
输出：
16:40:21.376 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'bean4'
Bean4{home='C:\Program Files\AdoptOpenJDK\jdk-11.0.10.9-hotspot', version='11.0.10'}
```

## 附录5 常见的Bean工厂后处理器

BeanFactory的作用：为BeanFactory提供扩展

**ConfigurationClassPostProcessor**

这个Bean工厂后处理器可以解析`@ComponentScan`，`@Bean`，`@Import`等

以解析@ComponentScan注解为例，简化一下，大概的实现是这样的：

+ 第一步：查找指定类上是否加上了@ComponentScan注解
+ 第二步：如果加上了，则获取注解属性中扫描的所有包路径，循环遍历
+ 第三步：将包路径转换为Spring能够识别的文件路径，并判断这些包下的类是否加上了@Component注解或者@Component注解的派生注解（类似于@Service, @Controller, @Mapper这样的）
+ 第四步：如果加上了上述注解，则创建相关类的BeanDefinition，并通过生成Bean名称的工具类生成Bean的名称
+ 第五步：将这些Bean加入到BeanFactory的容器中

```java
//查找目标类上是否有指定注解
        ComponentScan componentScan = AnnotationUtils.findAnnotation(Config.class, ComponentScan.class);
        if(componentScan != null) {
            //获取注解的basePackage属性，因为可以设置多个包路径，所以是个数组
            for (String s : componentScan.basePackages()) {
                System.out.println(s);
                //将包路径转换为文件路径
                //比如com.java.another.demo6.bag -> classpath*:com/java/another/demo6/bag/**/*.class
                String path = "classpath*:" + s.replace(".", "/") + "/**/*.class";
                System.out.println(path);
                //获取类的源信息
                CachingMetadataReaderFactory factory = new CachingMetadataReaderFactory();
                Resource[] resources = context.getResources(path);
                //根据注解生成bean的名字
                AnnotationBeanNameGenerator generator = new AnnotationBeanNameGenerator();
                for (Resource resource : resources) {
//                    System.out.println(resource);
                    //获取类的源信息
                    MetadataReader reader = factory.getMetadataReader(resource);
                    //获取扫描的类的类名
                    System.out.println("类名: " + reader.getClassMetadata().getClassName());
                    boolean hasComponent = reader.getAnnotationMetadata().hasAnnotation(Component.class.getName());
                    boolean hasComponentSon = reader.getAnnotationMetadata().hasMetaAnnotation(Component.class.getName());
                    //判断类上是否加了xxx注解
                    System.out.println("是否加上@Component注解" + hasComponent);
                    System.out.println("是否加上@Component的派生注解" + hasComponentSon);

                    if(hasComponent || hasComponentSon) {
                        //如果加了@Component注解以及其派生注解，则创建Bean定义
                        AbstractBeanDefinition bd = BeanDefinitionBuilder
                                .genericBeanDefinition(reader.getClassMetadata().getClassName())
                                .getBeanDefinition();
                        //将bean加入到容器中
                        DefaultListableBeanFactory beanFactory = context.getDefaultListableBeanFactory();
                        String name = generator.generateBeanName(bd, beanFactory);
                        beanFactory.registerBeanDefinition(name, bd);

                    }
                }

            }
        }
```

## 附录6 Spring中的Scope

即Bean的作用域，常见的有5中：

* singleton，容器启动时创建（未设置延迟），容器关闭时销毁
* prototype，每次使用时创建，不会自动销毁，需要调用 DefaultListableBeanFactory.destroyBean(bean) 销毁
* request，每次请求用到此 bean 时创建，请求结束时销毁
* session，每个会话用到此 bean 时创建，会话结束时销毁
* application，web 容器用到此 bean 时创建，容器停止时销毁

**Scope为Singleton注入其他scope bean失效的问题**

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext context =
            new AnnotationConfigApplicationContext("com.java.another.demo2");

    E e = context.getBean(E.class);
    log.info("{}", e.getF1());
    log.info("{}", e.getF1());
    log.info("{}", e.getF1());
    log.info("{}", e.getF1());
    context.close();

}

@Component  //不加@Scope注解默认就是单例的
public class E {

    @Autowired
    private F1 f1;

    public F1 getF1() {
        return f1;
    }
}

@Scope("prototype")
@Component
public class F1 {}

输出：
17:01:45.912 [main] INFO com.java.another.demo2.ScopeTest - com.java.another.demo2.F1@2c35e847
17:01:45.914 [main] INFO com.java.another.demo2.ScopeTest - com.java.another.demo2.F1@2c35e847
17:01:45.915 [main] INFO com.java.another.demo2.ScopeTest - com.java.another.demo2.F1@2c35e847
17:01:45.915 [main] INFO com.java.another.demo2.ScopeTest - com.java.another.demo2.F1@2c35e847
```

原因：对单例对象来说依赖注入仅仅发生了一次，E使用的是第一次依赖注入的F1，所以得到的F1对象都是相同的

解决方法：在依赖注入的注解上加上@Lazy注解，@Lazy注解在依赖注入的时候使用的是代理对象，虽然E中的代理F1是同一个，但每次使用代理对象的任意方法时，代理对象都会新建一个F1对象

```java
@Component
public class E {
    @Lazy   //解决scope失效的问题
    @Autowired
    private F1 f1;

    public F1 getF1() {
        return f1;
    }
}

输出：第一行是注入到E中的F1的代理对象
17:10:38.602 [main] INFO com.java.another.demo2.ScopeTest - class com.java.another.demo2.F1$$EnhancerBySpringCGLIB$$cfa5578d
17:10:38.603 [main] INFO com.java.another.demo2.ScopeTest - com.java.another.demo2.F1@6cc4cdb9
17:10:38.627 [main] INFO com.java.another.demo2.ScopeTest - com.java.another.demo2.F1@7f2cfe3f
17:10:38.627 [main] INFO com.java.another.demo2.ScopeTest - com.java.another.demo2.F1@5038d0b5
17:10:38.628 [main] INFO com.java.another.demo2.ScopeTest - com.java.another.demo2.F1@2ad48653
```

还有一种解决办法就是在@Scope注解中添加一个属性proxyMode，代理模式，原理还是一样的，仍然是创建代理对象

```java
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class F1 {
}
```

上面两种方法总体来说给目标对象注入代理对象，再由代理对象去创建一个个多例对象，下面两种方法不需要采用代理的方式，直接由工厂去生成：

```java
@Component
public class E {

    @Lazy   //解决scope失效的问题
    @Autowired
    private F1 f1;

    @Autowired
    private ObjectFactory<F2> f2;    //生成F2多例对象的工厂
    
    @Autowired
    private ApplicationContext context;  //也可以直接注入spring容器

    public F1 getF1() {
        return f1;
    }

    public F2 getF2() {
        return f2.getObject();
    }
}
```

## 附录7 Spring中的AOP

### 原生AspectJ

```java
@Service
@Slf4j
public class MyService {

    public void foo() {
        log.info("待增强的方法 -- MyService.foo()");
    }

}
@Aspect  //原生AspectJ的注解
@Slf4j
public class MyAspect {

    @Before("execution(* com.java.spring.demo.service.MyService.foo())")
    public void before() {
        log.info("before()...");
    }

}
```

原生的AspectJ并不是由代理对象去增强MyService中的foo方法的，而是直接由AspectJ编译器去改写了MyService类，在foo方法执行之前增加了一个方法调用，但这种方法使用的比较少，但使用这样的好处就是，能够突破一些代理的限制，比如：代理通常是通过重写方法实现的，但子类不能重写父类的静态方法，静态方法就不能通过代理对象去做增强，但AspectJ编译器可以，因为它是直接对方法进行改写，而不是重写

### jdk动态代理

jdk动态代理的使用：

```java
public class ProxyTest {
    public static void main(String[] args) {
        //演示jdk代理，只能针对接口代理
        ClassLoader classLoader = ProxyTest.class.getClassLoader();   //类加载器，用于加载在运行期间动态生成的字节码
        Class[] classes = {Foo.class};   //实现接口的数组
        //间接调用被代理类中的方法
        Target target = new Target();
        InvocationHandler handler = (proxy, method, args1) -> {
            log.info("before()...");
            Object invoke = method.invoke(target, args1);
            log.info("after()...");
            return invoke;
        };

        Foo proxy = (Foo) Proxy.newProxyInstance(classLoader, classes, handler);
        proxy.foo();
    }

    interface Foo {
        void foo();
    }
    //被代理对象
    static class Target implements Foo {

        @Override
        public void foo() {
            log.info("目标方法 -- foo()");
        }
    }
}
```

jdk动态代理的特点：

+ jdk的代理对象与目标对象之间是兄弟关系（指都实现了同一个接口）
+ 代理对象与目标对象之间不能进行强制类型转换
+ 目标类可以使用final修饰，因为代理对象和目标对象之间不是继承关系

### cglib代理

cglib动态代理的使用：

```java
public class CglibProxyTest {

    public static void main(String[] args) {
        //cglib动态代理，通过类的继承关系进行代理
        Target target = new Target();

        Class<Target> father = Target.class;   //代理对象的父类型
        MethodInterceptor interceptor = new MethodInterceptor() {
            /**
             *
             * @param o  代理类自己
             * @param method  代理类中增强的方法
             * @param objects 方法执行的参数
             * @param methodProxy  方法对象
             * @return 方法执行结果
             * @throws Throwable
             */
            @Override
            public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                log.info("before()...");
                Object result = method.invoke(target, objects);
                //Object result =  methodProxy.invoke(target, objects);   //spring用的就是这种
                log.info("after()...");

                return result;
            }
        };
        Target proxy = (Target) Enhancer.create(father, interceptor);
        proxy.foo();
    }

    static class Target {
        public void foo() {
            log.info("目标方法 --- foo()");
        }
    }
}
```

cglib动态代理的特点：

+ 代理类和目标类是父子关系
+ 目标类就不能加final关键字，因为final类不能创建子类
+ 目标类中的方法也不能用final修饰，因为代理类是通过方法重写的方式增强方法的
+ 在MethodInterceptor中（作用和InvocationHandler类似）提供了一个MethodProxy，可以不通过反射去调用目标方法

### Spring中jdk动态代理的原理

**最简单的代理**

首先模仿spring，先手写一个类似于spring中的动态代理的代码：

```java
public class JDKProxyTheoryTest {
    //测试接口
    interface Foo {
        void foo();
    }
    //将不确定的增强功能抽取出来单独写成一个接口
    interface InvocationHandler {
        void invoke();
    }
	//目标类和目标方法
    static class TargetClass implements Foo {

        @Override
        public void foo() {
            System.out.println("target class function foo()");
        }
    }

    public static void main(String[] args) {
        Foo proxy = new $Proxy0(() -> {
            //1. 功能增强
            System.out.println("添加的功能");
            //2. 调用目标方法
            new TargetClass().foo();
        });

        proxy.foo();
    }
}

输出：
添加的功能
target class function foo()
```

以及目标类的代理类：

```java
public class $Proxy0 implements JDKProxyTheoryTest.Foo {
	//传入需要增强的功能
    private JDKProxyTheoryTest.InvocationHandler handler;

    public $Proxy0(JDKProxyTheoryTest.InvocationHandler handler) {
        this.handler = handler;
    }

    //调用增强之后的方法即可
    @Override
    public void foo() {
        handler.invoke();
    }
}
```

**简单代理存在的问题**

由于上面的代码中的接口只有一个方法，那么增强方法在调用目标方法的时候可以直接调用到目标方法，但是实际上接口一般会有多个方法，那么当代理对象在调用方法的时候，就不能确定被调用的方法了

解决方法就是可以使用反射中的方法对象，在代理对象中，可以将要调用的方法作为参数传递给增强方法

```java
public class JDKProxyTheoryTest {
    //测试接口
    interface Foo {
        void foo() throws NoSuchMethodException;
        void bar() throws NoSuchMethodException;
    }
    //将不确定的增强功能抽取出来单独写成一个接口
    interface InvocationHandler {
        //method代表被增强的方法，args为方法所需要的参数
        void invoke(Method method, Object[] args);
    }

    static class TargetClass implements Foo {

        @Override
        public void foo() {
            System.out.println("target class function foo()");
        }

        @Override
        public void bar() {
            System.out.println("target class function bar()");
        }
    }

    public static void main(String[] args) {
        Foo proxy = new $Proxy0((method, params) -> {
            //1. 功能增强
            System.out.println("添加的功能");
            //2. 调用目标方法
            try {
                //在调用时，通过反射中的方法对象进行调用
                method.invoke(new TargetClass(), params);
            } catch (Exception e) {
                e.printStackTrace();
            }
        });

        try {
            proxy.foo();
            proxy.bar();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }
    }
}

输出：
添加的功能
target class function foo()
添加的功能
target class function bar()
```

修改后的代理类：

```java
public class $Proxy0 implements JDKProxyTheoryTest.Foo {

    private JDKProxyTheoryTest.InvocationHandler handler;

    public $Proxy0(JDKProxyTheoryTest.InvocationHandler handler) {
        this.handler = handler;
    }

    @Override
    public void foo() throws NoSuchMethodException {
        //获取被增强方法的方法对象
        Method foo = JDKProxyTheoryTest.Foo.class.getDeclaredMethod("foo");
        handler.invoke(foo, new Object[0]);
    }

    @Override
    public void bar() throws NoSuchMethodException {
        Method bar = JDKProxyTheoryTest.Foo.class.getDeclaredMethod("bar");
        handler.invoke(bar, new Object[0]);
    }
}
```

**进一步改进**

上述的程序中，接口中方法都没有返回值，实际情况中可能有些方法是有返回值的，所以需要修改InvocationHandler中的invoke方法，使其返回一个结果，为了通用，返回Object，spring中设计这个InvocationHandler时，还会将代理对象传入（一般用不太到）

```java
//将不确定的增强功能抽取出来单独写成一个接口
interface InvocationHandler {
    //method代表被增强的方法，args为方法所需要的参数
    Object invoke(Object proxy, Method method, Object[] args);
}
```

实际上，这个InvocationHandler的设计已经和Spring中的很像了（Spring中的InvocationHandler是java.lang.reflect包下的），而Spring中在设计代理类的时候，还会让代理类去继承一个Proxy的类（java.lang.reflect），这个类中内含了一个InvocationHandler，所以实际上的代理对象可以这么写：

```java
public class $Proxy0 implements JDKProxyTheoryTest.Foo {

    //可以在静态代码块中统一获取一次方法对象
    static Method foo;
    static Method bar;
    static Method add;
    
    static {
        try {
            
        } catch (NoSuchMethodException e) {
            //检查型异常需要变成运行时异常抛出
            throw new NoSuchMethodError(e.getMessage());
        }
    }
    
    public $Proxy0(JInvocationHandler handler) {
        super(handler);
    }

    @Override
    public void foo() throws NoSuchMethodException {
        try {
            handler.invoke(this, foo, new Object[0]);
        } catch (RuntimeException | Error e) {
            throw e;
        } catch (Throwable e) {
            throw new UndeclareThrowableException(e);
        }
    }

    @Override
    public void bar() throws NoSuchMethodException {
        try {
            handler.invoke(this, bar, new Object[0]);
        } catch (RuntimeException | Error e) {
            throw e;
        } catch (Throwable e) {
            throw new UndeclareThrowableException(e);
        }
        
    }

    @Override
    public int add(){
        try {
            Object result = handler.invoke(this, add, new Object[0]);
        	return (int) result;
        } catch (RuntimeException | Error e) {
            throw e;
        } catch (Throwable e) {
            throw new UndeclareThrowableException(e);
        }
    }
}
```

### Spring中的cglib动态代理

结合JDK动态代理的实现，cglib动态代理类似于：

```java
public class Target {  //目标类
    public void save() {
        System.out.println("target class save()");
    }

    public void save(int i) {
        System.out.println("target class save(int) : " + i);
    }

    public void save(long l) {
        System.out.println("target class save(long) : " + l);
    }
}

//代理类，继承目标类
public class Proxy extends Target{
	//cglib代理中在MethodInterceptor中实现增强功能
    private MethodInterceptor methodInterceptor;

    static Method save0;
    static Method save1;
    static Method save2;
    static {
        try {
            save0 = Target.class.getDeclaredMethod("save");
            save1 = Target.class.getDeclaredMethod("save", int.class);
            save2 = Target.class.getDeclaredMethod("save", long.class);
        } catch (NoSuchMethodException e) {
            throw new NoSuchMethodError(e.getMessage());
        }
    }

    public Proxy(MethodInterceptor methodInterceptor) {
        this.methodInterceptor = methodInterceptor;
    }

    ///////////////////////////带增强功能的方法
    @Override
    public void save() {
        try {
            methodInterceptor.intercept(this, save0, new Object[0], null);
        } catch (Throwable e) {
            throw new UndeclaredThrowableException(e);
        }
    }

    @Override
    public void save(int i) {
        try {
            methodInterceptor.intercept(this, save1, new Object[]{i}, null);
        } catch (Throwable e) {
            throw new UndeclaredThrowableException(e);
        }
    }

    @Override
    public void save(long l) {
        try {
            methodInterceptor.intercept(this, save2, new Object[]{l}, null);
        } catch (Throwable e) {
            throw new UndeclaredThrowableException(e);
        }
    }
}
```



## 附录8 Spring IOC & AOP总结

### IOC

`IOC`就是控制反转（`inverse of control`），是一种设计思想，将原本在程序中手动创建对象的控制权交给spring框架来管理

`IOC`有两层意思：

+ 控制：指的是对象创建（实例化，管理）的权力
+ 反转：将对象的控制权交给外部（Spring框架）管理

具体来说是将对象间的相互依赖关系交给IOC容器来管理，并由IOC容器来完成对象的注入工作，这样做的目的就是简化开发，减少各个模块之间的耦合

```java
public class PeopleService {
    //如果没有使用IOC容器的话，程序中就需要自己new对象，如果后续要进行功能的扩展，需要在这里修改实例化的对象
    private PeopleMapper peopleMapper = new PeopleMapperImpl();  //new PeopleMapperImpl1()  new PeopleMapperImpl2()  ...
}

public interface PeopleMapper {}

public class PeopleMapperImpl implements PeopleMapper {}

////////////////////////////////////////////////////////////////////////////////
//如果有了IOC容器的话，就不需要在程序中显示地new对象
public class PeopleService {
    @Autowired   //只需要加上注解标注，spring会从ioc容器中根据参数的类型或者名称从容器中获取对应的对象注入
    private PeopleMapper peopleMapper;
}
```

所以总得来说，`IOC`解决了两个问题：

+ 对象之间的耦合度、依赖程度降低
+ 资源管理变得很容易

从控制反转会引出另一个名词：依赖注入（`dependency injection`），是spring中IOC容器的一种实现方式

### AOP

`AOP`就是面向切面编程（`Aspect oriented programming `），是一种编程思想，能够将那些与业务无关，但是多个业务之间需要共同调用的代码（例如事务管理，日志管理，权限管理等等）封装起来，便于减少系统的重复代码，降低模块之间的耦合度

通过`AOP`的描述，能够发现它解决了两个问题：

+ 代码重复问题
+ 业务逻辑与其他功能代码混合在一起，代码臃肿的问题

简单地说`AOP`就是：在不改变原有业务逻辑的情况下，向业务代码中注入其他功能

Spring中实现`AOP`有几种方案：

+ AspectJ，Spring AOP集成了AspectJ，可以使用其中的AOP代理的功能
+ Spring JDK动态代理，处理接口的代理
+ Spring Cglib动态代理，处理类与类之间的代理

> Spring常用的是后面两种，与AspectJ的区别就是：
>
> + jdk和cglib动态代理都是运行时增强，用到了反射
> + aspectJ代理是编译时增强，直接修改了文件的字节码



## 附录9 Spring / Springboot的常用注解

### @SpringBootApplication

项目创建之后，这个注解会默认加在启动类上，这是整个Springboot项目的基石

```java
@SpringBootApplication
public class StudentManageApplication {
    public static void main(String[] args) {
        SpringApplication.run(StudentManageApplication.class, args);
    }
}
```

`@SpringBootApplication` = `@ComponentScan` + `@SpringBootConfiguration` + `@EnableAutoConfiguration`

+ `@ComponentScan`：包扫描注解，指定扫描规则
+ `@SpringBootConfiguration`：其实就是一个`@Configuration`注解，表示这个启动类是一个配置类
+ `@EnableAutoConfiguration`：这个注解又是由两个注解组成，分别是：
  + `@AutoConfigurationPackage`：实际上是一个`@Import(AutoConfigurationPackage.Register.class)`注解，指定了默认的包扫描路径，也就是启动类所在的包
  + `@Import(AutoConfigurationImportSelector.class)`：向项目中导入一系列的开发场景，也就是相应的配置类`xxxAutoConfiguration`，总共有127个左右，根据版本的不同有所变动，这些配置类再配合一些条件注解`@Conditionxxx`，使一些场景生效

这个注解其实也解释了Springboot的自动装配原理

### Spring bean相关的注解

+ `@Autowired`：管理依赖注入的注解，可以加在属性上，默认按照类型注入，这是Spring框架提供的一个注解
+ `@Component`、`@Repository`、`@Service`、`@Controller`：将某个类标识为Spring组件，生成一个JavaBean放到IOC容器中，由Spring进行管理，不同的注解标识不同的应用层，其中`@Component`是通用的
+ `@Scope`：标识Bean的作用域，常标注在方法上（配置类中向容器中加入Bean的方法）
+ `@Configuration`：声明配置类

## 附录10 Spring事务相关总结

### Spring对事务的支持

Spring框架对事务的支持取决于使用的数据库是否支持事务，比如MySQL数据库，如果使用的是innodb存储引擎，那么就是支持事务的；如果使用的是myisam存储引擎，那么就不支持事务了

Spring支持两种方式的事务管理：

+ 编程式事务：通过`TransactionTemplate`和`TransactionManager`手动管理事务，实际应用中很少使用，比如使用`TransactionManager`管理事务

  ```java
  @Autowired
  private PlatformTransactionManager transactionManager;
  
  public void testTransaction() {
  
    TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
            try {
                 // ....  业务代码
                transactionManager.commit(status);
            } catch (Exception e) {
                transactionManager.rollback(status);
            }
  }
  ```

+ 声明式事务：通过AOP的功能实现，比如`@Transaction`注解，代码的侵入性较小

  ```java
  public class PeopleServiceImpl implements PeopleService {
      @Transactional(propagation=propagation.PROPAGATION_REQUIRED)
      public void method() {
          //业务代码
      }
  }
  ```

### Spring事务管理器

Spring并不直接管理事务，而是提供了多种事务管理器：

+ `PlatformTransactionManager`（平台）事务管理接口，Spring事务管理的核心，通过这个接口，Spring为各个平台，比如JDBC，JPA等都提供了对应的事务管理器
+ `TransactionDefinition`事务定义信息（事务隔离级别、传播行为、超时回滚等），这个类定义了一些事务的基本属性：
  + 隔离级别
  + 传播行为
  + 回滚规则
  + 是否只读
  + 事务超时
+ `TransactionStatus`事务状态，用来记录事务的状态

### Spring事务传播行为

事务传播行为是为了解决业务层**方法之间**互相调用的事务问题，当某个事务方法被另一个事务方法调用时，必须指定事务应该如何传播，比如：

```java
public class A {
    @Autowired
    private B b;
    
    @Transaction
    public void aMethod() {
        //业务代码...
        b.bMethod();
        //业务代码...
    }
}

public class B {
    @Transaction
    public void bMethod() {
        
    }
}
```

在上面的例子中，A类的`aMethod()`方法调用了B类中的`bMethod()`方法，这个时候就涉及到了两个事务方法之间调用，如果`bMethod()`方法出现异常回滚，如果配置事务传播行为让`aMethod()`跟着回滚，就是事务传播行为要解决的问题

在`TransactionDefinition`定义中包括了如下几个表示传播行为的常量：

```java
public interface TransactionDefinition {
    int PROPAGATION_REQUIRED = 0;
    int PROPAGATION_SUPPORTS = 1;
    int PROPAGATION_MANDATORY = 2;
    int PROPAGATION_REQUIRES_NEW = 3;
    int PROPAGATION_NOT_SUPPORTED = 4;
    int PROPAGATION_NEVER = 5;
    int PROPAGATION_NESTED = 6;
    ......
}
```

Spring也有一个枚举类去定义这些常量，就是`Propagation`，常用的事务传播行为有这两种：

+ `TransactionDefinition.PROPAGATION_REQUIRED`：这是使用最多的一种事务传播行为，平常使用的`@Transaction`注解默认的就是这个传播行为，按照官方的说法，意思是如果当前方法存在事务，则加入该事务；如果当前方法没有事务，则新建一个事务，也就是说：
  + 如果外部方法没有开启事务的话，`Propagation.REQUIRED`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。
  + 如果外部方法开启事务并且是`Propagation.REQUIRED`的话，所有`Propagation.REQUIRED`修饰的内部方法和外部方法均属于同一事务 ，只要一个方法回滚，整个事务均回滚。
  + 以上面的例子说明，如果`aMethod()`和`bMethod()`使用的都是`PROPAGATION_REQUIRED`传播行为的话，两者使用的就是同一个事务，只要其中一个方法回滚，整个事务均回滚。
+ `TransactionDefinition.PROPAGATION_REQUIRES_NEW`：不管怎么样，新建一个事务，如果当前方法存在事务，则把事务挂起，也就是说不管外部方法是否开启事务，`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰
  + 以上面的例子说明，如果`bMethod()`使用`Propagation.REQUIRES_NEW`修饰的话，会开启一个独立的事务，也就是说，如果`aMethod()`出现异常回滚了，`bMethod()`不会跟着回滚，因为`bMethod()`开启了一个新事务，并且与`aMethod()`的事务相互独立

> 提到事务就需要关注事务的隔离级别，Spring给事务规定了五个隔离级别，定义在`TransactionDefinition`中，分别是（其实就比SQL的隔离级别多了一个默认的隔离级别）：
>
> - **`TransactionDefinition.ISOLATION_DEFAULT`** :使用后端数据库默认的隔离级别，MySQL 默认采用的 `REPEATABLE_READ` 隔离级别 Oracle 默认采用的 `READ_COMMITTED` 隔离级别.
> - **`TransactionDefinition.ISOLATION_READ_UNCOMMITTED`** :最低的隔离级别，使用这个隔离级别很少，因为它允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
> - **`TransactionDefinition.ISOLATION_READ_COMMITTED`** : 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
> - **`TransactionDefinition.ISOLATION_REPEATABLE_READ`** : 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
> - **`TransactionDefinition.ISOLATION_SERIALIZABLE`** : 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

### @Transaction注解

`@Transaction`注解可以标注在方法（必须在public方法上才会生效）、类、接口上

`@Transaction`注解的工作机制由AOP实现，也就是动态代理，如果目标对象实现了接口，则会采用jdk动态代理；如果目标对象没有实现接口，则会使用cglib动态代理

存在的问题：若同一类中的其他没有 `@Transactional` 注解的方法内部调用有 `@Transactional` 注解的方法，有`@Transactional` 注解的方法的事务会失效。这是由AOP自己造成的，在调用事务方法时，由于使用了动态代理，实际上调用的是`TransactionInterceptor` 类中的 `invoke()`方法，如果本类中直接调用事务方法就是`this.xxxMethod()`，并不会使事务生效



