---
title: Tomcat
date: 2020-02-05 16:49:01
tags: Tomcat
category: 中间件
summary: Tomcat
top: true
cover: true
author: 张文军
---

![](/images/favicon.png)
# Tomcat
## 1、Tomcat架构说明
Tomcat是一个基于JAVA的WEB容器，其实现了JAVA EE中的 Servlet 与 jsp 规范，与Nginx apache 服务器不同在于一般用于动态请求处理。在架构设计上采用面向组件的方式设计。即整体功能是通过组件的方式拼装完成。另外每个组件都可以被替换以保证灵活性。
Tomcat是一个由一系列可配置的组件构成的Web容器，而Catalina是Tomcat的servlet容器。
Catalina 是Servlet 容器实现，包含了涉及到安全、会话、集群、管理等Servlet 容器架构的各个方面。它通过松耦合的方式集成Coyote，以完成按照请求协议进行数据读写。同时，它还包括我们的启动入口、Shell程序等。
## 2、Catalina 地位
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/Tomcat/20200205045908956.png)
Tomcat 本质上就是一款 Servlet 容器， 因此Catalina 才是 Tomcat 的核心 ， 其他模块
都是为Catalina 提供支撑的。 比如 ： 通过Coyote 模块提供链接通信，Jasper 模块提供
JSP引擎，Naming 提供JNDI 服务，Juli 提供日志服务。

Catalina 的主要组件结构如下：

![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/Tomcat/20200205050146541.png)
如上图所示，Catalina负责管理Server，而Server表示着整个服务器。Server下面有多个
服务Service，每个服务都包含着多个连接器组件Connector（Coyote 实现）和一个容器
组件Container。在Tomcat 启动的时候， 会初始化一个Catalina的实例。

Catalina 各个组件的职责：


| 组件 |  职责 |
| ----- | ---- |
| Catalina |  负责解析Tomcat的配置文件 , 以此来创建服务器Server组件，并根据命令来对其进行管理 |
| Server |  服务器表示整个Catalina Servlet容器以及其它组件，负责组装并启动  Servlet引擎,Tomcat连接器。Server通过实现Lifecycle接口，提供了一种优雅的启动和关闭整个系统的方式 |
| Service | 服务是Server内部的组件，一个Server包含多个Service。它将若干个Connector组件绑定到一个Container（Engine）上 |
| Connector | 连接器，处理与客户端的通信，它负责接收客户请求，然后转给相关的容器处理，最后向客户返回响应结果 |
| Container  | 容器，负责处理用户的servlet请求，并返回对象给web用户的模块 |

![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/Tomcat/20200208013011514.png)

## 3、Tomcat 启动流程

![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/Tomcat/20200208013155480.png)

步骤 :
1） 启动tomcat ， 需要调用 bin/startup.bat (在linux 目录下 , 需要调用 bin/startup.sh)
， 在startup.bat 脚本中, 调用了catalina.bat。
2） 在catalina.bat 脚本文件中，调用了BootStrap 中的main方法。
3）在BootStrap 的main 方法中调用了 init 方法 ， 来创建Catalina 及 初始化类加载器。
4）在BootStrap 的main 方法中调用了 load 方法 ， 在其中又调用了Catalina的load方
法。
5）在Catalina 的load 方法中 , 需要进行一些初始化的工作, 并需要构造Digester 对象, 用
于解析 XML。
6） 然后在调用后续组件的初始化操作 。。。
加载Tomcat的配置文件，初始化容器组件 ，监听对应的端口号， 准备接受客户端请求

## 4、Tomcat 请求处理流程

Tomcat是用Mapper组件来完成这个任务的
Mapper组件的功能就是将用户请求的URL定位到一个Servlet，它的工作原理是：
	Mapper组件里保存了Web应用的配置信息，其实就是容器组件与访问路径的映射关系，比如Host容器里配置的域名、Context容器里的Web应用路径，以及Wrapper容器里Servlet映射的路径，你可以想象这些配置信息就是一个多层次的Map。当一个请求到来时，Mapper组件通过解析请求URL里的域名和路径，再到自己保存的Map里去查找，就能定位到一个Servlet。请你注意，一个请求URL最后只会定位到一个Wrapper容器，也就是一个Servlet。

下面的示意图中 ， 就描述了 当用户请求链接 `http://www.itcast.cn/bbs/findAll` 之后, 是如何找到最终处理业务逻辑的servlet 。
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/Tomcat/20200208013715593.png)




