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

![](https://i.imgur.com/hC39Y72.png)

	* zookeeper
	* namenode
	* datanode
	* HM
* 访问Web页面

![](https://i.imgur.com/GMr57OF.png)
![](https://i.imgur.com/lZ4pvpw.png)

各个目录的含义（官方文档）：

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

