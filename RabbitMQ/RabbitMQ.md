# RabbitMQ

## 一、MQ的基本概念

### 1.1 什么是MQ

MQ（message queue），从字面意思上看，本质是一个队列，FIFO先进先出，只不过队列中存放的内容是message，是一种进程间通信或者一个进程中不同线程的通信方式

在互联网架构中，MQ是一种非常常见的上下游“逻辑解耦 + 物理解耦”的消息通信服务，主要解决应用耦合，异步消息，流量削锋等问题

**基本概念**

大多数应用中，可以通过消息服务中间件来提升系统异步通信、扩展解耦的能力

消息服务中两个重要概念：

+ 消息代理：就是安装了消息中间件的服务器，生产者要发消息给装有消息中间件的服务器，如果想要获取消息，也需要连接上这台服务器来获取消息
+ 目的地：消息的生产者想要把消息发到哪里去

当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目的地

消息队列主要有两种形式的目的地：

+ 队列：点对点消息通信
+ 主题：发布 / 订阅消息通信

> **点对点通信：**
>
> + 消息发送者发送消息之后，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，消息读取后被移出队列
> + 消息只有唯一的发送者和接收者，但可以有多个服务监听消息队列，谁先强到这个消息，谁就去处理这个消息
>
> **发布订阅式**
>
> + 发送者发送消息到主题，多个接收者监听这个主题
> + 所有接收者都会收到这个消息
>
> 所有的消息中间件都需要具备这两种模式

**消息队列协议**

JMS（Java Message Service）Java消息服务：基于JVM消息代理的规范，ActiveMQ就是基于这个实现的

AMQP（Advanced Message Queuing Protocol）高级消息队列协议，兼容JMS，RabbitMQ是AMQP的实现

两者的对比：

<img src="RabbitMQ.assets/image-20220630155418422.png" alt="image-20220630155418422" style="zoom:80%;" />



### 1.2 MQ的作用

#### 1.2.1 流量削峰

假设现在有一个订单系统，1s内能够处理10000次请求，这个处理能力在正常情况下完全没有问题，但在流量高峰期，如果1s内有20000个请求进来，订单系统是处理不了的，只能使用服务降级和服务熔断，限制用户下单

使用消息队列，将这些大量的请求全部发送到消息队列中，然后系统立即给用用户响应，之后后台系统再去从消息队列中获取这些请求，慢慢处理

原始情况：当请求在系统的处理能力范围之内，那么系统就可以正常处理请求；一旦请求次数超过系统能够处理的范围，就会使得系统宕机

```mermaid
graph LR;
用户 -->|请求|订单系统
```

加入MQ：MQ可以接收用户的请求并缓存起来，采用先进先出的方式将缓存的请求发送至订单系统

```mermaid
graph LR;
用户 --> |请求|MQ(MQ接受请求)
MQ --> 订单系统
```



#### 1.2.2 应用解耦

以电商应用为例，应用中有支付系统，库存系统，物流系统等，用户创建订单之后，请求被订单系统处理，而订单系统又需要调用支付系统，库存系统，物流系统等

```mermaid
graph LR;
订单系统 --> 支付系统
订单系统 --> 库存系统
订单系统 --> 物流系统
订单系统 --> ......
```

如果由订单系统直接调用这些子系统，那么如果任何一个子系统处理出现异常，都会影响订单系统的执行

现在使用消息队列来转发请求

```mermaid
graph LR;
订单系统 --> MQ(MQ)
MQ --> 支付系统
MQ --> 库存系统
MQ --> 物流系统
MQ --> ......
```

这样系统间调用的问题就会减少，当支付系统出现异常需要进行修复，那么物流系统需要处理的请求会被缓存到消息队列中，用户的下单操作可以正常完成，当物流系统修复之后，从消息队列中取出请求继续进行处理，中途用户感受不到物流系统的故障，提升系统的可用性

#### 1.2.3 异步处理

订单系统需要调用物流系统，但用户又需要快速知道自己是否下单成功，如果使用同步方式进行，订单系统需要在物流系统处理完毕之后再给用户返回信息，使用消息队列之后：

```mermaid
graph LR;
订单系统 --> MQ
物流系统 --> MQ(MQ)
订单系统 --> 物流系统
```

在将请求发送至物流系统之后，订单系统可以监听消息队列中是否有物流系统的完成消息即可

### 1.3 MQ的分类

#### 1.3.1 ActiveMQ

优点：单机吞吐量万级，时效毫秒级，可用性高，基于主从架构实现高可用

现在用的不多，高吞吐量场景较少使用

#### 1.3.2 kafka

为大数据服务的消息中间件

百万级TPS（每秒事务处理量）的吞吐量，性能卓越，可用性非常高

kafka是分布式的，一个数据多个副本，少数机器宕机不会丢失数据

有优秀的第三方Kafka Web界面管理工具，对硬件要求比较高

#### 1.3.3 RocketMQ

阿里巴巴的开源产品，使用Java实现

单机吞吐量十万级，可用性非常高，分布式架构，消息可以做到0丢失，支持10亿级别的消息堆积

## 二、RabbitMQ

### 2.1 RabbitMQ的基本概念

RabbitMQ是一个消息中间件，它的基本功能就是接收并转发消息

以快递站作为类比，当商家需要将快递发送给用户时，先把快递放到快递站中，然后快递员会将快递从快递站中取出，送到收件人手里，RabbitMQ就相当于一个快递站

```mermaid
graph LR;
商家 --> 快递站
快递站 --> |快递员|用户
```

RabbitMQ并不处理消息，只是接收，存储和转发消息数据



#### RabbitMQ的工作流程

（1）生产者与消费者首先与消息队列的消息代理建立连接，并开辟多个信道

（2）生产者发送消息给消息代理中指定虚拟主机的交换机，消息由消息头和消息体组成

（3）交换机接收到消息之后根据消息的路由键决定将消息发送给哪个消息队列

（4）消费者通过信道监听各个消息队列，随时从消息队列中获取自己想要的消息

<img src="RabbitMQ.assets/image-20220715204931865.png" alt="image-20220715204931865" style="zoom:80%;" />

**几个基本概念**

+ Connection：网络连接，比如TCP连接
+ Binding：绑定关系，用于消息队列和交换机之间的关联，一个绑定就是基于路由键将交换机和消息队列连接起来的路由规则，通俗来理解可以将交换机看作一张路由表，交换机和队列之间的绑定可以是多对多关系
+ 信道：channel，多路复用连接中的一条独立的双向数据流通道，信道是建立在真实的TCP连接内的虚拟连接，AMQP命令都是通过信道发送出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成的
+ 虚拟主机：表示一批交换机和消息队列，可以简单理解为就是一个mini版的RabbitMQ，拥有自己的队列，交换机，绑定和权限机制，有点类似于操作系统中的虚拟机，一个RabbitMQ可以划分多个虚拟主机，各个主机之间相互隔离，互不影响



### 2.2 四大核心概念

先用一幅图表示这四大核心概念的关系

```mermaid
graph LR;
subgraph MQ
交换机 --> |绑定关系|队列1
交换机 --> |绑定关系|队列2
end
消费者 --> 交换机
队列1 --> 消费者1
队列2 --> 消费者2

```

#### 2.1 生产者

产生数据发送消息的程序就是生产者

**消息 --- Message**

消息由消息头和消息体组成，消息是不透明的

+ 消息头有一系列的可选属性组成，这些属性包括routing key（路由键），priority（相对于其他消息的优先级）等
+ 消息体是消息的具体信息

#### 2.2 交换机

交换机是RabbitMQ中非常重要的一个部件，一方面接收来自生产者的消息，另一方面将消息推送到各个队列中

交换机必须确切知道如何处理它接收到的消息，是将消息推送到特定队列还是推送到多个队列

MQ中，一个交换机对应多个队列

**Binding绑定**

用于消息队列和交换机之间的关联，一个绑定就是基于路由键将交换机和消息队列连接起来的路由规则

可以将交换机理解为由一个绑定构成的路由表

> **交换机的类型**
>
> （1）直接交换机，是指路由键的完全匹配，只有当消息的routing key和绑定关系的键完全一致才能匹配，所以是点对点的，direct交换机只能将消息发送给完全匹配的队列
>
> （2）fanout交换机，广播类型交换机，不关心路由键是什么，直接将消息转发给与它绑定的所有队列中
>
> （3）topic交换机，将队列的绑定关系和某个模式进行匹配，如果路由键符合队列的绑定关系，就将消息发送给路由键所匹配的所有队列中
>
> ![image-20220630171803098](RabbitMQ.assets/image-20220630171803098.png)

#### 2.3 队列

队列是RabbitMQ内部使用的一种数据结构，尽管消息流经RabbitMQ和应用程序，但消息只能存储在队列中

队列本质上是一个很大的消息缓冲区，生产者可以将消息发送到一个队列，消费者可以尝试从一个队列中接收数据

#### 2.4 消费者

消费者在大多数情况下是一个等待接收消息的程序，在实际生产环境上，生产者，消费者和消息中间件很多时候并不在一台机器上，同一个应用程序既可以是生产者同时又可以是消费者

### 2.3 RabbitMQ的模式

RabbitMQ的工作原理

```mermaid
graph LR;
subgraph connection1
channel1
channel2
channel3
end
subgraph connection2
channel_a
channel_b
channel_c
end
subgraph Broker代理
交换机1 --> |绑定| 队列1
交换机1 --> |绑定| 队列2
交换机2 --> |绑定| 队列3
交换机2 --> |绑定| 队列4
end
生产者1 --> channel1
生产者2 --> channel3

channel1 --> 交换机1
channel3 --> 交换机2

队列2 --> channel_a
队列3 --> channel_b

channel_a --> 消费者1
channel_b --> 消费者2


```

+ Connection：TCP连接，生产者发送消息或者消费者接收消息，都必须跟Broker建立连接才能通信
  + 一个客户端只需要建立一条连接
  + 连接为长连接，一直保持的连接
+ Channel：是建立在真实TCP连接中的虚拟连接，消息都是通过信道发送或者接收的，建立信道是为了复用TCP连接
  + 在一条连接中建立多条信道
  + 信道负责数据的传输
+ 虚拟主机：指一批交换机和队列，主要目的是为了系统的隔离

<img src="RabbitMQ.assets/image-20220630165746797.png" alt="image-20220630165746797" style="zoom:80%;" />

#### 2.3.1 简单模式

一个生产者对应一个消费者

```mermaid
graph LR;
生产者((生产者)) --> MQ
MQ --> 消费者((消费者))
```

**简单模式的演示**

新建一个springboot工程，加上rabbitMQ的依赖

```xml
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit-test</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
</dependency>
```

主要编写两个类：Producer生产者和Consumer消费者

生产者向rabbitMQ发送消息

```java
/** 消息生产者
 * @author ProgZhou
 * @createTime 2022/04/14
 */
public class Producer {

    //消息队列的名称
    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) {
        //创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //工厂IP，即需要连接的MQ的IP
        factory.setHost("127.0.0.1");

        //连接用户名
        factory.setUsername("admin");

        //连接密码
        factory.setPassword("123456");

        //获取连接
        try {
            Connection connection = factory.newConnection();
            //获取信道
            Channel channel = connection.createChannel();
            /*
            * 生成一个队列：
            * 1. String queue：队列名称
            * 2. boolean durable：队列中的消息是否持久化，默认情况消息存储在内存中
            * 3. boolean exclusive：队列是否只供一个消费者进行消费，即是否进行消息共享
            *       true代表队列中的消息仅供一个消费者接收
            *       false代表消息可供多个消费者共享
            * 4. boolean autoDelete：是否自动删除，当所有消费者都与这个队列断开连接时，这个队列会自动删除
            * 5. Map<String, Object> argument
            * */
            channel.queueDeclare(QUEUE_NAME, false, false,false,null);

            //发送消息
            String message = "hello world";
            /*
            * 发送消息
            * 1. String exchange：交换机名称
            * 2. String routingKey：路由key
            * 3. BasicProperties props：一些基本的消息配置
            * 4. byte[] body：发送的消息
            * */
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println("message publish successfully");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

消费者从MQ中接收消息

```java
/** 消费者，接收消息
 * @author ProgZhou
 * @createTime 2022/04/14
 */
public class Consumer {

    //消息队列的名称
    public static final String QUEUE_NAME = "hello";

    //接收消息
    public static void main(String[] args) {
        ConnectionFactory factory = new ConnectionFactory();

        factory.setHost("127.0.0.1");

        factory.setUsername("admin");

        factory.setPassword("123456");

        try {
            Connection connection = factory.newConnection();

            Channel channel = connection.createChannel();
            /*
            * 消费者接收消息
            * 1. String queue：队列的名称
            * 2. DeliverCallback deliverCallback：当消息被分配时回调
            * 3. CancelCallback cancelCallback：当消费者取消时，回调
            * */
            //接收消息的接口
            DeliverCallback dc = (consumerTag, message) ->{
                //输出消息
                System.out.println(new String(message.getBody()));
            };
            //当取消消息后的回调
            CancelCallback cc = (consumerTag) ->{
                System.out.println("consumer is interrupted");
            };
            channel.basicConsume(QUEUE_NAME, dc, cc);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

效果：

当生产者向MQ中发送消息后：RabbitMQ会时刻监视消息的状态

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\生产者生产消息.jpg)

当消费者从MQ中接收消息之后

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\消费者接收消息.jpg)



#### 2.3.2 工作模式

在高并发的情况下，如果只采用一个生产者对应一个消费者的模式，会导致消费者不能够及时处理消息队列中的过多的消息导致消息堆积，这时，可以采用多个消费者，使用轮询的方式将消息分发给不同的消费者

```mermaid
graph LR;
生产者((生产者)) --> MQ
MQ --> 消费者1((消费者1))
MQ --> 消费者2((消费者2))
MQ --> 消费者3((消费者3))
MQ --> ...
```

一个消息只能被一个消费者接收，可以采用轮询的方式，第一个消息发送给消费者1，第二个消息发送给消费者2以此类推

工作模式示例：

```java
/** 连接工厂创建信道的工具类
 * @author ProgZhou
 * @createTime 2022/04/14
 */
public abstract class RabbitMqUtil {
    public static final String QUEUE_NAME = "hello";

    public static Channel getChannel() throws Exception{
        ConnectionFactory factory = new ConnectionFactory();

        factory.setHost("127.0.0.1");

        factory.setUsername("admin");

        factory.setPassword("123456");

        Connection connection = factory.newConnection();
        return connection.createChannel();
    }
}
```

Producer.java

```java
/** 消息生产者
 * @author ProgZhou
 * @createTime 2022/04/15
 */
public class Provider {
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();

        //声明一个队列
        channel.queueDeclare(RabbitMqUtil.QUEUE_NAME, false, false, false, null);

        //发送消息，从控制台接收消息并推送至MQ
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String msg = scanner.next();
            channel.basicPublish("", RabbitMqUtil.QUEUE_NAME, null, msg.getBytes());
            System.out.println("provider send message: " + msg);
        }
    }
}
```

Consumer.java，由于需要模拟多个消费者的场景，需要写成多线程的形式

```java
/** 消费者，写两个一样的类充当多个消费者
 * @author ProgZhou
 * @createTime 2022/04/15
 */
public class Consumer01 {
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();

        System.out.println("consumer01 waiting to get message...");

        //消费者接收到消息之后如何处理消息
        DeliverCallback dc = (consumerTag, message) -> {
            String msg = new String(message.getBody());

            System.out.println("consumer01 receive: " + msg);

        channel.basicConsume(RabbitMqUtil.QUEUE_NAME, false, dc, (consumerTag) ->{});
    }
}
```

视图：

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\RabbitMQ工作模式.jpg)

输出结果：

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\生产者发送消息.jpg)

消费者2接收到的消息：

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\工作模式消费者轮询接收消息.jpg)

消费者是轮流接收到消息的，即第一条消息发送给消费者1，第二条消息发送给消费者2...

**不公平分发**

从上面的例子可以发现，消费者1的处理能力很强，消费者2的处理能力很弱，但这两者处理消息的总数是一样的，这就导致了消费者1在很长一部分的时间里是空闲的，浪费了资源

为了解决这样的问题，RabbitMQ支持不公平分发的制度，可以简单的理解为能者多劳，就是处理能力强的消费者可以多接收消息处理，处理能力弱的消费者就少接收消息

```java
//设置消费者的不公平分发，需要注意的是，不公平分发制度只有在手动应答的时候才生效
channel.basicQos(1);
```

效果如下：消费者1接收到的消息会比消费者2多

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\不公平分发1.jpg)

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\不公平分发2.jpg)

**预取值**

不公平分发允许用户设置每个消费者的预取值，意思是规定某个消费者接收多少消息

比如现在MQ中有7条消息，现在规定消费者1接收2条，消费者2接收5条

当队列中的7条消息未处理完时，消费者1只能处理两条消息，尽管已经处理完成，也不能接收新的消息

#### 2.3.3 发布订阅模式

**交换机**：之前示例中，生产者都是直接将消息发送给队列，但实际上，生产者的消息不会直接发送到队列上，而是将消息发送给交换机，交换机接收生产者发送的消息，并将其推送到指定的队列

> 如果生产者有特殊的需求，比如生产者希望自己发送的消息被多个消费者消费，队列是不能完成这样的工作的，因为一个队列中的每个消息只能被消费一次

```mermaid
graph LR;
P((Producer)) --> |msg|exchange
exchange --> |routingKey1|queue1
exchange --> |routingKey2|queue2
```

加入交换机之后，RabbitMQ中各个组件的功能为：

- **生产者**：发送消息
- **交换机**：将收到的消息根据路由规则路由到特定队列
- **队列**：用于存储消息
- **消费者**：收到消息并消费

**交换机与队列绑定**：交换机通过routingKey与队列进行绑定，routingKey可以自己随意指定，当消息发送到交换机，交换机根据消息指定的routingKey将消息发送给指定的队列

**交换机的类型**

+ 直接交换机（direct exchange）
+ 扇形交换机（fanout exchange）
+ 主题交换机（topic exchange）
+ 头交换机（header exchange）

> 还有一种无名交换机，就是RabbitMQ默认的交换机，之前的示例中，channel.basicPublish("", RabbitMqUtil.QUEUE_NAME, null, msg.getBytes());第一个参数为交换机的类型，当为空串的时候，默认分配的就是无名交换机



**扇形交换机**

将接收到的消息广播到它所绑定的所有队列中，也就是只要和交换机绑定的队列都可以收到生产者发送的消息

```java
//fanout交换机生产者
public class EmitLog {
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();

        //声明一个交换机，交换机名字 + 交换机类型
        channel.exchangeDeclare("Logs", "fanout");

        Scanner scanner = new Scanner(System.in);

        while (scanner.hasNext()){
            String string = scanner.next();

            channel.basicPublish("Logs", "", null, string.getBytes(StandardCharsets.UTF_8));

            System.out.println("send message: " + string);
        }
    }

}
```

```java
//fanout交换机消费者
public class ReceiveLogs1 {
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();

        //声明一个交换机
        channel.exchangeDeclare("Logs", "fanout");

        //声明一个队列，临时队列，当消费者断开与队列的连接时，队列就会删除
        String queue1 = channel.queueDeclare().getQueue();

        //绑定交换机，队列名 + 交换机名 + routingKey，routingKey = ""的话消息队列可以自动生成
        channel.queueBind(queue1, "Logs", "");
        System.out.println("ReceiveLogs01 waiting to receive message...");

        DeliverCallback dc = (consumerTag, message) -> {
            String msg = Arrays.toString(message.getBody());
            System.out.println("ReceiveLogs01 message: " + msg);
        };

        CancelCallback cc = consumerTag -> {};
        //消息接收
        channel.basicConsume(queue1, true, dc,cc);

    }
}
```

结果：

生产者发送消息

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\fanout交换机生产者.jpg)

消费者接收消息，可以看到两个消费者都收到了相同的消息

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\fanout交换机消费者1.jpg)

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\fanout消费者2.jpg)

fanout队列与队列的保证

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\fanout交换机绑定.jpg)



**直接交换机**（待修改）

direct交换机会将消息推送到指定的routKey所绑定的队列中

```java
//direct交换机生产者
public class DirectLogs {
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();

        //声明一个交换机，direct类型
        channel.exchangeDeclare("direct_logs", BuiltinExchangeType.DIRECT);

        Scanner scanner = new Scanner(System.in);

        while (scanner.hasNext()){
            String string = scanner.next();
            //使用随机数生成，对应发送给哪个routingKey绑定的交换机
            int i = (int)(Math.random() * 3 + 1);
            if(i == 1){
                System.out.print("routingKey: info ");
                channel.basicPublish("direct_logs", "info", null, string.getBytes(StandardCharsets.UTF_8));
            } else if(i == 2){
                System.out.print("routingKey: warning ");
                channel.basicPublish("direct_logs", "warning", null, string.getBytes(StandardCharsets.UTF_8));
            } else{
                System.out.print("routingKey: error ");
                channel.basicPublish("direct_logs", "error", null, string.getBytes(StandardCharsets.UTF_8));
            }


            System.out.println("send message: " + string);
        }
    }
}
```

```java
//direct交换机消费者
public class ReceiveLogDirect01 {
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();

        //声明一个队列
        channel.queueDeclare("console", false, false, false, null);

        //绑定交换机，一个队列可以使用多个routingKey绑定同一个交换机
        channel.queueBind("console", "direct_logs", "info");
        channel.queueBind("console", "direct_logs", "warning");

        System.out.println("ReceiveLogDirect01 waiting to receive message...");
        //接收消息
        DeliverCallback dc = (consumerTag, message) -> {
            System.out.println("ReceiveLogs01 consume message: " + new String(message.getBody() ));
        };

        CancelCallback cc = consumerTag ->{
            System.out.println("failed to received message: " + consumerTag);
        };

        //接收消息
        channel.basicConsume("console", true, dc, cc);

    }
}

//另一个队列不同的部分：队列名以及绑定规则不同
        channel.queueDeclare("disk", false, false, false, null);

        channel.queueBind("disk", "direct_logs", "error");
```

结果：

生产者发送消息

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\direct交换机生产者发送消息.jpg)

消费者接收消息：

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\direct消费者1接收消息.jpg)

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\direct交换机生产者2接收消息.jpg)



**主题交换机**

直接交换机只能定向对routingKey匹配的队列进行转发消息，如果现在想消息发送到多个队列但不是全部队列，假设现在发送的消息的routingKey为info.base，info.advantage，但某个队列现在只想接收info.base这个消息这时候，direct交换机就办不到了

Topic交换机所绑定队列的routingKey有一定的规则：

+ 必须是一个单词列表，单词与单词之间使用"."隔开

  > 比如info.base，info.advantage，stock.usd.nys等

+ 在匹配中*可以代替一个单词

+ #可以代替零个或多个单词

  > 比如\*.\*.rabbit可以匹配三个单词的routingKey，并且以rabbit结尾



### 2.4 RabbitMQ消息应答机制

消费者在处理消息时需要一定的时间，如果有一个消费者处理一个比较耗时的任务并且只处理了一部分就突然宕机了，会发生什么情况呢？

为了保证消息在发送过程中不丢失，RabbitMQ引入了消息应答机制

消息应答就是：消费者在接收到消息并且处理该消息之后，告诉RabbitMQ消息处理完毕，可以将消息删除

**基本机制**

生产者把消息发布到交换机，消息最终达到队列被消费者接收，而队列的绑定关系和消息的路由键决定交换机要把消息发送到哪个队列种

#### 2.4.1 自动应答方式

RabbitMQ将消息传送给消费者后，默认消息处理成功，会将这个消息从内存中删除，并不在乎消费者是否真的处理完这个消息

```mermaid
graph LR;
Provider --> MQ((MQ))
MQ --> |消息1|Consumer1
MQ --> |消息2|Consumer2
```

> 当消息发送给消费者1之后便立即删除消息，如果消费者1宕机，那么应用就丢失了消息1

#### 2.4.2 手动应答方式

RabbitMQ在将消息传递给消费者后，并不会立即将消息从队列中删除，而是等待消费者处理是否成功，当消费者处理完消息后，向MQ发送应答，RabbitMQ接收到应答之后，认为消息处理成功，这时才将消息从MQ中删除

```mermaid
graph LR;
Provider --> MQ((MQ))
MQ --> |宕机|Consumer1
MQ --> |消息1|Consumer2
```

> 当MQ发送消息之后，并不会立即删除消息，等待消费者的应答，如果消费者1宕机，那么MQ可以将消息1发送给消费者2进行处理，并继续等待应答

#### 2.4.3 消息自动重新入队

如果消费者由于某些原因失去连接（消费者通道关闭，连接关闭或TCP连接丢失），导致消费者未发送ACK确认帧，RabbitMQ了解到消息并未完全处理，将消息重新排队，如果此时其他消费者可处理消息，便将这个未处理完的消息发送给另一个消费者，这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\消息重新入队.jpg)

演示：

消费者：有两个消费者，消费者01睡眠时间为1s，消费者02睡眠时间为30s

```java
/** 消息应答 -- 消费者01
 * @author ProgZhou
 * @createTime 2022/04/15
 */
public class Consumer01 {
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();

        System.out.println("consumer01 waiting to get message...");

        //消费者接收到消息之后如何处理消息
        DeliverCallback dc = (consumerTag, message) -> {
            String msg = new String(message.getBody());

            //消费者02只有睡眠的时间与消费者01不同
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("consumer01 receive: " + msg);
            /*
            * 消息处理完之后的手动应答
            * 1. long deliveryTag: 消息的标号
            * 2. boolean multiple: 是否批量应答
            * */
            channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
        };

        channel.basicConsume(RabbitMqUtil.QUEUE_NAME, false, dc, (consumerTag) ->{});
    }
}
```

当生产者发送消息之后，关闭消费者2，就可以看到原本应该被消费者2接收的dd消息重新发送给消费者1了

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\消费者2宕机.jpg)

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\消费者1重新接收.jpg)



### 2.5 RabbitMQ持久化

在RabbitMQ中，如果遇到RabbitMQ服务停止或者挂掉，那么其中的消息将会出现丢失的情况，为了保证在RabbitMQ服务重启的情况下不丢失消息，可以将MQ中的交换机，队列和消息都设置为可持久化的，这样能够保证在大部分情况下消息不被丢失

#### 2.5.1 队列持久化

在声明队列的时候，将boolean durable参数设置为true，表示队列持久化

```java
//声明一个队列，支持队列持久化
channel.queueDeclare(RabbitMqUtil.QUEUE_NAME, true, false, false, null);
```

队列持久化之后，在RabbitMQ的界面会有显示

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\队列持久化.jpg)

> 如果原先这个队列存在并且没有设置持久化，在代码中修改持久化参数之后重新连接的话会报错，需要先将之前的队列删除后再创建

效果：关闭RabbitMQ再重启，可以发现这个队列仍然存在

#### 2.5.2 消息持久化

如果想要在MQ重启之后查看队列中的消息，需要将消息也设置为持久化，只设置队列持久化是不够的

设置消息持久化需要在生产者发布消息时，将prop属性设置为MessageProperties.PERSISTENT_TEXT_PLAIN

```java
//设置消息持久化
channel.basicPublish("", RabbitMqUtil.QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, msg.getBytes());
```

让生产者先发送几条消息，然后关闭RabbitMQ，再重启，可以发现消息仍然在队列中

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\消息持久化.jpg)

#### 2.5.3 交换机持久化

目前还没有用到交换机，交换机的持久化可以在声明交换机的时候进行

```java
//前两个参数分别为交换机的名字和交换机的类型，true表示进行交换机持久化
channel.exchangeDeclare("exchange", "topic", true);
```



**设置了交换机，队列，消息持久化后，能保证数据不丢失吗？**

不能

其一消息丢失并不仅仅是RabbitMQ宕机的原因引起的，像之前提到过的，如果不设置消费者手动应答的话，当消费者没有处理完消息，那么这个消息也就找不到了

其二，消息在存入RabbitMQ中，还需要有一段时间才能将消息持久化到磁盘中，这就像Redis的两种持久化方式一样，如果在持久化的过程中RabbitMQ宕机，那么消息仍然会丢失

#### 2.5.4 发布确认

上面提到了，尽管RabbitMQ设置了交换机持久化、队列持久化和消息持久化，但这三者还是不能保证消息百分之百不丢失，这是由于，生产者发送消息之后，是不知道RabbitMQ是否收到，或者说是否保存下来，由此引出了这个发布确认模式

**基本介绍**

发布确认模式就是将生产者与MQ连接的信道设置为confirm模式，一旦进入confirm模式，所有生产者发送的消息就会先被保存到一个map中，并分配一个唯一的序列号

当MQ确认收到消息之后，就会给生产者发送一个确认（包含消息的序列号，表示收到了哪条消息），这就使得生产者就明确知道哪条消息被MQ收到了，对于没有收到确认的消息，生产者可以选择重发，或者做其他操作

对于发布确认模式，总共有三种形式：

+ 单个确认，指的是每个消费者发送一条消息后进入等待状态，收到MQ发送来的确认之后再发送下一条消息；如果在规定的时间内没有收到确认，则认为该消息发送失败，重发该消息，代码如下：

  ```java
  //单个确认 发送完成耗时 --- 220ms
  public static void publishMessageSingle() throws Exception{
      Channel channel = RabbitMqUtil.getChannel();
  
      //队列名称
      String queueName = "single_confirm";
      channel.queueDeclare(queueName, true, false, false, null);
  
      //开启发布确认
      channel.confirmSelect();
  
      //记录开始时间
      long begin = System.currentTimeMillis();
  
      //批量发送消息
      for (int i = 0; i < 100; i++) {
          String message = String.valueOf(i);
          channel.basicPublish("", queueName, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
          //调用waitForConfirms，等待MQ确认
          boolean b = channel.waitForConfirms();
          if(b){
              System.out.println("Message published successfully.");
          }
      }
  
      long end = System.currentTimeMillis();
      System.out.println("time: " + (end - begin));
  }
  ```

  > 单个确认的优点就是简单，并且当消息发送失败时，可以明确的知道哪个消息发送失败，快速重发
  >
  > 缺点就是效率很低，因为每发送一个消息就需要等待MQ的确认

+ 批量确认，和sql中的批量操作相似，指定一个消息确认数，比如10，表示发送10条消息之后再统一进行确认，代码如下：

  ```java
  //批量发布确认  发布完成耗时 --- 70ms
  public static void publishMessageBatch() throws Exception{
      Channel channel = RabbitMqUtil.getChannel();
  
      //队列名称
      String queueName = "batch_confirm";
      channel.queueDeclare(queueName, true, false, false, null);
  
      //开启发布确认
      channel.confirmSelect();
  
      //记录开始时间
      long begin = System.currentTimeMillis();
  
      //批量确认消息的大小，即发布多少条以后确认一次
      int batchSize = 10;
  
      //批量发送消息
      for (int i = 1; i <= 100; i++) {
          String message = String.valueOf(i);
          channel.basicPublish("", queueName, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
          if(i % batchSize == 0){
              boolean b = channel.waitForConfirms();
              if(b){
                  System.out.println("batch confirm successfully.");
              }
          }
      }
  
      long end = System.currentTimeMillis();
      System.out.println("time: " + (end - begin));
  }
  ```

  > 批量确认的优点就是效率得到了很大的提高，从发布完成耗费时间就能看出来，提高了2倍多
  >
  > 缺点就是，如果出现了消息发送失败的情况，无法确认哪一条消息发布失败，一个解决方法就是生产者将整个批量的消息重发

+ 异步确认，以上两种发布确认模式都或多或少优点缺陷，异步发布模式就很好地综合了两种发布模式的优点。生产者只管发布消息，所发布的消息会全部存储在信道的一个map中，并分配一个序列号，当MQ收到消息之后，向生产者发送确认，确认需要携带消息的序列号，这就解决了批量发布分不清哪些消息发布成功，哪些消息发布失败的问题，代码如下：

  ```java
  //异步发布确认   发布完成耗时 --- 10ms
  public static void publishMessageAsyn() throws Exception{
      Channel channel = RabbitMqUtil.getChannel();
  
      //队列名称
      String queueName = "asyn_confirm";
      channel.queueDeclare(queueName, true, false, false, null);
  
      //使用处理并发的有序map来存储发送的消息
          // ConcurrentSkipListMap<Long, String> skipListMap = new ConcurrentSkipListMap<>();
      
      //开启发布确认
      channel.confirmSelect();
  
      //消息确认成功回调函数
      ConfirmCallback ackConfirm = (deliveryTag, multiple) -> {
          //可以将发送成功的消息从存储的skipListMap中删除，这样当消息发送完毕之后，map中剩余的消息就是发送失败的消息
  //            if(multiple){
  //                //如果是批量发布的话，可以一次性确认多条消息
  //                ConcurrentNavigableMap<Long, String> navigableMap = skipListMap.headMap(deliveryTag, true);
  //                navigableMap.clear();
  //            }
          //具体逻辑具体分析
          System.out.println("message publish successfully: " + deliveryTag);
      };
      //消息确认失败回调函数
      ConfirmCallback nackConfirm = (deliveryTag, multiple) -> {
          //具体逻辑具体分析
          System.out.println("message publish failed: " + deliveryTag);
      };
  
  
      //准备消息监听器，判断哪些消息发送成功，哪些消息发送失败，也可以只监听发送成功的消息
      channel.addConfirmListener(ackConfirm, nackConfirm);
  
      //记录开始时间
      long begin = System.currentTimeMillis();
      //批量发送消息
      for (int i = 1; i <= 100; i++) {
          String message = String.valueOf(i);
          channel.basicPublish("", queueName, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
          // channel.getNextPublishSeqNo()获得的是下一个发布的序列号，当前序列号需要减一
          // skipListMap.put(channel.getNextPublishSeqNo() - 1, message);
      }
  
  
      //记录结束时间
      long end = System.currentTimeMillis();
      System.out.println("time: " + (end - begin));
  
  }
  ```
  
  
  
  
  
  
  
  
  
  

### 2.6 死信队列

#### 2.6.1 死信的概念

死信，即无法被消费的消息，一般来说，由生产者发送的消息会直接投递到rabbitmq中的队列中，然后消费者从队列中取出消息并进行消费，但某些时候由于特定的原因导致queue中的某些消息无法被消费而一直堆积在队列中，这就是死信

应用场景：为了保证订单业务的消息数据不丢失，需要使用RabbitMQ的死信队列机制，当消费发生异常时，将消息投入死信队列中；用户在商城下单成功并点击支付后在指定时间内未支付，支付将失效



**什么时候会产生死信**

+ 消息过期
+ 队列达到最大长度，无法接收新的消息
+ 消息被拒绝

#### 2.6.2 死信代码架构

```mermaid
graph LR;
subgraph MQ
	正常交换机((正常交换机)) --> |routingKey| 正常队列
	正常队列 --> |死信|死信交换机
	死信交换机((死信交换机)) --> 死信队列
end
生产者 --> 正常交换机
正常队列 --> 消费者1
死信队列 --> 消费者2
```



```java
//死信 --- 消费者1
public class ReceiveLogs01 {
    //声明正常交换机名
    public static final String NORMAL_EXCHANE = "normal_exchange";
    //声明正常队列的名称
    public static final String NORMAL_QUEUE = "normal_queue";
    //声明死信交换机的名称
    public static final String DEAD_EXCHANGE = "dead_exchange";
    //声明死信队列的名称
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();

        //声明普通交换机，直接交换机
        channel.exchangeDeclare(NORMAL_EXCHANE, BuiltinExchangeType.DIRECT);
        Map<String, Object> arguments = new HashMap<>();
        //key的值是固定的
        arguments.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        //设置死信队列的routingKey
        arguments.put("x-dead-letter-routing-key", "lisi");
        //声明普通队列,需要参数与死信交换机建立联系
        channel.queueDeclare(NORMAL_QUEUE, false, false, false, arguments);


        //声明死信交换机
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
        //声明死信队列

        channel.queueDeclare(DEAD_QUEUE, false, false, false, null);

        //绑定普通队列和普通交换机
        channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANE, "zhangsan");

        //绑定死信队列和死信交换机
        channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "lisi");


        System.out.println("Consumer01 waiting to receive message...");
        //消费者接收消息
        DeliverCallback dc = (consumerTag, message) -> {
            String msg = new String(message.getBody());
            System.out.println("Consumer01 receive message: " + msg);
        };

        CancelCallback cc = consumerTag -> {
            System.out.println("failed to receive message: " + consumerTag);
        };

        channel.basicConsume(NORMAL_QUEUE, true, dc, cc);


    }
}
```



```java
//死信 --- 生产者，将消息发送给普通交换机
public class Provider {
    //声明正常交换机名
    public static final String NORMAL_EXCHANE = "normal_exchange";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();

        channel.exchangeDeclare(NORMAL_EXCHANE, BuiltinExchangeType.DIRECT);

        AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().expiration("10000").build();
        //发消息
        for (int i = 0; i < 10; i++) {

            String message = "info" + i;

            System.out.println("Provider send message: " + message);

            channel.basicPublish(NORMAL_EXCHANE, "zhangsan", properties, message.getBytes());
        }
    }
}

```

效果：发送消息十秒钟之后，过期的消息会全部进入死信队列

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\死信队列.jpg)



#### 2.6.3 延迟队列

延迟队列，队列内部是有序的，用来存储需要在指定时间里被处理的消息

延时队列，最重要的特性就体现在它的延时属性上，跟普通的队列不一样的是，普通队列中的元素总是等着希望被早点取出处理，而延时队列中的元素则是希望被在指定时间得到取出和处理，所以延时队列中的元素是都是带时间属性的，通常来说是需要被处理的消息或者任务。

简单来说，延时队列就是用来存放需要在指定时间被处理的元素的队列。

**延迟队列的应用场景**

+ 订单在十分钟之内未支付则自动取消
+ 用户在注册之后，如果三天内没有进行登录则进行提醒
+ 预定会议之后，需要在预定的时间点前十分钟通知各个参与会议的人员

> 这些场景都有一个特点，需要在某个事件发生之后或者之前的指定时间点完成某一项任务，如：发生订单生成事件，在十分钟之后检查该订单支付状态，然后将未支付的订单进行关闭

**RabbitMQ中的TTL**

`TTL`是RabbitMQ中一个消息或者队列的属性，表明`一条消息或者该队列中的所有消息的最大存活时间`，单位是毫秒。换句话说，如果一条消息设置了TTL属性或者进入了设置TTL属性的队列，那么这条消息如果在TTL设置的时间内没有被消费，则会成为“死信”

如果同时配置了队列的TTL和消息的TTL，那么较小的那个值将会被使用。



## 三、SpringBoot整合RabbitMQ



Spring对RabbitMQ的支持

+ Spring-rabbit提供了对遵循AMQP协议消息中间件的支持
+ 需要ConnectionFactory的实现来连接消息代理
+ 提供RabbitTemplate来发送消息
+ @RabbitListener注解在方法上监听消息代理发布的消息
+ @EnableRabbit开启支持

Springboot自动配置

+ RabbitAutoConfiguration
+ 这个自动配置类会为容器种注入RabbitTemplate，AmqpAdmin，CachingConnectionFactory（RabbitMQ的连接工厂）等

AmqpAdmin的使用：

```java
@Autowired   
AmqpAdmin amqpAdmin;

@Test
public void createExchange() {
	//创建一个直接类型的交换机
    /*
    需要的参数：
    	String name: 交换机的名字
    	boolean durable: 交换机是否持久化
    	boolean autoDelete: 是否自动删除
    	Map<String, Object> arguments: 需要的一些参数
    */
    DirectExchange directExchange = new DirectExchange("hello.java.exchange", true, false, null);
    amqpAdmin.declareExchange(directExchange) //声明一个交换机，需要传入一个Exchange接口
}

@Test
public void createQueue() {
    //创建一个队列
    /*
    需要参数：
    	String name: 队列的名字
    	boolean durable: 是否持久化
    	boolean exclusive: 是否是连接独占的，也就是是不是一条连接只能连接一个队列，一般都不是
    	boolean autoDelete: 是否自动删除
    	Map<String, Object> arguments: 携带的参数
    */
    Queue queue = new Queue("hello.java.queue", true, false, true, null);
    amqpAdmin.declareQueue(queue);
}

@Test
public void createBinding() {
    //创建队列和交换机的绑定关系
    /*
    需要参数：
    	String destination: 目的地
    	DestinationType destinationType: 目的地类型
    	String exchange: 待绑定的交换机
    	String routingKey: 路由键
    	Map<String, Object> arguments: 携带的参数
    将exchange(交换机的名称)指定的交换机与destination目的地(交换机或者队列的名称)进行绑定，目的地可以是交换机也可以是队列
    */
    Binding bind = new Binding("hello.java.queue", DestinationType.QUEUE, "hello.java.exchange", "hello.java", null);
    amqpAdmin.declareBinding(bind);
}
```

RabbitTemplate的使用

```java
@Autowired
RabbitTemplate rabbitTemplate;

@Test
public void sendMessage() {
    //测试发送消息
    /*
    需要参数：
    	String exchange: 发送给的交换机名称
    	String routingKey: 消息携带的路由键
    	Object message: 发送的消息，发送的消息需要实现Serializable接口，也可以转成json数据传送
    如果想要消息能够接收或者发送json数据，就需要给容器中放一个json数据的转换器
    可以在配置类中直接添加一个
    @Bean
    public MessageConvert messageConvert(){
 		return new Jackson2JsonMessageConvert();
 	}    
 */
    rabbitTemplate.convertAndSend("hello.java.exchange", "hello.java", "hello world!");   //这个方法可以发送任何类型的消息
}
```



### 3.1 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.rabbitmq</groupId>
    <artifactId>demo02</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo02</name>
    <description>demo02</description>

    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.72</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.10.5</version>
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
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit-test</artifactId>
            <scope>test</scope>
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
                    <mainClass>com.rabbitmq.demo02.Demo02Application</mainClass>
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

### 3.2 application.yml

```yml
spring:
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: admin
    password: 123456
```

### 3.3 代码

队列示例：

```mermaid
graph LR;
Provider --> |message|X((X))
X --> |XA| queueA
X --> |XB| queueB
queueA --> |YD| Y((Y Dead)) 
queueB --> |YD| Y((Y Dead))
Y --> |routingKey|dead_queue
dead_queue --> Consumer
```



**配置类**

```java
/** RabbitMq延迟队列配置
 * @author ProgZhou
 * @createTime 2022/04/21
 */
@Configuration
public class RabbitMqTtlQueueConfig {

    //普通交换机的名称
    public static final String NORMAL_EXCHANGE = "X_normal_exchange";
    //死信交换机的名称
    public static final String DEAD_EXCHANGE = "Y_dead_exchange";

    //普通队列名称
    public static final String NORMAL_QUEUE_1 = "Qa_normal_queue";
    public static final String NORMAL_QUEUE_2 = "Qb_normal_queue";

    //死信队列名称
    public static final String DEAD_QUEUE = "Qd_dead_queue";

    //声明普通交换机
    @Bean("xExchange")
    public DirectExchange getExchange_x(){
        return new DirectExchange(NORMAL_EXCHANGE);
    }

    //声明死信交换机
    @Bean("yExchange")
    public DirectExchange getExchange_y(){
        return new DirectExchange(DEAD_EXCHANGE);
    }

    //声明普通队列
    @Bean("normalQueue_a")
    public Queue normalQueue_a(){
        Map<String, Object> map = new HashMap<>();
        //设置死信交换机
        map.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        //设置死信routingKey
        map.put("x-dead-letter-routing-key", "YD");
        //设置TTL
        map.put("x-message-ttl", 10000);

        return QueueBuilder.durable(NORMAL_QUEUE_1).withArguments(map).build();
    }
    //声明普通队列
    @Bean("normalQueue_b")
    public Queue normalQueue_b(){
        Map<String, Object> map = new HashMap<>();
        //设置死信交换机
        map.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        //设置死信routingKey
        map.put("x-dead-letter-routing-key", "YD");
        //设置TTL
        map.put("x-message-ttl", 40000);

        return QueueBuilder.durable(NORMAL_QUEUE_2).withArguments(map).build();
    }

    //声明死信队列
    @Bean("deadQueue")
    public Queue deadQueue(){
        return QueueBuilder.durable(DEAD_QUEUE).build();
    }

    //交换机与队列绑定
    @Bean
    public Binding queueBindAX(@Qualifier("normalQueue_a") Queue Qa,
                               @Qualifier("xExchange") DirectExchange x){
        return BindingBuilder.bind(Qa).to(x).with("XA");
    }
    @Bean
    public Binding queueBindBX(@Qualifier("normalQueue_b") Queue Qb,
                               @Qualifier("xExchange") DirectExchange x){
        return BindingBuilder.bind(Qb).to(x).with("XB");
    }

    //绑定死信交换机与死信队列
    @Bean
    public Binding queueBindDY(@Qualifier("deadQueue") Queue Qd,
                               @Qualifier("yExchange") DirectExchange y){
        return BindingBuilder.bind(Qd).to(y).with("YD");
    }


}
```

消费者

```java
/** 消费者
 * @author ProgZhou
 * @createTime 2022/04/21
 */
@Component
@Slf4j
public class DeadLetterQueueConsumer {

    @RabbitListener(queues = "Qd_dead_queue")
    public void receiveD(Message message, Channel channel){
        String msg = new String(message.getBody());
        log.info("当前时间: {}, 收到死信队列的消息: {}", new Date(), msg);
    }

}
```

生产者  --- Controller

```java
/** 在一般情况下，生产者由web发送消息
 * @author ProgZhou
 * @createTime 2022/04/21
 */
@Slf4j
@RestController
@RequestMapping("/ttl")
public class SendMessageController {

    @Autowired
    RabbitTemplate rabbitTemplate;


    //发消息
    @GetMapping("/send/{message}")
    public void sendMessage(@PathVariable("message") String message){
        log.info("当前时间：{}, 发送一个消息给两个TTL队列:{}", new Date(), message);

        rabbitTemplate.convertAndSend("X_normal_exchange", "XA", "10s " + message);
        rabbitTemplate.convertAndSend("X_normal_exchange", "XB", "40s " + message);

    }

}
```

测试结果：消息为abcd

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\发送消息.jpg)

![](C:\Users\86198\Desktop\JavaEE\RabbitMQ\image\监听结果.jpg)





整合springboot之后使用可以使用RabbitTemplate发送消息
