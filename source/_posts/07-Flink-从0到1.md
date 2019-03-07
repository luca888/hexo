title: 从0到1学会Apache Flink-调试Flink程序
comments: true
date: 2019-03-04 10:05:14
categories: flink
tags:
	- 入门
---
###### 安装flink本地模式 ######
以win10为例:

为了能够运行Flink，唯一的要求是安装一个有效的Java 8.x.

下载Flink:
>https://flink.apache.org/downloads.html#binaries

<!--more-->

解压:
>tar -zxvf flink-*.tgz

> cd flink-1.7.1

启动:

打开cmd:
>cd flink-1.7.1\bin

>start-cluster.bat 

修改程序获得本地环境:
>      final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();


替换为:

>       final StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironment();

删除:
>       env.execute("Socket Window WordCount");


debug运行程序.
