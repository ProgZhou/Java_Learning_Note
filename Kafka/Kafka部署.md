# Kafka部署

## 一、安装包准备



kafka安装包官网下载地址：http://kafka.apache.org/downloads

将安装包上传至linux服务器指定目录，如/data/kafka

```shell
#解压安装包
tar -xzf kafka_2.13-3.4.0.tgz
```

**解压完的目录结构**

```
[root@user kafka_2.13-3.4.1]# ls
bin  config  libs  LICENSE  licenses  logs  NOTICE  site-docs  zookeeper
```

## 二、修改zookeeper配置

> kafka需要注册到zookeeper上，在启动kafka之前，需要启动zookeeper

在单机上部署zookeeper集群：

+ 复制三份zookeeper配置文件：zookeeper2181.properties、zookeeper2182.properties、zookeeper2183.properties

+ 修改配置文件：

  ```properties
  #zookeeper数据目录
  dataDir=/data/kafka/kafka_2.13-3.4.1/zookeeper/data/2181
  #zookeeper日志目录
  dataLogDir=/data/kafka/kafka_2.13-3.4.1/zookeeper/logs/2181
  #zookeeper启动端口 不同配置文件端口不同
  clientPort=2181
  # disable the per-ip limit on the number of connections since this is a non-production config
  maxClientCnxns=60
  # Disable the adminserver by default to avoid port conflicts.
  # Set the port to something non-conflicting if choosing to enable this
  admin.enableServer=false
  # admin.serverPort=8080
  
  
  initLimit=10
  syncLimit=5
  tickTime=2000
  
  #需要在各自的dataDir目录下创建对应的myid文件，内容分别为'1'、'2'、'3'
  server.1=10.216.0.62:2881:3881
  server.2=10.216.0.62:2882:3882
  server.3=10.216.0.62:2883:3883
  ```

+ 启动zookeeper(可以先前台启动观察是否报错)

  ```shell
  nohup ./zookeeper-server-start.sh ../config/zookeeper2181.properties &
  nohup ./zookeeper-server-start.sh ../config/zookeeper2182.properties &
  nohup ./zookeeper-server-start.sh ../config/zookeeper2183.properties &
  ```

## 三、修改kafka配置

在单机上部署kafka集群：

+ 复制三份kafka配置文件：server9092.properties、server9093.properties、server9094.properties

+ 修改配置文件：

  ```properties
  # The id of the broker. This must be set to a unique integer for each broker.
  broker.id=1
  
  ############################# Socket Server Settings #############################
  #如果需要外网连接，此处需要配置回环地址
  listeners=PLAINTEXT://0.0.0.0:9092   
  advertised.listeners=PLAINTEXT://localhost:9092
  # The number of threads that the server uses for receiving requests from the network and sending responses to the network
  num.network.threads=3
  
  # The number of threads that the server uses for processing requests, which may include disk I/O
  num.io.threads=8
  
  # The send buffer (SO_SNDBUF) used by the socket server
  socket.send.buffer.bytes=102400
  
  # The receive buffer (SO_RCVBUF) used by the socket server
  socket.receive.buffer.bytes=102400
  
  # The maximum size of a request that the socket server will accept (protection against OOM)
  socket.request.max.bytes=104857600
  
  ############################# Log Basics #############################
  # A comma separated list of directories under which to store log files
  log.dirs=/data/kafka/kafka_2.13-3.4.1/logs/9092
  
  # The default number of log partitions per topic. More partitions allow greater
  # parallelism for consumption, but this will also result in more files across
  # the brokers.
  num.partitions=10
  num.recovery.threads.per.data.dir=1
  
  ############################# Internal Topic Settings  #############################
  # The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
  # For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
  offsets.topic.replication.factor=2
  transaction.state.log.replication.factor=2
  transaction.state.log.min.isr=2
  
  ############################# Log Flush Policy #############################
  
  ############################# Log Retention Policy #############################
  log.retention.hours=168
  log.retention.check.interval.ms=300000
  
  ############################# Zookeeper #############################
  zookeeper.connect=localhost:2181,localhost:2182,localhost:2183
  
  zookeeper.connection.timeout.ms=18000
  
  group.initial.rebalance.delay.ms=0
  ```

+ 后台启动kafka

  ```
  nohup ./kafka-server-start.sh ../config/server9092.properties &
  nohup ./kafka-server-start.sh ../config/server9093.properties &
  nohup ./kafka-server-start.sh ../config/server9094.properties &
  ```

  