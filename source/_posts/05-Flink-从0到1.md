title: 从0到1学会Apache Flink-创建项目
comments: true
date: 2019-03-04 17:05:14
categories: flink
tags:
	- 入门
---

######环境准备
已安装了Maven 3.0.4（或更高版本）和Java 8.x.

######创建项目:
---
* 方法一

命令行方式创建maven项目:
>mvn archetype:generate                                
>      -DarchetypeGroupId=org.apache.flink              
>      -DarchetypeArtifactId=flink-quickstart-java      
>      -DarchetypeVersion=1.7.1

<!--more-->

建设项目:
>mvn clean package

打包完成后在target目录下会生成jar文件.

* 方法二

使用开发工具创建maven项目:

点击 file > new > project

![](https://i.imgur.com/aVfKDq0.png)

点击maven > create from archetype > flink-quickstart-java
![](https://i.imgur.com/EjRJw7D.png)

如果没有flink-quickstart-java点击 add archetype

![](https://i.imgur.com/1LpWaUw.png)

一直next直到finish,项目结构如下:

![](https://i.imgur.com/YsO2OOh.png)




