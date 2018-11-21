## Hue

>Hue is a Web interface for analyzing data with Apache Hadoop。[Hue](http://gethue.com/)是由Cloudera开源的大数据WEB工具。

**功能特性**

![](https://i.imgur.com/4YCXkzH.png)

* 免费的开源的
* 可以用于生产环境的
* 百分百兼容

**Hue架构**

![](https://i.imgur.com/EsYoORQ.png)

**Hue安装**

依赖环境准备

![](https://i.imgur.com/jo7tUZ8.png)

注意：在联网状态下装，避免问题

![](https://i.imgur.com/KE9V5qz.png)

修改基础配置

/opt/cdh-5.3.6/hue-3.7.0-cdh5.3.6/desktop/conf

![](https://i.imgur.com/z5rDSPO.png)

	 # Set this to a random string, the longer the better.
	  # This is used for secure hashing in the session store.
	  secret_key=jFE93j;2[290-eiw.KEiwN2s3['d;/.q[eIW^y#e=+Iei*@Mn<qW5o
	
	  # Webserver listens on this address and port
	  http_host=master
	  http_port=8888
	
	  # Time zone name
	  time_zone=Asia/Shanghai

注意：Hue依赖与Python，我们需要装上python


