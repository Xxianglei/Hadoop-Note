## Hive初识 
![](https://i.imgur.com/tIUvVFX.png)

**MapReduce的不便性** 

* MapReduce is hard to program
* 【八股文】格式编程，三大部分
* No Schema, lack of query lanaguages, eg. SQL
	*  数据分析，针对DBA、SQL语句，如何对数据分析
	*  MapReduce编程成本高
	*  FaceBook 实现并开源Hive 


**Hive简介** 

Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射
成一张表，并提供类SQL查询功能。也是由Facebook开源用于解决海量结构化日志的数据统计。

构建在Hadoop之上的数据仓库 

* 使用HQL作为查询接口；
* 使用HDFS存储；
* 使用MapReduce计算；

**Hive的特点**  

* 处理的数据存储在HDFS
* 分析数据底层的实现MapReduce
* 执行程序运行的YARN
* 灵活性扩展性比较好，支持UDF，自定义存储格式
* 适合离线数据处理

Hive运行本质：将HQL转化成MapReduce程序 

**Hive架构** 

![](https://i.imgur.com/nzpHOcL.png)

CLI和JDBC作为客户端，在写完HQL后交给Driver以及下部分和MetaStore执行，负责将其转为MR来执行。

SQLParser->QueryOptimizer->Physical Plan->Execution

元数据: Metastore 

*  元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/
分区字段、表的类型（是否是外部表）、表的数据所在目录等；
* 默认存储在自带的derby数据库中，推荐使用采用MySQL存储Metastore；

## 动手搭建Hive环境 

记着启动Hadoop，Yarn(我把RM改到了master方便启动)

![](https://i.imgur.com/oSCgX7d.png)

![](https://i.imgur.com/F9zwzxJ.png)

![](https://i.imgur.com/6dOtUlX.png)

简单使用: 

1. show databases ;
2. use default;
3. show tables ;
4. create table student(id int, name string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
5. load data local inpath '/opt/datas/student.txt'into table student ;
7. select * from student ;
8. select id from student ;

默认将元数据放在derby（内存数据库，只供一个人使用为了满足多人开发，我们需要将使用MySQL）
## MySQL安装

metastore支持使用哪些数据库

![](https://i.imgur.com/x7Xgdg0.png)

* 检查是否已经装过mysql
	* rpm -qa|grep mysql 有就卸载没有就跳过

* 安装Mysql
	* chmod u+x ./*  添加执行权限
![](https://i.imgur.com/sEW8uG8.png)
* 安装server
	* rpm -ivh MySQL-server-5.6.24-1.el6.x86_64.rpm
  
* 安装client 
	* rpm -ivh MySQL-client-5.6.24-1.el6.x86_64.rpm

* 进入MySQL mysql -uroot -proot
* 设置开机启动chkconfig mysql on
![](https://i.imgur.com/ZTYJpTb.png)

flush privileges;刷新一下配置

设置任何机器可以连接，主要是Windows下连接。

**常见错误** 

>mysql -uroot -p
输入密码出现Access denied for user 'root'@'localhost'(using password: YES)错误。每次安装Mysql就是噩梦，今天总算找了一个比较好的解决方案。

* 关闭Mysql service mysqld stop
* 跳过密码验证 /usr/bin/mysqld_safe --skip-grant-tables（卡住了就另开一个窗口）
* 无密码登录 mysql -u root
* 修改密码
	* use mysql
	* update user set password=PASSWORD("你的密码") where User = 'root';
* 重启mysql service mysqld restart
* 正常进入mysql -uroot -proot

二：报错![](https://i.imgur.com/sEtwMZ7.png) 

    * 卸载它
    * 然后重新安装Mysql
    * 出现如下异常：
    *  FATAL ERROR: please install the following Perl     modules before executing /usr/bin/mysql_install_db: 
     Data::Dumper 

	* 解决方案：# yum install -y perl-Module-Install.noarch
	* 再卸载安装失败的Mysql
	* 重新安装
	* 查看随机密码 cat /root/.mysql_secret 
      ld6z5j8P

原因： 
在centos7环境中没有找到离线安装的办法，只能在线安装，需要调整到可联网状态。虚拟机中的话，可以调整到nat模式下。



