>这一节我能说多少说多少吧，信息量太大。Hbase的官方文档我感觉写的挺乱糟的，反正用起来没有Hadoop的舒服。最终还是一个原则不会再查！
## Hbase架构
---
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125214753900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)

## Hbase Region定位
---
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125214839478.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2018112521490568.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)

## Hbase表创建
---
>help 'create'  引号‘’是必须的

![](https://i.imgur.com/INRSS6Q.png)

* namespace 命名空间类似与数据库的概念。ns1：t1指定ns1命名空间下的t1表

**如何创建一个命名空间？**

![](https://i.imgur.com/zmm8Rez.png)

* create_namespace 'ns1'
* list_namespace 'ns1'

**如何创建一张表？**
>｛｝里写的是列蔟的名字
* create 'ns1:t1', 'cf'  =  create 'ns1:t1', {NAME => 'f1'}
* create 'ns1:t2', {NAME => 'cf1'}, {NAME => 'cf2'}, {NAME => 'cf3'} = create 'ns1:t3', 'cf1', 'cf2', 'cf3'

**查看某命名空间下的表**

* list_namespace_tables 'ns1'
* describe 'ns1:t3'
![](https://i.imgur.com/oA9I1N9.png)

## Hbase表设计
---
>能不能再建表的时候多建几个Region？你知道为什么要多建几个Region吗？HBase 表的预分区？
>
首先我们都知道，当我们创建一张表的时候，Hbase会自动为表分配一个Region（不知道的看我前面的文章），那么此时问题就来了，当我们需要向这张表导入大量的数据（file/datas -> hfile  -> bulk load into hbase table）的时候，RegionServer会管理Region进行split，这时候RegionServer的负载是很大的，效率变低的同时可靠性也会出现问题，所以给出如下解决方案：

	解决方案：
		创建表时，多创建一些Region（依据表的数据rowkey进行设计，结合业务）
	五个Region
		被多个RegionServer进行管理
		要点：
			在插入数据时，会向五个Region中分别插入对应的数据，均衡了
这也就是对Hbase表的预分区处理。---->Region划分，依赖于rowkey，所谓的预分区就是预先预估一些rowkey。

**预分区方法**		

* 方式一： create 'bflogs', 'info', SPLITS => ['20151001000000000', '20151011000000000', '20151021000000000']

![](https://i.imgur.com/y4UaAm6.png)

* 方式一：create 'bflogs2', 'info', SPLITS_FILE => '/opt/datas/bflogs-split.txt'

![](https://i.imgur.com/Ig8LV7Z.png)

* 方式三：create 't11', 'f11', {NUMREGIONS => 2, SPLITALGO => 'HexStringSplit'}
* create 't12', 'f12', {NUMREGIONS => 4, SPLITALGO => 'UniformSplit'}
>  NUMREGIONS region数量，SPLITALGO自定义的类，rowkey他给你生成不太实用。

**Hbase表属性介绍**
>调几个大头的说一说，其他的就百度去吧，反正我不懂，哈哈。

当我们 desc 'user' 查看到一个表有以下属性：
	 'user', 
	{
		NAME => 'info', 
		DATA_BLOCK_ENCODING => 'NONE', 
		BLOOM FILTER => 'ROW', 
		REPLICATION_SCOPE => '0',
		**VERSIONS => '1',**
		**COMPRESSION => 'NONE',**
		MIN_VERSIONS => '0', 
		TTL => 'FOREVER',
		KEEP_DELETED_CELLS => 'false', 
		BLOCKSIZE => '65536', 
		**IN_MEMORY => 'false',**
		**BLOCKCACHE => 'true'**
	}
   
  * VERSIONS --->数据版本号（多版本你懂的）
  * COMPRESSION--->Hbase在hdfs上存储的文件压缩格式
  * IN_MEMORY       BLOCKCACHE  ---->缓存

这也就又得说Snappy的压缩配置了，其实我没有去配过，所以建议大家还是看看官方文档怎么说的吧
                          
**[Snappy](http://hbase.apache.org/book.html#compression)压缩配置**

* hadoop checknative 

> 查看hadoop文件压缩格式


* 配置HBase
> hadoop-snappy jar  -> 放入到Hbase的lib目录

* 需要将本地库 native 放到Hbase lib下


	* Hbaselib目录下创建一个 native
	* 设置软连接 ln -s /opt/moudles/hadoop/lib/native ./Linux-amd-64

* 重启Hbase集群
* 设置hbase-site.xml
>内容不详细，见谅！

**使用Snnapy压缩**
>
create 'test2',{NAME=>'CF2',COMPRESS=>'SNAPPY'}

**BLOCKCHACHE**
>读->memstore->查不到->blockcache->查不到->磁盘（并且把结果放入BlockCache）blockcache采用的是LRU算法，达到上限后回淘汰老的一批数据。
	
	RegionServer   -  12G
		>> MemStore    40%
			write
		>> BlockCache数据缓存  40%
			read
		>> other       20%

读的时候会读到缓存里面，下一次读更快。

		当用户去读取数据
			>> MemStore
			>> BlockCache    ->  每个RegionServer只有一个BlockCache
			>> hfile
			>> meger
			返回数据集



**IN_MEMORY**

>保存meta表元数据，如果将数据量很大的用户表设置为inmemory的话，可能会导致meta表缓存失败，进而对整个集群性能产生影响。


## Hbase表管理

---
**Hbase写入数据**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125213239109.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)
**Hbase读取数据**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125213334679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)

## Hbase集成Hive使用
---
>把Hbase里面的数据拿出来给Hive分析。

	两种方式
	>> 管理表
		创建hive表的时候，指定数据存储在hbase表中。
	CREATE TABLE hbase_table_1(key int, value string) 
	STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
	WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf1:val")
	TBLPROPERTIES ("hbase.table.name" = "xyz");
	
	
	
	>> 外部表
		现在已经存在一个HBase表，需要对表中数据进行分析。
	CREATE EXTERNAL TABLE hbase_user(id int, name string,age int) 
	STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
	WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,info:name,info:age")
	TBLPROPERTIES ("hbase.table.name" = "user");
	
	本质：
		Hive就是HBase客户端。
		是不是需要一些配置，jar包

	export HBASE_HOME=/opt/modules/hbase-0.98.6-hadoop2
	export HIVE_HOME=/opt/modules/hive-0.13.1/lib
	ln -s $HBASE_HOME/lib/hbase-server-0.98.6-hadoop2.jar $HIVE_HOME/hbase-server-0.98.6-hadoop2.jar
	ln -s $HBASE_HOME/lib/hbase-client-0.98.6-hadoop2.jar $HIVE_HOME/hbase-client-0.98.6-hadoop2.jar
	ln -s $HBASE_HOME/lib/hbase-protocol-0.98.6-hadoop2.jar $HIVE_HOME/hbase-protocol-0.98.6-hadoop2.jar
	ln -s $HBASE_HOME/lib/hbase-it-0.98.6-hadoop2.jar $HIVE_HOME/hbase-it-0.98.6-hadoop2.jar 
	ln -s $HBASE_HOME/lib/htrace-core-2.04.jar $HIVE_HOME/htrace-core-2.04.jar
	ln -s $HBASE_HOME/lib/hbase-hadoop2-compat-0.98.6-hadoop2.jar $HIVE_HOME/lib/hbase-hadoop2-compat-0.98.6-hadoop2.jar
	ln -s $HBASE_HOME/lib/hbase-hadoop-compat-0.98.6-hadoop2.jar $HIVE_HOME/lib/hbase-hadoop-compat-0.98.6-hadoop2.jar
	ln -s /opt/modules/hbase-0.98.6-hadoop2/lib/high-scale-lib-1.1.1.jar /opt/modules/hive-0.13.1/lib/high-scale-lib-1.1.1.jar

hive-site.xml

![](https://i.imgur.com/Cb4dxml.png)

## Hbase继承Sqoop 
---
>示例模板
>
	sqoop import \
	--connect jdbc:mysql://master:3306/testdb\
	--username root -P \
	--tale customercontactinfo \
	--columns "customernum,contactinfo" \
	--hbase-table customercontactindo \
	--column-family ContactInfo \
	--hbase-row-key customernum -m 1

**附录：**

部分图文来至<<Hadoop 海量数据处理>>。


---
## 总结

总是学不会 ，再聪明一点，记得自我保护， 必要时候讲些官方文档！😂😂😂 您要是都看到这了，恭喜你看懵逼了吧。