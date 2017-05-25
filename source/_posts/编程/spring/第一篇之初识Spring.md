---
title: 第一篇之初始Spring
date: 2017-05-25 11:40
categories: Spring的那点事
tags: Spring
---

### 初识Spring
#### 简单了解Spring
Spring是一个轻量级的企业开源框架，于2003年兴起，由Rod Johnson创建！其目的是为了简化企业及应用程序的开发，Spring框架的核心是一个Ioc容器。

### Spring框架结构
![image](\images\spring\1\spirng架构图.JPG)

#### Core Container(核心容器)
由core，Bean，上下文和表达式语言模块组成
- Core模块：Spring的核心类库，主要实现Ioc功能。
- Beans模块：模块提供Bean Factory，提倡面向接口编程，所有的依赖关系都有Bean Factory来维护。
- Context模块：模块建立在由核心和 Bean 模块提供的坚实基础上，它是访问定义和配置的任何对象的媒介。ApplicationContext 接口是上下文模块的重点。
- EL模块：提供强大的表达式语言支持。
#### Date Access/Integration(数据访问/集成模块)
包括JDBC，ORM，OXM，JMS 和事务处理模块
- JDBC：提供JDBC的JdbcTemplate，减少传统JDBC冗余的编码和事务控制。
- ORM：提供对象关系映射API，包括 JPA，JDO，hibernate 和 MyBatis，提供了集成层。
- OXM：提供了一个对Object/XML映射实现，将java对象映射成XML数据，或者将XML数据映射成java对象，Object/XML映射实现包括JAXB、Castor、XMLBeans和XStream。
- JMS：Java Messaging Service
- Transactions：用于Spring管理事务，支持编程和声明性的事物管理。
#### Web
由 Web，Servlet，Struts 和 Portlet 组成
- Web：提供了基本的面向 web 的集成功能，例如多个文件上传的功能和使用 servlet 监听器和面向 web 应用程序的上下文来初始化 IoC 容器。
- Web-Servlet：提供了一个Spring MVC Web框架实现,即：模型-视图-控制器（MVC）。
- Web-Struts：提供了与Struts无缝集成。
- Portlet：提供Portlet环境中实现MVC。
#### Aop&Aspects
- Aop：提供了面向方面的编程实现，允许自定义方法拦截器和切入点对代码进行干净地解耦，比如业务代码和日志代码的解耦。
- Aspects：提供了与 AspectJ 的集成。
#### Instrumentaiion
模块在一定的应用服务器中提供了类 instrumentation 的支持和类加载器的实现。
#### Test
Spring支持Junit和TestNG测试框架，而且还额外提供了一些基于Spring的测试功能，比如在测试Web框架时，模拟Http请求的功能。
### Spring的特点
- 轻量级的容器：Spring容器是非侵入式的，对象创建和装配和生命周期完全由容器负责。
- 事务管理：Spring的事务管理可以让我们专注于业务逻辑的开发。
- AOP支持：方便面向切面编程，把通用的功能提取出来。
- JDBC抽象和ORM框架的支持：Spring简化了传统的JDBC冗余编码,并且非常方便集成第三方ORM，如Mybatis。
- Web支持：非常方便集成web框架,且Spirng提供了SpringMVC,可以无缝集成。