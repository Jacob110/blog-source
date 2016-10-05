title: JAX-WS (SOAP) 开发 WebService
date: 2015-09-24 21:49:21
category: Java
tags: 
	- java
	- jax-ws
	- soap
	- webservice
---


## SOAP 简介
`SOAP` 简单对象访问协议(Simple Object Access Protocol)是一种标准化的通讯规范，主要用于Web服务(Web Service)中。是一个基于XML的协议，它包括四个部分

- SOAP封装(envelop)，封装定义了一个描述消息中的内容是什么，是谁发送的，谁应当接受并处理它以及如何处理它们的框架；
- SOAP编码规则（encoding rules），用于表示应用程序需要使用的数据类型的实例; 
- SOAP RPC表示(RPC representation)，表示远程过程调用和应答的协定;
- SOAP绑定（binding），使用底层协议交换信息。

<!--more-->

## JAX-WS

Java API for XML Web Services(JAX-WS)是一组用来创建Web服务的API，主要是基于`SOAP`规范的。JAX-WS 提供了许多注解来简化 WebService 服务端和客户端的开发。

![](/images/jax-ws-01.gif)

## `IDEA` 下开发 WebService

定义接口
~~~
package com.jacob.ws.service;

import javax.jws.WebMethod;
import javax.jws.WebService;

@WebService
public interface FirstService {
    @WebMethod
    public String sayHi(String name);
}
~~~

实现类

~~~
package com.jacob.ws.service.imp;

import com.jacob.ws.service.FirstService;
import javax.jws.WebMethod;
import javax.jws.WebService;

@WebService(endpointInterface = "com.jacob.ws.service.FirstService")
public class FirstServiceImp implements FirstService {

    public String sayHi(String name) {
        return "Hi," + name + ",this is first JAX-WS Service";
    }
}

~~~

### 发布
两种发布方式，一种是 Java 类中中定义 `Endpoit` 来发布，另一种是在 Web 容器中发布

### 1. Endpoint 发布

~~~
package com.jacob.ws.endpoit;

import com.jacob.ws.service.imp.FirstServiceImp;

import javax.xml.ws.Endpoint;

public class FirstServicePublisher {

    public static void main(String[] arg){
        System.out.println("Service is publishing...");
        Endpoint.publish("http://localhost:9999/ws/firstservice", new FirstServiceImp());
        System.out.println("Service is published.");
    }
}

~~~


### 2. Web 应用中发布

新建一个 Web项目，打开模块设置(`Modeule Settings`) 添加 `WebService` 的 `Facets` 如图
![](/images/jax-ws-05.png)

然后选择要添加的 `WebService` 类型，如图
![](/images/jax-ws-06.png)

添加完以后会自动生成 相关的配置文件，需要添加两个依赖项：

~~~
<dependency>
	<groupId>com.sun.xml.ws</groupId>
 	<artifactId>jaxws-rt</artifactId>
	<version>2.1.3</version>
</dependency>

<dependency>
	<groupId>javax.xml.ws</groupId>
	<artifactId>jaxws-api</artifactId>
	<version>2.2.11</version>
</dependency>

~~~

`sun-jaxws.xml` 中添加 `endpoint`

~~~
<?xml version="1.0" encoding="UTF-8"?>
<endpoints xmlns='http://java.sun.com/xml/ns/jax-ws/ri/runtime' version='2.0'>
    <endpoint name="FirstService" implementation="com.jacob.ws.service.imp.FirstServiceImp"
              url-pattern="/firstws"></endpoint>
</endpoints>

~~~

`web.xml` 中添加 `listner`

~~~
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>JaxWebService</display-name>
    <listener>
        <listener-class>com.sun.xml.ws.transport.http.servlet.WSServletContextListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>WSServlet</servlet-name>
        <display-name>WSServlet</display-name>
        <description>JAX-WS endpoint</description>
        <servlet-class>com.sun.xml.ws.transport.http.servlet.WSServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>WSServlet</servlet-name>
        <url-pattern>/firstws</url-pattern>
    </servlet-mapping>
</web-app>

~~~
这里的 `url-pattern` 必须跟上面 `endpoint` 中的一样

启动server 浏览器输入 url http://localhost:8080/JaxWebService/firstws 如图
![](/images/jax-ws-02.png)

## Client 调用

新建一个项目，或者直接在刚才的 项目中也可以。右键选择 WebService->Generate Java Code From Wsdl

![](/images/jax-ws-04.png)
wsdl url 输入刚才发布的 service url ,确定后会自动生成客户端调用相关的类,项目结构

![](/images/jax-ws-client.png)

编写测试类：

~~~
public class FirstServiceClient {
    public static void main(String[] args){
        FirstServiceImpService firstServiceImp=new FirstServiceImpService();
        System.out.println(firstServiceImp.getFirstServiceImpPort().sayHi("jacob"));
    }
}
~~~
控制台输出

~~~
Hi,jacob,this is first JAX-WS Service
~~~


## 可能会碰到的问题
编译、打包都正常，但是启动 `Server` 报错

~~~
com.sun.xml.ws.model.RuntimeModelerException: runtime modeler error: Wrapper class com.jacob.ws.service.jaxws.SayHi is not found. Have you run APT to generate them?
~~~

**Solution**

服务接口添加 `@SOAPBinding` 注解，如下

~~~
@SOAPBinding(style = SOAPBinding.Style.RPC, use= SOAPBinding.Use.LITERAL)
~~~







