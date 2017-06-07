---
title: 第七篇之Spring Transaction
date: 2017-06-07 11:12
categories: Spring的那点事
tags: Spring
---

## 数据库事务概念

> 数据库事务(Database Transaction) ，是指作为单个逻辑工作单元执行的一系列操作，要么完全地执行，要么完全地不执行。

### 相关属性
名称|解释
---|---
原子性(Atomicity)|事务必须是原子工作单元。对于其数据修改，要么全都执行，要么全都不执行
一致性(Consistency)|事务在完成时，必须使所有的数据都保持一致状态
隔离性(Isolation)|由并发事务所作的修改必须与任何其它并发事务所作的修改隔离，这需要事务隔离级别来指定隔离性
持久性(Durability)|事务完成之后，它对于系统的影响是永久性的

### 隔离级别

在数据库操作中，为了有效保证并发读取数据的正确性，提出的事务隔离级别。
名称|解释
---|---
未提交读(Read Uncommitted)|最低隔离级别，一个事务能读取到别的事务未提交的更新数据，很不安全，可能出现丢失更新、脏读、不可重复读、幻读
提交读(Read Committed)|一个事务能读取到别的事务提交的更新数据，不能看到未提交的更新数据，不可能出现丢失更新、脏读，但可能出现不可重复读、幻读
可重复读(Repeatable Read)|保证同一事务中先后执行的多次查询将返回同一结果，不受其他事务影响，不可能出现丢失更新、脏读、不可重复读，但可能出现幻读
序列化(Serializable)|最高隔离级别，不允许事务并发执行，而必须串行化执行，最安全，不可能出现更新、脏读、不可重复读、幻读

> 隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大。对于多数应用程序，可以优先考虑把数据库系统的隔离级别设为Read Committed。它能够避免脏读取，而且具有较好的并发性能。尽管它会导致不可重复读、幻读和第二类丢失更新这些并发问题，在可能出现这类问题的个别场合，可以由应用程序采用悲观锁或乐观锁来控制。

## Spring提供的事务管理

Spring并不直接管理事务，而是提供了多种事务管理器，将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现

Spring事务管理涉及的接口的如下
![image](\images\spring\1\spring_transaction.jpg)
图片来源于网络

### PlatformTransactionManager接口
#### PlatformTransactionManager接口：

```
package org.springframework.transaction;

public interface PlatformTransactionManager {

//返回一个已经激活的事务或创建一个新的事务（根据给定的TransactionDefinition类型参数定义的事务属性）
TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
//提交
void commit(TransactionStatus status) throws TransactionException;
//回滚
void rollback(TransactionStatus status) throws TransactionException;

}

```
#### TransactionDefinition接口

这个接口提供定义事务属性

```
package org.springframework.transaction;

import java.sql.Connection;
public interface TransactionDefinition {

	int PROPAGATION_REQUIRED = 0;

	int PROPAGATION_SUPPORTS = 1;
	
	int PROPAGATION_MANDATORY = 2;

	int PROPAGATION_REQUIRES_NEW = 3;

	int PROPAGATION_NOT_SUPPORTED = 4;

	int PROPAGATION_NEVER = 5;

	int PROPAGATION_NESTED = 6;

	int ISOLATION_DEFAULT = -1;

	int ISOLATION_READ_UNCOMMITTED = Connection.TRANSACTION_READ_UNCOMMITTED;
	
	int ISOLATION_READ_COMMITTED = Connection.TRANSACTION_READ_COMMITTED;

	int ISOLATION_REPEATABLE_READ = Connection.TRANSACTION_REPEATABLE_READ;

	int ISOLATION_SERIALIZABLE = Connection.TRANSACTION_SERIALIZABLE;

	int TIMEOUT_DEFAULT = -1;
    //返回定义的事务传播行为
	int getPropagationBehavior();
    //返回定义的事务隔离级别
	int getIsolationLevel();
    //返回定义的事务超时时间
	int getTimeout();
    //返回定义的事务是否是只读的
	boolean isReadOnly();
    //返回定义的事务名字
	String getName();

}


```

#### TransactionStatus接口

这个接口提供简单的控制事务执行和查询事务状态的方法

```
package org.springframework.transaction;
import java.io.Flushable;
public interface TransactionStatus extends SavepointManager, Flushable {
    //返回当前事务状态是否是新事务
	boolean isNewTransaction();
    //返回当前事务是否有保存点
	boolean hasSavepoint();
    //设置当前事务应该回滚
	void setRollbackOnly();
    //回当前事务是否应该回滚
	boolean isRollbackOnly();
    //用于刷新底层会话中的修改到数据库，一般用于刷新如Hibernate/JPA的会话，可能对如JDBC类型的事务无任何影响
	@Override
	void flush();
    //当前事务否已经完成
	boolean isCompleted();

}


```

### Spring内置事务管理器实现
名称|解释
---|---
DataSourceTransactionManager|位于org.springframework.jdbc.datasource包中，数据源事务管理器，提供对单个javax.sql.DataSource事务管理，用于Spring JDBC抽象框架、iBATIS或MyBatis框架的事务管理
JdoTransactionManager|位于org.springframework.orm.jdo包中，提供对单个javax.jdo.PersistenceManagerFactory事务管理，用于集成JDO框架时的事务管理
JpaTransactionManager|位于org.springframework.orm.jpa包中，提供对单个javax.persistence.EntityManagerFactory事务支持，用于集成JPA实现框架时的事务管理
HibernateTransactionManager|于org.springframework.orm.hibernate3包中，提供对单个org.hibernate.SessionFactory事务支持，用于集成Hibernate框架时的事务管理
JtaTransactionManager|位于org.springframework.transaction.jta包中，提供对分布式事务管理的支持，并将事务管理委托给Java EE应用服务器事务管理器
OC4JjtaTransactionManager|位于org.springframework.transaction.jta包中，Spring提供的对OC4J10.1.3+应用服务器事务管理器的适配器，此适配器用于对应用服务器提供的高级事务的支持
WebSphereUowTransactionManager|位于org.springframework.transaction.jta包中，Spring提供的对WebSphere 6.0+应用服务器事务管理器的适配器，此适配器用于对应用服务器提供的高级事务的支持
WebLogicJtaTransactionManager|位于org.springframework.transaction.jta包中，Spring提供的对WebLogic 8.1+应用服务器事务管理器的适配器，此适配器用于对应用服务器提供的高级事务的支持


### Spring事务属性
事务管理器接口PlatformTransactionManager通过getTransaction(TransactionDefinition definition)方法来得到事务，这个方法里面的参数是TransactionDefinition类，这个类就定义了一些基本的事务属性

#### 传播行为

事务的第一个方面是传播行为（propagation behavior）。当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。

Spring定义了七种传播行为：

传播行为|含义
---|---
PROPAGATION_REQUIRED|表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务
PROPAGATION_SUPPORTS|表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行
PROPAGATION_MANDATORY|表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常
PROPAGATION_REQUIRED_NEW|表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager
PROPAGATION_NOT_SUPPORTED|表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager
PROPAGATION_NEVER|表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常
PROPAGATION_NESTED|表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。

#### 隔离级别

隔离级别定义了一个事务可能受其他并发事务影响的程度

隔离级别|含义
---|---
ISOLATION_DEFAULT|默认隔离级别，即使用底层数据库默认的隔离级别
ISOLATION_READ_UNCOMMITTED|未提交读；
ISOLATION_READ_COMMITTED|提交读，一般情况下我们使用这个
ISOLATION_REPEATABLE_READ|可重复读
ISOLATION_SERIALIZABLE|序列化

## Spring事务测试

> Spring提供了对编程式事务和声明式事务的支持，编程式事务允许用户在代码中精确定义事务的边界，而声明式事务（基于AOP）有助于用户将操作与事务规则进行解耦。 
简单地说，编程式事务侵入到了业务代码里面，但是提供了更加详细的事务管理；而声明式事务由于基于AOP，所以既能起到事务管理的作用，又可以不影响业务代码的具体实现。


### 编程式事务

Spring提供两种方式的编程式事务管理，分别是：

1.使用PlatformTransactionManager

2.使用TransactionTemplate
> TransactionTemplate模板类用于简化事务管理，事务管理由模板类定义，而具体操作需要通过TransactionCallback回调接口或TransactionCallbackWithoutResult回调接口指定，通过调用模板类的参数类型为TransactionCallback或TransactionCallbackWithoutResult的execute方法来自动享受事务管理，TransactionTemplate是线程安全的。

#### 使用PlatformTransactionManager
##### xml配置

```
<!-- 定义一个某个框架平台的TransactionManager,此处使用JDBC -->
 <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">    
    <property name="dataSource" ref="dataSource"/>  
</bean> 

```

##### 测试代码
```
@Test
public void testPlatformTransactionManager(){
 	PlatformTransactionManager ptm = ac.getBean("transactionManager",PlatformTransactionManager.class);
 	UserServiceImpl userDaoImpl = ac.getBean("userServiceImpl",UserServiceImpl.class);
 	//定义事务属性
 	DefaultTransactionDefinition dtd = new DefaultTransactionDefinition();
 	//设置隔离级别
 	dtd.setIsolationLevel(DefaultTransactionDefinition.ISOLATION_DEFAULT);
	//设置传播行为
 	dtd.setPropagationBehavior(DefaultTransactionDefinition.PROPAGATION_REQUIRED);
 	//获取事务状态
 	TransactionStatus ts = ptm.getTransaction(dtd);
	List<Map<String, Object>> queryUser;
	try {
		//业务逻辑的处理
		queryUser = userDaoImpl.queryUser();
		System.out.println("testMapperFactoryBean:"+queryUser.toString());
		//提交
		ptm.commit(ts);
	} catch (Exception e) {
		//回滚
		ptm.rollback(ts);
	}	
}

```

#### 使用TransactionTemplate

##### xml配置

```
<!-- 定义一个某个框架平台的TransactionManager,此处使用JDBC -->
 <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">    
    <property name="dataSource" ref="dataSource"/>  
</bean> 

<!-- 使用TransactionTemplate -->
<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
	<!-- 设置transactionManager -->
	<constructor-arg name="transactionManager" ref="transactionManager"></constructor-arg>
</bean>

```

##### 测试代码
```
@Test
public void testTransactionTemplate(){
	UserServiceImpl userDaoImpl = ac.getBean("userServiceImpl",UserServiceImpl.class);
	TransactionTemplate tt = ac.getBean("transactionTemplate",TransactionTemplate.class);
			List<Map<String, Object>> queryUser = tt.execute(new TransactionCallback<List<Map<String, Object>>>() {

		@Override
		public List<Map<String, Object>> doInTransaction(TransactionStatus status) {
			//业务逻辑的处理
			try {
				return userDaoImpl.queryUser();
			} catch (Exception e) {
				//异常回滚
				status.setRollbackOnly();
			}
			return null;
		}
	});
	System.out.println("testTransactionTemplate:"+queryUser.toString());
}

```

### 声明式事务

> 声明式事务支持，使用该方式后最大的获益是简单，事务管理不再是令人痛苦的，而且此方式属于无侵入式，对业务逻辑实现无影响。

#### xml配置

```
<!-- 定义一个某个框架平台的TransactionManager,此处使用JDBC -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">    
    <property name="dataSource" ref="dataSource"/>  
</bean> 


<!-- ****************************声明式事务***************************************** -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
	<tx:attributes>
		<tx:method name="update*" propagation="REQUIRED" isolation="DEFAULT"/>
		<tx:method name="*" propagation="REQUIRED" isolation="DEFAULT" read-only="true"/>
	</tx:attributes>
</tx:advice>

<aop:config proxy-target-class="true">
	<aop:pointcut expression="execution(* com.szl.springorm.service.impl.*.*(..))" id="txPointcut"/>
	<aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
</aop:config>
		
```

#### 测试代码

```

/**
 * 声明式事务测试
 * @author SongZhangLiang
 */
@Test
public void testTransaction(){
	UserServiceImpl userDaoImpl = ac.getBean("userServiceImpl",UserServiceImpl.class);
	List<Map<String, Object>> queryUser = userDaoImpl.queryUser();
	System.out.println("testPlatformTransactionManager:"+queryUser.toString());
}

```

#### 声明式事务XML配置详解

属性|含义
---|---
`<tx:advice`|事务通知定义，用于指定事务属性，其中`transaction-manager`属性指定事务管理器，并通过`<tx:attributes >`指定具体需要拦截的方法
`<tx:method name="update*">`|表示将拦截以update开头的方法，被拦截的方法将应用配置的事务属性：propagation="REQUIRED"表示传播行为是Required，isolation="DEFAULT"表示隔离级别依赖底层事务系统
`<tx:method name="*">`|表示将拦截其他所有方法，被拦截的方法将应用配置的事务属性：propagation="REQUIRED"表示传播行为是Required，isolation="DEFAULT"表示隔离级别依赖底层事务系统，read-only="true"表示事务只读
`<aop:config>`|AOP相关配置
`<aop:pointcut/>`|切入点定义，定义名为"serviceMethod"的aspectj切入点，切入点表达式为`execution(* com.szl.springorm.service.impl.*.*(..))`表示拦截impl包及子包下的任何类的任何方法
`<aop:advisor>`|Advisor定义，其中切入点为txPointcut，通知为txAdvice


### @Transactional实现事务管理

> 对声明式事务管理，Spring提供基于@Transactional注解方式来实现，但需要Java 5+。
 注解方式是最简单的事务配置方式，可以直接在Java源代码中声明事务属性，且对于每一个业务类或方法如果需要事务都必须使用此注解。
 
#### XML配置

配置支持声明式事务

```
<tx:annotation-driven transaction-manager="transactionManager"/>

```

#### 业务代码

业务逻辑层代码，我们在方法上使用`@Transactional`注解
```
@Transactional
@Override
public List<Map<String, Object>> queryUser() {
	return userDao1.queryUser();
}

```
 
#### 测试代码

```
/**
 * 声明式事务测试_注解
 * @author SongZhangLiang
 */
@Test
public void testTransactionAnnotation(){
	com.szl.springtransaction.service.impl.UserServiceImpl userDaoImpl = ac.getBean("userServiceImpl1",com.szl.springtransaction.service.impl.UserServiceImpl.class);
	List<Map<String, Object>> queryUser = userDaoImpl.queryUser();
	System.out.println("testTransactionAnnotation:"+queryUser.toString());
}

```

#### @Transactional配置介绍

Spring提供的`<tx:annotation-driven/>`用于开启对注解事务管理的支持，从而能识别Bean类上的@Transactional注解元数据，其具有以下属性：

- transaction-manager：指定事务管理器名字，默认为transactionManager，当使用其他名字时需要明确指定
- proxy-target-class：表示将使用的代码机制，默认false表示使用JDK代理，如果为true将使用CGLIB代理
- order：定义事务通知顺序，默认Ordered.LOWEST_PRECEDENCE，表示将顺序决定权交给AOP来处理。

Spring使用`@Transaction`来指定事务属性，可以在接口、类或方法上指定，如果类和方法上都指定了`@Transaction`，则方法上的事务属性被优先使用，具体属性如下：

- value：指定事务管理器名字，默认使用`<tx:annotation-driven/>`指定的事务管理器，用于支持多事务管理器环境
- propagation：指定事务传播行为，默认为Required，使用Propagation.REQUIRED指定
- isolation：指定事务隔离级别，默认为“DEFAULT”，使用Isolation.DEFAULT指定
- readOnly：指定事务是否只读，默认false表示事务非只读
- timeout：指定事务超时时间，以秒为单位，默认-1表示事务超时将依赖于底层事务系统
- rollbackFor：指定一组异常类，遇到该类异常将回滚事务
- rollbackForClassname：指定一组异常类名字，其含义与`<tx:method>`中的rollback-for属性语义完全一样
- noRollbackFor：指定一组异常类，即使遇到该类异常也将提交事务，即不回滚事务
- noRollbackForClassname：指定一组异常类名字，其含义与`<tx:method>`中的no-rollback-for属性语义完全一样

#### @Transactional注意事项
- 如果在接口、实现类或方法上都指定了@Transactional 注解，则优先级顺序为方法>实现类>接口
- 默认只对RuntimeException异常回滚
- 在使用Spring代理时，默认只有在public可见度的方法的@Transactional 注解才是有效的，其它可见度（protected、private、包可见）的方法上即使有@Transactional 注解也不会应用这些事务属性的，Spring也不会报错，如果你非要使用非公共方法注解事务管理的话，可考虑使用AspectJ




