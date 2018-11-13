>昨天初步介绍了Hadoop，以及Hadoop的单机模式、伪分布式应用。今天主要记录一下对HDFS文件系统的Java Api操作以及Mapreduce的编程。时间紧张，废话就不多说了，趁着脑子还清醒，赶紧做个笔记。🙈🙉🙊

## HDFS
--- 
前面提过HDFS是起源于Google的GFS论文，其主要特点是易于扩展、运行在大量廉价机器上提过容错机制、提供高性能的文件存储。
>上架构图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111193209571.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)
图中我们可以看到：
##### NameNode
* Namenode是一个单一节点的中心服务器，主要用于存储一些元数据，负责管理文件系统的namespace以及客户端对文件的访问。
* Datanode负责数据的存取（数据流不经过namenode）仅仅通过Datanode，Namenode只是保存的一些元数据。
* 副本的存放位置由namenode决定，读取时候namenode尽量让用户先读取最近的副本，来减少时延。
* 同时Namenode还管理数据块的复制，周期性的接收datanode传来的心跳。
##### DataNode
* Datanode以文件存储在磁盘上（数据本身+元数据）
* Datanode在向namenode注册后周期性（一小时）的发送报告
* Datanode每3s向namenode发送心跳，超过10分钟没收到心跳，namenode会认为节点损坏，然后开始恢复备份。
为此，我简单说一下Namenode和Datanode的通信写作方式。主要有两个。
1. 心跳机制
2. 块的报告

对于HDFS的文件放置策略，见官网
>[放置策略](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html) 简而言之就是：在HDFS上，文件是以块保存的，默认存入三个副本。默认是在HDFS客户端上放第一个副本，第二个副本放在与第一个不同且随机的另外的选择的机架中的节点上。第三个副本与第二个副本在同一个机架但是不同节点上。
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181113175119582.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)
##### 数据损坏的处理 
当DN读取block时，他会计算checksum，如果发现和创建时不一样，说明文件以及损坏。此时客户端就通过读取其他节点来获取完整的数据（备份）。对于Namenode而言，他将标记损坏的节点，并且复制block来恢复备份。最后Datanode在其文件创建后一段时间再次验证checksum。
## Name启动过程
----



 
先说一说为什么格式化文件系统
>目的是生成fsimage文件（文件系统的元数据信息）

> 第一次启动 

0->format----->生成fsimage文件

1->fsiamge 一个永久性检测点

2->start namenode 第一次启动namenode
* read fsimage 

3->start datanode

  * 向namenode注册
  * block report

4->mkdir  put  get ... 

这些改变会写到编辑日志文件里edit（存储整个文件系统，元数据变化的信息）

> 第二次启动

1. statrt namenode
	1. read[fsimage]
	2. read[edits]
	3. 读完后在本地生成一个新的fsimage
	4. 生成一个编辑日志（空）
	
2. 和上面一样操作写到新的edit日志里
> secondarynamenode 如何辅助 
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181112194336868.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)
每隔一段时间将编辑日志和镜像文件合并一下，可以大大提高启动时文件读取速度。 

* 本地磁盘
	* fsimage
	* edits
	* 合成新的 fsimage
	
secondary定期拷贝到他的目录下进行合并，生成一个新的，然后复制给namenode，最后重命名为fsiamge，最后删除掉原来的 。
注意：SecondaryNamenode不可理解为Namenode的“热备”，这是一个错误的认识。

## 数据读取
----
#### 读
>读的过程如下图，比较简单，就不赘述。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2018111317575226.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)
#### 写

>打开要写入的文件，请求送至Namenode，简历改文件的元数据。此时新的元数据还没有和任何数据块关联，这时客户端收到“打开成功”，可以开始写入数据。
>>写入的数据自动拆分为数据包，放入内存队列。由客户端独立的线程读取数据包，向Namenode发送请求，求得一组datanode如下图。
>>>接着向Datanode中写入数据，在建立好的复制管道中实现数据复制备份。同时做出相应的应答。最终完成写入过程。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181113180539620.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)


## 安全模式safemode
----
发生在namenode生成加载完两个文件后，就开始进入到安全模式，然后等待datanode发送块的报告。  

直到`total blocks/total blocks=99.999%`此时安全模式才会退出。（30s）左右的稳定时间 


操作： 

* 查看文件系统的文件
* 不能改变文件系统的命名空间（元数据不能改变）

	* 创建文件夹
	* 上传文件
	* 删除文件
	
结束后web上会显示 ：

* Security is off.

* Safemode is off.

## 手动进入安全模式
----
* hdfs dfsadmin -safemode enter 

进入后客户端除了读啥都干不了

*  hdfs dfsadmin -safemode get 

查看是否是安全模式

* hdfs dfsadmin -safemode leave

离开安全模式

## Java Api开发HDFS
----
* Linux下eclipse安装
* Linux下Maven安装
*  创建maven项目
* Hadoop jar依赖
* Junit jar依赖 
* 添加配置文件到main/resources下,直接把机器上改好的拿出来就行
* 添加log4j的配置文件消除警告（自愿）

```
package com.ibeifeng.hadoop.senior.hdfs;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;

/**
 * 
 * @author xianglei
 * 
 */
public class HdfsApp {

	/**
	 * 获取文件系统
	 * 
	 * @return
	 * @throws Exception
	 */
	public static FileSystem getFileSystem() throws Exception {
		// core-site.xml,core-defautl.xml,hdfs-site.xml,hdfs-default.xml
	  // 读取配置文件信息
		Configuration conf = new Configuration();

		// 获取文件系统，传入配置文件参数
		FileSystem fileSystem = FileSystem.get(conf);

		// System.out.println(fileSystem);
		return fileSystem;
	}

	/**
	 * 读取数据
	 * 
	 * @param fileName
	 * @throws Exception
	 */
	public static void read(String fileName) throws Exception {

		// 调用获取文件系统静态方法
		FileSystem fileSystem = getFileSystem();

		// String fileName = "/user/input/mapreduce/wordcount/input/wc.input";
		// 设置路径
		Path readPath = new Path(fileName);

		// 打开文件
		FSDataInputStream inStream = fileSystem.open(readPath);

		try {
			// 读取文件  用Hadoop封装好的工具IOUtils 
			IOUtils.copyBytes(inStream, System.out, 4096, false);
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			// 关闭流
			IOUtils.closeStream(inStream);
		}
	}

	public static void main(String[] args) throws Exception {

		// String fileName = "/user/input/mapreduce/wordcount/input/wc.input";
		// read(fileName);
		FileSystem fileSystem = getFileSystem();

		// 写出的路径
		String putFileName = "/user/input/put-wc.inut";
		Path writePath = new Path(putFileName);

		// HDFS读出流
		FSDataOutputStream outStream = fileSystem.create(writePath);

		// 本地文件路径下的文件作为流的输入
		FileInputStream inStream = new FileInputStream(//
				new File("/opt/modules/hadoop-2.7.3/wc.input")//
		);

	
		try {
			//从输入流读到输出流
			IOUtils.copyBytes(inStream, outStream, 4096, false);
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			// 关闭流
			IOUtils.closeStream(inStream);
			IOUtils.closeStream(outStream);
		}
		}
}

```
HDFS的操作条理清晰，看上去是比较简单的，在此我就不多说了。
# Yarn
---


Yarn 数据操作系统（云操作系统）类似与Windows操作系统为各个软件提供服务 负责整个集群管理调度监控（为各种框架分配资源进行隔离Container 容器：将任务放入其中进行隔离）Hadoop 2.x 引入Yarn后等于是引入了操作系统，可以在此之上运行更多的计算框架如Storm....
>Yarn架构图

![](https://i.imgur.com/7l67Wcy.png)

ResourceManager作为全局资源管理器，整个集群只有一个，负责集群资源的统一管理和调度分配。
* 处理客户端请求
* 监控/启动ApplicationMaster
* 监控NodeManager
* 资源调度分配

Nodemanager会定时向RM发送各个节点资源的使用情况，以及各个container的运行状况。
* 处理RM的命令
* 处理AM的命令
* 单个节点上的资源管理和任务管理

Yarn的各种资源管理是交给Application master做的。

* 资源申请，分配任务
* 任务监控、容错
* 协调RM的资源，通过Nm监视容器的执行和资源的使用（CPU内存）

Container资源抽象，封装某节点上多维度的资源

* 对任务环境的抽象
* 描述一系列信息
* 任务运行资源
* 任务启动命令 

 ApplicationMaster 协调应用进程
 * 生命周期随着应用开始而开始结束而结束
 * 协调应用进程
 * 向RM申请资源，在对应的NodeManager启动Container来执行任务。
 * 监控Container状态。
 
 AM的启动过程室友RM完成的。当Job提交后，RM会在集群中任选一个Container，启动作为AM。
 
 ## Yarn工作流程  
 ---
 >资源调度有RM完成，资源隔离由NM（container）实现 

 
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181113181434685.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)

* 用户向YARN中提交应用程序，其中包括ApplicationMaster程序、启动ApplicationMaster命令、用户程序等 
* resourceManager为该应用程序分配第一个container，并与对应的nodeManager通信，要求它在这个container中启动应用程序的ApplicationMaster 
* ApplicationMaster首先向ResourceManager注册，这样用户可以直接通过ResourceManager查看应用程序的运行状态，然后它将为各个任务申请资源，并监控它的运行状态，直到运行结束，即重复4~7 
* ApplicationMaster采用轮询的方式通过RPC协议向resourceManager申请和领取资源 
* 一旦ApplicationMaster申请到资源后，便与对应的Nodemanager通信，要求它启动任务。 
* NodeManager为任务设置好运行环境(包括环境变量、JAR包、二进制程序等)后，将任务启动命令写到一个脚本中，并通过运行该脚本启动任务 
* 各个任务通过某个RPC协议向ApplicationMaster汇报自己的状态和进度，以让ApplicationMaster随时掌握各个任务的运行状态，从而可以在任务失败时重新启动任务。在应用程序运行过程中，用户可随时通过RPC向ApplicationMaster查询应用程序的当前运行状态 
* 应用程序运行完成后，ApplicationMaster向resourceManager注销并关闭自己 

 
## Yarn特殊配置
---


Yarn运行用户配置每个节点上可以用的物理内存资源

![](https://i.imgur.com/fZpB40z.png)

![](https://i.imgur.com/QkHRAEA.png)

 一般来说可以修改一下虚拟内存和虚拟CPU其他的默认。

虚拟CPU是为了弥补CPU配置的差异，将好的两个i7CPU虚拟为三个i5。

## Yarn生态系统
---

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181113181743653.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)

## MapReduce编程
---
>input  -> map  -> reduce  -> output 在整个过程中数据传输的流通格式是以键值对的方式 及`<key,value>`。

		input；	`<key,vlaue>` 
		
		map (MapTask)
		
		output: `<key,value>` 
		
		input: `<key,value>` 
		
		reduce (ReduceTask)
		
		output: `<key,value>`

## MapReduce过程
---
一.  

* 初始偏移量0
* 偏移量11得到 

hadoop yarn				    ->  <0,hadoop yarn>  
hadoop mapreduce			->  <11,hadoop mapreduce> 

二. 

	<0,hadoop yarn> 
	
	hadoop yarn -> split -> hadoop,yarn
	
	<hadoop,1> 
	<yarn,1>

三. 

		map ->  shuffle  -> reduce 

* 分组、排序 

* 将相同key的value合并在一起，放到一个集合中。

		<hadoop,1> 
						->		<hadoop,list(1,1)>
		<hadoop,1>

注意：如果用HDFS上文件作为输入，会首先使用FileInputFormat对文件切分形成一个分片（逻辑上的分片，不在磁盘存储），每个分片作为一个map输入，然后才是解析为键值对。

##### Word Count MR代码

```
public class WordCountMapReduce extends Configured implements Tool {

	// step 1: Map Class
	/**
	 * 
	 * public class Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT> <0,hadoop yarn>
	 */
	public static class WordCountMapper extends
			Mapper<LongWritable, Text, Text, IntWritable> {
		private Text mapOutputKey = new Text();
		private final static IntWritable mapOuputValue = new IntWritable(1);

		@Override
		public void map(LongWritable key, Text value, Context context)
				throws IOException, InterruptedException {
			// line value
			String lineValue = value.toString();

			// split (hadoop yarn)
			// String[] strs = lineValue.split(" "); neicunshoubuliao
			StringTokenizer stringTokenizer = new StringTokenizer(lineValue);

			// iterator
			while (stringTokenizer.hasMoreTokens()) {
				// get word value
				String wordValue = stringTokenizer.nextToken();
				// set value
				mapOutputKey.set(wordValue);
				;
				// output (hadoop,1)
				context.write(mapOutputKey, mapOuputValue);
			}
		}

	}

	// step 2: Reduce Class
	/**
	 * 
	 * public class Reducer<KEYIN,VALUEIN,KEYOUT,VALUEOUT>
	 */
	public static class WordCountReducer extends
			Reducer<Text, IntWritable, Text, IntWritable> {

		private IntWritable outputValue = new IntWritable();

		@Override
		public void reduce(Text key, Iterable<IntWritable> values,
				Context context) throws IOException, InterruptedException {
			// sum tmp
			int sum = 0;
			// iterator
			for (IntWritable value : values) {
				// total
				sum += value.get();
			}
			// set value
			outputValue.set(sum);

			// output
			context.write(key, outputValue);
		}

	}

	// step 3: Driver ,component job
	public int run(String[] args) throws Exception {
		// 1: get confifuration new Configuration
		// extend Configured
		Configuration configuration = getConf();

		// 2: create Job
		Job job = Job.getInstance(configuration, //
				this.getClass().getSimpleName());
		// run jar
		job.setJarByClass(this.getClass());

		// 3: set job
		// input -> map -> reduce -> output
		// 3.1: input
		Path inPath = new Path(args[0]);
		FileInputFormat.addInputPath(job, inPath);

		// 3.2: map
		job.setMapperClass(WordCountMapper.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);

		// 3.3: reduce
		job.setReducerClass(WordCountReducer.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);

		// 3.4: output
		Path outPath = new Path(args[1]);
		FileOutputFormat.setOutputPath(job, outPath);

		// 4: submit job true printf log
		boolean isSuccess = job.waitForCompletion(true);

		return isSuccess ? 0 : 1;

	}

	// step 4: run program
	public static void main(String[] args) throws Exception {
		// 1: get confifuration
		Configuration configuration = new Configuration();

		// int status = new WordCountMapReduce().run(args);
		// youhua
		int status = ToolRunner.run(configuration,//
				new WordCountMapReduce(),//
				args);

		System.exit(status);
	}

}
```

对这个代码简单分析一下：
>主要分为四块，也算是一个很死板的又很标准的写法
1. Mapper 
		map主要处理输入的字符串，将其分割为单个字符，最终的输出格式<hadoop ,1>。
2. Reducer
		reduce主要将key相同的求和计算单词出现的次数。
3. Driver
		run主要处理配置文件的获取，Job的创建，路径的设置，Map、reduce的组装和Job的提交
4. Main

## MapReduce 工作机制
--- 
![在这里插入图片描述](https://img-blog.csdnimg.cn/2018111318390868.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)


## Shuffle过程 
---
![](https://i.imgur.com/HyObpwA.jpg) 

 	* step 1:
 		input
 			InputFormat
 				* 读取数据
 				* 转换成<key,value>
 			FileInputFormat
 				* TextInputFormat
 	* step 2：
 		map
 			ModuleMapper
 				map(KEYIN,VALUEIN,KEYOUT,VALUEOUT)
 			* 默认情况下
 				KEYIN: LongWritable
 				VUALEIN: TEXT
 	* step 3:
 		shuffle
 			* process
 				* map,output<key,value>
 					* memory 默认100M
 					* spill，溢写到磁盘中，可能有很多文件
 						* 分区parttition
 						* 排序sort
 				* 很多小文件，spill
 					* 合并，merge
 					* 排序
 					大文件 -> Map Task 运行的机器的本地磁盘
 				---------------------------------------
 				* copy
 				Reduce Task，回到Map  Task运行的机器上，拷贝要处理的数据
 				* 合并，merge，排序
 				* 分组group
 					将相同Key 的value放在一起
 		总的来说
 			* 分区
 				partitioner（map输出的那些数据分配给那些reduce处理）
 			* 排序
 				sort（）
 			* copy --   用户无法干涉
 				拷贝
 			* 分组
 				group
 			* 可设置
	 			* 压缩
	 				compress - 可设置
	 			* combiner(合并)
	 				Map Task端的Reduce
                * 可通过配置文件配置
                mapred-site.xml
                mapreduce.map.output.compress	false	Should the outputs of the maps be compressed before being sent across the network. Uses SequenceFile compression.
                mapreduce.map.output.compress.codec	org.apache.hadoop.io.compress.DefaultCodec
               
              * 可以通过代码解决
                configuration.set("mapreduce.map.output.compress", "true");
        		configuration.set("mapreduce.map.output.compress.codec", "org.apache.hadoop.io.compress.SnappyCodec");

 	* step 4：
 		reduce
 			reduce(KEYIN,VALUEIN,KEYOUT,VALUEOUT)
 			map输出<key,value>数据类型与reduce输入<key,value>一致
 	* step 5:
 		output
 			OutputFormat
 		FileOutputFormat
 			TextOutputFormat
 			每个<key,value>对，输出一行，key与value中间分隔符为\t，默认调用Key和Value的toString()方法
<font color="red">Shuffle过程是贯穿于map和reduce两个过程的</font>
##### Shuffle过程的基本要求：

　　1.完整地从map task端拉取数据到reduce task端

　　2.在拉取数据的过程中，尽可能地减少网络资源的消耗

　　3.尽可能地减少磁盘IO对task执行效率的影响

那么，Shuffle的设计目的就要满足以下条件：

　　1.保证拉取数据的完整性

　　2.尽可能地减少拉取数据的数据量

　　3.尽可能地使用节点的内存而不是磁盘
　　
## MapReduce调优
----
* 设置reduce个数，默认是1，一个多累啊，自己测试看什么情况最优秀（必须要做）
	* 通过程序设置 ：
	* job.setNumReduceTasks(2);
	* 修改配置文件：
	* mapreduce.job.reduces 
     1 
     

什么情况最优呢，条件reduce数量，调到一个运行速度最好的数量。 

* Map Task 输出压缩
* Combine合并，在中间进行了一次合并
![在这里插入图片描述](https://img-blog.csdnimg.cn/2018111318204336.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)

*Shuffle  参数调节 

>调节shuffle阶段的排序因子，map输出在本地文件，小文件合并的触发条件


*  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181113183553748.png)

>排序大小，也就是设置内存缓冲区的大小,可以解决不需要放到内存的情况，如前者改为1M后者0.8表示百分之80就开始网本地写

* ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181113183216464.png)

* ![在这里插入图片描述](https://img-blog.csdnimg.cn/2018111318323096.png)

> 虚拟cpu核数条件，满足如聚类分类算法的硬件需求 
> 
![在这里插入图片描述\](https://img-blog.csdnimg.cn/20181113183307695.png)
!\[](https://img-blog.csdnimg.cn/2018111318332978.png)

---
这些配置官网一大堆，自己琢磨琢磨官方文档吧。

##  总结
----
对于Hadoop编程这块，MapReduce算是最重要的吧，感觉除了这个其他的东西还是很简单的。所以呢我感觉MR这个程序强撸上10遍才能满足我。由于时间关系，这篇文章还存在一些不足（这会实在是静不下心❤），后面等我舒服了在花点时间将里面的部分内容进行一个梳理主要是启动过程和MR程序的思想。大体上还是没什么问题的。🙃
