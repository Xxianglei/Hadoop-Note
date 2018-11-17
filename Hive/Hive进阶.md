>这一块的内容自我感觉算不上进阶。除了一些看似需要死记硬背但实际上我并不会去背的东西，真没啥好写的。只能全当做笔记了。另外值得提一嘴的就是那个UDF，其实只要有那么一点点Java基础看起来就很简单。不要因为编程两个字就选择性忽略了。单学习来讲UDF编程还是很基础的，在实际项目中视不同的需求可能会麻烦一点。但再难毕竟是Java老本行。


## Hive关于库的操作
----
![](https://i.imgur.com/PAIqzIA.png)
![](https://i.imgur.com/0idzmZp.png)
![](https://i.imgur.com/xCLIzhd.png)

**一堆CDAUS：**

	create database db_hive_01 ;
	create database if not exists db_hive_02 ;      //推荐这种写法很标准
	create database if not exists db_hive_03 location '/user/beifeng/hive/warehouse/db_hive_03.db' ;
	
	show databases ;
	show databases like 'db_hive*' ;
	
	use db_hive ;
	
	desc database db_hive_03 ;
	desc database extended db_hive_03 ;           // 查看拓展信息
	（数据库删除的同时数据库的目录也没有了）
	drop database db_hive_03 ;                          //有表存在就删不了
	drop database db_hive_03 cascade;            //级联删除，有表也能删
	drop database if exists db_hive_03 ;

这些是在是没什么好说的照搬笔记。

## Hive关于表的操作
----

![](https://i.imgur.com/nNryGvg.png)
![](https://i.imgur.com/4rTu7ax.png)
![](https://i.imgur.com/Dho3XkM.png)
* 创建表 🌰
  	 1.	基础的表创建格式
			create table IF NOT EXISTS default.xl_log_20181117(
			ip string COMMENT 'remote ip address'//注释 , 
			user string ,
			req_url string COMMENT 'user request url')  
			COMMENT 'BeiFeng Web Access Logs'
			ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' // 行分割
	        STORED AS TEXTFILE ;  //数据格式（默认如此可以不写）
  2.  //加载数据到表里面
	load data local inpath '/opt/datas/xl-log.txt' into table default.bf_log_20150913;
	3. //用查询的数据创建一个新表
	create table IF NOT EXISTS default.xl_log_20150913_sa
	AS select ip,req_url from default.xl_log_20150913 ;
	4. // 拷贝表结构创建一个新的表，引出一个分表的概念
	create table IF NOT EXISTS default.xl_log_20150913_sa
	like default.xl_log_20150913 ;



* 删除表🌰

![](https://i.imgur.com/z90JnZX.png)


---
	//建表
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
	//加载本地数据到表中
	load data local inpath '/opt/datas/emp.txt' overwrite into table emp ;
	load data local inpath '/opt/datas/dept.txt' overwrite into table dept ;
	
	create table if not exists default.dept_cats
	as select * from dept ;
	
	//清除表的数据
	truncate table dept_cats ;  
	
    // 拷贝表结构建表
	create table if not exists default.dept_like
	like
	default.dept ; 
	//修改表名
	alter table dept_like rename to dept_like_rename ;
	//删除表
	drop table if exists dept_like_rename ;
---
## Hive表的类型
---
1.管理表（manged_table）

* 内部表也称之为MANAGED_TABLE；
* 默认存储在/user/hive/warehouse下，也可以通过location指定；
* **删除表时，会删除表数据以及元数据**；


2.托管表（external） 

* 外部表称之为EXTERNAL_TABLE；
* 在创建表时可以自己指定目录位置(LOCATION)；
* **删除表时，只会删除元数据不会删除表数据**；

>使用外部表的场景:上述特点中可以看出在使用外部表的情况下删除表的时候是不会删除表中数据的。同时可以指定自己的目录，**通常必须指定**。这样可供多部门使用。

例子🌰：

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
---

>分区表实际上就是对应一个HDFS文件系统上的独立的文件夹，该文件夹下是该分区所有的数据文件。Hive中的分区就是分目录，把一个大的数据集根据业务需要分割成更下的数据集（将表进行薯片分割）。

使用场景：如对日志进行分析，可以按照时间进行分区，提高分析效率。分析10G数据和分析1G数据你说谁快？

例子🌰：

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
	partitioned by (month string,day string)     //不能与列名重复
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' ;

	load data local inpath '/opt/datas/emp.txt' into table default.emp_partition partition (month='201509',day='13') ;
	此时HDFS目录结构为：
	/user/hive/warehouse/emp_partition/month=201509/day=13

	select * from emp_partition where month = '201509' and day = '13' ;
	
	//统计几天的数据可以使用union
	select * from emp_partition where month = '201509' and day = '14' union
	select * from emp_partition where month = '201509' and day = '15' union
	select * from emp_partition where month = '201509' and day = '16' ;
		
例子🌰：

	create table IF NOT EXISTS default.dept_part(
	deptno int,
	dname string,
	loc string
	)
	partitioned by (day string)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
	
	加载数据到表下面但是此时你没有指定哪个分区，如果你去select你会发现查不出来数据。对此有一下两种解决方案。

	第一种方式
	1.dfs -mkdir -p /user/hive/warehouse/dept_part/day=20150913 ;
	  dfs -put /opt/datas/dept.txt /user/hive/warehouse/dept_part/day=20150913 ;	
	2.hive (default)> msck repair table dept_part ;//修复表
	
	第二种方式
	1. dfs -mkdir -p /user/hive/warehouse/dept_part/day=20150914 ;
	   dfs -put /opt/datas/dept.txt /user/hive/warehouse/dept_part/day=20150914 ;
	2. alter table dept_part add partition(day='20150914');//（常用）
	
	//查看一个表的分区数
	show partitions dept_part ;

## Hive的数据迁移
---

**加载数据** 

![](https://i.imgur.com/nufQor3.png)

* 原始文件存储的位置
	* 本地 local
	* 文件系统hdfs
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


	1. insert overwrite local directory '/opt/datas/hive_exp_emp'
	   select * from default.emp ;

	2. insert overwrite local directory '/opt/datas/hive_exp_emp2'
    	ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' COLLECTION ITEMS TERMINATED BY '\n'
	    select * from default.emp ;
	
	3. bin/hive -e "select * from default.emp ;" > /opt/datas/exp_res.txt
	

	4. insert overwrite directory '/user/beifeng/hive/hive_exp_emp'

	   select * from default.emp ;
	   
	5. sqoop后面介绍
		

##Hive查询

[详见官网](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select)

	SELECT [ALL | DISTINCT] select_expr, select_expr, ...
	FROM table_reference
	[WHERE where_condition]
	[GROUP BY col_list]
	[CLUSTER BY col_list
	  | [DISTRIBUTE BY col_list] [SORT BY col_list]
	]
	[LIMIT number]

	select * from emp ;
	select t.empno, t.ename, t.deptno from emp t ;
	
	select * from emp limit 5 ;
	select t.empno, t.ename, t.deptno from emp t where  t.sal between 800 and 1500 ;
	
	select t.empno, t.ename, t.deptno from emp t where comm is null ;
	
	select count(*) cnt from emp ;
	select max(sal) max_sal from emp ;
	select sum(sal) from emp ;
	select avg(sal) from emp ;

	 group by分组 /having 条件
	 
	group by
	* 每个部门的平均工资
	select t.deptno, avg(t.sal) avg_sal from emp t group by t.deptno ;
	* 每个部门中每个岗位的最高薪水
	select t.deptno, t.job, max(t.sal) avg_sal from emp t group by t.deptno, job ;
	
	having
		* where 是针对单条记录进行筛选
		* having 是针对分组结果进行筛选
	求每个部门的平均薪水大于2000的部门
	select deptno, avg(sal) from emp group by deptno ;
	select deptno, avg(sal) avg_sal from emp group by deptno having avg_sal > 2000;

	join连接
		两个表进行连接
		m  n
		m表中一条记录和n表中的一条记录组成一条记录
	等值jion
		join ... on
	select e.empno, e.ename, d.deptno, d.dname from emp e join dept d on e.deptno = d.deptno ;
	
	左连接
	left join
	select e.empno, e.ename, d.deptno, d.dname  from emp e left join dept d on e.deptno = d.deptno ;
	
	右连接
	right join
	select e.empno, e.ename, e.deptno, d.dname  from emp e right join dept d on e.deptno = d.deptno ;
	
	
	全连接（左+右）
	full join
	select e.empno, e.ename, e.deptno, d.dname  from emp e full join dept d on e.deptno = d.deptno ;

Hive 数据迁移新特性（类似与Get操作）

	Export 
		导出，将Hive表中的数据，导出到外部
	Import
		导入，将外部数据导入Hive表中
	
	EXPORT TABLE default.emp TO  '/user/beifeng/hive/export/emp_exp'（自动创建） ;
	
	export_target_path：	//指的是HDFS上路径
    //创建时使用import导入数据
	create table db_hive.emp like default.emp ;
	import table db_hive.emp from '/user/beifeng/hive/export/emp_exp';

Hive 数据排序

	order by
		对全局数据的一个排序，仅仅只有个reduce
		select * from emp order by empno desc ;
	
	sort by
		对每一个reduce内部数据进行排序的，全局结果集来说不是排序
	
		set mapreduce.job.reduces= 3;
		select * from emp sort by empno asc ;
		insert overwrite local directory '/opt/datas/sortby-res' select * from emp sort by empno asc ;
	
	 distribute by
		分区partition
		类似于MapReduce中分区partition,对数据进行分区，结合sort by进行使用
		insert overwrite local directory '/opt/datas/distby-res' select * from emp distribute by deptno sort by empno asc ;
	
	    注意事项：
		distribute by 必须要在sort by前面。
	
	 cluster by
		当distribute by和sort by 字段相同时，可以使用cluster by ;
		insert overwrite local directory '/opt/datas/cluster-res' select * from emp cluster by empno ;

## Hive UDF编程 
---

>UDF全称User Definition Function，用户自定义函数。使用极其简单。

编程步骤： 
1. 导入依赖

		<dependencies>
			<!-- Hadoop Client -->
			<dependency>
				<groupId>org.apache.hadoop</groupId>
				<artifactId>hadoop-client</artifactId>
				<version>${hadoop.version}</version>
			</dependency>
	
			<!-- Hive Client -->
			<dependency>
				<groupId>org.apache.hive</groupId>
				<artifactId>hive-jdbc</artifactId>
				<version>${hive.version}</version>
			</dependency>
			<dependency>
				<groupId>org.apache.hive</groupId>
				<artifactId>hive-exec</artifactId>
				<version>${hive.version}</version>
			</dependency>
	
			<!-- Junit 4.x -->
			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>4.10</version>
				<scope>test</scope>
			</dependency>
		</dependencies>
3. 继承org.apache.hadoop.hive.ql.UDF 

4. 需要实现evaluate函数；evaluate函数支持重载；
	public class LowerUDF extends UDF {
		
		public Text evaluate(Text str){
			// validate 
			if(null == str.toString()){
				return null ;
			}
			// lower
			return new Text (str.toString().toLowerCase())  ;
		}	
		public static void main(String[] args) {
			System.out.println(new LowerUDF().evaluate(new Text("HIVE")));
		}
    	}


	**如何运行**
	1. 方法一：
		add jar /opt/datas/hiveudf.jar ;//在hive运行下添加jar文件
		create temporary function my_lower as "com.beifeng.senior.hive.udf.LowerUDF" ;//注册临时函数，指定用哪个类做处理
		select ename, my_lower(ename) lowername from emp limit 5 ;//调用函数
	 2. 方法二：
		CREATE FUNCTION self_lower AS 'com.beifeng.senior.hive.udf.LowerUDF' USING JAR 'hdfs://hadoop-senior.ibeifeng.com:8020/user/beifeng/hive/jars/hiveudf.jar';  //把本地jar传上去，改文件路径需要提前创建
		select ename, self_lower(ename) lowername from emp limit 5 ;
		
## 总结
-----
卒！