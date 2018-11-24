## Hbase

![](http://hbase.apache.org/images/hbase_logo_with_orca.png)

[HBase](http://hbase.apache.org/book.html#quickstart)是大数据（Hadoop）分布式的可扩展的大数据量的数据库。

* 存储数据
* 检索数据
* 查询数据

传统关系型数据库RDBMS

	Oracle/MySQL

Hbase能做的什么

* 海量数据的存储
* 海量数据的查询

Hbase在上亿的数据表里能做到秒级别查询（实时查询）

使用场景

* 交通数据
* 账单数据
* 游戏数据
* 电商交易数据


Hbase在生态圈中的位置

![](https://i.imgur.com/jdVak8j.png)

Hbase列存储

每条数据有唯一的标识符

	rowkey    -. 主键

面向列的数据库

	一条数据的结构:

	rowkey + columnfamily + column01  + timestamp ：  value   => cell(单元)

为什么Hbase能这么快呢？

>关键点在于表中rowkey的设计。

![](https://i.imgur.com/zirGOCH.png)

Hbase中数据是以字节数组进行存储，不存在数据类型。默认数据三份（HDFS）

数据模型

![](https://i.imgur.com/jcc3v6i.png)
![](https://i.imgur.com/4cKERxy.png)

Hbase架构

![](https://i.imgur.com/UTpOfCZ.png)

可见HDFS，Zookeeper。有多少个RS，管理的内容都存在ZK的ZNode上面，其次ZK帮助用户找到这某张表在哪个RS做一个检索功能。ZK的诞生于Hbase的需求。

Hbase安装部署[指导](http://hbase.apache.org/book.html#quickstart)

* 解压安装
* 配置hbase-env.sh

	![](https://i.imgur.com/yEcCB6v.png)
* 配置hbase-site.xml
	
	![](https://i.imgur.com/95WoJZE.png)

同时关闭默认ZK管理，由我们自己来配置。注释掉就行。

	# export HBASE_MANAGES_ZK=true

* 修改regionservers

![](https://i.imgur.com/Ly9vIIc.png)

* 启动Hbase（提前启动Hadoop）


hbase-deamon.sh start master

![](https://i.imgur.com/hC39Y72.png)

	* zookeeper
	* namenode
	* datanode
	* HM
* 访问Web页面

![](https://i.imgur.com/GMr57OF.png)
![](https://i.imgur.com/lZ4pvpw.png)

各个目录的含义（官方文档）：

* WAL意为Write ahead log用来做灾难恢复

* 有必要的话就换hadoop的jar


![](https://i.imgur.com/UBCOs8p.png)

Hbase Shell使用

![](https://i.imgur.com/NWYERgE.png)


* list 查看表
* help ‘’查看帮助细节  help 'create'
* 创建表 create ‘user’，‘info’
* 描述表 describe ‘user’
* 插数据 put 'user','1001','info:name','xianglei' （表名，rowkey，列蔟:列名，值）
* 查看数据（三种方式） 
	* 依据rowkey查询，最快的

* 从memstore写到storefile flush 'user'
* 手动compact compact 'user'

		get
![](https://i.imgur.com/JuNoytH.png)
![](https://i.imgur.com/QmDGu2O.png)

	* 范围查询（用的最多）

		scan 'user',{STARTROW=>'1002'}
![](https://i.imgur.com/gGBBMNb.png)

	* 全表扫描

		scan

		![](https://i.imgur.com/KuqLu8p.png)
		![](https://i.imgur.com/4dmLsUC.png)

* 删除数据
	
	delete 'user','1001','info:name'
	deleteall 'user','1001'

Hbase 物理模型

![](https://i.imgur.com/2rBdlXq.png)
![](https://i.imgur.com/EHWJHMW.png)
![](https://i.imgur.com/whudTDV.png)
![](https://i.imgur.com/fgYvVmF.png)
![](https://i.imgur.com/AYHNwqa.png)

>每个column family存储在HDFS上的一个单独文件中；Key 和 Version number在每个 column family中均由一份；空值不会被保存。

HBase数据写入流程

	put —> cell 
		* 0) wal(预写日志) -> hdfs
		* 1) memstore（溢写）
		* 2) storefile -> hdfs

Hbase Region理解

	Region
		rowkey
		startrowkey[  - stoprowkey )
	0011
	..			Region-00
	1001
	..			Region-01  1009
	1011
	..			Region-02
	1021
	..			Region-03
	1031
				Region-04
	
	1041
	
	
	>>>>>
	默认的情况下，创建一个表时，会给一个表创建一个Region
	如下：

	startkey
		null
					1200001
	endkey
		null
	当插入数据达到一个阀值，增加一个新的region。


Hbase 架构细节

![](https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=3299396373,2098068075&fm=26&gp=0.jpg)

客户端读取数据首先连接ZK，他需要去ZK找到这个表（region）所在的位置
这里Zk存储了下图内容：然后去meta表扫描信息，比如找到user的rowkey然后找到他对应的region，得到管理他的regionserver。Master连接ZK是需要知道哪些RS是活着的。有几个列蔟就有几个Store，Store里面有一个memstore和很多个storeFile。

Hlog存在的意义是，当集群突然挂掉memstore数据丢失后，当重启机器时Hlog重新读取文件系统的数据，并且恢复回来。

数据上传->Hlog->memstore->storefile
数据读取:先到memstore读（没有溢写到）再到storefile最后合并。

![](https://i.imgur.com/6llPzyg.png)

RS->一个RS对应一个目录

![](https://i.imgur.com/YV2ZlqZ.png)

* meta-region-server


	存储meta表对应的region被哪个regionserver管理的信息
![](https://i.imgur.com/1pkIJ9Y.png)


	HBase新版本中，有了类似于RDBMS中DataBase的概率
	命令空间
		用户自定义的表，默认情况下命名空间
			default
		系统自带的元数据表的命名空间
			hbase
![](https://i.imgur.com/eWaLM5z.png)

user-table拥有多个

	regions

hbase:meta

					RegionServer    RegionServer

client -> zookeepr -> hbase:meta -> user-table -> put/get/scan

	get /hbase/meta-region-server

Hbase数据存储

* 所有数据存储在Hdfs上
	* Hfile
	* Hlog 


![](https://i.imgur.com/G4RGRoI.png)
![](https://i.imgur.com/H8SSmx0.png)

Hbase Java API

* jar依赖 server client
* 配置文件导入 hdfs-seite.xml core-site.xml hbase-site.xml


实例代码:

	package com.beifeng.senior.hadoop.hbase;
	
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.hbase.Cell;
	import org.apache.hadoop.hbase.CellUtil;
	import org.apache.hadoop.hbase.HBaseConfiguration;
	import org.apache.hadoop.hbase.client.Delete;
	import org.apache.hadoop.hbase.client.Get;
	import org.apache.hadoop.hbase.client.HTable;
	import org.apache.hadoop.hbase.client.Put;
	import org.apache.hadoop.hbase.client.Result;
	import org.apache.hadoop.hbase.client.ResultScanner;
	import org.apache.hadoop.hbase.client.Scan;
	import org.apache.hadoop.hbase.util.Bytes;
	import org.apache.hadoop.io.IOUtils;
	
	/**
	 * CRUD Operations
	 * 
	 * @author xianglei
	 *
	 */
	public class HBaseOperation {

	public static HTable getHTableByTableName(String tableName) throws Exception {
		// 读取默认配置文件信息
		Configuration configuration = HBaseConfiguration.create();

		//获取表的实例 
		HTable table = new HTable(configuration, tableName);

		return table;

	}

	public void getData() throws Exception {
		String tableName = "user"; // 默认省略->命名框架+名称：default.user / hbase:meta

		HTable table = getHTableByTableName(tableName);

		// 获取一个有着Rowkey的Get 对应shell -->get 'user','10002','info:name'
		Get get = new Get(Bytes.toBytes("10002")); 

		
		// get 查询  指定（ 列蔟 列名）
		get.addColumn(//
				Bytes.toBytes("info"), //
				Bytes.toBytes("name"));
		get.addColumn(//
				Bytes.toBytes("info"), //
				Bytes.toBytes("age"));

		// 获取数据
		Result result = table.get(get);

		// Key : rowkey + cf + c + version
		// Value: value
		// 打印 列蔟：列名->值
		for (Cell cell : result.rawCells()) {
			System.out.println(//
			Bytes.toString(CellUtil.cloneFamily(cell)) + ":" //
			+ Bytes.toString(CellUtil.cloneQualifier(cell)) + " ->" //
			+ Bytes.toString(CellUtil.cloneValue(cell)));
		}

		// 关闭资源
		table.close();

	}

	/**
	 * 建议 把tablename 和 columnfamily 定义为 常量 , 写到一个常量工具类里面 HBaseTableContent
	 * 
	 * 对于插入的列名和值建议使用 Map<String,Obejct> 使用for进行add
	 * 
	 * @throws Exception
	 */
	public void putData() throws Exception {
		String tableName = "user"; 

		HTable table = getHTableByTableName(tableName);

		Put put = new Put(Bytes.toBytes("10004"));

		// put 'user','10004','info:name','zhaoliu'
		put.add(//
				Bytes.toBytes("info"), //
				Bytes.toBytes("name"), //
				Bytes.toBytes("zhaoliu")//
		);

		put.add(//
				Bytes.toBytes("info"), //
				Bytes.toBytes("age"), //
				Bytes.toBytes(25)//
		);
		put.add(//
				Bytes.toBytes("info"), //
				Bytes.toBytes("address"), //
				Bytes.toBytes("shanghai")//
		);
		//放入数据
		table.put(put);

		table.close();
	}

	public void delete() throws Exception {
		String tableName = "user"; // default.user / hbase:meta

		HTable table = getHTableByTableName(tableName);

		Delete delete = new Delete(Bytes.toBytes("10004"));
		/* 删除info列蔟下的address列的所有数据deleteColumn删最新的deleteColumns删除所有的
		 * delete.deleteColumn(Bytes.toBytes("info"),//
		 * Bytes.toBytes("address"));
		 */
		
		// rowkey为10004行info列蔟所有数据
		delete.deleteFamily(Bytes.toBytes("info"));

		table.delete(delete);

		table.close();
	}

	public static void main(String[] args) throws Exception {
		String tableName = "user"; 

		HTable table = null;
		ResultScanner resultScanner = null;

		try {
			table = getHTableByTableName(tableName);

			Scan scan = new Scan();
			
			// Range 范围扫描 包头不包尾
			scan.setStartRow(Bytes.toBytes("10001"));
			scan.setStopRow(Bytes.toBytes("10003")) ;
			// 或者这样写
			//Scan scan2 = new Scan(Bytes.toBytes("10001"),Bytes.toBytes("10003"));
			
			// PrefixFilter
			// PageFilter
			//设置过滤器 会影响查询速度
			//scan.setFilter(filter) ;
			
			//数据读出后缓存到blockcache下一次读就直接从缓存拿
			//每一次获取多少列
			//scan.setCacheBlocks(cacheBlocks);
			//scan.setCaching(caching);
			
			// 查询哪些列哪些列蔟
			//scan.addColumn(family, qualifier)
			//scan.addFamily(family)

			resultScanner = table.getScanner(scan);

			for (Result result : resultScanner) {
				// result里面存的是cell
				System.out.println(Bytes.toString(result.getRow()));
				//System.out.println(result);
				for (Cell cell : result.rawCells()) {
					System.out.println(//
					Bytes.toString(CellUtil.cloneFamily(cell)) + ":" //
					+ Bytes.toString(CellUtil.cloneQualifier(cell)) + " ->" //
					+ Bytes.toString(CellUtil.cloneValue(cell)));
				}
				System.out.println("---------------------------------------");
			}

		} catch (Exception e) {
			e.printStackTrace();
		} finally {

			IOUtils.closeStream(resultScanner);
			IOUtils.closeStream(table);
		}

	}

	}

Hbase架构剖析

![](https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=3299396373,2098068075&fm=26&gp=0.jpg)

Client

* 集群访问入口
* 使用RPC机制与HM和HR进行通信
* 与HM进行通信进行管理类操作
* 与HR进行数据读写操作
* 维护cache加快对Hbase的访问


Zookeeper

* 保证集群只有一个HM，Hbase没有单节点故障的概念，Hbase可以有多个HM但是只有一个去管理
* 存储所有HR的寻址入口 meta
* 实时监控HS的上下线信息，并且通知HM
* 存储Hbase的schema的table元数据


HMaster

* Hbase没有单节点故障的概念，Hbase可以有多个HM但是只有一个去管理
* 管理用户对表的操作
* 管理HR的负载均衡，调整Region分布
* Region Split后，负责新的Region的分布
* HS失效后，负责失效的HS上的Region的迁移工作


HRegion Server

* 维护region处理IO请求
* 切分大Region
* 客户端访问数据不需要HM参与，寻址访问ZK和HS，数据读写访问HS，HM仅仅维护table和Region的元数据。

![](http://i.imgur.com/5tqPWmW.jpg)

Hbase集成MapReduce
	
	/opt/modules/hbase-0.98.6-hadoop2/lib/hbase-common-0.98.6-hadoop2.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/protobuf-java-2.5.0.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/hbase-client-0.98.6-hadoop2.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/hbase-hadoop-compat-0.98.6-hadoop2.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/hbase-server-0.98.6-hadoop2.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/hbase-protocol-0.98.6-hadoop2.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/high-scale-lib-1.1.1.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/zookeeper-3.4.5.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/guava-12.0.1.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/htrace-core-2.04.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/netty-3.6.6.Final.jar

继承所需jar包如上

	export HBASE_HOME=/opt/modules/hbase-0.98.6-hadoop2
	export HADOOP_HOME=/opt/modules/hadoop-2.5.0
	HADOOP_CLASSPATH=`${HBASE_HOME}/bin/hbase mapredcp` $HADOOP_HOME/bin/yarn jar $HBASE_HOME/lib/hbase-server-0.98.6-hadoop2.jar
	此jar下面自带的
	  CellCounter: Count cells in HBase table
	  completebulkload: Complete a bulk data load.
	  copytable: Export a table from local cluster to peer cluster
	  export: Write table data to HDFS.
	  import: Import data written by Export.
	  importtsv: Import data in TSV format.
	  rowcounter: Count rows in HBase table
	  verifyrep: Compare the data from tables in two different clusters. WARNING: It doesn't work for incrementColumnValues'd cells since the timestamp is changed after being appended to the log.
	
	TSV
		tab
		>> student.tsv
		1001	zhangsan	26	shanghai
	CSV
		逗号
		>> student.csv
		1001,zhangsan,26,shanghai
	
	completebulkload     ★ ★ ★ ★ ★ ★ 直接变成hfile
		file  csv
		  |
		hfile
		  |
		load

代码实例

package com.beifeng.senior.hadoop.hbase;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.Mutation;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
import org.apache.hadoop.hbase.mapreduce.TableMapper;
import org.apache.hadoop.hbase.mapreduce.TableReducer;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class User2BasicMapReduce extends Configured implements Tool {
	
	// Mapper Class
	public static class ReadUserMapper extends TableMapper<Text, Put> {

		private Text mapOutputKey = new Text();

		@Override
		public void map(ImmutableBytesWritable key, Result value,
				Mapper<ImmutableBytesWritable, Result, Text, Put>.Context context)
						throws IOException, InterruptedException {
			// get rowkey
			String rowkey = Bytes.toString(key.get());

			// set map的输出key
			mapOutputKey.set(rowkey);

			// --------------------------------------------------------
			Put put = new Put(key.get());

			// iterator
			for (Cell cell : value.rawCells()) {
				// add family : info
				if ("info".equals(Bytes.toString(CellUtil.cloneFamily(cell)))) {
					// add column: name
					if ("name".equals(Bytes.toString(CellUtil.cloneQualifier(cell)))) {
						put.add(cell);
					}
					// add column : age
					if ("age".equals(Bytes.toString(CellUtil.cloneQualifier(cell)))) {
						put.add(cell);
					}
				}
			}

			// context write
			context.write(mapOutputKey, put);
		}

	}

	// Reducer Class
	public static class WriteBasicReducer extends TableReducer<Text, Put, //
	ImmutableBytesWritable> {

		@Override
		public void reduce(Text key, Iterable<Put> values,
				Reducer<Text, Put, ImmutableBytesWritable, Mutation>.Context context)
						throws IOException, InterruptedException {
			for(Put put: values){
			// 输出已经定义好了
				context.write(null, put);
			}
		}

	}

	// Driver
	public int run(String[] args) throws Exception {
		
		// create job
		Job job = Job.getInstance(this.getConf(), this.getClass().getSimpleName());
		
		// set run job class
		job.setJarByClass(this.getClass());
		
		// set job
		Scan scan = new Scan();
		scan.setCaching(500);        // 1 is the default in Scan, which will be bad for MapReduce jobs
		scan.setCacheBlocks(false);  // don't set to true for MR jobs
		// set other scan attrs

		// set input and set mapper
		TableMapReduceUtil.initTableMapperJob(
		  "user",        // input table
		  scan,               // Scan instance to control CF and attribute selection
		  ReadUserMapper.class,     // mapper class
		  Text.class,         // mapper output key
		  Put.class,  // mapper output value
		  job //
		 );
		
		// set reducer and output
		TableMapReduceUtil.initTableReducerJob(
		  "basic",        // output table
		  WriteBasicReducer.class,    // reducer class
		  job//
		 );
		
		job.setNumReduceTasks(1);   // at least one, adjust as required
		
		// submit job
		boolean isSuccess = job.waitForCompletion(true) ;
		
		
		return isSuccess ? 0 : 1;
	}
	
	
	public static void main(String[] args) throws Exception {
		// get configuration
		Configuration configuration = HBaseConfiguration.create();
		
		// submit job
		int status = ToolRunner.run(configuration,new User2BasicMapReduce(),args) ;
		
		// exit program
		System.exit(status);
	}

	}

	export HBASE_HOME=/opt/modules/hbase-0.98.6-hadoop2
	export HADOOP_HOME=/opt/modules/hadoop-2.5.0
	HADOOP_CLASSPATH=`${HBASE_HOME}/bin/hbase mapredcp` $HADOOP_HOME/bin/yarn jar $HADOOP_HOME/jars/hbase-mr-user2basic.jar

Hbase 数据迁移

>Hbase数据来至Logs或者RDMS等。数据迁移的方式有


* Put API
* bilk load tool
* MapReduce job

样本数据
	
	10001	zhangsan35	male	beijing	0109876543
	10002	lisi	32	male	shanghia	0109876563
	10003	zhaoliu	35	female	hangzhou	01098346543
	10004	qianqi	35	male	shenzhen	01098732543

1. importTSV
	
		export HBASE_HOME=/opt/modules/hbase-0.98.6-hadoop2

		export HADOOP_HOME=/opt/modules/hadoop-2.5.0

		HADOOP_CLASSPATH=`${HBASE_HOME}/bin/hbase mapredcp`:${HBASE_HOME}/conf ${HADOOP_HOME}/bin/yarn jar \
		${HBASE_HOME}/lib/hbase-server-0.98.6-hadoop2.jar importtsv \
		-Dimporttsv.columns=HBASE_ROW_KEY,\
		info:name,info:age,info:sex,info:address,info:phone \
		student \
		hdfs://masterm:8020/user/hbase/importtsv

2. bulk load

>


* 消除了对Hbase集群的插入压力
* 提高了Job的运行速度，降低了Job的执行时间

		importtsv默认支持设置bulk load方式写入数据

		export HBASE_HOME=/opt/modules/hbase-0.98.6-hadoop2
	
		export HADOOP_HOME=/opt/modules/hadoop-2.5.0
	
		HADOOP_CLASSPATH=`${HBASE_HOME}/bin/hbase mapredcp`:${HBASE_HOME}/conf \
		        ${HADOOP_HOME}/bin/yarn jar \
		${HBASE_HOME}/lib/hbase-server-0.98.6-hadoop2.jar importtsv \
		-Dimporttsv.columns=HBASE_ROW_KEY,\
		info:name,info:age,info:sex,info:address,info:phone \
		-Dimporttsv.bulk.output=hdfs://master:8020/user/hbase/hfileoutput \
		student2 \
		hdfs://master:8020/user/hbase/importtsv
		
		========================================================	==============
		
		export HBASE_HOME=/opt/modules/hbase-0.98.6-hadoop2
	
		export HADOOP_HOME=/opt/modules/hadoop-2.5.0
	
		HADOOP_CLASSPATH=`${HBASE_HOME}/bin/hbase mapredcp`:${HBASE_HOME}/conf \
		${HADOOP_HOME}/bin/yarn jar \
		${HBASE_HOME}/lib/hbase-server-0.98.6-hadoop2.jar \
		completebulkload \
		hdfs://hadoop-senior.ibeifeng.com:8020/user/beifeng/hbase/hfileoutput \
		student2

是mv过去的不是拷贝的！
