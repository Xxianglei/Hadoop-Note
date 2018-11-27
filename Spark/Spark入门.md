## Spark概述

**Spark是什么？**

>Spark是一个快速的通用的针对大数据集的计算框架
>处理的数据放在内存里面，计算时采用DAG有向无环图。为什么叫通用的，是因为他可以做Hadoop的各种功能模块，不需要其他框架辅助。

**Sprak特性**

![](https://i.imgur.com/iXQH0hp.png)

* 计算比Mapreduce快一百倍。
* 数据放在磁盘比MR快10倍。
* Spark简单易用支持 R java python Scala
* 通用性继承了SQL streaming 和复杂计算
* Spark可以运行在任何地方

![](http://spark.apache.org/images/spark-runs-everywhere.png)


**MapReduce与Spark的类似框架**

	MapReduce
		Hive		Storm			Mahout		Griph
	
	Spark Core
		Spark SQL	Spark Streaming	Spark ML	Spark GraphX	Spark R

SparkStreaming 比storm 用的多。Spark不可代替MR。

MR中数据传输是kv形式，Spark是1以RDD（弹性分布式数据集）的形式。

**Spark生态系统**

![](https://i.imgur.com/EMEj44G.png)

Hive 底层的引擎运行可以直接转换成 MapReduce Tez Spark
BlinkDB 是一个在误差界限内或者一定返回时间内的SQL查询

**MR与Spark的区别**


![](https://i.imgur.com/d9Kw1vj.png)

![](https://i.imgur.com/XK1KARs.png)

## Spark编译、测试

>The Maven-based build is the build of reference for Apache Spark. Building Spark using Maven requires Maven 3.5.4 and Java 8. Note that support for Java 7 was removed as of Spark 2.2.0.

* 下载源码
* [编译细节](http://spark.apache.org/docs/latest/building-spark.html)
	* SBT 编译
	* Maven 编译
	* 打包编译make-distribution.sh


* Maven安装部署

	* 下载地址
	http://maven.apache.org/download.cgi
	*  解压
	tar -zvxf apache-maven-3.0.5
	* 配置环境变量
	export MAVEN_HOME=/opt/modules/maven-3.0.5
	export PATH=$PATH:$MAVEN_HOME/bin
	* 验证
	mvn -version


* 解压 tar -zvxf spark-1.3
	
	### MAVEN 编译
	$ /opt/modules/spark-1.3.0-src/build/mvn clean package -DskipTests -Phadoop-2.4 -Dhadoop.version=2.5.0-cdh5.3.6 -Pyarn -Phive-0.13.1 -Phive-thriftserver
	
	### 打包编译，使用CDH 5.3.6版本的HADOOP
	./make-distribution.sh --tgz -Phadoop-2.4 -Dhadoop.version=2.5.0-cdh5.3.6 -Pyarn -Phive-0.13.1 -Phive-thriftserver
	
	### 打包编译，使用Apache版本的HADOOP
	./make-distribution.sh --tgz -Phadoop-2.4 -Dhadoop.version=2.5.0 -Pyarn -Phive-0.13.1 -Phive-thriftserver

编译spark注意：1）替换maven 仓库jar包 2）配置域名解析服务 3）修改make-distribution.sh 4）执行打包编译


## Spark运行方式

>Local Standalone Yarn  Mesos

安装scala

配置环境变量

![](https://i.imgur.com/cyFN3uB.png)

解压spark-1.3.0-bin-2.5.0-cdh5.3.6.tar.gz

直接启动测试官方例子（本地模式）

![](https://i.imgur.com/u6ZN5uF.png)
![](https://i.imgur.com/QulmeIr.png)

	//一行一行的读 把值封装到RDD
	 var testFile = sc.textFile("README.md")
	
	testFile: org.apache.spark.rdd.RDD[String] = README.md MapPartitionsRDD[1] at textFile at <console>:21
	
	可以将函数A作为参数传递给函数B,此时这个函数B叫做高阶函数
	textFile.filter((line: String) => line.contains("Spark"))
	textFile.filter(line => line.contains("Spark"))
	textFile.filter(line => line.contains("Spark"))
	textFile.filter(_.contains("Spark"))
	下划线表示任意元素

	匿名函数
	(line: String) => line.contains("Spark")
	def func01(line: String) {
		line.contains("Spark")
	} 
	
	def func01(line: String) => line.contains("Spark")
	
	
	filter(f: T => Boolean)
	(line: String) => line.contains("Spark")
	
	T:
		(line: String)
	Boolean:
		line.contains("Spark")
函数没有参数时，括号可以省略。

发现命令行是Scala

![](https://i.imgur.com/7XWrSWw.png)


Web端口 http://192.168.1.205:4040 开多个的时候从4040端口暂用 端口号往后变。


## Spark部署

>Spark Standalone 安装部署模式指的是Spark本身自带的集群管理。

它也是分布式的

主节点 Master
从节点 Work Nodes

![](https://i.imgur.com/DdF7NeG.png)

需要安装的内容

1）JAVA

2）SCALA

3）HDFS

4）SPARK

