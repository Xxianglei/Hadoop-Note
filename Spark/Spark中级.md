## Spark RDD

>弹性的分布式数据集，可以理解为一个Java类，里面放的都是数据。RDD代表一个不可变的对元素分区的集合。并且RDD可以被并行计算。

![](https://i.imgur.com/ENizg3K.png)
**Spark RDD特性**

* 分为若干个区
* 每个分片用一个函数计算
* RDD直接是一个依赖关系
* 对于K-V的RDD可指定一个分区，告诉它如何分片
* 要运行的计算/执行最好在哪几个机器上运行

RDD lineage 有生命线 ，它保存了如何转换得来的信息。

处理RDD split进行计算时，split数据在哪里，我们尽量在那台机器上进行计算，移动计算，不是移动数据。

**RDD 操作**

RDD的创建

* Parallelized Collections

![](https://i.imgur.com/I5GUQCV.png)

* External Datasets

![](https://i.imgur.com/NdTfMHy.png)

![](https://i.imgur.com/qnczj4V.png)

**Transformation**
![](https://i.imgur.com/VS0bIBC.png)
**Action**
![](https://i.imgur.com/aeOm4Ep.png)

**RDD的依赖**

![](https://i.imgur.com/kRJecyR.png)

* 窄依赖



![](https://i.imgur.com/jn8ZvCZ.png)


* 宽依赖



![](https://i.imgur.com/35PrWCh.png)

**RDD Shuffle**



![](https://i.imgur.com/bA8Zkv7.png)

>The shuffle is Spark’s mechanism for re-distributing data 

那些操作会引起Shuffle？

* 具有重新调整分区操作 ，eg：repartition,coalesce
* *BeyKey eg: groupByKey ,reduceByKey
* 关联操作 eg： join ，cogroup


**Spark 内核**

![](https://i.imgur.com/ZW9eQPo.png)

![](https://i.imgur.com/4uuaw8J.png)

**DAG调度**

>阶段的划分是按是否有shuffle来判断的。


![](https://i.imgur.com/zrqKJBB.png)

*  接收用户提交的job
*  构建 Stage，记录哪个 RDD 或者 Stage 输出被物化
*  重新提交 shuffle 输出丢失的 stage
*  将 Taskset 传给底层调度器

**Task 调度**

![](https://i.imgur.com/zKQzO6I.png)

* 提交 taskset( 一组 task) 到集群运行并监控
* 为每一个 TaskSet 构建一个 TaskSetManager 实例管理这个 TaskSet
的生命周期
* 数据本地性决定每个 Task 最佳位置 (process-local, node-local, 
rack-local and then any) 
* 推测执行，碰到 straggle 任务需要放到别的节点上重试出现 shuffle 
输出 lost 要报告 fetch failed 错误

**Partition和Task**

![](https://i.imgur.com/A8UrtK9.png)

* Task是Executor中的执行单元
* Task处理数据常见的两个来源：外部存储以及shuffle数据
* Task可以运行在集群中的任意一个节点上
* 为了容错，会将shuffle输出写到磁盘或者内存中

**案例**

* 排序
* Top key

	
		## WordCount
		val rdd = sc.textFile("hdfs://master:8020/user/mapreduce/wordcount/input/wc.input")
		val wordcount = rdd.flatMap(_.split(" ")).map((_,1)).reduceByKey(_ + _)
		wordcount.collect
		wordcount.saveAsTextFile("hdfs://master:8020/user/mapreduce/wordcount/sparkOutput99")
		
		======result
		(hive,2)
		(mapreduce,1)
		(mapreduce2,1)
		(hadoop,5)
		(hdfs,1)
		发现：
			Spark 运行WordCoun程序，并没有像MapReduce程序那样，对Key进行排序。
		## Key Sort
		wordcount.sortByKey().collect    ## 默认情况是 升序
		wordcount.sortByKey(true).collect	
		wordcount.sortByKey(false).collect		
			
		===============================================================	
		------result
		(hive,2)
		(mapreduce,1)
		(mapreduce2,1)
		(hadoop,5)
		(hdfs,1)	
		需求：
			按照value值进行降序
		## Value Sort
		wordcount.map(x => (x._2,x._1)).sortByKey(false).collect
		wordcount.map(x => (x._2,x._1)).sortByKey(false).map(x => (x._2,x._1)).collect
			
		## Top N
		wordcount.map(x => (x._2,x._1)).sortByKey(false).map(x => (x._2,x._1)).take(3)

