---
title: dubbo+zookeeper
date: 2017-09-23 11:41
categories: dubbo+zookeeper
tags: dubbo
---

### dubbo简介

dubbo介绍请参考[官网](https://dubbo.gitbooks.io/dubbo-user-book/preface/background.html)

本文会使用zookeeper作为注册中心，其中有3个项目，皆使用MAVEN进行管理

1. dubbo-api项目

提供统一的接口，为jar包，供consumer和provider引用

2. dubbo-provider项目

服务提供者，为jar包，包含api接口的实现类，提供服务的实现逻辑

3. dubbo-consumer

服务消费者，为WAR包，使用tomcat进行部署，前端浏览器访问，可以访问到dubbo-provider项目提供的服务

### zookeeper

> Zookeeper 是 Apacahe Hadoop 的子项目，是一个树型的目录服务，支持变更推送，适合作为 Dubbo 服务的注册中心，工业强度较高，可用于生产环境，并推荐使用 

#### 安装zookeeper

- 下载安装

**zookeeper启动会默认使用conf/zoo.cfg配置文件,我们需要copy一份并重新命名为zoo.cfg**

```
wget http://www.apache.org/dist//zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
tar zxvf zookeeper-3.4.9.tar.gz
cd zookeeper-3.4.9
cp conf/zoo_sample.cfg conf/zoo.cfg

```

- zoo.cfg配置

```
[root@localhost conf]# vim zoo.cfg 

tickTime=2000
initLimit=10
syncLimit=5
dataDir=/tmp/zookeeper
clientPort=2181

```

- 简单命令

```
#启动
./bin/zkServer.sh start

#停止
./bin/zkServer.sh stop

```

### dubbo-api项目

#### 项目结构

![image](/images/dubbo/dubboapi.jpg)

#### 接口

api只提供一个UserService的接口，非常简单

```
public interface UserService {

	String sayHello(String name);
}


```

#### 配置文件

在该项目的classpath路径下提供了一个xml配置，该配置文件其实就是dubbo消费者的相关配置，后边消费者项目引入api之后可以直接使用该文件，当然你也可以直接在消费者项目行配置以下内容

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

	
	<!-- 引入服务的配置 -->
	<!-- 
	  dubbo:reference 的一些属性的说明：
      1）interface调用的服务接口
      2）check 启动时检查提供者是否存在，true报错，false忽略
      3）registry 从指定注册中心注册获取服务列表，在多个注册中心时使用，值为<dubbo:registry>的id属性，多个注册中心ID用逗号分隔
      4）loadbalance 负载均衡策略，可选值：random,roundrobin,leastactive，分别表示：随机，轮循，最少活跃调用
	 -->
	<dubbo:reference id="userService" interface="com.szl.dubbo.api.UserService" cluster="failfast" check="false"/>

</beans>

```

### dubbo-provider项目

#### 项目结构

![image](/images/dubbo/dubboprovider.jpg)

#### 接口实现

实现UserService接口的方法

```
@Service("userService")
public class UserServiceImpl implements UserService {

	@Override
	public String sayHello(String name) {
		return "hello " + name + ",欢迎来到dubbo的世界";
	}
}

```


#### 配置文件

- dubbo.properties

备注:启动dubbo服务时,默认读取classpath下一个名为dubbo.properties文件的属性值，该文件配置信息如下。


```
#dubbo.container指定了dubbo的容器使用spring，dubbo内部有四种容器实现，Spring Container是其中一种，也是默认的容器
#dubbo.spring.config指定了dubbo在启动服务时加载的spring配置文件。
#dubbo.protocol.name和dubbo.protocol.port分别指定使用的协议名和端口
dubbo.container=spring
#指定spring的配置文件
dubbo.spring.config=classpath:user-provider.xml
dubbo.protocol.name=dubbo
dubbo.protocol.port=28511

```

- user-provider.xml

其实就是一个spring的配置文件,只不过引入了dubbo相关的配置

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	    http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
        http://code.alibabatech.com/schema/dubbo 
		http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

	<context:component-scan base-package="com.szl"></context:component-scan>
	<!-- 开启注解配置 -->
    <context:annotation-config/>

	<!-- 
	    dubbo:registry 标签一些属性的说明： 
	    1）register是否向此注册中心注册服务，如果设为false，将只订阅，不注册。 
		2）check注册中心不存在时，是否报错。
		3）subscribe是否向此注册中心订阅服务，如果设为false，将只注册，不订阅。 
		4）timeout注册中心请求超时时间(毫秒)。 
		5）address可以Zookeeper集群配置，地址可以多个以逗号隔开等。 
		
		dubbo:service标签的一些属性说明： 1）interface服务接口的路径 
		2）ref引用对应的实现类的Bean的ID 3）registry向指定注册中心注册，在多个注册中心时使用，值为<dubbo:registry>的id属性，多个注册中心ID用逗号分隔，如果不想将该服务注册到任何registry，可将值设为N/A 
		4）register 默认true ，该协议的服务是否注册到注册中心。 
	-->

	<!-- 提供方应用名称信息，这个相当于起一个名字，我们dubbo管理页面比较清晰是哪个应用暴露出来的 -->
	<dubbo:application name="dubbo-provider" />
	<!-- 使用zookeeper注册中心暴露服务地址 -->
	<dubbo:registry address="zookeeper://192.168.1.23:2181" />
	<!-- 指定了集群容错模式，此处为快速失败 -->
	<dubbo:provider cluster="failfast" />
	<!-- 要暴露的服务接口 -->
	<dubbo:service interface="com.szl.dubbo.api.UserService" ref="userService" />

</beans>

```

- pom.xml

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.szl</groupId>
	<artifactId>dubboapi</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>dubboapi</name>
	<url>http://maven.apache.org</url>

	<properties>
		<!-- 文件拷贝时的编码 -->
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<!-- 编译时的编码 -->
		<maven.compiler.encoding>UTF-8</maven.compiler.encoding>
	</properties>

	<dependencies>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>dubbo</artifactId>
			<version>2.5.5</version>
			<exclusions>
				<exclusion>
					<groupId>org.springframework</groupId>
					<artifactId>spring</artifactId>
				</exclusion>
			</exclusions>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-beans</artifactId>
			<version>4.3.2.RELEASE</version>
			<scope>provided</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

#### 启动服务提供者
##### 第一种启动方式

使用dubbo提供的启动方式(其实内部也是调用了第二种方式)

```
    public static void main(String[] args) {
    	 com.alibaba.dubbo.container.Main.main(args);
    }

```

##### 第二种启动方式

使用spring启动方式

```
public static void main(String[] args) {
	ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:user-provider.xml");
	context.start();
	//此处应阻塞，防止服务停止
}

```


##### 启动服务提供者

启动成功如下:

```
log4j:WARN No appenders could be found for logger (com.alibaba.dubbo.common.logger.LoggerFactory).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
[2017-09-23 11:24:30] Dubbo service server started!

```


### dubbo-consumer项目


#### 项目结构

![image](/images/dubbo/dubboconsumer.jpg)


#### UserController

UserController就是一个简单的控制器

```
@RestController
@RequestMapping(value = "user", produces = "application/json; charset=UTF-8")
public class UserController {

	@Autowired
	private UserService userService;

	@RequestMapping(value = "")
	public String sayHello(@RequestParam(name = "name", defaultValue = "访客") String name) {
		return userService.sayHello(name);
	}

}

```
#### 配置文件

- web.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0">
	<display-name>dubbo-consumer</display-name>
	
	
	<!-- 配置Spring配置文件路径 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
            classpath*:applicationContext.xml
        </param-value>
	</context-param>
	<!-- 配置Spring上下文监听器 -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<servlet>
		<servlet-name>dispatcher</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath*:spring-mvc.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>dispatcher</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>


	<welcome-file-list>
		<welcome-file>index.html</welcome-file>
		<welcome-file>index.htm</welcome-file>
		<welcome-file>index.jsp</welcome-file>
		<welcome-file>default.html</welcome-file>
		<welcome-file>default.htm</welcome-file>
		<welcome-file>default.jsp</welcome-file>
	</welcome-file-list>
</web-app>

```

- applicationContext.xml

spring配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	    http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
        http://code.alibabatech.com/schema/dubbo 
		http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

	<!-- 开启注解配置 -->
    <context:annotation-config/>
    <!-- 导入dubbo消费者配置文件-->
    <import resource="classpath:user-consumer.xml"/>

</beans>

```

- user-consumer.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	    http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
        http://code.alibabatech.com/schema/dubbo 
		http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    
    
	<dubbo:application name="dubbo-consumer" />
	<dubbo:registry address="zookeeper://192.168.1.23:2181" />
	<dubbo:consumer check="false" />

	<!-- 导入dubbo api消费者配置 -->
	<import resource="classpath*:user-reference.xml" />

</beans>

```

- spring-mvc.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:task="http://www.springframework.org/schema/task"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context-4.0.xsd
	http://www.springframework.org/schema/mvc
	http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
	http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
		http://www.springframework.org/schema/task
		http://www.springframework.org/schema/task/spring-task-3.2.xsd">

	<context:component-scan base-package="com.szl">
		<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller" /> 
    	<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service" />  
	</context:component-scan>
	
	<!-- 开启注解配置 -->
    <mvc:annotation-driven />
</beans>

```

- pom.xml
省略,请参考源码

- logback.xml
省略,请参考源码

#### 启动项目

- 使用tomcat启动该项目

- 访问一下

![image](/images/dubbo/dubboconsumer1.jpg)




### dubbo-admin

管理控制台,开源部分主要包含：路由规则，动态配置，服务降级，访问控制，权重调整，负载均衡，等管理功能,[安装参考](https://dubbo.gitbooks.io/dubbo-admin-book/content/install/admin-console.html)

打开控制台可以看到我们刚刚写的服务提供者和服务消费者

![image](/images/dubbo/dubboadmin1.jpg)

![image](/images/dubbo/dubboadmin2.jpg)

![image](/images/dubbo/dubboadmin3.jpg)

