## 高级Hadoop
Hadoop 2.x 部署 

* Local Mode
	* Distributed Mode
		* 伪分布式
			一台机器，运行所有的守护进程，
			从节点DataNode、NodeManager
		* 完全分布式
			有多个从节点
			DataNodes
			NodeManagers
			配置文件
				$HADOOP_HOME/etc/hadoop/slaves

----
![](https://i.imgur.com/EFPblZO.png)
##  完全分布式
	* 克隆机器
	* 修改ip
	* 修改主机名 
	* 设置好映射 
	
##  Hadoop文件配置

* hdfs 

		 * hadoop-env.sh
		 * core-site.xml
		 * hdfs-site.xml
		 * slaves
* yarn 

		 * yarn-env.sh
		 * yarn-site.xml
		 * slaves
* mapredue 

		 * mapred-env.sh
		 * mapred-site.xml	
		 
## SSH无密码登录 
## 集群基准测试（面试）

* 基本测试 

		&服务启动，是否可用，简单的应用
		&HDFS创建和删除是否能够成功
		&yarn run jar
		&mapreduce
* 基准测试 
		
			测试集群的性能
			&hdfs:写数据、读数据

* 监控集群 
	
			&cloudera
			&Cloudera Manager
				部署安装集群
				监控集群
				配置同步集群
				预警
## ntp同步时间 
* 找一台机器 

		时间服务器
* 所有的机器与这台机器时间进行定时的同步 

		比如，每日十分钟，同步一次时间

# rpm -qa|grep ntp

*  vi /etc/ntp.conf 
![](https://i.imgur.com/4xfVM91.png)
![](https://i.imgur.com/297Y3Z0.png)
![](https://i.imgur.com/5KDC2L0.png)
* vi /etc/sysconfig/ntpd
![](https://i.imgur.com/sUm9O7B.png)
* SYNC_HWCLOCK=yes
* [root@hadoop-senior hadoop-2.5.0]# service ntpd status
ntpd is stopped
* [root@hadoop-senior hadoop-2.5.0]# service ntpd start
Starting ntpd: [  OK  ]
* [root@hadoop-senior hadoop-2.5.0]# chkconfig ntpd on

## 分布式服务框架Zookeeper

## HA配置
--- 

配置HA要点 

* share edits
	JournalNode （日志节点）至少三个
* NameNode
	Active，Standby
* Client
	Proxy
* fence 
	
		同一时刻仅仅有一个NameNode对外提供服务
		使用的方式sshfence
			两个NameNode之间能够ssh无密码登录
			131(NameNode) ssh -> 132
			132(NameNode) ssh -> 131

规划集群 

![](https://i.imgur.com/uQMHpb1.png)


配置HA后不需要secondarynamenode了，为什么呢？配置HA后备份的Namenode完全替代了它。
       hdfs-site.xml
		<property>
		  <name>dfs.nameservices</name>
		  <value>mycluster</value>
		</property>

          // 命名空间

		<property>
		  <name>dfs.ha.namenodes.mycluster</name>
		  <value>nn1,nn2</value>
		</property>

        // 管理哪两个nn

		<property>
		  <name>dfs.namenode.rpc-address.mycluster.nn1</name>
		  <value>machine1.example.com:8020</value>
		</property>
		<property>
		  <name>dfs.namenode.rpc-address.mycluster.nn2</name>
		  <value>machine2.example.com:8020</value>
		</property>


        // nn在哪台机器

         // 配置两个nn访问端口Web
			<property>
			  <name>dfs.namenode.http-address.mycluster.nn1</name>
			  <value>machine1.example.com:50070</value>
			</property>
			<property>
			  <name>dfs.namenode.http-address.mycluster.nn2</name>
			  <value>machine2.example.com:50070</value>
			</property>


           // 共享日志的目录
			<property>
			  <name>dfs.namenode.shared.edits.dir</name>
			  <value>qjournal://node1.example.com:8485;node2.example.com:8485;node3.example.com:8485/mycluster</value>
			</property> 

           // 配置客户端访问
			<property>
			  <name>dfs.client.failover.proxy.provider.mycluster</name>
			  <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
			</property>

           //配置隔离方式 ssh隔离

		    <property>
		      <name>dfs.ha.fencing.methods</name>
		      <value>sshfence</value>
		    </property>
		
		    <property>
		      <name>dfs.ha.fencing.ssh.private-key-files</name>
		      <value>/home/exampleuser/.ssh/id_rsa</value>
		    </property> 

        使用的方式sshfence
		两个NameNode之间能够ssh无密码登录
		131(NameNode) ssh -> 132
		132(NameNode) ssh -> 131 

         // 配置日志文件写在本地那个目录
			<property>
			  <name>dfs.journalnode.edits.dir</name>
			  <value>
                 /opt/app/hadoop-2.7.3/data/dfs/jn
               </value>
			</property>

			//配置 core-site.xml
			
			 <property>
			  <name>fs.defaultFS</name>
			  <value>hdfs://mycluster</value>
			</property>

最后向其他两个节点同步文件

启动顺序

* ./hadoop-daemon.sh start journalnode（每个）
* bin/hdfs namenode -format（nn1）
* sbin/hadoop-daemon.sh start namenode
* ./hdfs namenode -bootstrapStandby（nn2）
*  ./hadoop-daemon.sh start datanode
*  ./hdfs haadmin -transitionToActive nn1

##### 千万不要手贱乱格式化，贼烦

## ZK实现自动故障转换

![](https://i.imgur.com/mV8KwnH.png)

* hdfs-site.xml 

          // 启动故障自动转移
		 <property>
		   <name>dfs.ha.automatic-failover.enabled</name>
		   <value>true</value>
		 </property>

        // 依赖ZK  需要知道ZK在哪
* core-site.xml

		<property>
		   <name>ha.zookeeper.quorum</name>
		   <value>zk1.example.com:2181,zk2.example.com:2181,zk3.example.com:2181</value>
		 </property>

![](https://i.imgur.com/PE3VzaF.png)
start-dfs.sh 启动一切  包括zkFc
![](https://i.imgur.com/DDrWG02.png)

zk挂掉不会影响集群，只是不能故障转移了，失去了对集群的选举

## HA太简单了

## Federation

![](https://i.imgur.com/mkONCnL.png)

NameNode
	
	能不能有多个NameNode
	NameNode 			NameNode 				NameNode
	元数据				元数据					元数据
	log				machine					电商数据/话单数据

三个把数据存在所有的DataNodes节点上

## 集中式缓存管理 

HDFS允许用户将一部分文件或目录缓存在HDFS中，Namenode会通知拥有对应快的DataNodes将其缓存在datanode的内存中

## 分布式拷贝

数据迁移，如将测试集群的数据拷贝到生产集群 

http://hadoop.apache.org/docs/stable/hadoop-distcp/DistCp.html

拷贝命令： 

 hadoop distcp -i hftp://sourceFS:50070/src hdfs://destFS:8020/dest
 
## Yarn的HA


![](http://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/images/rm-ha-overview.png) 


配置 

		<property>
		  <name>yarn.resourcemanager.ha.enabled</name>
		  <value>true</value>
		</property>
		<property>
		  <name>yarn.resourcemanager.cluster-id</name>
		  <value>cluster1</value>
		</property>
		<property>
		  <name>yarn.resourcemanager.ha.rm-ids</name>
		  <value>rm1,rm2</value>
		</property>
		<property>
		  <name>yarn.resourcemanager.hostname.rm1</name>
		  <value>master1</value>
		</property>
		<property>
		  <name>yarn.resourcemanager.hostname.rm2</name>
		  <value>master2</value>
		</property>
		<property>
		  <name>yarn.resourcemanager.webapp.address.rm1</name>
		  <value>master1:8088</value>
		</property>
		<property>
		  <name>yarn.resourcemanager.webapp.address.rm2</name>
		  <value>master2:8088</value>
		</property>
		<property>
		  <name>yarn.resourcemanager.zk-address</name>
		  <value>zk1:2181,zk2:2181,zk3:2181</value>
		</property>

命令
http://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/ResourceManagerHA.html

		 $ yarn rmadmin -getServiceState rm1
		 active
		
		 $ yarn rmadmin -getServiceState rm2
		 standby



		 $ yarn rmadmin -transitionToStandby rm1
		 Automatic failover is enabled for org.apache.hadoop.yarn.client.RMHAServiceTarget@1d8299fd
		 Refusing to manually manage HA state, since it may cause
		 a split-brain scenario or other incorrect state.
		 If you are very sure you know what you are doing, please
		 specify the forcemanual flag.

## RM restart


