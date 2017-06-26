---
title: 第八篇之Spirng WEB
date: 2017-06-15 10:25
categories: Spring的那点事
tags: Spring
---

### Spring Web框架

Spring提供了自己的Web框架（Spring MVC），也可以方便集成其他第三方WEB框架，比如Status2等;
首先我们来看一下Spring配置，该配置不是特定于任何一个Web框架的，下面以集成SpringMVC框架为列来进行介绍

### 新建WEB工程

![image](\images\spring\1\spring_web1.jpg)

### Sping通用配置

#### WEB.XML配置

##### web.xml初始化配置

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0">
</web-app>

```
`<web-app version="3.0">`表示采用Servlet 3.0规范的Web程序部署描述格式

##### 指定Web应用上下文实现


> 在Web环境中，Spring提供WebApplicationContext（继承ApplicationContext）接口用于配置Web应用，该接口应该被实现为在Web应用程序运行时只读，即在初始化完毕后不能修改Spring Web容器（WebApplicationContext），但可能支持重载。
Spring提供XmlWebApplicationContext实现，并在Web应用程序中默认使用该实现，可以通过在web.xml配置文件中使用如下方式指定：

```
<!-- 指定Sping容器的实现 -->
<context-param>
	<param-name>contextClass</param-name>
	<param-value>  
   		org.springframework.web.context.support.XmlWebApplicationContext  
	</param-value>
</context-param> 

```
> 如上指定是可选的，只有当使用其他实现时才需要显示指定。

**容器的实现可以参看第二篇Spring IOC**


##### 指定Spring配置文件

容器默认情况下将加载/WEB-INF/applicationContext.xml配置文件，也可以使用如下形式在web.xml中定义要加载自定义的配置文件，多个配置文件用“,”。

```
<!--  指定Spring加载文件位置 -->
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>classpath:applicationContext.xml</param-value>
</context-param>

```
##### 加载和关闭Spring Web容器

Spring使用ContextLoaderListener监听器来加载和关闭Spring Web容器，即使用如下方式在web.xml中指定：

```
<!-- Spring 监听器 -->
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

```
> ContextLoaderListener监听器将在Web应用启动时使用指定的配置文件初始化Spring Web容器，在Web应用关闭时销毁Spring Web容器

##### 在Web环境中获取Spring Web容器

- 使用 WebApplicationContextUtils

```
WebApplicationContextUtils.getWebApplicationContext(servletContext);  
或  
WebApplicationContextUtils.getRequiredWebApplicationContext(servletContext);  

```

如果当前Web应用中的ServletContext 中没有相应的Spring Web容器，对于getWebApplicationContext()方法将返回null,而getRequiredWebApplicationContext()方法将抛出异常

- 使用 ApplicationContextAware

###### 代码

```
package com.szl.springweb.util;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class ApplicationContextUtil implements ApplicationContextAware {
	
	private static ApplicationContext applicationContext;
	
	private ApplicationContextUtil() {
		
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext)
			throws BeansException {
		ApplicationContextUtil.applicationContext = applicationContext;
	}
	
	public static ApplicationContext getApplicationContext(){
		return applicationContext;
	}

}

```

###### 测试

```
/**
 * 
 * 测试从ApplicationContextAware中获取上下文bean
 */
@Test
public void testApplicationContextAware(){
	UserServiceImpl userDaoImpl = ApplicationContextUtil.getApplicationContext().getBean("userServiceImpl",UserServiceImpl.class);
	List<Map<String, Object>> queryUser = userDaoImpl.queryUser();
	System.out.println("SpringWeb:"+queryUser.toString());
}

```

只要类实现ApplicationContextAware 接口即可，这样就可获取上下文中所有bean。

### 集成SpingMVC

#### WEB.XML配置

Spring提供的前端控制器（DispatcherServlet），所有的请求都有经过它来统一分发。

```
<!-- Sping MVC配置 -->
<servlet>
	<servlet-name>DispatcherServlet</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<!-- 指定SpingMVC的配置文件 -->
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath*:dispatcher_servlet.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
	<servlet-name>DispatcherServlet</servlet-name>
	<url-pattern>/</url-pattern>
</servlet-mapping>

```
#### dispatcher_servlet.xml配置

这个是指SpringMVC的配置

```
<!-- 扫描Controller所在的包 -->
<context:component-scan base-package="com.szl.springweb.controller">
	<!-- 如果扫描的是所有的包,需要以下配置,排除Service -->
	<!-- <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/> -->
</context:component-scan>

<!-- 相当于注册RequestMappingHandlerMapping和RequestMappingHandlerAdapter -->
<mvc:annotation-driven/>

<!-- 定义跳转的文件的前后缀 ，视图模式配置-->
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<!-- 这里的配置我的理解是自动给后面action的方法return的字符串加上前缀和后缀，变成一个 可用的url地址 -->
	<property name="prefix" value="/WEB-INF/jsp/" />
	<property name="suffix" value=".jsp" />
</bean>

```

#### Controller代码

```

@Controller
public class UserController {
	
    @Autowired
    @Qualifier(value="userServiceImpl")
    private UserService UserService;
    
    @RequestMapping(value = "queryUser",produces="application/json; charset=UTF-8")
    @ResponseBody
    public String queryUserList(){
    	List<Map<String, Object>> queryUser = UserService.queryUser();
    	System.out.println(queryUser.toString());
    	return JSONObject.toJSONString(queryUser);
    }

}

```

#### 启动tomcat,访问controller

![image](\images\spring\1\mvcurl.jpg)

#### 总结

1.此处只是简单的整合SpirngMVC,返回JSON数据;

2.SpringMVC的详细功能介绍可以参看[官网](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc)


### 异常总结

我们在UserController代码中只用了 `@Qualifier(value="userServiceImpl")`，因为`@Autowired`默认根据类型自动装配，如果当Spring上下文中存在不止一个userServiceImpl类型的bean时，就会抛出BeanCreationException异常，见以下异常代码，所以我们需要使用`@Qualifier`来配合指定具体的Bean。

```
org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type [com.szl.springorm.service.UserService] is defined: expected single matching bean but found 2: userServiceImpl,userServiceImpl1
	org.springframework.beans.factory.config.DependencyDescriptor.resolveNotUnique(DependencyDescriptor.java:172)
	org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1065)
	org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1019)
	org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:566)
	org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:88)
	org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:349)
	org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1214)
	org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:543)
	org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:482)
	org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306)
	org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
	org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
	org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
	org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:776)
	org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:861)
	org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:541)
	org.springframework.web.servlet.FrameworkServlet.configureAndRefreshWebApplicationContext(FrameworkServlet.java:668)
	org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:634)
	org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:682)
	org.springframework.web.servlet.FrameworkServlet.initWebApplicationContext(FrameworkServlet.java:553)
	org.springframework.web.servlet.FrameworkServlet.initServletBean(FrameworkServlet.java:494)
	org.springframework.web.servlet.HttpServletBean.init(HttpServletBean.java:136)
	javax.servlet.GenericServlet.init(GenericServlet.java:158)
	org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502)
	org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)
	org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:616)
	org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:509)
	org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1104)
	org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:684)
	org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1520)
	org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1476)
	java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	java.lang.Thread.run(Thread.java:745)

```








