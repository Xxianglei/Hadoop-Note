## Hadoop协作框架
	
	对日志类型的海量数据
		* hdfs
		* mr ,  hive - hql
	
	大数据协作框架
	第一个问题
		hdfs
			文件来源哪里？数据存储到hdfs ？  海量
	
		现实数据来源两个方面
			* RDBMS（Oracle，MySQL，DB2...） ->  sqoop（SQL to  HADOOP）
			* 文件（apache，nginx日志数据） -> Flume(实时抽取数据)
	
	第二个问题
		对数据的分析任务Job，至少都是上千（互联网公司）
		调度任务  ？
			什么执行，多长执行一次，执行频率
		某一些业务的分析，需要许多job任务共同完成，相互依赖关系 ，工作流如何调度呢 ？
	Oozie
	
	
	第三个问题
		hadoop 2.x生态系统中重要的框架，8个，
		监控
		统一WEB UI界面，管理框架，监控框架
	Hue

## Sqoop

	Sqoop：SQL-to-Hadoop
	 连接传统关系型数据库和Hadoop的桥梁
	 把关系型数据库的数据导入到Hadoop与其相关的系统
	(如HBase和Hive)中
	 把数据从Hadoop系统里抽取并导出到关系型数据库里
	 利用MapReduce加快数据传输速度
	批处理方式进行数据传输

修改

![](https://i.imgur.com/iKy0erY.png)
![](https://i.imgur.com/zYibdov.png)
![](https://i.imgur.com/wsyA3ef.png)

**使用**

注意：需要把 mysql-connector-java-5.1.27-bin.jar 放到sqoop的lib目录下面。

![](https://i.imgur.com/9dA4lfo.png)

**帮助**

./sqoop help xx

	usage: sqoop list-tables [GENERIC-ARGS] [TOOL-ARGS]
	
	Common arguments:
	   --connect <jdbc-uri>                         Specify JDBC connect
	                                                string
	   --connection-manager <class-name>            Specify connection manager
	                                                class name
	   --connection-param-file <properties-file>    Specify connection
	                                                parameters file
	   --driver <class-name>                        Manually specify JDBC
	                                                driver class to use
	   --hadoop-home <hdir>                         Override
	                                                $HADOOP_MAPRED_HOME_ARG
	   --hadoop-mapred-home <dir>                   Override
	                                                $HADOOP_MAPRED_HOME_ARG
	   --help                                       Print usage instructions
	-P                                              Read password from console
	   --password <password>                        Set authentication
	                                                password
	   --password-alias <password-alias>            Credential provider
	                                                password alias
	   --password-file <password-file>              Set authentication
	                                                password file path
	   --relaxed-isolation                          Use read-uncommitted
	                                                isolation for imports
	   --skip-dist-cache                            Skip copying jars to
	                                                distributed cache
	   --username <username>                        Set authentication
	                                                username
	   --verbose                                    Print more information
	                                                while working

查看有哪些数据库	

	./sqoop list-databases --connect jdbc:mysql://192.168.1.205:3306 --username root --password root

数据导入导出

![](https://i.imgur.com/vog4UvH.png)

* 得到表的元数据 
* 提交map任务

![](https://i.imgur.com/dbUKPJS.png)

	bin/sqoop import \
	--connect jdbc:mysql://master:3306/test \
	--username root \
	--password root \
	--table my_user
	
不写目录的话，在hdfs的当前用户目录下面
	
		bin/sqoop import \
		--connect jdbc:mysql://master:3306/test \
		--username root \
		--password root \
		--table my_user \
		--target-dir /user/sqoop/imp_my_user \
		--num-mappers 1
设置map个数为1 

**控制文件格式** 

数据存储文件

	* textfile
	* orcfile
	* parquet
import hdfs : parquet

	bin/sqoop import \
	--connect jdbc:mysql://hadoop-senior.ibeifeng.com:3306/test \
	--username root \
	--password 123456 \
	--table my_user \
	--target-dir /user/beifeng/sqoop/imp_my_user_parquet \
	--fields-terminated-by ',' \
	--num-mappers 1 \
	--as-parquetfile

	>>>>>>>>>>>>>>>>>import hdfs : columns 指定列导入>>>>>>>>>>>>>>>>>>
	bin/sqoop import \
	--connect jdbc:mysql://master:3306/test \
	--username root \
	--password root \
	--table my_user \
	--target-dir /user/beifeng/sqoop/imp_my_user_column \
	--num-mappers 1 \
	--columns id,account

* 在实际的项目中，要处理的数据，需要进行初步清洗和过滤


		* 某些字段过滤
		* 条件
		* join

		bin/sqoop import \
		--connect jdbc:mysql://hadoop-senior.ibeifeng.com:3306/test \
		--username root \
		--password 123456 \
		--query 'select id, account from my_user where $CONDITIONS' \
		--target-dir /user/beifeng/sqoop/imp_my_user_query \
		--num-mappers 1

		>>>>>>>>>>>>>>>>>import hdfs : compress >>>>>>>>>>>>>>>>>>、
		压缩库你得配置编译源码Snappy
		bin/sqoop import \
		--connect jdbc:mysql://hadoop-senior.ibeifeng.com:3306/test \
		--username root \
		--password 123456 \
		--table my_user \
		--target-dir /user/beifeng/sqoop/imp_my_sannpy \
		--delete-target-dir \
		--num-mappers 1 \
		--compress \
		--compression-codec org.apache.hadoop.io.compress.SnappyCodec \
		--fields-terminated-by '\t'

		>>>>>>>>>>>>>>>>>import hdfs  increment>>>>>>>>>>>>>>>>>>
		增量数据的导入
			有一个唯一标识符，通常这个表都有一个字段，类似于插入时间createtime
			* query
		where createtime => 20150924000000000 and createtime < 20150925000000000
			* sqoop
		Incremental import arguments:
		   --check-column <column>        Source column to check for incremental
		                                  change
		   --incremental <import-type>    Define an incremental import of type
		                                  'append' or 'lastmodified'
		   --last-value <value>           Last imported value in the incremental
		                                  check column	
		bin/sqoop import \
		--connect jdbc:mysql://hadoop-senior.ibeifeng.com:3306/test \
		--username root \
		--password 123456 \
		--table my_user \
		--target-dir /user/beifeng/sqoop/imp_my_incr \
		--num-mappers 1 \
		--incremental append \
		--check-column id \
		--last-value 4
		
		>>>>>>>>>>>>>>>>>import hdfs direct  直接从mysql导出数据>>>>>>>>>>>>>>>>>>
		bin/sqoop import \
		--connect jdbc:mysql://hadoop-senior.ibeifeng.com:3306/test \
		--username root \
		--password 123456 \
		--table my_user \
		--target-dir /user/beifeng/sqoop/imp_my_incr \
		--num-mappers 1 \
		--delete-target-dir \
		--direct 


**导出数据**

![](https://i.imgur.com/axcklzz.png)

	===================导出数据RDBMS=====================================
	hive table 
	* table 
		hiveserver2进行jdbc方式查询数据
	* hdfs file
		export ->  mysql/oracle/db2/
	
	>>>>>>>>>>>>>>>>>export mysql table >>>>>>>>>>>>>>>>>>
	touch /opt/datas/user.txt
	vi /opt/datas/user.txt
	12,beifeng,beifeng
	13xuanyun,xuanyu
	
	bin/hdfs dfs -mkdir -p /user/beifeng/sqoop/exp/user/ 
	bin/hdfs dfs -put /opt/datas/user.txt /user/beifeng/sqoop/exp/user/
	
	
	bin/sqoop export \
	--connect jdbc:mysql://hadoop-senior.ibeifeng.com:3306/test \
	--username root \
	--password 123456 \
	--table my_user \
	--export-dir /user/beifeng/sqoop/exp/user/ \
	--num-mappers 1



		===================导入导出Hive====================================
		Hive数据存储在hdfs上
			schema
		table location / file
		
		>>>>>>>>>>>>>>>>>import hive table >>>>>>>>>>>>>>>>>>
		use default ;
		drop table if exists user_hive ;
		create table user_hive(
		id int,
		account string,
		password string
		)
		ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' ;
		
		bin/sqoop import \
		--connect jdbc:mysql://hadoop-senior.ibeifeng.com:3306/test \
		--username root \
		--password 123456 \
		--table my_user \
		--fields-terminated-by '\t' \
		--delete-target-dir \
		--num-mappers 1 \
		--hive-import \
		--hive-database default \
		--hive-table user_hive
		
		
		>>>>>>>>>>>>>>>>>export mysql table >>>>>>>>>>>>>>>>>>
		CREATE TABLE `my_user2` (
		  `id` tinyint(4) NOT NULL AUTO_INCREMENT,
		  `account` varchar(255) DEFAULT NULL,
		  `passwd` varchar(255) DEFAULT NULL,
		  PRIMARY KEY (`id`)
		);
		
        就变个路径
		
		bin/sqoop export \
		--connect jdbc:mysql://hadoop-senior.ibeifeng.com:3306/test \
		--username root \
		--password 123456 \
		--table my_user2 \
		--export-dir /user/hive/warehouse/user_hive \
		--num-mappers 1 \
		--input-fields-terminated-by '\t'
		格式
		=======================================================

![](https://i.imgur.com/nsPMgP2.png)

		shell scripts
			## step 1
			load data ...
			## step 2
			bin/hive -f xxxx
			## step 3
			bin/sqoop --options-file /opt/datas/sqoop-import-hdfs.txt 

