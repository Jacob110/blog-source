title: Redis学习笔记 安装配置(master/slave/sentinel)
date: 2015-11-05 16:07:52
category:
	- Redis
	- Java
tags:
	- redis
	- nosql
	- 数据库
	- javaweb
	- java
---


下载：
windows：https://github.com/MSOpenTech/redis/releases
Linux： http://download.redis.io/releases/redis-3.0.0.tar.gz

<!--more-->

## windows 安装

~~~
D:\Program Files\Redis>redis-server.exe redis.windows.conf
[208] 02 Nov 15:54:58.750 #
The Windows version of Redis allocates a large memory mapped file for sharing
the heap with the forked process used in persistence operations. This file
will be created in the current working directory or the directory specified by
the 'heapdir' directive in the .conf file. Windows is reporting that there is
insufficient disk space available for this file (Windows error 0x70).

You may fix this problem by either reducing the size of the Redis heap with
the --maxheap flag, or by moving the heap file to a local drive with sufficient
space.
Please see the documentation included with the binary distributions for more
details on the --maxheap and --heapdir flags.

Redis can not continue. Exiting.
~~~

根据提示需要设置下`maxheap`标识，打开`redis.windows.conf`文件，找到`maxheap`设置一下即可

~~~
# increased if it is smaller than 1.5*maxmemory. 
#  
# maxheap <bytes>
maxheap 1024000000
~~~

再次启动`redis-server`，提示如下，启动成功

~~~
D:\Program Files\Redis>redis-server.exe redis.windows.conf
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 2.8.2104 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in stand alone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 4632
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

[4632] 02 Nov 15:55:25.209 # Server started, Redis version 2.8.2104
[4632] 02 Nov 15:55:25.211 * The server is now ready to accept connections on po
rt 6379
~~~

### 客户端连接测试
使用自带客户端工具连接测试，打开`redis-cli.exe`

~~~
D:\Program Files\Redis>redis-cli.exe
127.0.0.1:6379> help
redis-cli 2.8.2104
Type: "help @<group>" to get a list of commands in <group>
      "help <command>" for help on <command>
      "help <tab>" to get a list of possible help topics
      "quit" to exit
~~~
测试一些基本命令

~~~
127.0.0.1:6379> set flash barry
OK
127.0.0.1:6379> get flash
"barry"
127.0.0.1:6379> set batman 11
OK
127.0.0.1:6379> get batman
"11"
127.0.0.1:6379> del flash
(integer) 1
127.0.0.1:6379> get flash
(nil)
127.0.0.1:6379>
~~~

## Linux下安装

### 下载解压
[下载地址](http://download.redis.io/releases/redis-3.0.0.tar.gz) 上传到Linux，然后解压

~~~
tar -xzvf redis-3.0.0.tar.gz
~~~

### 编译
进入到src目录下 执行make

~~~
vm-vmw7073-app:/opt/app/redis-3.0.0/src$>make
rm -rf redis-server redis-sentinel redis-cli redis-benchmark redis-check-dump redis-check-aof *.o *.gcda *.gcno *.gcov redis.info lcov-html
(cd ../deps && make distclean)
make[1]: Entering directory `/opt/app/redis-3.0.0/deps'
(cd hiredis && make clean) > /dev/null || true
(cd linenoise && make clean) > /dev/null || true
(cd lua && make clean) > /dev/null || true
(cd jemalloc && [ -f Makefile ] && make distclean) > /dev/null || true
(rm -f .make-*)
make[1]: Leaving directory `/opt/app/redis-3.0.0/deps'
(rm -f .make-*)
...
...
Hint: It's a good idea to run 'make test' ;)
~~~

### 启动server
~~~
vm-vmw7073-app:/opt/app/redis-3.0.0/src$>./redis-server
~~~

### 修改配置
上面启动是在当前窗口启动，这样肯定不方便，改成在后台运行，再改一下默认端口，将`redis.conf` 文件复制一份，重命名

~~~
mkdir conf
cp redis.conf conf/
mv redis.conf redis_6688.conf
~~~

编辑配置，将daemonize改为yes

~~~

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes

# When running daemonized, Redis writes a pid file in /var/run/redis.pid by
# default. You can specify a custom pid file location here.
pidfile /var/run/redis.pid

# Accept connections on the specified port, default is 6379.
# If port 0 is specified Redis will not listen on a TCP socket.
port 6388
~~~

重新启动server

~~~
vm-vmw7073-app:/opt/app/redis-3.0.0/src$./redis-server ../conf/redis.conf 
~~~

查看下进程中有没有

~~~
vm-vmw7073-app:/opt/app/redis-3.0.0/src$>ps -ef | grep redis
cargo    31323     1  0 11:03 ?        00:00:00 ./redis-server *:6388                 
cargo    31352 27670  0 11:03 pts/1    00:00:00 grep redis
~~~

### 客户端连接

~~~
vm-vmw7073-app:/opt/app/redis-3.0.0/src$>./redis-cli -p 6388
127.0.0.1:6388> keys *
(empty list or set)
127.0.0.1:6388> set flash barry
OK
127.0.0.1:6388> get flash
"barry"
127.0.0.1:6388> 
~~~

~~~
vm-vmw7073-app:/opt/app/redis-3.0.0/src$>./redis-server ../conf/redis_6388.conf
~~~


### 基本命令

返回所有key

~~~
127.0.0.1:6379> keys *
1) "CTAS"
~~~

查看key存储的值类型

~~~
127.0.0.1:6379> type CTAS
hash
~~~

返回哈希表key中的所有域

~~~
127.0.0.1:6379> hkeys CTAS
1) "AGT|CTAS_SHAMU|201508|GOJ"
2) "AGT|CTAS_SHAMU|201505"
3) "AGT|CTAS_SHAMU|201507|GOJ"
4) "AGT|CTAS_SHAMU|201507"
5) "AGT|CTAS_SHAMU|201505|GOJ"
6) "AGT|CTAS_SHAMU|201508"
7) "AGT|CTAS_SHAMU|201506|GOJ"
8) "AGT|CTAS_SHAMU|201506"
~~~

更多命令参考这里 http://redisdoc.com/


## master-slave 配置

修改slave节点的配置,将`slaveof` 改为master的ip和端口

~~~
vi redis.conf
# slaveof <masterip> <masterport>
slaveof 10.6.53.37 6388
~~~

配置灰常简单，然后分别重启master和slave的server，添加和删除测试一下
![](http://7xngxf.com1.z0.glb.clouddn.com/redis-001.png)

查看进程和复制状态是否正常：

~~~
127.0.0.1:6388> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=10.6.53.36,port=6688,state=online,offset=2264,lag=0
master_repl_offset:2264
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:2263
~~~

## sentinel 配置

~~~
vm-vmw7073-app:/opt/app/redis-3.0.0/conf$>vi sentinel.conf

port 6400

sentinel monitor mymaster 10.6.53.37 6400 1
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-sysncs mymaster 1
~~~

### 启动sentinel

~~~
vm-vmw7073-app:/opt/app/redis-3.0.0/src$>./redis-sentinel ../conf/sentinel.conf 
4162:X 03 Nov 15:02:31.501 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
4162:X 03 Nov 15:02:31.501 # Redis can't set maximum open files to 10032 because of OS error: Operation not permitted.
4162:X 03 Nov 15:02:31.501 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.0.0 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6400
 |    `-._   `._    /     _.-'    |     PID: 4162
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

4162:X 03 Nov 15:02:31.503 # Sentinel runid is 4ff4197908a7cbdda863a182767166ca7e5ad1cc
4162:X 03 Nov 15:02:31.503 # +monitor master mymaster 10.6.53.37 6400 quorum 1
4162:X 03 Nov 15:02:33.594 * +sentinel sentinel 10.6.53.37:6400 10.6.53.37 6400 @ mymaster 10.6.53.37 6400
~~~

PING:返回PONG
~~~
127.0.0.1:6400> PING
PONG
~~~

sentinel masters 列出所有被监视的主Redis服务实例，以及这些主服务实例的当前状态。

~~~
127.0.0.1:6400> sentinel masters
1)  1) "name"
    2) "mymaster"
    3) "ip"
    4) "10.6.53.37"
    5) "port"
    6) "6400"
    7) "runid"
    8) "3b128618ae45f5e2ad04bfa7dcf6a6cbf6bec0bc"
    9) "flags"
   10) "master"
   ...
~~~

		