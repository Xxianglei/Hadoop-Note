>这一讲主要是对Hbase JavaApi使用的介绍，编程还是挺简单的，重点在于理解编程实现的过程。其次深入讲解了Hbase的架构。以及Hbase如何实现数据的迁移。

## Hbase Java API
>Hbase提供了java开发的接口，可以使用java语言对Hbase数据库进行操作。
---
* jar包依赖 server client
* 配置文件导入 hdfs-seite.xml core-site.xml hbase-site.xml

>代码内容很简单，我也打上了注释就不多说了。

实例代码:
```java
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
```

## Hbase架构组件剖析
-----

![](http://i.imgur.com/5tqPWmW.jpg))

* Client

	* 集群访问入口
	* 使用RPC机制与HM和HR进行通信
	* 与HM进行通信进行管理类操作
	* 与HR进行数据读写操作
	* 维护cache加快对Hbase的访问


- Zookeeper

	* 保证集群只有一个HM，Hbase没有单节点故障的概念，Hbase可以有多个HM但是只有一个去管理
	* 存储所有HR的寻址入口 meta
	* 实时监控HS的上下线信息，并且通知HM
	* 存储Hbase的schema的table元数据


* HMaster

	* Hbase没有单节点故障的概念，Hbase可以有多个HM但是只有一个去管理
	* 管理用户对表的操作
	* 管理HR的负载均衡，调整Region分布
	* Region Split后，负责新的Region的分布
	* HS失效后，负责失效的HS上的Region的迁移工作


* HRegion Server

	* 维护region处理IO请求
	* 切分大Region
	* 客户端访问数据不需要HM参与，寻址访问ZK和HS，数据读写访问HS，HM仅仅维护table和Region的元数据。



## Hbase集成MapReduce
---
>Hbase继承所需的jar包

```	shell
	/opt/modules/hbase-0.98.6-hadoop2/lib/hbase-common-0.98.6-hadoop2.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/protobuf-java-2.5.0.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/hbase-client-0.98.6-hadoop2.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/hbase-hadoop-compat-0.98.6-hadoop2.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/hbase-server-0.98.6-hadoop2.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/hbase-protocol-0.98.6-hadoop2.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/high-scale-lib-1.1.1.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/zookeeper-3.4.5.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/guava-12.0.1.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/htrace-core-2.04.jar:/opt/modules/hbase-0.98.6-hadoop2/lib/netty-3.6.6.Final.jar
```

**使用方法：**

	export HBASE_HOME=/opt/modules/hbase-0.98.6-hadoop2
	export HADOOP_HOME=/opt/modules/hadoop-2.5.0
	HADOOP_CLASSPATH=`${HBASE_HOME}/bin/hbase mapredcp` $HADOOP_HOME/bin/yarn jar $HBASE_HOME/lib/hbase-server-0.98.6-hadoop2.jar
	
	 // 此jar下面自带的
	  CellCounter: Count cells in HBase table
	  completebulkload: Complete a bulk data load.
	  copytable: Export a table from local cluster to peer cluster
	  export: Write table data to HDFS.
	  import: Import data written by Export.
	  importtsv: Import data in TSV format.
	  rowcounter: Count rows in HBase table
	  verifyrep: Compare the data from tables in two different clusters. WARNING: It doesn't work for incrementColumnValues'd cells since the timestamp is changed after being appended to the log.
	
	TSV 格式
		tab分隔
		>> student.tsv
		1001	zhangsan	26	shanghai
	CSV 格式
		逗号分隔
		>> student.csv
		1001,zhangsan,26,shanghai
	
	completebulkload     ★ ★ ★ ★ ★ ★ 直接变成hfile
		file  csv
		  |
		hfile
		  |
		load

**代码实例**
```java
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
```
运行：

	export HBASE_HOME=/opt/modules/hbase-0.98.6-hadoop2
	export HADOOP_HOME=/opt/modules/hadoop-2.5.0
	HADOOP_CLASSPATH=`${HBASE_HOME}/bin/hbase mapredcp` $HADOOP_HOME/bin/yarn jar $HADOOP_HOME/jars/hbase-mr-user2basic.jar

## Hbase 数据迁移
---

Hbase数据来至于Logs或者RDMS等。数据迁移的方式有
* Put API
* bilk load tool
* MapReduce job

准备样本数据
	
			10001	zhangsan35	male	beijing	0109876543
			10002	lisi	32	male	shanghia	0109876563
			10003	zhaoliu	35	female	hangzhou	01098346543
			10004	qianqi	35	male	shenzhen	01098732543

1. **importTSV**
	
		export HBASE_HOME=/opt/modules/hbase-0.98.6-hadoop2

		export HADOOP_HOME=/opt/modules/hadoop-2.5.0

		HADOOP_CLASSPATH=`${HBASE_HOME}/bin/hbase mapredcp`:${HBASE_HOME}/conf ${HADOOP_HOME}/bin/yarn jar \
		${HBASE_HOME}/lib/hbase-server-0.98.6-hadoop2.jar importtsv \
		-Dimporttsv.columns=HBASE_ROW_KEY,\
		info:name,info:age,info:sex,info:address,info:phone \
		student \
		hdfs://masterm:8020/user/hbase/importtsv

2. **bulk load方式快速加载巨量数据**

>1、准备数据文件
Bulk Load的第一步。会执行一个Mapreduce作业，当中使用到了HFileOutputFormat输出HBase数据文件：StoreFile。
HFileOutputFormat的作用在于使得输出的HFile文件能够适应单个region。使用TotalOrderPartitioner类将map输出结果分区到各个不同的key区间中，每一个key区间都相应着HBase表的region。
2、导入HBase表
第二步使用completebulkload工具将第一步的结果文件依次交给负责文件相应region的RegionServer，并将文件move到region在HDFS上的存储文件夹中。一旦完毕。将数据开放给clients。
假设在bulk load准备导入或在准备导入与完毕导入的临界点上发现region的边界已经改变，completebulkload工具会自己主动split数据文件到新的边界上。可是这个过程并非最佳实践，所以用户在使用时须要最小化准备导入与导入集群间的延时，特别是当其它client在同一时候使用其它工具向同一张表导入数据。

bulkload的优点

* 消除了对Hbase集群的插入压力
* 提高了Job的运行速度，降低了Job的执行时间

使用方法
	
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
		
		=======================================================================
		
		export HBASE_HOME=/opt/modules/hbase-0.98.6-hadoop2
	
		export HADOOP_HOME=/opt/modules/hadoop-2.5.0
	
		HADOOP_CLASSPATH=`${HBASE_HOME}/bin/hbase mapredcp`:${HBASE_HOME}/conf \
		${HADOOP_HOME}/bin/yarn jar \
		${HBASE_HOME}/lib/hbase-server-0.98.6-hadoop2.jar \
		completebulkload \
		hdfs://hadoop-senior.ibeifeng.com:8020/user/beifeng/hbase/hfileoutput \
		student2

由于BulkLoad是绕过了Write to WAL，Write to MemStore及Flush to disk的过程，所以并不能通过WAL来进行一些复制数据的操作。

----

## 总结

这一块的东西挺多的。详见官网[Hbase](http://abloz.com/hbase/book.html#perf.writing)，我还得去看看官网的内容再来总结。