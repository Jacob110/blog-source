title: Zookeeper学习 安装部署
date: 2015-10-31 10:42:38
category:
	- Zookeeper
	- Java
tags:
	- zookeeper
	- javaweb
	- java
---

ZooKeeper是Hadoop的正式子项目，它是一个针对大型分布式系统的可靠协调系统，提供的功能包括：配置维护、名字服务、分布式同步、组服务等。ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

<!--more-->

官网 http://zookeeper.apache.org/

下载 http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.6/

## 单机模式配置
windows为例
解压后重命名为 zookeeper ，修改zoo.cfg，修改如下

~~~
tickTime=2000
initLimit=10
syncLimit=5
dataDir=D:\\Java\\zookeeper\\data
dataLogDir=D:\\Java\\zookeeper\\logs
clientPort=7011
~~~

需要手动创建两个目录 dataDir 和 dataLogDir，设置端口为7011

启动server
[](zoo-001)

启动客户端连接
~~~
D:\Java\zookeeper\bin>zkCli.cmd -server localhost:7011
~~~

查看节点
~~~
[zk: localhost:7011(CONNECTED) 9] ls /
[zkclitest, zookeeper]
~~~

创建节点
~~~
[zk: localhost:7011(CONNECTED) 11] create /testcli hello
Created /testcli
~~~
再查看
~~~
[zk: localhost:7011(CONNECTED) 12] ls /
[testcli, zkclitest, zookeeper]
~~~
退出客户端
~~~
[zk: localhost:7011(CONNECTED) 13] quit
Quitting...
2015-10-30 14:59:57,525 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x150b78
055c70000 closed
2015-10-30 14:59:57,525 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread
@512] - EventThread shut down
~~~

## 集群部署

zookeeper不仅可以单击提供服务，同时也支持多机组成集群来提供服务。集群模式有两种

- 伪集群：一台物理机器运行多个zookeeper实例
- 集群：在多个物理机上运行，每个物理机只运行一个zookeeper实例

简单说下集群配置方式

环境：
- linux
- 172.28.20.101，172.28.20.102  两台机器

测试资源有限，在上述两台机器各部署两个zookeeper 实例。

### 下载解压
将之前本机(windows)下载好的zookeeper上传到服务器，并解压重命名为zk2
~~~
tar -xzvf zookeeper-3.4.6.tar.gz
mv zookeeper-3.4.6 zk2
~~~

### 配置zoo.cfg
打开bin目录，将zoo_sample.cfg 备份并改名为zoo.cfg
~~~
cp zoo_sample.cfg zoo.cfg
~~~

zoo.cfg 是zookeeper的核心配置文件，zk启动时候是通过配置文件的参数来初始化的。下面是本文的配置，

~~~
[hadoop@dc1 conf]$ vi zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/hadoop/zk2/data
dataLogDir=/home/hadoop/zk2/logs
clientPort=7012

server.1=dc1:7021:7031
server.2=dc2:7021:7031
server.3=dc2:7022:7032
server.4=dc1:7022:7032

~~~

- dataDir：数据存储目录
- dataLogDir：日志存储目录
这两个目录需要手动先创建好，否则会使用默认的目录，可能会与其他实例重复
- clientPort:服务端口，同一台机器 端口不能重复
- server.A=B:C:D
A: zk服务编号，数字
B：服务所在IP地址
C：zk leader选举端口
D：zk 各实例之间通信端口

除了上述coo.cfg配置文件外，集群模式需要在dataDir目录下配置一个文件myid，如果没有，需要新建。文件内容就是上面server.A中A的数值，表示当前启动的server编号。

这样101上的一个zk实例就配置好了，参照这个配置，将另外3个实例也配置好。

### 启动server
分别启动四个实例

启动101机器上两个zk实例

启动zk1

~~~
[hadoop@dc1 bin]$ ./zkServer.sh start
JMX enabled by default
Using config: /home/hadoop/zk1/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
~~~

启动zk2
~~~
[hadoop@dc1 bin]$ ./zkServer.sh start
JMX enabled by default
Using config: /home/hadoop/zk2/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
~~~
jps 看下进程
~~~
[hadoop@dc1 bin]$ jps
9969 QuorumPeerMain
10508 QuorumPeerMain
10542 Jps
~~~

用客户端连到server

~~~
[hadoop@dc1 bin]$ ./zkCli.sh -server localhost:7012
Connecting to localhost:7012
2015-10-29 10:00:54,496 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT
2015-10-29 10:00:54,515 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:host.name=dc1
2015-10-29 10:00:54,515 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:java.version=1.6.0_29
2015-10-29 10:00:54,533 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:java.vendor=Oracle Corporation
2015-10-29 10:00:54,533 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:java.home=/opt/app/cargo/jrockit-jdk1.6.0/jrockit-jdk1.6.0_29-R28.1.5-4.0.1/jre
2015-10-29 10:00:54,533 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:java.class.path=/home/hadoop/zk2/bin/../build/classes:/home/hadoop/zk2/bin/../build/lib/*.jar:/home/hadoop/zk2/bin/../lib/slf4j-log4j12-1.6.1.jar:/home/hadoop/zk2/bin/../lib/slf4j-api-1.6.1.jar:/home/hadoop/zk2/bin/../lib/netty-3.7.0.Final.jar:/home/hadoop/zk2/bin/../lib/log4j-1.2.16.jar:/home/hadoop/zk2/bin/../lib/jline-0.9.94.jar:/home/hadoop/zk2/bin/../zookeeper-3.4.6.jar:/home/hadoop/zk2/bin/../src/java/lib/*.jar:/home/hadoop/zk2/bin/../conf:.:/opt/app/cargo/jrockit-jdk1.6.0/jrockit-jdk1.6.0_29-R28.1.5-4.0.1/lib/dt.jar:/opt/app/cargo/jrockit-jdk1.6.0/jrockit-jdk1.6.0_29-R28.1.5-4.0.1/lib/tools.jar
2015-10-29 10:00:54,533 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:java.library.path=/opt/app/cargo/jrockit-jdk1.6.0/jrockit-jdk1.6.0_29-R28.1.5-4.0.1/jre/lib/amd64/jrockit:/opt/app/cargo/jrockit-jdk1.6.0/jrockit-jdk1.6.0_29-R28.1.5-4.0.1/jre/lib/amd64:/opt/app/cargo/jrockit-jdk1.6.0/jrockit-jdk1.6.0_29-R28.1.5-4.0.1/jre/../lib/amd64
2015-10-29 10:00:54,533 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:java.io.tmpdir=/tmp
2015-10-29 10:00:54,533 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:java.compiler=<NA>
2015-10-29 10:00:54,533 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:os.name=Linux
2015-10-29 10:00:54,533 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:os.arch=amd64
2015-10-29 10:00:54,533 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:os.version=2.6.32-431.23.3.el6.x86_64
2015-10-29 10:00:54,533 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:user.name=hadoop
2015-10-29 10:00:54,533 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:user.home=/home/hadoop
2015-10-29 10:00:54,533 [myid:] - INFO  [Main Thread:Environment@100] - Client environment:user.dir=/home/hadoop/zk2/bin
2015-10-29 10:00:54,536 [myid:] - INFO  [Main Thread:ZooKeeper@438] - Initiating client connection, connectString=localhost:7012 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@9a031d8
Welcome to ZooKeeper!
JLine support is enabled
2015-10-29 10:00:54,649 [myid:] - INFO  [Main Thread-SendThread(localhost:7012):ClientCnxn$SendThread@975] - Opening socket connection to server localhost/0:0:0:0:0:0:0:1:7012. Will not attempt to authenticate using SASL (java.lang.SecurityException: 无法定位登录配置)
[zk: localhost:7012(CONNECTING) 0] 2015-10-29 10:00:54,701 [myid:] - INFO  [Main Thread-SendThread(localhost:7012):ClientCnxn$SendThread@852] - Socket connection established to localhost/0:0:0:0:0:0:0:1:7012, initiating session
2015-10-29 10:00:54,754 [myid:] - INFO  [Main Thread-SendThread(localhost:7012):ClientCnxn$SendThread@1235] - Session establishment complete on server localhost/0:0:0:0:0:0:0:1:7012, sessionid = 0x450b15327460000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
~~~
连接成功，可以测试一些基本命令，ls/get/create 等等

## Java应用中连接zk Server


### 添加依赖

maven项目中 在pom.xml 添加依赖
~~~
<!--zookeeper-->
<dependency>
	<groupId>com.github.sgroschupf</groupId>
	<artifactId>zkclient</artifactId>
	<version>0.1</version>
	<scope>${jar-scope}</scope>
</dependency>
~~~
### 示例代码
~~~
@Test
public void testZkClient() {
	ZkClient zkClient = new ZkClient("172.28.20.101:7011,172.28.20.101:7012,172.28.20.102:7011");
	String node = "/app/local/zkclitest";
	if (!zkClient.exists(node)) {
		zkClient.createPersistent(node, "hello zk");
	}
	System.out.println(zkClient.readData(node));
}
~~~
在zk服务器通过zkCli连接可以看到刚才创建的节点：

~~~
[zk: localhost:7011(CONNECTED) 2] ls /app/local 
[ciq-web, mfoc-web, zkclitest, ctas-web, ccsp-bill]
[zk: localhost:7011(CONNECTED) 3] 
~~~

## 参考资料
http://www.cnblogs.com/sunddenly/p/4033574.html
http://www.cnblogs.com/shanyou/p/3221990.html
