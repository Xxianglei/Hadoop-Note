## HiveServer2
	bin/hiveserver2

	bin/beeline

	!connect jdbc:hive2://hadoop-senior.ibeifeng.com:10000 beifeng beifeng org.apache.hive.jdbc.HiveDriver
	
	bin/beeline -u jdbc:hive2://hadoop-senior.ibeifeng.com:10000/default
	
**JDBC**

	package com.beifeng.senior.hive.jdbc;
	
	import java.sql.SQLException;
	import java.sql.Connection;
	import java.sql.ResultSet;
	import java.sql.Statement;
	import java.sql.DriverManager;
	
	public class HiveJdbcClient {
		private static final String DRIVERNAME = "org.apache.hive.jdbc.HiveDriver";
	
		/**
		 * @param args
		 * @throws SQLException
		 */
		public static void main(String[] args) throws SQLException {
			try {
				Class.forName(DRIVERNAME);
			} catch (ClassNotFoundException e) {
				e.printStackTrace();
				System.exit(1);
			}
	
			Connection con = DriverManager.getConnection(//
					"jdbc:hive2://hadoop-senior.ibeifeng.com:10000/default",//
					"root",
					"root"//
				);
			Statement stmt = con.createStatement();
			String tableName = "emp";
	
			// select * query
			String sql = "select * from " + tableName;
			System.out.println("Running: " + sql);
			ResultSet res = stmt.executeQuery(sql);
			while (res.next()) {
				System.out.println(String.valueOf(res.getInt(1)) + "\t" + res.getString(2));
			}
	
		}
	}

	HiveServer2 JDBC 

			将分析的结果存储在hive表（result），前段通过DAO代码，进行数据的查询。

## 数据压缩 
>压缩的好处 通常情况下 block  -> map 10G ,10 block
压缩后5G , 5 block

**MR压缩的位置**

![](https://i.imgur.com/0GwPSOn.png)

* map输入压缩
* map输出到磁盘进行压缩
* reduce输出进行压缩

压缩配置的位置，map的输入，map输出（中间结果），输出的数据。

**Hadoop支持的压缩格式 **
![](https://i.imgur.com/9k0cDfc.png)

**数据压缩**
![](https://i.imgur.com/CFssGEY.png)
	数据压缩后使数据量变小
	* map存到本地磁盘，减少IO
	* reduce去拷贝减少网络IO
	
1) 安装sanppy（谷歌开源） 

2) 编译haodop 2.x源码 

	1.mvn package -Pdist,native -DskipTests -Dtar -Drequire.snappy
	2./opt/modules/hadoop-2.5.0-src/target/hadoop-2.5.0/lib/native
3）替换hadoop/lib下的native
4）检查是否支持了sanppy bin/hadoop checknative
5）配置 

* **在hadoop命令行运行时配置**
![](https://i.imgur.com/2ZXXafT.png)
* yarn jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.0.jar wordcount -Dmapreduce.map.output.compress=true -Dmapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec /user/beifeng/mapreduce/wordcount/input /user/beifeng/mapreduce/wordcount/output22


* **Hive中设置压缩**

![](https://i.imgur.com/EU7fPqY.png)
![](https://i.imgur.com/CksHe6u.png)

## 数据存储

	file_format:
	  : 
	  | SEQUENCEFILE（行）
	  | TEXTFILE （行）   -- (Default, depending on hive.default.fileformat configuration)
	  | RCFILE   （列 ）  -- (Note: Available in Hive 0.6.0 and later)
	  | ORC      （列 常）   -- (Note: Available in Hive 0.11.0 and later)
	  | PARQUET   （列 常）  -- (Note: Available in Hive 0.13.0 and later)
	  | AVRO       （列 ） -- (Note: Available in Hive 0.14.0 and later)
	  | INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname
![](https://i.imgur.com/GvnsWmd.png)
* 按行存储数据
* 按列存储数据（更多时候是对列操作的比如对每一列统计数量等，减少加载到内存中的数据）


**行列存储**
![](https://i.imgur.com/GlWqyDe.png)

**ORC**

		TEXTFILE

		create table page_views(
		track_time string,
		url string,
		session_id string,
		referer string,
		ip string,
		end_user_id string,
		city_id string
		)
		ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
		STORED AS TEXTFILE ;
		
		load data local inpath '/opt/datas/page_views.data' into table page_views ;
		dfs -du -h /user/hive/warehouse/page_views/ ;
		18.1 M  /user/hive/warehouse/page_views/page_views.data
		
		ORC 
		
		create table page_views_orc(
		track_time string,
		url string,
		session_id string,
		referer string,
		ip string,
		end_user_id string,
		city_id string
		)
		ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
		STORED AS orc ;
		
		insert into table page_views_orc select * from page_views ;
		dfs -du -h /user/hive/warehouse/page_views_orc/ ;
		2.6 M  /user/hive/warehouse/page_views_orc/000000_0

		parquet
		create table page_views_parquet(
		track_time string,
		url string,
		session_id string,
		referer string,
		ip string,
		end_user_id string,
		city_id string
		)
		ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
		STORED AS PARQUET ;
		
		insert into table page_views_parquet select * from page_views ;
		dfs -du -h /user/hive/warehouse/page_views_parquet/ ;
		13.1 M  /user/hive/warehouse/page_views_parquet/000000_0
		
		select session_id,count(*) cnt from page_views group by session_id order by cnt desc limit 30 ;
		
		select session_id,count(*) cnt from page_views_orc group by session_id order by cnt desc limit 30 ;
		
		-------
		select session_id from page_views limit 30 ;
		select session_id from page_views_orc limit 30 ;
		select session_id from page_views_parquet limit 30 ;
		
		========================================================
		create table page_views_orc_snappy(
		track_time string,
		url string,
		session_id string,
		referer string,
		ip string,
		end_user_id string,
		city_id string
		)
		ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
		STORED AS orc tblproperties ("orc.compress"="SNAPPY");
		
		insert into table page_views_orc_snappy select * from page_views ;
		dfs -du -h /user/hive/warehouse/page_views_orc_snappy/ ;
		
		--------------
		create table page_views_orc_none(
		track_time string,
		url string,
		session_id string,
		referer string,
		ip string,
		end_user_id string,
		city_id string
		)
		ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
		STORED AS orc tblproperties ("orc.compress"="NONE");
		
		insert into table page_views_orc_none select * from page_views ;
		
		dfs -du -h /user/hive/warehouse/page_views_orc_none/ ;
		
		--------------------
		set parquet.compression=SNAPPY ;
		create table page_views_parquet_snappy(
		track_time string,
		url string,
		session_id string,
		referer string,
		ip string,
		end_user_id string,
		city_id string
		)
		ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
		STORED AS parquet;
		insert into table page_views_parquet_snappy select * from page_views ;
		dfs -du -h /user/hive/warehouse/page_views_parquet_snappy/ ;

注意:这里我改了native文件，因为安装snappy的需要。可能会涉及到版本不匹配（我用的Hadoop版本的确和教程不一样）而在hive load数据时候报错，最后我改回来了。慌得一批。自己重新编译一个。

## Hive优化

* FetchTask 直接抓取，不需要走MP

>配置的属性 hive.fetch.task.conversi

![](https://i.imgur.com/DuzlgHH.png)

默认是 minimal-> * 、查分区数据、Limit都不走MP。

* 大表拆分
* 外部表分区表
* 数据格式控制数据压缩
* SQL语句优化
* MapReduce优化
	* Reduce Number
	* JVM 重用 
	* 推测执行

