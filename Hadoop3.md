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

