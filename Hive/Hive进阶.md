## Hive数据库
![](https://i.imgur.com/PAIqzIA.png)
![](https://i.imgur.com/0idzmZp.png)
![](https://i.imgur.com/xCLIzhd.png)
	create database db_hive_01 ;
	create database if not exists db_hive_02 ;  -->   标准
	create database if not exists db_hive_03 location '/user/beifeng/hive/warehouse/db_hive_03.db' ;
	
	show databases ;
	show databases like 'db_hive*' ;
	
	use db_hive ;
	
	desc database db_hive_03 ;
	desc database extended db_hive_03 ;
	（数据库删除的同时数据库的目录也没有了）
	drop database db_hive_03 ; //有表存在就删不了
	drop database db_hive_03 cascade;//级联删除，有表也能删
	drop database if exists db_hive_03 ;
## Hive表操作

![](https://i.imgur.com/nNryGvg.png)
![](https://i.imgur.com/4rTu7ax.png)
![](https://i.imgur.com/Dho3XkM.png)
* 创建表 

    //基础的表创建格式
	create table IF NOT EXISTS default.xl_log_20181117(
	ip string COMMENT 'remote ip address'//注释 , 
	user string ,
	req_url string COMMENT 'user request url')  
	COMMENT 'BeiFeng Web Access Logs'
	ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' // 行分割
	STORED AS TEXTFILE ;//数据格式（默认如此可以不写）
    //加载数据到表里面
	load data local inpath '/opt/datas/bf-log.txt' into table default.bf_log_20150913;
	//用查询的数据创建一个新表
	create table IF NOT EXISTS default.bf_log_20150913_sa
	AS select ip,req_url from default.bf_log_20150913 ;
	// 拷贝表结构创建一个新的表，引出一个分表的概念
	create table IF NOT EXISTS default.bf_log_20150913_sa
	like default.bf_log_20150913 ;



* 删除表

![](https://i.imgur.com/z90JnZX.png)

## 例子

	==================================================================
	员工表
	create table IF NOT EXISTS default.emp(
	empno int,
	ename string,
	job string,
	mgr int,
	hiredate string,
	sal double,
	comm double,
	deptno int
	)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
	部门表
	create table IF NOT EXISTS default.dept(
	deptno int,
	dname string,
	loc string
	)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
	
	load data local inpath '/opt/datas/emp.txt' overwrite into table emp ;
	load data local inpath '/opt/datas/dept.txt' overwrite into table dept ;
	
	create table if not exists default.dept_cats
	as select * from dept ;
	
	truncate table dept_cats ;  //清除表的数据
	
	create table if not exists default.dept_like
	like
	default.dept ;  // 拷贝表结构
	
	alter table dept_like rename to dept_like_rename ;
	
	drop table if exists dept_like_rename ;
	
	==================================================================
## Hive表类型

1.管理表（manged_table）

* 内部表也称之为MANAGED_TABLE；
* 默认存储在/user/hive/warehouse下，也可以通过location指定；
* 删除表时，会删除表数据以及元数据；


2.托管表（external） 

* 外部表称之为EXTERNAL_TABLE；
* 在创建表时可以自己指定目录位置(LOCATION)；
* 删除表时，只会删除元数据不会删除表数据；

>使用外部表的场景:上述特点中可以看出在使用外部表的情况下删除表的时候是不会删除表中数据的。同时可以指定自己的目录，通常必须指定。这样可供多部门使用。

例子：

	create EXTERNAL table IF NOT EXISTS default.emp_ext2(
	empno int,
	ename string,
	job string,
	mgr int,
	hiredate string,
	sal double,
	comm double,
	deptno int
	)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
	location '/user/beifeng/hive/warehouse/emp_ext2';

## 分区表


>分区表实际上就是对应一个HDFS文件系统上的独立的文件夹，该文件夹
下是该分区所有的数据文件。Hive中的分区就是分目录，把一个大的数据
集根据业务需要分割成更下的数据集

使用场景：如对日志进行分析，可以按照时间进行分区，提高分析效率。

例子：

	create EXTERNAL table IF NOT EXISTS default.emp_partition(
	empno int,
	ename string,
	job string,
	mgr int,
	hiredate string,
	sal double,
	comm double,
	deptno int
	)                  //二级分区.
	partitioned by (month string,day string)  //不能与列名重复
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' ;

	load data local inpath '/opt/datas/emp.txt' into table default.emp_partition partition (month='201509',day='13') ;
	此时HDFS目录结构为：
	/user/hive/warehouse/emp_partition/month=201509/day=13

	select * from emp_partition where month = '201509' and day = '13' ;
	
	//统计几天的数据可以使用union
	select * from emp_partition where month = '201509' and day = '14' union
	select * from emp_partition where month = '201509' and day = '15' union
	select * from emp_partition where month = '201509' and day = '16' ;
		
例子：

	create table IF NOT EXISTS default.dept_nopart(
	deptno int,
	dname string,
	loc string
	)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
	
	dfs -put /opt/datas/dept.txt /user/hive/warehouse/dept_nopart ;
	
	select * from dept_nopart ;
	
	可以查到数据
	------------------------------------------------------
	create table IF NOT EXISTS default.dept_part(
	deptno int,
	dname string,
	loc string
	)
	partitioned by (day string)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';

	第一种方式
	dfs -mkdir -p /user/hive/warehouse/dept_part/day=20150913 ;
	dfs -put /opt/datas/dept.txt /user/hive/warehouse/dept_part/day=20150913 ;
	
	//修复表
	hive (default)> msck repair table dept_part ;
	
	第二种方式
	dfs -mkdir -p /user/hive/warehouse/dept_part/day=20150914 ;
	dfs -put /opt/datas/dept.txt /user/hive/warehouse/dept_part/day=20150914 ;

	（常用）
	alter table dept_part add partition(day='20150914');
	
	//查看一个表的分区数
	show partitions dept_part ;

## 数据迁移

**加载数据** 

![](https://i.imgur.com/nufQor3.png)

* 原始文件存储的位置
	* 本地 local
	* hdfs
* 对表的数据是否覆盖
	* 覆盖 overwrite
	* 追加
* 分区表加载，特殊性 

	partition (partcol1=val1,...)

1）加载本地文件到hive表

load data local inpath '/opt/datas/emp.txt' into table default.emp ;
2）加载hdfs文件到hive中

load data inpath '/user/beifeng/hive/datas/emp.txt' into table default.emp ;

3）加载数据覆盖表中已有的数据

load data inpath '/user/beifeng/hive/datas/emp.txt' overwrite into table default.emp ;
4）创建表时通过insert加载

create table default.emp_ci like emp ;
insert into table default.emp_ci select * from default.emp ;

5）创建表的时候通过location指定加载 

	create EXTERNAL table IF NOT EXISTS default.emp_ext2(
	empno int,
	ename string,
	job string,
	mgr int,
	hiredate string,
	sal double,
	comm double,
	deptno int
	)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
	location '/user/beifeng/hive/warehouse/emp_ext2';

**导出数据**


	insert overwrite local directory '/opt/datas/hive_exp_emp'
	select * from default.emp ;

	insert overwrite local directory '/opt/datas/hive_exp_emp2'
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' COLLECTION ITEMS TERMINATED BY '\n'

	select * from default.emp ;
	
	bin/hive -e "select * from default.emp ;" > /opt/datas/exp_res.txt
	
	-----------------------------------------------------------
	insert overwrite directory '/user/beifeng/hive/hive_exp_emp'

	select * from default.emp ;
	
	============================================================
	sqoop
		hdfs/hive -> rdbms
		rdbms -> hdfs/hive/hbase
