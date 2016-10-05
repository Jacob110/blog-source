title: jboss eap 6 domain(域) 模式配置
date: 2015-10-23 21:29:39
category: 
	- Java Web
	- JBoss
tags: 
	- jboss
	- java
	- web开发
	- 应用服务器
---

jboss eap 6 提供了两种操作模式：standalone(独立服务模式) 和managed domain(受管域).两种模式的主要区别在于对服务器的管理，而不是它们为满足最终用户请求而提供的功能。

<!--more-->

## Domain 模式

domain 模式是通过单个控制点来管理多个 jboss eap 6 实例的模式。集中管理的jboss eap 6 服务器集合被称作域的成员，域里所有的jboss eap 6 实例共享一个公共的管理策略，以及配置。

所有的jboss server 可以划分成不同的 server group(服务器组)，每个server group可以包含若干 jboss server，在所有这些server中，指定一个作为主控制器，及 master server，其他的作为 slave server。

![](http://7xngxf.com1.z0.glb.clouddn.com/jboss-domain-006.png)

## 环境准备

- windows7
- jboss eap 6

两台机器 172.21.0.201 作为 master server，172.21.0.57 作为 slave server

## 安装jboss
两台机器分别安装 jboss eap 6，任意目录即可。[下载地址](https://www.jboss.org/products/eap/download/)
下面用 `jboss_home="D:\Java\jboss-eap-6.3" `代替jboss 安装目录

## master server 配置

### 修改host.xml
打开 `"jboss_home\domain\configuration"` 中的 host.xml

~~~
<interfaces>
    <interface name="management">
        <inet-address value="${jboss.bind.address.management:172.21.0.201}"/>
    </interface>
    <interface name="public">
        <inet-address value="${jboss.bind.address:172.21.0.201}"/>
    </interface>
    <interface name="unsecure">
        <inet-address value="${jboss.bind.address.unsecure:172.21.0.201}"/>
    </interface>
</interfaces>

~~~

将其中的 `127.0.0.1` 替换成机器真是 ip，否则远程机器无法通过浏览器访问jboss 控制台

`jboss_home\bin\domain.bat` 以domain 模式启动 jboss

进入控制台 `http://172.21.0.201:9990/console`
首次进入会提示输入用户名密码，需要先创建一下，进入 `jboss_home\bin\add-user.bat` 根据提示创建账号密码。

### 创建 Group和Server
在控制台的 Domain Tab 页，可以看到默认的一些 group和server，先全部remove掉，然后创建自己的group 和server 如图
![](http://7xngxf.com1.z0.glb.clouddn.com/jboss-domain-001.png)

如果server 无法romve，可能是该server处于 started状态，需要先停掉，参考下图
![](http://7xngxf.com1.z0.glb.clouddn.com/jboss-domain-003.png)

创建server group 和server
[](http://7xngxf.com1.z0.glb.clouddn.com/jboss-domain-002.png)

创建的时候会有个 `Port Offset`，即端口偏移量，默认端口是 `8080`,比如这里设置成 `1`,那么访问的时候 端口号就是 `8081`

## Slave Server 配置
切换到Slave 机器(172.21.0.57)  进入 jboss domain 配置目录 `jboss_home\domain\configuration`

### 修改host.xml
修改host.xml,现将原来的host.xml 和 host-slave.xml 改名备份，再将host-slave.xml 改名为 host.xml，打开 host.xml

~~~
<management-interfaces>
    <native-interface security-realm="ManagementRealm">
        <!-- modify port or conflict with master port-->
        <socket interface="management" port="${jboss.management.native.port:9099}"/>
    </native-interface>
</management-interfaces>
~~~
修改其中的端口，改成其他不用的端口，不能与master server 的端口相同，否则会起冲突报错。

~~~
<domain-controller>
   <remote host="${jboss.domain.master.address:172.21.0.201}" port="${jboss.domain.master.port:9999}" security-realm="ManagementRealm"/>
</domain-controller>
~~~
将其中连接 master的地址改为 master server 的真实 ip，本文为 `172.21.0.201`

~~~
<interfaces>
    <interface name="management">
        <inet-address value="${jboss.bind.address.management:0.0.0.0}"/>
    </interface>
    <interface name="public">
       <inet-address value="${jboss.bind.address:0.0.0.0}"/>
    </interface>
    <interface name="unsecure">
        <!-- Used for IIOP sockets in the standard configuration.
             To secure JacORB you need to setup SSL -->
        <inet-address value="${jboss.bind.address.unsecure:0.0.0.0}"/>
    </interface>
</interfaces>
~~~
将原先的 `127.0.0.1`修改为`0.0.0.0`或者本机真是IP

~~~
<servers>
    <server name="slave-server-one" group="my-group-one">
        <socket-bindings port-offset="2"/>
    </server>
    <server name="slave-server-two" group="my-group-two">
        <!-- server-two avoids port conflicts by incrementing the ports in
             the default socket-group declared in the server-group -->
        <socket-bindings port-offset="3"/>
    </server>
</servers>
~~~
手动给Slave Server 创建两个Server，并且指定group，group为master server上已经创建好的，同样指定port-offeset

最后给 Slave server 添加个名字，以便在 master server 控制台显示，如下

~~~
<host name="slave57" xmlns="urn:jboss:domain:1.6">
~~~

## 安全认证配置

切回到master server上，通过 `jboss_home\bin\add-user.bat` 添加一个名为 `slave57`的管理员帐号，记住最后生成 密钥，如下图
![](http://7xngxf.com1.z0.glb.clouddn.com/jboss-005.png)

再切回到 slave server上，将刚才的密钥添加到 host.xml 中 如下
~~~
<server-identities>
     <!-- Replace this with either a base64 password of your own, or use a vault with a vault expression -->
     <secret value="d2VibG9naWMxMjMh"/>
</server-identities>
~~~

好了，配置完了，依次以 domain 模式启动 master、slave 上的jboss，不出意外会在master 控制台看到名为 `slave57`的 slave server
![](http://7xngxf.com1.z0.glb.clouddn.com/jboss-domain-007.png)

## 应用部署

部署一个最简单 hello world 例子试试，如图
![](http://7xngxf.com1.z0.glb.clouddn.com/jboss-domain-008.png)
将打包好的 war包上传，然后分配(assign)到任意一个 group即可，不出意外，通过相关 server group 下的 jboss server 就都可以访问了。本例中 两个group，6个server,如图
![](http://7xngxf.com1.z0.glb.clouddn.com/jboss-domain-009.png)

这样可以通过6个url都可以访问

- 172.21.0.57:8083/gotimp/
- 172.21.0.57:8082/gotimp/
- 172.21.0.201:8084/gotimp/
- 172.21.0.201:8083/gotimp/
- 172.21.0.201:8082/gotimp/
- 172.21.0.201:8081/gotimp/

## 总结
Domain模式的主要目的是使得多台jboss服务器的配置可以集中于一点，统一配置、统一部署。不同于群集(Cluster),群集的目标是让多台服务器分摊压力，当一台或多台服务器当机时，服务可以继续保持运转。
