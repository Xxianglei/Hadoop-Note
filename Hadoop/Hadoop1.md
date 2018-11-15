> 距离第一次接触大数据已经快一年了，中间参加了为期4个月左右的中国软件杯，拿了个国家三等奖，还算是为我时间的牺牲得到了一点回报。暑假到前半个月，一直在学JavaWeb，接触了后台之后对很多知识有了更深入的理解，同时也对大数据的应用有了更加清晰的认识。目前准备花一个月的时间将大数据相关的知识总结一下捋一下，然后再做一个推荐系统或者合适的大数据开发项目。最后就开始苦逼的备战面试复习了。
> >如果你想入门大数据，对大数据的生态圈有一个比较全面的了解，我相信我的文章应该对你有所帮助，下面就带你走进大数据（臭屌丝）的世界。
 ---

 Hadoop吹牛逼的话我就不多介绍了，文章核心还是放在对于Hadoop的运用以及理解上面。
 ## Apache Hadoop
 ---
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111185444304.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)
 ---
> Hadoop是一个由Apache基金会所开发的分布式系统基础架构。
用户可以在不了解分布式底层细节的情况下，开发分布式程序。充分利用集群的威力进行高速运算和存储。
[1]  Hadoop实现了一个分布式文件系统（Hadoop Distributed File System），简称HDFS。HDFS有高容错性的特点，并且设计用来部署在低廉的（low-cost）硬件上；而且它提供高吞吐量（high throughput）来访问应用程序的数据，适合那些有着超大数据集（large data set）的应用程序。HDFS放宽了（relax）POSIX的要求，可以以流的形式访问（streaming access）文件系统中的数据。
Hadoop的框架最核心的设计就是：HDFS和MapReduce。HDFS为海量的数据提供了存储，而MapReduce则为海量的数据提供了计算。 -----百度百科

说的挺高级挺牛逼的，其实的确也牛逼😂。不得不佩服老外咋能搞出这玩意。Hadoop是05年出来的，诞生于谷歌三大论文GFS MapReduce BigTable ->Hdfs MapReduce Hbase。总之： Hadoop可以简单认识为可扩展的分布式计算（大数据存储分析网络数据）

## Hadoop的发展
---
>Hadoop目前经历了1.x、2.x、到现在的3.x我主要说一下1.x和2.x的区别吧，这两个版本的区别算是比较明显的。

##### Hadoop1.0和Hadoop2.0版本之间还是存在很大的差异的。
整体上，1.0版本是由分布式存储系统HDFS和分布式计算框架MapReduce组成，其中HDFS由一个NameNode和多个DataNode组成，MapReduce由一个JobTracker和多个TaskTracker组成。

2.0版本中，引入HDFS Fedration，它让多个NameNode分管不同的目录进而实现访问隔离和横向拓展，同时解决了NameNode单点故障问题；引入资源管理框架Yarn，可以为各类应用程序进行资源管理和调度。

## Hadoop的案例
---
* 阿里 最开始 用的Hadoop-0.19.0 开始非常早
* 大数据经典案例  《啤酒与尿布》 -->百度去吧，懒得打字！😂
## Hadoop企业搭建核心架构
>此图也是我大数据复习阶段的核心内容。

![](https://i.imgur.com/CugfLXF.png)

对于这里面的部分框架我想做个简单的说明，也是我自己的理解。

* Hive 大数据仓库 方便开发人员对数据处理简化写MR的过程 底层任是MR
* Sqoop（SQLto Hadoop 数据转换） oozie（任务调度） flume（文件收集 实时搜集） hue（web工具） 大数据辅助框架
* Hbase 分布式数据库（一个表几十亿上百亿 秒级别查询）  就是一个数据库 做到实时查询  可以说是一个DBA的角色  
* spark 数据处理框架 运行在yarn上处理Hdfs 不可替代Hadoop 必须看源码 社区特别火   核心RDD 

## Hadoop核心模块
---
* Hadoop Common
为其他hadoop模块提供基础设施
* Hadoop Yarn
任务调度与资源管理框架
* Hadoop Hdfs
高可靠高吞吐量的分布式文件系统
* Hadoop Mapreduce
分布式离线并行计算框架

下面我按此顺序一一介绍：
1. Common是为其他hadoop模块提供基础设施，一般情况没人碰他吧，反正我没碰过。
2.  Yarn 数据操作系统（云操作系统）类似与Windows操作系统为各个软件提供服务 负责整个集群管理调度监控（为各种框架分配资源进行隔离Container 容器：将任务放入其中进行隔离）
>架构图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111194440133.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)
* ResourceManager:处理客户端请求；启动/监控ApplicationMaster;监控NodeManager;资源分配与调度
* ApplicationMaster:数据切分；为应用程序申请资源，并分配给内部任务；任务监控与容错
* NodeManager:单个节点上的资源管理；处理来自ResourceManager的命令；处理来自ApplicationMaster的命令
* Container:对任务运行环境的抽象，封装了CPU、内存等多维资源以及环境变量、启动命令等任务运行相关的信息
3. HDFS 官方一点的说法是高可靠高吞吐量的分布式文件系统。以流式数据访问模式来存储超大文件，运行于商用硬件集群上，是管理网络中跨多台计算机存储的文件系统。简单的理解就是一个超级大硬盘，百度云盘，360云盘估计也是如此。 
HDFS不适合用在：要求低时间延迟数据访问的应用，存储大量的小文件，多用户写入，任意修改文件。
值得一提的是：在HDFS上数据是以block的方式存储的
HDFS数据块：HDFS上的文件被划分为块大小的多个分块，作为独立的存储单元，称为数据块。默认64M.
>如：128 M
200M
blk_0001：128M
blk_0002 : 72M

##### 使用数据块的好处是：
* 一个文件的大小可以大于网络中任意一个磁盘的容量。文件的所有块不需要存储在同一个磁盘上，因此它们可以利用集群上的任意一个磁盘进行存储。
* 简化了存储子系统的设计，将存储子系统控制单元设置为块，可简化存储管理，同时元数据就不需要和块一同存储，用一个单独的系统就可以管理这些块的元数据。
* 数据块适合用于数据备份进而提供数据容错能力和提高可用性。
>上架构图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111193209571.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)
##### 在这里主要注意的是 
1. HDFS文件系统由三部分组成：Namenode（是存在内存当中的，本地磁盘也有他的备份
fsimage：镜像文件edites：编辑日志  ） Datanode Secondary Namenode（用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照）。
2. 在HDFS上，文件是以块保存的，默认存入三个副本。默认是在HDFS客户端上放第一个副本，第二个副本放在与第一个不同且随机的另外的选择的机架中的节点上。第三个副本与第二个副本在同一个机架但是不同节点上。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181113175119582.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)
3. 数据的存取（数据流不经过namenode）仅仅通过Datanode，Namenode只是保存的一些元数据。

</br>
4. Mapreduce离线计算框架 

>Mapreduce的核心思想是“分而治之”，学过分治法的应该很熟悉，比如他将一个10G的数据量分配（map）给10个CPU计算，最后将结果收集（reduce）到一起，这就是Mapreduce工作的基本流程。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111195350115.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)

##### Mapreduce原理
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111195603372.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)
##### Map任务处理

　　1.1 读取HDFS中的文件。每一行解析成一个<k,v>。每一个键值对调用一次map函数。        

　　1.2 覆盖map()，接收1.1产生的<k,v>，进行处理，转换为新的<k,v>输出。　

　　1.3 对1.2输出的<k,v>进行分区。默认分为一个区.

　　1.4 对不同分区中的数据进行排序（按照k）、分组。

　　1.5对分组后的数据进行归约。

##### Reduce任务处理

　　2.1 多个map任务的输出，按照不同的分区，通过网络copy到不同的reduce节点上。

　　2.2 对多个map的输出进行合并、排序。覆盖reduce函数，接收的是分组后的数据，实现自己的业务逻辑，处理后，产生新的<k,v>输出。

　　2.3 对reduce输出的<k,v>写到HDFS中。

-----
基本内容介绍完了，接下来说说Hadoop到底怎么玩?Hadoop有三种运行方式，单机模式，伪分布式模式，分布式模式，我们来一一学习。
## Hadoop安装与配置
>http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html 这个网站是权威我基本上也是看着这个来的。
##### Hadoop的运行需要java环境
1. 安装JDK，照着操作行了，没什么难点。
*  rpm -qa|grep java 
xxx yyy zzz为你要卸载的插件，插件之间以空格隔开
rpm -e --nodeps xxx yyy zzz
* rpm -ivh jdk-8u172-linux-x64.rpm 
*  vim /etc/profile
export JAVA_HOME=/usr/java/default
export PATH=$JAVA_HOME/bin:$PATH
* source /etc/profile
* echo $JAVA_HOME
##### 上传Hadoop 解压安装
##### 直接开始玩单节点
* $ mkdir input
* $ cp etc/hadoop/*.xml input
* $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep input output 'dfs[a-z.]+'
* $ cat output/*
*  bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount wcinput wcoutput
（经典案例  统计单词个数）
##### 伪分布式
* 核心配置文件
![](https://i.imgur.com/ZU5EPOJ.png)
这些都是需要我们修改的文件，我们一个一个来操作。
* core-site.xml 
 ![](https://i.imgur.com/NBWT4eM.png)
前者配置 dfs文件系统 指定了namenode运行的节点
后者配置文件临时目录 覆盖到默认的目录
* hdfs-site.xml
 ![](https://i.imgur.com/p3CHWQ8.png)
配置文件备份数  在此为一份  因为目前是伪分布式  只有一个节点
* yarn-site.xml
![](https://i.imgur.com/VuRX3ww.png)
前者配置 运行MapReduce 
后者配置 yarn主机 
* 修改mapred-site.xml

```<configuration>
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
设置MapReduce运行在yarn上
* 格式化hdfs文件系统 
bin/hdfs namenode -format
注意：格式化确认输入Y
* 启动Hadoop
sbin/hadoop-daemon.sh start namenode 
sbin/hadoop-daemon.sh start datanode
* 启动Yarn
sbin/yarn-daemon.sh start resourcemanager
sbin/yarn-daemon.sh start nodemanager

### 注意
1. 最好修改三个配置文件脚本   mapred-env.sh yarn-env.sh  hadoop-env.sh
* 配置好JDK路径 /usr/java/jdk1.8.0_91
2. There are 0 datanode(s) running and no node(s) are excluded in this operation.

 出现上述问题可能是格式化两次hadoop，导致没有datanode

解决办法是：找到hadoop安装目录下 hadoop-2.4.1/data/dfs/data里面的current文件夹删除

然后从新执行一下 hadoop namenode -format

再使用start-dfs.sh和start-yarn.sh 重启一下hadoop

用jps命令看一下就可以看见datanode已经启动了
### 安装常见问题解决
* bin/hdfs namenode -format 

原因：core-site.xml 文件部分内容写错  或者主机名与ip的映射写错

* namenode启动出错 

原因：自查日志文件（logs）
## 结束
到此伪分布式的基础配置基本上就结束了，对于分布式：多搞几台机器  修改DataNode个数  配置slave（指定DataNode）文件  配置hosts文件 ssh  OK  OK不？  反正我不想打字写分布式配置了。

----
## 其他配置
1. Yarn上的历史服务查看（日志聚集）
修改yarn-site.xml
![](https://i.imgur.com/vedphYN.png)
前者  启动日志聚集 
后者  设计报废时间

* 启动yarn yarn historyserver 再重跑任务 ，最后访问查看。
* 在sbin目录下 ./mr-jobhistory-daemon.sh start historyserver
* 在sbin目录下 ./mr-jobhistory-daemon.sh stop historyserver
2.  hdfs垃圾回收
hdfs-site.xml 
![](https://i.imgur.com/d8NODuH.png)
(写成具体值否则出错)
在垃圾箱中可以恢复文件
3. Secondarynamenode
![](https://i.imgur.com/xN1SeN9.png)
  ![](https://i.imgur.com/iHm2cF0.png)

3. 分开启动
![](https://i.imgur.com/yyzS8D9.png)
4.  一键启动
start-all.sh（不推荐） 

5. 需要配置ssh秘钥  
* cd ~
* ssh-keygen -t rsa
* ssh-copy-id master
* 测试
* ssh master
* exit



###  友情提醒
<font color="red"> 我的配置中用的是IP 没有写主机名master,访问路径啥的可能会出现问题，你参照着官网的内容，别一股脑照着全般了。</font>

----
## 总结
---
关于Hadoop的学习，这仅仅是个入门，Hadoop核心的内容在后面几篇文章里面，此文主要给没有基础的朋友了解Hadoop以及简单使用Hadoop。说实话，这篇文章排版有一点点的乱，后面有时间在一点点修订补充吧。虽然时间挺紧张的，但是我感觉花个一俩小时总结也是挺主要的，这点时间成本还是挺值得。🤔
