---
title: 第二篇之Spirng IOC
date: 2017-05-26 13:49
categories: Spring的那点事
tags: Spring
---

### 了解IOC
#### IOC是什么
控制反转（Inversion of Control），不是一种技术，而是一种思想，是面向对象编程中的一种设计原则，用来降低代码之间的耦合度。
#### IOC的好处
对象的创建和依赖由容器负责，对象与对象之间是松耦合的，利于功能复用。
#### IOC和DI
- DI即依赖注入（Dependency Injection），由容器动态的将某个依赖关系注入到组件之中，它们是spring核心思想的不同方面的描述。
- IOC是目的，DI是手段，IOC让程序员不需要去new对象，由IOC容器负责，当需要使用某些组件的时候由框架注入（DI）进来。

#### 打印Hello Ioc

1.定义HelloIoc的接口
```
package com.szl.springioc.dao;

public interface HelloIoc{
	
	public void sayHello();
	
} 

```
2.接口定义完成，实现接口来完成打印“Hello Ioc”的功能
```
package com.szl.springioc.dao.impl;

import com.szl.springioc.dao.HelloIoc;

public class HelloIocImpl implements HelloIoc{

	@Override
	public void sayHello() {
		System.out.println("hello Ioc");
	}
} 

```
3.接下来我们通过配置文件让Spring Ioc来管理它们，我们在工程的resources目录建立一个HelloIoc.xml文件，如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx" xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	    http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">
	
	<bean id="helloIoc" class="com.szl.springioc.dao.impl.HelloIocImpl" />

</beans>

```
4.现在我们可以实例化容器，从容器中获取对象，来实现我们的功能
```
package com.szl.SpringIoc.test;

import org.junit.Before;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.szl.springioc.dao.HelloIoc;
import com.szl.springioc.dao.impl.HelloIocImpl;

public class SpringTest {
	ApplicationContext ac;
	@Before
	public void init(){
		//读取配置文件从而实例化IOC容器
		ac = new ClassPathXmlApplicationContext("HelloIoc.xml");
	}

	@Test
	public void testHello() {
		//从容器中获取bean
		HelloIoc hi = (HelloIocImpl) ac.getBean("helloIoc");
		//执行业务逻辑
		hi.sayHello();
	}
}

```
执行Junit Test后，我们可以在控制台看到程序已经打印出“Hello Ioc”。

---

### IOC容器的XML文件配置
#### Bean标签

```
<beans>
	<!-- 使用ID作为标识符 -->
	<bean id="helloIoc"  class="com.szl.springioc.dao.impl.HelloIocImpl" />
	<!-- 使用NAME作为标识符 -->
	<bean name="helloIocName" class="com.szl.springioc.dao.impl.HelloIocImpl" />
	<!-- 不使用任何标识符 ,仅使用类的全限定名-->
	<bean class="com.szl.springioc.dao.impl.HelloIocImpl" />
	<!-- 同时使用ID和NAME，如果名称相同,容器会自动处理 -->
	<bean id="helloIocSame" name="helloIocSame"  class="com.szl.springioc.dao.impl.HelloIocImpl" />
</beans>

```
Spring IoC容器目的就是管理Bean，`<bean>`标签主要用来进行Bean定义，主要有以下属性：
- id:  Bean的命名，使用id作为“标识符”，必须唯一。
- name:  Bean的命名，使用name作为“标识符”，必须唯一。
- scope：Bean的作用域。
- class:类的全限定名。

测试实例如下：
```
public class SpringTest {
	ApplicationContext ac;
	@Before
	public void init(){
		//读取配置文件从而实例化IOC容器
		ac = new ClassPathXmlApplicationContext("HelloIoc.xml");
	}

	@Test
	public void testBean(){
		//测试bean使用id
		HelloIoc hiId = (HelloIocImpl) ac.getBean("helloIoc");
		hiId.sayHello();
		
		//测试bean使用name
		HelloIoc hiName = (HelloIocImpl) ac.getBean("helloIocName");
		hiName.sayHello();
		
		//测试bean仅使用类的全限定名
		HelloIoc hi = ac.getBean(HelloIocImpl.class);
		hi.sayHello();
		
		//测试Bean同时使用id和name
		HelloIoc hiIdAndName = (HelloIocImpl) ac.getBean("helloIocSame");
		hiIdAndName.sayHello();
	}
}

```
#### Bean的作用域
作用域|说明
---|---
singleton|默认值，Spring已单例模式创建Bean的实例，即容器中该Bean的实例只有一个
prototype|每次从容器中获取Bean时，都会创建一个新的实例
request|用于WEB应用环境，针对每次HTTP请求都会创建一个实例
session|用于WEB应用环境，同一个回话共享一个实例
global session|仅在Protlet的WEB应用中使用，同一个全局会话共享一个实例，对于非Protlet环境，等同于session


### 解读IOC容器
1.org.springframework.beans和org.springframework.context是Spring Ioc的基本组成，BeanFactory是整个IOC容器的最基本接口。  
2.BeanFactory接口有3个类：
- AutowireCapableBeanFactory  
该接口的功能是主要实现了Bean的自动装配功能，为实例Bean暴露了装配的功能
- HierarchicalBeanFactory  
定义了BeanFactory的父子链结构 
- ListableBeanFactory  
该接口的功能是用来列出所有Bean的名称、类型、注解等信息   

3.ApplicationContext接口继承了HierarchicalBeanFactory和ListableBeanFactory，所以ApplicationContext包含BeanFactory的所有功能，而已在国际化支持、资源访问（如URL和文件）、事件传播等方面进行了良好的支持。

4.实例化ApplicationContext的常用实现类
- ClassPathXmlApplicationContext  
从类路径ClassPath中寻找指定的XML配置文件，找到并装载完成ApplicationContext的实例化工作。
- FileSystemXmlApplicationContext  
从指定的文件系统路径中寻找指定的XML配置文件，找到并装载完成ApplicationContext的实例化工作。
- XmlWebApplicationContext  
从Web应用中寻找指定的XML配置文件，找到并装载完成ApplicationContext的实例化工作。

这些实现类的主要区别就是装载Spring配置文件实例化ApplicationContext容器的方式不同，在ApplicationContext实例化后，同样通过getBean方法从ApplicationContext容器中获取装配好的Bean实例以供使用。

**注：在Java项目中通过ClassPathXmlApplicationContext类手动实例化ApplicationContext容器通常是不二之选。但对于Web项目就不行了，Web项目的启动是由相应的Web服务器负责的，因此，在Web项目中ApplicationContext容器的实例化工作最好交给Web服务器来完成。**
















