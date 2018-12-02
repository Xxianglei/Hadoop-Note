## Spark HistoryServer



> 应用在运行时候可以用4040端口提供WEB监控。那当应用完成或者说运行失败的时候怎么去监控呢？那就是HistoryServer。



![](https://i.imgur.com/JNu6o7H.png)



**配置**



> 配置在spark-env.sh中的SPARK_HISTORY_OPTS,可以理解为服务端的配置



![](https://i.imgur.com/vV4AdoC.png)



> 配置在spark-defaults.conf,可以理解为客户端的配置



![](https://i.imgur.com/lctUGYj.png)

- 启动 HistoryServer

  > ./start-history-server.sh

## Spark on Yarn

> the driver program 运行在什么地方？
>
> > 本地，应用运行的这台机器，如图一
> >
> > work 节点的某台机器上，如图二

![](https://i.imgur.com/u4yAJ27.png)


![](https://i.imgur.com/egWiBEQ.png)

**Shell 命令**

```shell
/spark-submit \
--master spark://master:7077 \
--deploy-mode client \
jars / sparkAPP.jar 
```



```shell
/spark-submit \
--master spark://master:7077 \
--deploy-mode cluster \
jars / sparkAPP.jar
```

------

#### Spark on Yarn架构图

![](https://i.imgur.com/0QZdKVS.png)

> 客户端提交给RM	
>
> RM-->AM(Spark)
>
> Spark APP --> Executors
>
> > SAM->RM 申请运行所需要的资源
>
> Driver program 
>
> > DAG->DAGScheduler->StageScheduler->TaskSet


![](https://i.imgur.com/eiUibZN.png)

**Shell 命令**

首先启动Yarn

将spark-shell运行在yarn上，以client 模式

```shell
./bin/spark-shell --master yarn --deploy-mode client
```
![](https://i.imgur.com/wyObzbE.png)

```shell
./bin/spark-submit  \
    --master yarn-cluster \
    jars/sparkApp.jar
```

## Spark Streaming



> Streaming：是一种数据传送技术，它把客户机收到的数据变成一个稳定连续的
> 流，源源不断地送出，使用户听到的声音或看到的图象十分平稳，而且用户在
> 整个文件送完之前就可以开始在屏幕上浏览文件。

流式计算

- Apache Strom
- Spark Streaming
- Apache Samza

上述三种实时计算系统都是开源的分布式系统，具有低延迟、可扩展和容错性
诸多优点，它们的共同特色在于：允许你在运行数据流代码时，将任务分配到
一系列具有容错能力的计算机上并行运行。此外，它们都提供了简单的API来
简化底层实现的复杂程度。



**Spark Streaming 是什么**





> Spark Streaming is an extension(扩展) of the core Spark API that enables scalable（可扩展性）, high-throughput（高吞吐量）, fault-tolerant stream processing of live data streams（流式处理）.


![](https://i.imgur.com/LWWtQdi.png)



> 原始数据流->分布式流式计算系统->输出数据


![](https://i.imgur.com/lTchvs6.png)

**官方例子**

- 从Socket实时读取数据，进行实时处理
- rpm -ivh nc-1.84-22.el6.x86_64.rpm
- 运行nc针对于端口号9999
  $ nc -lk 9999
- 运行Demo
  bin/run-example streaming.NetworkWordCount master 9999

![](https://i.imgur.com/kVHgVUp.png)



**代码例子：**


![](https://i.imgur.com/TIeZn2M.png)

## Spark Streaming是如何工作的

- 流数据分批，底层实质上还是一个批处理。每一个batch交给core处理。

![](https://i.imgur.com/AIlLuOx.png)

- 二

![](https://i.imgur.com/ghtRHVW.png)

![](https://i.imgur.com/1nbEhca.png)

![](https://i.imgur.com/VsaA0EV.png)

![](https://i.imgur.com/Kx7lGKg.png)


## Spark Streaming编程



- 方式一：IDE编程用的比较多

![](https://i.imgur.com/NJxvSOJ.png)

- 方式二：spark shell用的比较多

![](https://i.imgur.com/Zt5HAxO.png)

基本步骤：

- 通过创建一个·Dstream定义一个数据流
- 定义流式计算，进行一些转换和输出
- 开始接收数据并且处理它   streamingContex。start()
- 等待应用处理的介绍    streamingContex.awaitTermination()
- 可以手动停止    streamingContext.stop（）

**代码模板**

```scala
import org.apache.spark._
import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._ 

val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
val ssc = new StreamingContext(conf, Seconds(5))

// 读取数据 实质创建了一个Dstream
val lines = ssc.socketTextStream("localhost", 9999)

// 数据处理
val words = lines.flatMap(_.split(" "))
val pairs = words.map(word => (word, 1))
val wordCounts = pairs.reduceByKey(_ + _)
wordCounts.print()

//启动应用
ssc.start()             // 开始计算
ssc.awaitTermination()  // 一直等到计算终止

```



**HDFS上的数据操作**



```scala
import org.apache.spark._
import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._ 

val ssc = new StreamingContext(sc, Seconds(5))

// 从HDFS上面读取文本数据
val lines = ssc.textFileStream("hdfs://master:8020/user/streaming/input/hdfs/")

// 数据处理
val words = lines.flatMap(_.split("\t"))
val pairs = words.map(word => (word, 1))
val wordCounts = pairs.reduceByKey(_ + _)
wordCounts.print()
 
ssc.start()             // 开始计算
ssc.awaitTermination()  //  一直等到计算终止
```

测试：

- 创建 wc.txt
- 创建文件目录：./hdfs dfs -mkdir -p /user/streaming/input/hdfs （监控的文件夹）
- 上传文件 ：./hdfs dfs -put /opt/datas/wc.txt /user/streaming/input/hdfs
- 开启 ./spark-shell --master local[2]
- 拷贝代码运行



```scala
import org.apache.spark._
import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._ 

val ssc = new StreamingContext(sc, Seconds(5))

// read data
val lines = ssc.textFileStream("hdfs://master:8020/user/streaming/input/hdfs/")

// process
val words = lines.flatMap(_.split(" "))
val pairs = words.map(word => (word, 1))
val wordCounts = pairs.reduceByKey(_ + _)
wordCounts.saveAsTextFiles("hdfs://master:8020/user/streaming/output/")
 
ssc.start()             // Start the computation
ssc.awaitTermination()  // Wait for the computation to terminate
```



如何在Spark-shell中执行某个scala代码，快速开发可以创建一个xx.scala的文件 

执行      :load /opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/HDFSWordCount.scala

## Spark Streaming集成Flume

> Flume有三个组件
> ​	Source  --->   Channel  --->   Sink(Spark Streaming) 作为接受者，接受数据进行处理。



导入必要的包：

![](https://i.imgur.com/kEaAgK7.png)
代码：

```scala
import org.apache.spark._
import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._ 
import org.apache.spark.streaming.flume._
import org.apache.spark.storage.StorageLevel

val ssc = new StreamingContext(sc, Seconds(5))

// read data
val stream = FlumeUtils.createStream(ssc, "master", 9999, StorageLevel.MEMORY_ONLY_SER_2)

stream.count().map(cnt => "Received " + cnt + " flume events." ).print()

ssc.start()             // Start the computation
ssc.awaitTermination()  // Wait for the computation to terminate
```

运行shell 加载依赖包

```shell
./spark-shell --jars \
/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/spark-streaming-flume_2.10-1.3.0.jar,/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/flume-avro-source-1.5.0-cdh5.3.6.jar,/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/flume-ng-sdk-1.5.0-cdh5.3.6.jar

```

开启flume

```shell
bin/flume-ng agent -c conf -n a2 -f conf/flume-spark-push.sh -Dflume.root.logger=DEBUG,console
```

## Spark Streaming 集成Kafka集群

![](https://i.imgur.com/eeIDr2y.png)

**Kafka架构**

![](https://i.imgur.com/0Ijjk7E.png)

- 生产者：是能够发布消息到话题的任何对象
- Topic：是特定类型的消息流。消息是字节的有效负载，话题是消息的分类名或种子名
- 消费者：可以订阅一个或者多个Topic并且从Broker拉数据
- 代理（Kafka集群）：已发布的消息保存在一组服务器中

## Kafka 安装配置

基本环境：

![](https://i.imgur.com/JcLBIKY.png)

- 解压 tar -zxvf kafka_2.10-0.8.2.1.tgz -C /opt/cdh-5.3.6/
- 更换 lib下的 zookeeper jar包为自己合适的 zookeeper jar 


![](https://i.imgur.com/hp7CPzP.png)

- 修改配置文件 vi server.properties

  ```xml
  ############################# Server Basics #############################
  
  # The id of the broker. This must be set to a unique integer for each broker.
  broker.id=0
  
  
  # The port the socket server listens on
  port=9092
  
  host.name=master
  
  # java.net.InetAddress.getCanonicalHostName().
  #advertised.host.name=<hostname routable by clients>
  
  # The number of threads handling network requests
  num.network.threads=3
  
  # The number of threads doing disk I/O
  
  socket.request.max.bytes=104857600
  # A comma seperated list of directories under which to store log files
  log.dirs=/opt/cdh-5.3.6/kafka_2.10-0.8.2.1/tmp/kafka-logs
  
  # parallelism for consumption, but this will also result in more files across
  # the brokers.
  num.partitions=1
  # There are a few important trade-offs here:
  
  # The number of messages to accept before forcing a flush of data to disk
  #log.flush.interval.messages=10000
  
  
  # from the end of the log.
  
  # The minimum age of a log file to be eligible for deletion
  log.retention.hours=168
  
  # A size-based retention policy for logs. Segments are pruned from the log as long as the remaining
  # segments don't drop below log.retention.bytes.
  #log.retention.bytes=1073741824
  
  # The maximum size of a log segment file. When this size is reached a new log segment will be created.
  log.segment.bytes=1073741824
  
  # The interval at which log segments are checked to see if they can be deleted according
  # to the retention policies
  log.retention.check.interval.ms=300000
  
  # By default the log cleaner is disabled and the log retention policy will default to just delete segments after their retention expires.
  # If log.cleaner.enable=true is set the cleaner will be enabled and individual logs can then be marked for log compaction.
  log.cleaner.enable=false
  
  ############################# Zookeeper #############################
  
  # Zookeeper connection string (see zookeeper docs for details).
  # This is a comma separated host:port pairs, each corresponding to a zk
  # server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
  # You can also append an optional chroot string to the urls to specify the
  # root directory for all kafka znodes.
  zookeeper.connect=master:2181
  
  # Timeout in ms for connecting to zookeeper
  zookeeper.connection.timeout.ms=6000
  ```



- 启动Kafka

 ./kafka-server-start.sh  ../config /server.properties



**简单实用**

启动Kafka集群 放到后台去启动
​	nohup bin/kafka-server-start.sh config/server.properties & 

创建topic命令
​	./kafka-topics.sh --create --zookeeper master:2181 --replication-factor 1 --partitions 1 --topic test

查看已用topic
​	./kafka-topics.sh --list --zookeeper master:2181

生产数据

修改配置 producer.properties

![](https://i.imgur.com/5iAmjll.png)

​	./kafka-console-producer.sh --broker-list master:9092 --topic test

消费数据
​	./kafka-console-consumer.sh --zookeeper master:2181 --topic test --from-beginning

![](https://i.imgur.com/gF7uiCz.png)

![](https://i.imgur.com/2E0Olzq.png)



**Spark Streaming与kafka集成细节**



```scala
===============Spark Streaming + Kafka Integration 基于接收者=================
import java.util.HashMap
import org.apache.spark._
import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._ 
import org.apache.spark.streaming.kafka._

val ssc = new StreamingContext(sc, Seconds(5))

val topicMap = Map("test" -> 1)

// read data
val lines = KafkaUtils.createStream(ssc, "master:2181", "testWordCountGroup", topicMap).map(_._2)

val words = lines.flatMap(_.split(" "))
val wordCounts = words.map(x => (x, 1)).reduceByKey(_ + _)
wordCounts.print()

ssc.start()             // Start the computation
ssc.awaitTermination()  // Wait for the computation to terminate

------------------
bin/spark-shell --jars \
/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/spark-streaming-kafka_2.10-1.3.0.jar,/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/kafka_2.10-0.8.2.1.jar,/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/kafka-clients-0.8.2.1.jar,/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/zkclient-0.3.jar,/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/metrics-core-2.2.0.jar

第两种方式 基于
===============Spark Streaming + Kafka Integration=================
import kafka.serializer.StringDecoder
import org.apache.spark._
import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._ 
import org.apache.spark.streaming.kafka._

val ssc = new StreamingContext(sc, Seconds(5))

val kafkaParams = Map[String, String]("metadata.broker.list" -> "master:9092")
val topicsSet = Set("test")

// read data
val messages = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](ssc, kafkaParams, topicsSet)

val lines = messages.map(_._2)
val words = lines.flatMap(_.split(" "))
val wordCounts = words.map(x => (x, 1)).reduceByKey(_ + _)
wordCounts.print()

ssc.start()             // Start the computation
ssc.awaitTermination()  // Wait for the computation to terminate

------------------
bin/spark-shell --jars \
/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/spark-streaming-kafka_2.10-1.3.0.jar,/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/kafka_2.10-0.8.2.1.jar,/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/kafka-clients-0.8.2.1.jar,/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/zkclient-0.3.jar,/opt/cdh-5.3.6/spark-1.3.0-bin-2.5.0-cdh5.3.6/externallibs/metrics-core-2.2.0.jar
```

