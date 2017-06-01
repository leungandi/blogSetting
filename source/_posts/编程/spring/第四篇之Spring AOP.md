---
title: 第四篇之Spring AOP
date: 2017-05-28 15:23
categories: Spring的那点事
tags: Spring
---

### AOP概念
AOP是Aspect Oriented Programming的缩写，也就是面向切面编程的意思，其设计思想来源于代理模式，在此基础上进行封装扩展，最终形成了一些功能强大的AOP框架，如AspectJ。

- 面向切面编程

提供从另一个角度来考虑程序结构从而完善面向对象编程(OOP)，简单理解面向切面编程，就是在不改变原程序的基础上为代码增加新的功能，如增加日志输出、启动数据库事物等等；

- 基本概念

名称|解释
---|---
切入点（Pointcut）| 可以插入增强处理的方法
连接点（Jointpoint）| 可以插入增强处理的方法，方法处称为连接点
通知（Advice）| 在连接点执行的行为
切面（Aspect）| 切入点和通知的集合

- 增强处理（Advice）

名称|解释
---|---
Before Advice|前置通知，在切入点选择的连接点方法之前执行
After Advice|后置通知，在切入点选择的连接点方法之后执行
After returning Advice|后置返回通知，在切入点选择的连接点的方法正常执行完毕时执行的通知，必须是连接点处的方法没抛任何异常
After throwing Advice|后置异常通知，在切入点选择的连接点的方法抛出异常时执行的通知，必须是连接点处的方法抛出异常
Around Advices|环绕通知，在切入点选择的连接点的方法前后都可以执行的通知

### AOP的日志输出

#### 业务代码

即目标类，需要加入通知的类。

我们此处模拟向数据库插入用户信息，定义的代码如下：

- 模拟dao的实现

```
package com.szl.springaop.dao.impl;

import com.szl.springaop.dao.UserDao;

public class UserDaoImpl implements UserDao{

	/**
	 * 模拟向数据库插入用户信息
	 */
	@Override
	public void insertUser() {
		System.out.println("向数组库插入用户信息");
	}
	


}
	

```
- 模拟service的实现

```
package com.szl.springaop.service.impl;

import com.szl.springaop.dao.UserDao;
import com.szl.springaop.service.UserService;

public class UserServiceImpl implements UserService {

	private UserDao userDao;
	
	public UserDao getUserDao() {
		return userDao;
	}

	public void setUserDao(UserDao userDao) {
		this.userDao = userDao;
	}

	/**
	 * 模拟向数据库插入用户数据
	 */
	@Override
	public void insetUser() {
		userDao.insertUser();
	}

}


```

#### 日志代码

即切面支持类，切面是切入点和通知的组合，而切面是通过配置方式定义的，因此在定义切面前，我们需要定义切面支持类，切面支持类提供了通知实现


```
package com.szl.springaop.aop;

public class AopLog {
	
	/**
	 * 前置通知
	 * @author Andrew Song
	 */
	public void beforeAdvice(){
		System.out.println("-----Hello,this is beforeAdvice-----");
	}
	/**
	 * 后置通知
	 * @author Andrew Song
	 */
	public void afterAdvice(){
		System.out.println("-----Hello,this is afterAdvice-----");
	}
		
	/**
	 * 后置返回通知
	 * @author Andrew Song
	 */
	public void afterReturningAdvice(){
		System.out.println("-----Hello,this is afterReturningAdvice-----");
	}
	/**
	 * 后置异常通知
	 * @author Andrew Song
	 */
	public void afterThrowingAdvice(){
		System.out.println("-----Hello,this is afterThrowingAdvice-----");
	}
	
	/**
	 * 环绕通知
	 * @author Andrew Song
	 */
	public void aroundAdvice(){
		System.out.println("-----Hello,this is aroundAdvice-----");
	}
	
}

```

#### xml中的配置

- 配置切面代码如下：

```
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

    <bean id="userDao" class="com.szl.springaop.dao.impl.UserDaoImpl" />
	<bean id="userService" class="com.szl.springaop.service.impl.UserServiceImpl">
		<property name="userDao" ref="userDao"></property>
	</bean>

	<bean id="aopLog" class="com.szl.springaop.aop.AopLog"></bean>
    
	<aop:config proxy-target-class="true">
		<aop:pointcut expression="execution(* com.szl.springaop.service.impl.*.*(..))" id="servicePointcut"/>
		<aop:aspect ref="aopLog">
			<aop:before method="beforeAdvice" pointcut-ref="servicePointcut"/>
		</aop:aspect>
	</aop:config>
</beans>

```
- 配置介绍
1. 在spring容器中使用AOP配置，我们要在<beans>标签中引入AOP的命名空间，以导入与AOP配置相关的标签；
2. 与AOP相关的配置都放在<aop:config>标签中；
3.  `<aop:pointcut>`是切入点标签，可以在expression属性中配置切入点；
4. `<aop:aspect>`是织入标签，在织入时要设置通知的类型,以上切面配置的意思是：在匹配"servicePointcut"的切入点方法之前织入"aopLog"对象的 "beforeAdvice方法"；

### 运行结果

```
-----Hello,this is beforeAdvice-----
向数组库插入用户信息

```
我们可以看到，前置通知已经正确的运行了。

**ps:其他通知的运行结果就不再演示，可参考实例代码**

### 其他配置

#### 获取连接点信息

- 切面支持类

```
/**
 * 前置通知,获取连接点信息
 * @author Andrew Song
 */
public void beforeAdviceJoinPoint(JoinPoint joinPoint){
	System.out.println("-----Hello,this is beforeAdviceJoinPoint-----");
	System.out.println("连接点对象："+joinPoint.getTarget().getClass().getName());
	System.out.println("连接点方法："+joinPoint.getSignature());
	System.out.println("连接点参数："+joinPoint.getArgs().toString());
}

```

- 输出结果

```
-----Hello,this is beforeAdviceJoinPoint-----
连接点对象：com.szl.springaop.service.impl.UserServiceImpl
连接点方法：void com.szl.springaop.service.impl.UserServiceImpl.insetUser()
连接点参数：[Ljava.lang.Object;@429bffaa
向数组库插入用户信息

```

#### 后置返回通知获取返回值

- 切面支持类

```
/**
 * 后置返回通知-获取返回值
 * @author Andrew Song
 */
public void afterReturningAdviceObject(Object objectVal){
	System.out.println("-----Hello,this is afterReturningAdvice-----");
	System.out.println("业务方法的返回值是："+objectVal);
}

```

- XML配置

```
<aop:after-returning method="afterReturningAdviceObject" pointcut-ref="servicePointcut" returning="objectVal"/>
```
**如果需要获取返回值，需要添加returning属性**

- 输出结果

```
-----Hello,this is afterReturningAdvice-----
业务方法的返回值是：你好，世界
```

#### 后置异常通知获取异常信息

- 切面支持类

```
/**
 * 后置异常通知-获取异常信息
 * @author Andrew Song
 */
public void afterThrowingAdviceException(Exception ex){
	System.out.println("-----Hello,this is afterThrowingAdvice-----");
	System.out.println("抛出的异常信息为："+ex.getMessage());
}
	
```

- XML配置

```
<aop:after-throwing method="afterThrowingAdviceException" pointcut-ref="servicePointcut" throwing="ex"/>
```
**如果需要获取返回的异常，需要添加throwing属性**

- 输出结果

```
-----Hello,this is afterThrowingAdvice-----
抛出的异常信息为：/ by zero
```

#### 强大的环绕通知

环绕通知是功能最强大的通知处理，Spring把目标方法的控制权全部都交给了它，在环绕通知处理过程中，可以获取或修改目标方法的参数、返回值，可以对它进行异常处理，甚至可以决定目标方法是否放行。

- 切面支持类

```
/**
 * 环绕通知
 * @author Andrew Song
 * @throws Throwable 
 */
public void aroundAdvicePjp(ProceedingJoinPoint pjp) throws Throwable{
	System.out.println("-----目标方法之前输出：Hello,this is aroundAdvicePjp-----");
	//获取目标方法的参数
//	Object[] args = pjp.getArgs();
	if(true){
		pjp.proceed();
	}
	System.out.println("-----目标方法之后输出：Hello,END-----");
}
	

```

- XML配置

```
<aop:around method="aroundAdvicePjp" pointcut-ref="servicePointcut"/>

```

- 输出结果

```
-----目标方法之前输出：Hello,this is aroundAdvicePjp-----
向数组库插入用户信息
-----目标方法之后输出：Hello,END-----
```

- 总结

环绕通知的第一个参数必须是一个ProceedingJoinPoint的类型，其他getArgs()可以获取目标方法的参数，调用它的proceed()方法就是调用目标方法，所以可以通过它给目标方法传参并获取返回结果，总而言之，控制proceed()方法是否执行就相当于控制了目标方法是否执行。

### 简化AOP配置

#### 使用schema

以上案列已经使用过schema形式实现了AOP的配置

#### 使用annotation

使用annotation之前，我们先了解AspectJ的概念，AspectJ是面向切面的框架，它扩展了JAV语言，@AspectJ使用JDK5.0注解和正规的AspectJ切点表达式语言。

#### 注解类型
名称|解释
---|---
@Aspect|声明切面
@Before|前置通知
@After|后置通知
@AfterReturning|后置返回通知
@AfterThrowing|后置异常通知


#### 测试demo

以前置通知为列子

- 切面

```
package com.szl.springaop.aop;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect //用@Aspect注解声明切面
public class AspectJLog {

    /**
     * 前置通知
     * @author Andrew Song
     */
    @Before("execution(* com.szl.springaop.service.impl.*.*(..))") //声明切入点和通知类型
    public void beforeAdvice(){
    	System.out.println("----Hello,this is beforeAdvice,使用注解");
    }

}

```

- test

```
@Test
public void testAdviceAspectJ() {
	//从容器中获取bean
	UserService us = ac.getBean("userService",UserServiceImpl.class);
	//执行业务逻辑
	us.insetUser();
}


```

- xml配置

```
<!--  Spring默认不支持@AspectJ风格的切面声明, 为了支持需要使用如下配置  -->
<aop:aspectj-autoproxy />
<bean id="aspectJLog" class="com.szl.springaop.aop.AspectJLog"></bean>

```

- 输出结果

```
----Hello,this is beforeAdvice,使用注解
向数组库插入用户信息

```

- 总结

1.配置文件需要使用`<aop:aspectj-autoproxy/>`来开启注解风格的@AspectJ支持

2.注解风格可以减少在XML的配置








