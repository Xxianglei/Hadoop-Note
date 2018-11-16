>沿着前面的内容，接下来的文章就是关于Hive从基础的搭建到高级应用的知识。鄙人在大二初学Hive的时候，只是觉得Hive和Mysql差不多，但是对于Hive为什么叫做数据仓库，以及Hive的UDF编程我并没有太多思考。所以啊，为了混口饭吃迟早还是要还的。所幸目前算是明白了数据仓库的含义同时对Hive的架构、使用有了全新的认识。

## Hive是什么
----- 
先看一看官方的解释
> hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。 其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析。----百度百科

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181116201148805.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)

**Hive简介** 

>Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射
成一张表，并提供类SQL查询功能。也是由Facebook开源用于解决海量结构化日志的数据统计。 

![](https://i.imgur.com/tIUvVFX.png)
构建在Hadoop之上的数据仓库，其数据也存储在HDFS之上(  /user/hive/warehouse )

    <property>
		<name>hive.metastore.warehouse.dir</name>
		<value>/user/hive/warehouse</value>
	</property> 
	
**"数据仓库工具"的含义**:
* 我们先看官方对于数据仓库的定义
>数据仓库，英文名称Data Warehouse，简写为DW。数据仓库顾名思义，是一个很大的数据存储集合，出于企业的分析性报告和决策支持目的而创建，对多样的业务数据进行筛选与整合。它为企业提供一定的BI（商业智能）能力，指导业务流程改进、监视时间、成本、质量以及控制。数据仓库的输入方是各种各样的数据源，最终的输出用于企业的数据分析、数据挖掘、数据报表等方向。

用一句话来讲Hive就是对数据处理或管理的工具。为什么这么说呢？

我们都知道Hive对于数据的处理是限于对数据的查询、统计、Hive是没有事务机制的。由于 Hive 是针对数据仓库应用设计的，而数据仓库的内容是读多写少的。因此，Hive 中不支持对数据的改写和添加，所有的数据都是在加载的时候中确定好的。由此可见数据仓库工具的含义了。

**Hive的特点**  

* 处理的数据存储在HDFS
* 分析数据底层的实现MapReduce
* 执行程序运行的YARN
* 灵活性扩展性比较好，支持UDF，自定义存储格式
* 适合离线数据处理
* 使用HQL作为查询接口；

Hive运行本质：**将HQL转化成MapReduce程序** 

**Hive架构** 

![](https://i.imgur.com/nzpHOcL.png)

这里CLI和JDBC作为客户端，在写完HQL后交给Driver以及下部分和MetaStore执行，负责将其转为MR来执行。

*SQLParser->QueryOptimizer->Physical Plan->Execution*

元数据: Metastore 

*  元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/
分区字段、表的类型（是否是外部表）、表的数据所在目录等；
* 默认存储在自带的derby数据库中，推荐使用采用MySQL存储Metastore；




## 为什么使用Hive
----

MapReduce的不便性

*  MapReduce is hard to program
* 编程方式死板八股文格式编程，三大部分
* No Schema, lack of query lanaguages, eg. SQL
	*  数据分析，针对DBA、SQL语句，如何对数据分析
	*  MapReduce编程成本高
	*  FaceBook 实现并开源Hive 

总而言之，使用Hive基本上可以不再手撸MapReduce代码了（又臭又长），其次HQL和SQL很类似对于一般人来讲上手也快。同时也降低了学习成本。


## 动手搭建Hive环境 
---
>介绍完了Hive，我们来动手安装配置一个Hive。这里有个“噩梦”->MySQL的安装，我相信初学Hive你一定懂我。不过这次有了‘’三年‘’的开发经验，基本上是‘“逢山开路遇水架桥”式的成功安装。我这块也做了一个特殊的整理。也帮助我们数据库工作室的学弟们不死在这个坑里。

1. 记着启动Hadoop，Yarn(我把RM改到了master方便启动)。
2. 解压文件 export是方便你随地都可以开启Hive，你愿意弄就弄吧。
![](https://i.imgur.com/oSCgX7d.png)
3. 修改hive-env.sh (主要是设置下面两项、conf目录最后加个conf，我路径有问题后来改了)
![](https://i.imgur.com/6dOtUlX.png)

4. 在Hdfs文件系统上创建两个路径（他说的must，懂吧！）
![](https://i.imgur.com/F9zwzxJ.png)

简单的配置就结束了，来来来，先用一用看看。

1. show databases ;
2. use default;
3. show tables ;
4. create table student(id int, name string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
5. load data local inpath '/opt/datas/student.txt'into table student ;
6. select * from student ;
7. select id from student ;

你可以尝试另起一个窗口在开启Hive，这时你会发现，用不了崩了。原因在下面。
注意：Hive默认是将元数据放在derby（内存数据库，只供一个人使用为了满足多人开发，我们需要将使用MySQL）所以多人开启自然崩

## MySQL安装
>derby多人使用满足不了，那么我们换个路子用MySQL解决，在此你也可以用Oracle等

元数据metastore支持使用哪些数据库，找了一张图。

![](https://i.imgur.com/x7Xgdg0.png)

**Mysql安装细节**

* 检查是否已经装过mysql（必须）
	* rpm -qa|grep mysql 有就卸载没有就跳过
* 安装Mysql，为rpm包添加权限
	* chmod u+x ./*  添加执行权限
![](https://i.imgur.com/sEW8uG8.png)
* 安装server (这里多半会出现问题)
	* rpm -ivh MySQL-server-5.6.24-1.el6.x86_64.rpm
  
* 安装client 
	* rpm -ivh MySQL-client-5.6.24-1.el6.x86_64.rpm

* 进入MySQL mysql -uroot -proot

* 设置开机启动chkconfig mysql on
* 设置登录信息
![](https://i.imgur.com/ZTYJpTb.png)

flush privileges;刷新一下配置

设置任何机器可以连接，主要是Windows下连接。

你要是顺利的走到了这里，你基本上就可以舒口气了。但是很多人会在安装Server的时候崩溃，甚至放弃！☠

**MySQL安装常见错误梳理** 

常见错误一：

	安装成功但是登录失败
>mysql -uroot -p
输入密码出现Access denied for user 'root'@'localhost'(using password: YES)错误。每次安装Mysql就是噩梦，今天总算找了一个比较好的解决方案。


	解决方案：
* 关闭Mysql service mysqld stop
* 跳过密码验证 /usr/bin/mysqld_safe --skip-grant-tables（卡住了就另开一个窗口）
* 无密码登录 mysql -u root
* 修改密码
	* use mysql
	* update user set password=PASSWORD("你的密码") where User = 'root';
* 重启 mysql service mysqld restart
* 正常进入mysql -uroot -proot

常见错误二:

	安装Server发生冲突如下图
![](https://i.imgur.com/sEtwMZ7.png)  

	解决方案：
 * 卸载它 rpm -e --nodpes xxxx
*  然后重新安装 Server
 * 如果出现如下异常（多半会）：
 *  FATAL ERROR: please install the following Perl     modules before executing /usr/bin/mysql_install_db: 
 Data::Dumper 

* 解决方案：# yum install -y perl-Module-Install.noarch
* 再卸载安装失败的Mysql-server（先rpm -qa|grep Mysql）
* 重新安装 Mysql-server
* 注意查看随机密码 cat /root/.mysql_secret 
 ld6z5j8P

这套方案基本上是我最快的解决办法，如果你一筹莫展了你可以尝试一下，如果你要是有更好的办法，欢迎交流。

## Hive元数据配置
-----
>安好了MySQL，下面就是配置Hive的元数据。在MySQL里面自动创建metastore，存放元数据。

修改conf目录下hive-site.xml的配置的核心如下。（不要问我hive-site.xml在哪，我相信你没有那么菜。）
![](https://i.imgur.com/HblI2OO.png)

1.修改配置文件
    
    
	<configuration>
	<property>
	  <name>javax.jdo.option.ConnectionURL</name>
	  <value>jdbc:mysql://master:3306/metastore?createDatabaseIfNotExist=true</value>
	  <description>JDBC connect string for a JDBC metastore</description>
	</property>                   设置Mysql连接地址（metastore）如果不存在就创建这个数据库。
	
	<property>
	  <name>javax.jdo.option.ConnectionDriverName</name>
	  <value>com.mysql.jdbc.Driver</value>
	  <description>Driver class name for a JDBC metastore</description>
	</property>                  设置连接驱动 （别忘了把JDBC的jar包导入到lib目录）  
	
	<property>
	  <name>javax.jdo.option.ConnectionUserName</name>
	  <value>root</value>
	  <description>username to use against metastore database</description>
	</property>                  设置登录名
	
	<property>
	  <name>javax.jdo.option.ConnectionPassword</name>
	  <value>root</value>
	  <description>password to use against metastore database</description>
	</property>                设置登录密码

	</configuration>

2. 拷贝mysql驱动jar包，到Hive安装目录的lib下 

	$ cp mysql-connector-java-5.1.27-bin.jar /opt/modules/hive-0.13.1/lib/

4. 启动hive


现在是配置的hive metastore Mysql 与我们hive安装在同一台机器上

##### 注意：
* hive-env.sh中conf的路径不要写错了，否则默认加载会一直使用derby，你配置的mysql失效（我就弄错了）
* jdbc的jar包需要导入lib下，否则报错（一定不要忘）

## Hive日志文件配置
-----
>为了方便查看在Hive的使用中出现错误时的日志信息，对于日志文件存放路径的配置是很有必要的。

1. 重命名 hive-log4j.properties （conf目录下）
2. 修改设置log文件存放路径，如下图

![](https://i.imgur.com/EmBgH8k.png)
运行后查看是否正常生成log文件
![](https://i.imgur.com/2tu99zq.png)

同时如果你不想进行全局设置的话，你可以设置本次会话生效（在启动时），日志可以在hive界面查看。
![](https://i.imgur.com/S1dLTQg.png)
>下面这一步在开发环境也是一个很有帮助的配置，可以方便你查看目前使用的数据库以及查询时数据字段的描述

在cli命令行上显示当前数据库，以及查询表的行头信息 

	conf/hive-site.xml
		<property>
			<name>hive.cli.print.header</name>
			<value>true</value>
			<description>Whether to print the names of the columns in query output.</description>
		</property>

		<property>
			<name>hive.cli.print.current.db</name>
			<value>true</value>
			<description>Whether to include the current database in the Hive prompt.</description>
		</property>

## Hive常用帮助
----
![](https://i.imgur.com/PMrs1DS.png) 

命令行输入	bin/hive -help 调出Hive操作的帮助信息。看着很多但是不需要去背，在需要用的时候查一下就行了。用多了自然就记住了。

	usage: hive
	 -d,--define <key=value>          Variable subsitution to apply to hive
	                                  commands. e.g. -d A=B or --define A=B
	    --database <databasename>     Specify the database to use
	 -e <quoted-query-string>         SQL from command line
	 -f <filename>                    SQL from files
	 -H,--help                        Print help information
	 -h <hostname>                    connecting to Hive Server on remote host
	    --hiveconf <property=value>   Use value for given property
	    --hivevar <key=value>         Variable subsitution to apply to hive
	                                  commands. e.g. --hivevar A=B
	 -i <filename>                    Initialization SQL file
	 -p <port>                        connecting to Hive Server on port number
	 -S,--silent                      Silent mode in interactive shell
	 -v,--verbose                     Verbose mode (echo executed SQL to the
	                                 console)


*  bin/hive -e </quoted-query-string/> 
 
	eg:不进入Hive命令运行HQL语句
		bin/hive -e "select * from db_hive.student ;"

* bin/hive -f </filename/>
	  eg: 批量运行SQL脚本。运行顺序是按次序运行的，在实际开发中可以方便HQL语句的管理。
	
		$ touch hivef.sql
			select * from db_hive.student ;
		$ bin/hive -f /opt/datas/hivef.sql 
		$ bin/hive -f /opt/datas/hivef.sql > /opt/datas/hivef-res.txt

* bin/hive -i </filename/> 
   与用户udf相互使用

* 在hive cli命令窗口中如何查看hdfs文件系统 

	hive (default)> dfs -ls / ;  

* 在hive cli命令窗口中如何查看本地文件系统 

	hive (default)> !ls /opt/datas ； 

  ![](https://i.imgur.com/iR1y3ZH.png) 

-----
Hive的初体验到此结束！
## 我有个疑问：
>**Hive或者Mysql中exit 和quit有区别吗？各位路过的大佬帮忙解释一下。**
## 总结
---
在Hive的配置里面尤其需要注意的就是Mysql的配置，以及元数据的配置。需要的就是一点细心和一点经验。如果初学Hive的你遇着同样的问题，哈！本文章可以一站式解决你的所有烦恼。同时如果有疑问的，欢迎相互交流！



