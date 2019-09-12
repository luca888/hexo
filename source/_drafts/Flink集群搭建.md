title: 非root用户搭建Flink集群

---

flink集群需要运行在jdk上,先安装jdk.

配置Linux系统的hostname:
	
	100.0.0.111	PVT01
	100.0.0.112	PVT02
	100.0.0.113	PVT03
	100.0.0.114	PVT04
	100.0.0.115	PVT05


把flink安装在/opt/flink目录下:
切换到root用户:
> sudo su -

进入opt目录:
> cd /opt

创建flink文件夹:
> mkdir flink

把flink文件夹的权限设为777,便于文件的操作:
> chmod -R 777 flink

把flink安装包上传到flink目录下,用到的安装包版本是flink-1.7.2-bin-hadoop27-scala_2.11.tgz,解压安装包.
> tar -zxvf flink-1.7.2-bin-hadoop27-scala_2.11.tgz

解压后得到文件夹:
> flink-1.7.2

配置文件配置conf/flink-conf.yaml:

	jobmanager.rpc.address：master节点的IP地址 
	taskmanager.heap.mb：每个TaskManager可用的总内存 
	taskmanager.numberOfTaskSlots：每台机器上可用CPU的总数 
	parallelism.default：每个Job运行时默认的并行度,最大值为全部task节点slots的总和. 
	taskmanager.tmp.dirs：临时目录 
	jobmanager.heap.mb：每个节点的JVM能够分配的最大内存 
	jobmanager.rpc.port: 6123 
	jobmanager.web.port: 8081

Linux机器逻辑cpu个数查询:
> cat /proc/cpuinfo| grep "processor"| wc -l

内存大小查询:
> cat /proc/meminfo |grep MemTotal

最终配置:

    jobmanager.rpc.address:PVT01
    taskmanager.heap.mb：2048
    taskmanager.numberOfTaskSlots：8
    parallelism.default：15
    jobmanager.heap.mb：2048
    jobmanager.web.port: 8081
    jobmanager.rpc.port: 6123

配置主节点masters:
> PVT01:8081

配置从节点slaves:

	PVT02
	PVT03
	PVT04
	PVT05

在其他节点创建flink文件夹.
把配置完成的flink-1.7.2文件夹通过spc发送到其他节点:
> scp -r /opt/flink/flink-1.7.2 user@PVT02:/opt/flink
> scp -r /opt/flink/flink-1.7.2 user@PVT03:/opt/flink
> scp -r /opt/flink/flink-1.7.2 user@PVT04:/opt/flink
> scp -r /opt/flink/flink-1.7.2 user@PVT05:/opt/flink

配置PVT01上普通用户user通过ssh免密登陆其他节点
切换到user用户,生成密钥
> ssh-keygen -t rsa

一直按回车,会在/home/user目录下生成.ssh文件夹,文件夹内包含id_rsa和id_rsa.pub

把id_rsa.pub分发到其他节点的/home/user/.ssh目录下,如果.ssh目录不存在先建文件夹.
> scp /home/user/.ssh/id_rsa.pub user@PVT02:/home/user/.ssh
> scp /home/user/.ssh/id_rsa.pub user@PVT03:/home/user/.ssh
> scp /home/user/.ssh/id_rsa.pub user@PVT04:/home/user/.ssh
> scp /home/user/.ssh/id_rsa.pub user@PVT05:/home/user/.ssh

在其他节点的/home/user/.ssh目录下创建authorized_keys文件
> touch /home/fmtt/.ssh/authorized_keys

把id_rsa.pub的内容追加到authorized_keys文件中
> cat /home/fmtt/.ssh/id_rsa.pub  >> /home/fmtt/.ssh/authorized_keys

用user用户在PVT01上启动集群:
> cd /opt/flink/flink-1.7.2
> ./bin/start-cluster.sh

在浏览器访问100.0.0.111:8081可以看到flink的UI页面.