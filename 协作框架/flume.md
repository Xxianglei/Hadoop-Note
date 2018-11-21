## Flume

>Flume is a distributed（分布式）, reliable（可靠的）, and available（可用的） service for efficiently collecting, aggregating, and moving large amounts of log data. It has a simple（简单的） and flexible（灵活的） architecture based on streaming data flows（流）. It is robust(健壮) and fault tolerant（容错） with tunable reliability mechanisms and many failover and recovery mechanisms. It uses a simple extensible data model that allows for online analytic application（在线数据分析应用）.


![](https://i.imgur.com/Lt1ZHd1.png)

分布式在这 flume可以去搜集很多个服务器上的数据。而不是说Flume向hdfs那种。

Cloudera 开发的框架可以做到实时收集数据的文件搜集框架。


![](http://flume.apache.org/_images/DevGuide_image00.png)

![](https://i.imgur.com/YV2Ao9T.png)

Agent


拿到数据封装在一个event中，然后放到管道中，在管道中（队列）做个缓存可以保证数据安全性，然后sink写到hdfs。

Events 

![](https://i.imgur.com/sMtVBa8.png)

* Event是Flume数据传输的基本单元
* Flume以事件的形式将数据从源头传送到最终的目的
* Event由可选的header和载有数据的一个byte array构成
	* 载有的数据对flume是不透明的
	* Header是容纳了key-value字符串对的无序集合，key在集合内是唯一的。
	* Header可以在上下文路由中使用扩展


Flume运行环境

* 运行在logs地方
* 系统:Linux
* JVM/JDK
* 轻量级服务（eg，zk，jn，zkfc，sqoop）

## Flume安装配置

* tar -zxvf flume-ng-1.5.0-cdh5.3.6.tar.gz -C /opt/cdh-5.3.6/

修改conf文件

![](https://i.imgur.com/rPxlNRu.png)
![](https://i.imgur.com/2J623OV.png)

* 把hdfs的jar包丢到lib下，数据传到hdfs的必须

![](https://i.imgur.com/jB7fJSN.png)

* 使用

		bin/flume-ng agent --conf conf --name agent-test --conf-file test.conf
	
		bin/flume-ng agent -c conf -n agent-test -f test.conf

* 常用内容

		bin/flume-ng 
		Usage: bin/flume-ng <command> [options]...
		
		commands:
		  agent                     run a Flume agent
		
		global options:
		  --conf,-c <conf>          use configs in <conf> directory
		  -Dproperty=value          sets a Java system property value
		
		agent options:
		  --name,-n <name>          the name of this agent (required)
		  --conf-file,-f <file>     specify a config file (required if -z missing) 

**测试使用**

一.编辑 a1.conf
![](https://i.imgur.com/dNawMgy.png)

	# Name the components on this agent
	a1.sources = r1
	a1.sinks = k1
	a1.channels = c1
	
	# Describe/configure the source
	a1.sources.r1.type = netcat
	a1.sinks = k1
	a1.channels = c1
	
	# Describe/configure the source
	a1.sources.r1.type = netcat
	a1.sources.r1.bind = master
	a1.sources.r1.port = 44444
	
	# Use a channel which buffers events in memory
	a1.channels.c1.type = memory
	a1.channels.c1.capacity = 1000
	a1.channels.c1.transactionCapacity = 100
	
	# Describe the sink
	a1.sinks.k1.type = logger
	a1.sinks.k1.maxByteToLog=1024
	
	
	# Bind the source and sink to the channel
	
	a1.sources.r1.channels = c1
	a1.sinks.k1.channel = c1

发送字符  装个软件talnet
	
	/etc/rc.d/init.d/xinetd restart

二.执行 flume-ng 
![](https://i.imgur.com/L8QYMJG.png)

	 ./flume-ng agent --c /opt/cdh-5.3.6/flume/conf --n a1  -Dflume.root.logger=INFO,console

查看已经打开的端口

  netstat -tnlp；

![](https://i.imgur.com/EMphuiO.png)

![](https://i.imgur.com/8pWUcmj.png)

测试：

![](https://i.imgur.com/kUmvUVt.png)

![](https://i.imgur.com/NFU5wN6.png)
说明：

![](https://i.imgur.com/446FmFx.png)

设计一个案例：

* 收集log

	hive运行的日志
	/opt/cdh-5.3.6/hive-0.13.1-cdh5.3.6/logs/hive.log
	tail -f 
* memory

* hdfs
	/user/beifeng/flume/hive-logs/

第二个agent程序
	
	bin/flume-ng agent \
	-c conf \
	-n a2 \
	-f conf/flume-tail.conf \
	-Dflume.root.logger=DEBUG,console

模板写法：

	# example.conf: A single-node Flume configuration
	
	# Name the components on this agent
	a2.sources = r2
	a2.sinks = k2
	a2.channels = c2
	
	# Describe/configure the source
	a2.sources.r2.type = exec
	a2.sources.r2.command = tail -f /opt/cdh-5.3.6/hive-0.13.1-cdh5.3.6/logs/hive.log
	a2.sources.r2.shell=/bin/bash -c
	
	# Use a channel which buffers events in memory  
	a2.channels.c2.type = memory
	a2.channels.c2.capacity = 1000
	a2.channels.c2.transactionCapacity = 100
	
	# Describe the sink
	a2.sinks.k2.type = hdfs
	a2.sinks.k2.hdfs.path = hdfs://master:8020/user/flume/hive-logs
	a2.sinks.k2.hdfs.filePrefix = events-
	a2.sinks.k2.hdfs.fileType=DataStream 
	a2.sinks.k2.hdfs.batchSize= 10
	
	a2.sinks.k2.hdfs.writeFormat=Text
	
	# Bind the source and sink to the channel
	
	a2.sources.r2.channels = c2
	a2.sinks.k2.channel = c2
	

	运行
	./flume-ng agent -c /opt/cdh-5.3.6/flume/conf -f /opt/cdh-5.3.6/flume/conf/hdfs.conf -n a2 -Dflume.root.logger=INFO,console

	开启hive生成日志文件

	查看文件系统日志收集情况

![](https://i.imgur.com/VWrmZ4V.png)

**HDFS Sink**

>This sink writes events into the Hadoop Distributed File System (HDFS). It currently supports creating text and sequence files. It supports compression in both file types. The files can be rolled (close current file and create a new one) periodically based on the elapsed time or size of data or number of events. It also buckets/partitions data by attributes like timestamp or machine where the event originated. The HDFS directory path may contain formatting escape sequences that will replaced by the HDFS sink to generate a directory/file name to store the events. Using this sink requires hadoop to be installed so that Flume can use the Hadoop jars to communicate with the HDFS cluster. Note that a version of Hadoop that supports the sync() call is required.


![](https://i.imgur.com/RLXk5rD.png)

>Note For all of the time related escape sequences, a header with the key “timestamp” must exist among the headers of the event (unless hdfs.useLocalTimeStamp is set to true). One way to add this automatically is to use the TimestampInterceptor.


![](https://i.imgur.com/vQbugOb.png)

	hdfs://master:8020/user/beifeng/flume/applogs/%Y%m%d

	hdfs.useLocalTimeStamp = true 

## 实战案例

>一般来说去监控某个日志文件的目录，不搜集变化的日志文件，只是收集完成的文件。（Spool Directory Source）


----
>Spooling Directory Paths通过监听某个目录下的新增文件，
并将文件的内容读取出来，实现日志信息的收集。实际生产
中会结合log4j来使用。被传输结束的文件会修改后缀名，添
加.COMPLETED后缀

![](https://i.imgur.com/ru4BSu7.png)

**实现**

* 监控目录
	* 日志目录，抽取完整的日志文件，写的日志文件不抽取
* 使用FileChannel
	* 本地文件系统缓冲，比内存安全性更高
* 数据存储HDFS
	* 存储对应hive表的目录或者hdfs目录

编辑conf文件：
	
	# example.conf: A single-node Flume configuration
	
	# Name the components on this agent
	a3.sources = r3
	a3.sinks = k3
	a3.channels = c3
	
	# Describe/configure the source
	a3.sources.r3.type = spooldir
	a3.sources.r3.spoolDir  =/opt/cdh-5.3.6/flume/spoollogs
	a3.sources.r3.ignorePattern=^(.)*\\.log*$
	a3.sources.r3.fileSuffix=delete
	
	# Use a channel 
	a3.channels.c3.type = file
	a3.channels.c3.checkpointDir= /opt/cdh-5.3.6/flume/filechannel/checkpoint
	a3.channels.c3.dataDirs= /opt/cdh-5.3.6/flume/filechannel/data
	
	
	# Describe the sink
	a3.sinks.k3.type = hdfs
	a3.sinks.k3.hdfs.path = hdfs://master:8020/user/flume/spool-logs/%y%m%d
	a3.sinks.k3.hdfs.filePrefix = events-
	a3.sinks.k3.hdfs.fileType=DataStream 
	a3.sinks.k3.hdfs.batchSize= 10
	
	a3.sinks.k3.hdfs.writeFormat=Text
	
	# Bind the source and sink to the channel
	
	a3.sources.r3.channels = c3
	a3.sinks.k3.channel = c3

执行：

	./flume-ng agent -c /opt/cdh-5.3.6/flume/conf -f /opt/cdh-5.3.6/flume/conf/app.conf -n a3 -Dflume.root.logger=INFO,console

