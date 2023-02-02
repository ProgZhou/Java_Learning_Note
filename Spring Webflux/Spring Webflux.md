# Spring Webflux

## 一、基本介绍

**Spring Webflux**是Spring5添加的新模块，用于web开发，功能与SpringMVC类似。Webflux使用当前一种比较流行的响应式编程

传统的web框架，比如SpringMVC是基于Servlet容器；而Webflux是一种异步非阻塞的框架，异步非阻塞的框架在Servlet3.1之后得以支持，核心是基于Reactor的相关API实现

Webflux的特点：

+ 非阻塞，在有限的资源下（内存），提供更高的吞吐量，在同一时间内能处理更多的请求
+ 函数式编程，基于Java8函数式编程的方式实现路由请求

SpringMVC与Webflux的对比：

<img src="Spring Webflux.assets/image-20230201090334896.png" alt="image-20230201090334896" style="zoom:80%;" />

两种框架都可以使用注解的方式编程，都可以运行在Tomcat等容器中

SpringMVC采用命令式编程（方便调试）

Webflux采用函数式响应编程

## 二、响应式编程

响应式编程是一种面向**数据流**和**变化传播**的编程范式

最典型的例子就是excel表格

| **A1** | **B1** | **C1**          |
| ------ | ------ | --------------- |
| 1      | 2      | `sum(A1, B1)` 3 |

C1单元格的值会随着A1、B1值变化而变化，比如A1的值变为6，则C1的值会变为8

**Java8中的响应式编程**

根据响应式编程的特点，与其相似的编程模式是观察者模式，而在Java8以及之前的版本中，观察者模式的实现由Observer和Observable两个类

```java
//Observer接口
public interface Observer {
    //每当观察到的对象发生变化时，都会调用此方法
    void update(Observable o, Object arg);
}

//Observable类，此类表示可观察对象或模型视图中的数据
public class Observable {
    //当此类的数据发生变化，则通知所有观察者
    public void notifyObservers(){
        
    }
}
```

