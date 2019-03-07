title: 从0到1学会Apache Flink-单机模式安装
comments: true
date: 2019-03-04 13:05:14
categories: flink
tags:
	- 入门
---

###### 环境准备 ######
为了能够运行Flink，唯一的要求是安装一个有效的Java 8.x.

您可以通过发出以下命令来检查Java的正确安装：

> java -version

如果你有Java 8，输出将如下所示：

>java version "1.8.0_111"

>Java(TM) SE Runtime Environment (build 1.8.0_111-b14)

>Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)

<!--more-->

###### 下载 ######
下载页面:
> https://flink.apache.org/downloads.html#binaries

下载后进行解压:
> tar -zxvf flink-*.tgz

> cd flink-1.7.1


###### 启动 ######
>./bin/start-cluster.sh  # Start Flink

打开浏览器访问localhost:8081,出现如下页面启动成功:
![](https://i.imgur.com/sP1OZQn.png)

也可以通过检查logs目录中的日志文件来验证系统是否正在运行：
>$ tail log/flink-*-standalonesession-*.log

>INFO ... - Rest endpoint listening at localhost:8081

>INFO ... - http://localhost:8081 was granted 

>leadership ...

>INFO ... - Web frontend listening at http://localhost:8081.

>INFO ... - Starting RPC endpoint for StandaloneResourceManager at akka://flink/user/resourcemanager .

>INFO ... - Starting RPC endpoint for StandaloneDispatcher at akka://flink/user/dispatcher .

>INFO ... - ResourceManager akka.tcp://flink@localhost:6123/user/resourcemanager was granted leadership ...

>INFO ... - Starting the SlotManager.

>INFO ... - Dispatcher akka.tcp://flink@localhost:6123/user/dispatcher was granted leadership ...

>INFO ... - Recovering all persisted jobs.

>INFO ... - Registering TaskManager ... under ... at the SlotManager.

###### 停止 ######
---
>./bin/stop-cluster.sh