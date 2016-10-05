title: Mac OS X 下安装MySQL 5.7
date: 2015-10-13 22:03:27
category: Java Web
tags: 
	- mysql
	- web开发
	- 数据库
---



### 下载安装包
[官网下载安装包](http://downloads.mysql.com/archives/community/) 选择相应的版本和格式，有 `.dmg` 和`压缩包`两种。

这里选择简单直接的 `.dmg`安装包，下载的时候可以将下载地址直接贴到迅雷，速度比较快。
<!--more-->
### 安装
安装很简单，直接双击下好的.dmg文件，一路next就可以了。
### 启动 MySQL
OK！安装够简单，接下来就是启动MySQL，以及具体使用了。

系统偏好设置->MySQL->Start MySQL Server

![image](http://7xngxf.com1.z0.glb.clouddn.com/mysql-01.jpg)

启动 Mysql

![image](http://7xngxf.com1.z0.glb.clouddn.com/mysql-02.png)

然后在终端中进入MySQL控制台

~~~
jacob@promote:~$ mysql -u root -p
Enter password:
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
~~~
这是什么情况，`root`用户的密码是啥，安装的时候也没有提示要设置密码。

Google之，说是初始安装后密码为空直接回车就可以了，试了下不行，提示如上。下面就来解决这个问题。
### MySQL修改密码
* 关闭服务

系统偏好设置->MySQL->Stop MySQL Server

- 安全模式进入MySQL

~~~
jacob@JacobdeMacBook-Pro:~$ sudo mysqld_safe --skip-grant-tables
~~~

重新打开一个终端 进入MySQL控制台

~~~
jacob@JacobdeMacBook-Pro:~$ mysql -u root
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.7-rc MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
~~~
提示已经成功进入控制台

修改密码，`sql`语句

~~~
mysql> update mysql.user set password=password('123456') where user='root';
~~~
坑爹的地方来了，输入后报如下错误

~~~
ERROR 1054 (42S22): Unknown column 'password' in 'field list'
~~~
神马情况，`'password'`列不存在，这个地方花了好多时间，原因其实很简单啊啊。

**MySQL 5.7 版本中 `user`表中的密码字段列名称变了，从`password`变成了`authentication_string`**
可以直接看一下`user`表中的字段

~~~
mysql> use mysql;
mysql> desc user;
~~~
部分字段如下

Field | Type | Null | Key | Default | Extra 
------| -----| -----| ----| --------| ------
Host                   | char(60)             | NO   | PRI |      | 
User                   | char(16)             | NO   | PRI |      |                  
**authentication_string**  | text                 | YES  |     | NULL |                  
password_expired       | enum('N','Y')        | NO   |     | N    |              
password_last_changed  | timestamp            | YES  |     | NULL |                   
password_lifetime      | smallint(5) unsigned | YES  |     | NULL | 

最后用如下如下语句修改              

~~~
mysql> update mysql.user set authentication_string=PASSWORD('123456') where user='root';
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
~~~

修改成功，再次进入控制台

~~~
mysql> show databases;
~~~
这次可以进去了，但是随便执行一条语句依然报错啊

~~~
ERROR 1820 (HY000): You must SET PASSWORD before executing this statement
~~~
按照提示再次设置密码

~~~
mysql> set password for root@localhost=password('12345');
Query OK, 0 rows affected, 1 warning (0.00 sec)
~~~

这次OK了，接下来就可以正常建表、查询 使用了。