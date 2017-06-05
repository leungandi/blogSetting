---
title: 第六篇之Spring ORM
date: 2017-06-5 14:56
categories: Spring的那点事
tags: Spring
---

## ORM是什么?

> 对象关系映射（英语：(Object Relational Mapping，简称ORM，或O/RM，或O/R mapping），是一种程序技术，用于实现面向对象编程语言里不同类型系统的数据之间的转换

常见的ORM框架有JPA、MYBATIS、HIBERNATE等。

## Spring对ORM的支持

Spring对ORM的支持主要表现在以下方面

### 一致的异常体系结构
Spring提供了一种方便的方法，把特定于某种技术的异常，如SQLException， 转化为自己的异常，这种异常属于以 DataAccessException为根的异常层次。这些异常封装了原始异常对象，这样就不会有丢失任何错误信息的风险。

###  一致的DAO抽象支持
为了便于以一种一致的方式使用各种数据访问技术，如JDBC、JDO和Hibernate， Spring提供了一套抽象DAO类扩展。这些抽象类提供了一些方法，通过它们可以 获得与当前使用的数据访问技术相关的数据源和其他配置信息。

- JdbcDaoSupport：JDBC数据访问对象的基类。 需要一个DataSource，同时为子类提供 JdbcTemplate。

- HibernateDaoSupport： Hibernate数据访问对象的基类。 需要一个SessionFactory，同时为子类提供 HibernateTemplate。也可以选择直接通过 提供一个HibernateTemplate来初始化。

- JdoDaoSupport：JDO数据访问对象的基类。 需要设置一个PersistenceManagerFactory， 同时为子类提供JdoTemplate。
- JpaDaoSupport：JPA数据访问对象的基类。 需要一个EntityManagerFactory，同时 为子类提供JpaTemplate。

### Spring事务管理
Spring对所有数据访问提供一致的事务管理，通过配置方式，简化事务管理。

---

**本文主要记录Spring和Mybatis的集成**

## Spring集成Mybatis?

### Mybatis介绍
[了解Mybatis,请点击前往](http://www.mybatis.org/mybatis-3/zh/index.html)

### 如何集成
#### 简单介绍
> MyBatis-Spring 会帮助你将 MyBatis 代码无缝地整合到 Spring 中。 使用这个类库中的类, Spring 将会加载必要的 MyBatis 工厂类和 session 类。 这个类库也提供一个简单的方式来注入 MyBatis 数据映射器和 SqlSession 到业务层的 bean 中。 而且它也会处理事务, 翻译 MyBatis 的异常到 Spring 的 DataAccessException 异常(数据访问异常,译者注)中。最终,它并 不会依赖于 MyBatis,Spring 或 MyBatis-Spring 来构建应用程序代码。
要使用 MyBatis-Spring 模块,你只需要包含 mybatis-spring-x.x.x.jar 文 件就可以了,并在类路径中加入相关的依赖。

> 如果你使用 Maven,那么在 pom.xml 中加入下面的代码即可:
```
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>x.x.x</version>
</dependency>
```
#### 配置
##### Spring XML配置

1.dataSource

配置数据源，此处我们使用dbcp2数据源

2.SqlSessionFactoryBean

- 用来集成Mybatis,它会创建SqlSessionFactory，SqlSessionFactory需要一个DataSource 
- `configLocation`用来指定Mybatis的配置文件路径及名称
- `mapperLocations`用来指定Mybatis的XML映射文件

新建orm_test.xml
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
       	<!-- 读取配置文件 -->
        <context:property-placeholder location="database.properties"/>
        
        <!-- 配置数据源 -->
        <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource"  
	    destroy-method="close">  
		    <property name="driverClassName" value="${jdbc.driver}" />  
		    <property name="url" value="${jdbc.url}" />  
		    <property name="username" value="${jdbc.username}" />  
		    <property name="password" value="${jdbc.password}" />  
		</bean>
 	 	
 	 	<!-- 在基本的 MyBatis 中,session 工厂可以使用 SqlSessionFactoryBuilder 来创建。而在 MyBatis-Spring 中,则使用 SqlSessionFactoryBean 来替代 -->
 	 	<!-- sqlSessionFactory配置 -->	 
 	 	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
 	 		<!-- 指定数据源 -->
	 	 	<property name="dataSource" ref="dataSource"></property>
	 	 	<!-- 指定mybatis配置文件 -->
			<property name="configLocation" value="classpath:configuration.xml"/> 
			<property name="mapperLocations" value="classpath*:com/szl/springorm/dao/impl/*.xml"></property>
 	 	</bean>
<beans> 	

```

##### Mybatis XML配置

新建configuration.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
	<settings>  
        <setting name="cacheEnabled" value="false"/>  
    </settings>  
</configuration>  

```

[更多详细配置介绍,点击进入官网](http://www.mybatis.org/mybatis-3/zh/configuration.html)

##### Mybatis 映射文件配置
- DAO接口
```
public interface UserDao {
	/**
	 * 增加用户
	 * @return
	 */
	int addUser(User user);
	
	/**
	 * 查询用户
	 * @return
	 */
	List<Map<String, Object>> queryUser();
	
	/**
	 * 删除用户
	 * @return
	 */
	int delUserById(Long id);

}

```

- 映射XML文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<!-- 命名空间,指向DAO接口 -->
<mapper namespace="com.szl.springorm.dao.UserDao" >
 
 <!-- 查询用户 -->
 <select id="queryUser" resultType="java.util.Map">
 	select * from t_users
 </select>
 
 <!-- 新增用户 -->
 <insert id="addUser" parameterType="com.szl.springorm.model.User">
 	INSERT INTO t_users(name,email)VALUES(#{user.name},#{user.email})
 </insert>
 
 <!-- 删除用户 -->
 <delete id="delUserById" parameterType="java.lang.Long">
 	delete from t_users where id=#{id}
 </delete>
 
</mapper>

```
[更多映射文件配置介绍,点击进入官网](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html)

##### 测试一下

```
public class SpringOrmTest {
	
	private ApplicationContext ac;
	
	@Before
	public void init() {
		ac = new ClassPathXmlApplicationContext("orm_test.xml");
	}
	
	/**
	 * 使用SqlSession进行数据库操作
	 * @author SongZhangLiang
	 */
	@Test
	public void testSqlSession() {
		/**
		 * 直接使用SqlSession进行数据库操作（使用指定的完全限定名“com.szl.springorm.dao.UserDao.queryUser”来调用映射语句）
		 */
		SqlSession ss = null;
		try {
			SqlSessionFactory ssf = ac.getBean("sqlSessionFactory",SqlSessionFactory.class);
			ss = ssf.openSession();
			List<Map<String, Object>> selectList = ss.selectList("com.szl.springorm.dao.UserDao.queryUser");
			System.out.println("testSqlSession:"+selectList.toString());
			ss.commit();
		} catch (Exception e) {
			//异常回滚
			System.out.println("异常："+e);
			ss.rollback();
		}
		finally {
			if(null != ss)
				ss.close();
		}
		/**
		 * 直接使用SqlSession进行数据库操作（使用Mapper 接口）
		 */
//		UserDao mapper = ss.getMapper(UserDao.class);
//		System.out.println("mapper:"+mapper.queryUser().toString());
	}

}
```

##### 总结

> 使用SqlSessionFactory来创建 SqlSession。一旦你获得一个session之后,你可以使用它来执行映射语句,提交或回滚连接,最后,当不再需要它的时候, 你可以关闭session

这样操作依旧繁琐，MyBatis-Spring提供了SqlSessionTemplate，qlSessionTemplate是 MyBatis-Spring 的核心。 这个类负责管理 MyBatis的SqlSession,调用MyBatis的SQL方法, 翻译异常。
SqlSessionTemplate 是线程安全的, 可以被多个 DAO 所共享使用。

### 使用SqlSessionTemplate

#### Spring XMl配置

```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  	<!-- 指定数据源 -->
  	<property name="dataSource" ref="dataSource"></property>
  	<!-- 指定mybatis配置文件 -->
	<property name="configLocation" value="classpath:configuration.xml"/> 
	<property name="mapperLocations" value="classpath*:com/szl/springorm/dao/impl/*.xml"></property>
</bean>
  	
<!-- 1.使用sqlSessionTemplate,注入sqlSessionFactory -->
<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"></constructor-arg>
</bean>

```

#### 测试一下

```

/**
 * 使用SqlSessionTemplate进行数据库操作
 * SqlSessionTemplate 是 MyBatis-Spring 的核心。
 * 这个类负责管理 MyBatis 的 SqlSession, 调用 MyBatis 的 SQL 方法, 翻译异常。 SqlSessionTemplate       是线程安全的, 可以被多个 DAO 所共享使用
 */
@Test
public void testSqlSessionTemplate(){
	SqlSession ss = ac.getBean("sqlSessionTemplate",SqlSession.class);
	List<Map<String, Object>> selectList = ss.selectList("com.szl.springorm.dao.UserDao.queryUser");
	System.out.println("testSqlSessionTemplate:"+selectList.toString());
}

```

### 使用sqlSessionDaoSupport

SqlSessionDaoSupport是一个抽象的支持类, 用来提供 SqlSession 。调用getSqlSession()方法会得到一个 SqlSessionTemplate,之后可以用于执行SQL方法

#### DAO的实现
**使用SqlSessionDaoSupport的方法，需要编写DAO的实现类**

```
/**
 * 
 * SqlSessionDaoSupport 是 一 个 抽象 的支 持 类, 提供 SqlSession.
 * 调 用 getSqlSession()方法你会得到一个 SqlSessionTemplate,之后可以用于执行 SQL 方法
 * 
 * 使用SqlSessionDaoSupport的方法，需要编写DAO的实现类
 * @author SongZhangLiang
 */
public class UserDaoImpl extends SqlSessionDaoSupport implements UserDao{

	@Override
	public int addUser(User user) {
		return this.getSqlSession().insert("com.szl.springorm.dao.UserDao.addUser", user);
	}

	@Override
	public List<Map<String, Object>> queryUser() {
		return this.getSqlSession().selectList("com.szl.springorm.dao.UserDao.queryUser");
	}

	@Override
	public int delUserById(Long id) {
		return this.getSqlSession().delete("com.szl.springorm.dao.UserDao.delUserById",id);
	}

}

```

#### Spring XML配置

```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <!-- 指定数据源 -->
  <property name="dataSource" ref="dataSource"></property>
  <!-- 指定mybatis配置文件 -->
<property name="configLocation" value="classpath:configuration.xml"/> 
<property name="mapperLocations" value="classpath*:com/szl/springorm/dao/impl/*.xml"></property>
</bean>

<!-- 2.使用sqlSessionDaoSupport,注入sqlSessionFactory-->
<bean id="userDaoImpl" class="com.szl.springorm.dao.impl.UserDaoImpl">
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>

```

#### 测试一下

```
/**
 * 使用SqlSessionDaoSupport进行数据库操作
 * DAO的实现类必须要继承SqlSessionDaoSupport
 * @author SongZhangLiang
 */
@Test
public void testSqlSessionDaoSupport(){
	UserDaoImpl userDaoImpl = ac.getBean("userDaoImpl",UserDaoImpl.class);
	List<Map<String, Object>> queryUser = userDaoImpl.queryUser();
	System.out.println("testSqlSessionDaoSupport:"+queryUser.toString());
}

```

### 使用MapperFactoryBean动态代理

为了代替手工使用 SqlSessionDaoSupport 或 SqlSessionTemplate 编写数据访问对象 (DAO)的代码，MyBatis-Spring 提供了一个动态代理的实现:MapperFactoryBean

#### 编写DAO

```
public interface UserDao1 {
/**
 * 增加用户
 * @return
 */
int addUser(User user);

/**
 * 查询用户
 * @return
 */
List<Map<String, Object>> queryUser();

/**
 * 删除用户
 * @return
 */
int delUserById(Long id);

}

```

#### 编写Service实现类

```
public class UserServiceImpl implements UserService {
	

	private UserDao1 userDao1;
	
    //增加set方法,方便注入
	public void setUserDao1(UserDao1 userDao1) {
		this.userDao1 = userDao1;
	}

	@Override
	public int addUser(User user) {
		return userDao1.addUser(user);
	}

	@Override
	public List<Map<String, Object>> queryUser() {
		return userDao1.queryUser();
	}

	@Override
	public int delUserById(Long id) {
		return userDao1.delUserById(id);
	}
	

```

#### Spring XML配置

```
<!-- 1.注册MapperFactoryBean -->	
<bean id="userDao1" class="org.mybatis.spring.mapper.MapperFactoryBean">
  <!-- 注册映射器,也就是DAO接口 -->
  <property name="mapperInterface" value="com.szl.springorm.dao.UserDao1"></property>
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
<!-- 2.service中直接注入userDao --> 	 	
<bean id="userServiceImpl" class="com.szl.springorm.service.impl.UserServiceImpl">
  <property name="userDao1" ref="userDao1"></property>
</bean>

```

#### 测试一下

```
/**
 * 使用MapperFactoryBean代理
 * 注意在这段代码中没有 SqlSession 或 MyBatis 的引用。也没有任何需要创建,打开或 关闭 session 的代码,MyBatis-Spring 会来关心它的。
 * @author SongZhangLiang
 */
@Test
public void testMapperFactoryBean(){
	UserServiceImpl userDaoImpl = ac.getBean("userServiceImpl",UserServiceImpl.class);
	List<Map<String, Object>> queryUser = userDaoImpl.queryUser();
	System.out.println("testMapperFactoryBean:"+queryUser.toString());
}

```

#### 总结

如果有大量的映射文件，那么就需要在XML中配置大量的MapperFactoryBean，这样XML会非常的臃肿，为此MyBatis-Spring提供了MapperScannerConfigurer， 它将会查找类路径下的映射器并自动将它们创建成MapperFactoryBean。

配置如下：
> basePackage 属性是让你为映射器接口文件设置基本的包路径。 你可以使用分号或逗号 作为分隔符设置多于一个的包路径。每个映射器将会在指定的包路径中递归地被搜索到。
MapperScannerConfigurer 属性不支持使用了 PropertyPlaceholderConfigurer 的属 性替换,因为会在 Spring 其中之前来它加载。但是,你可以使用 PropertiesFactoryBean 和 SpEL 表达式来作为替代。

```
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <!--basePackage属性是为映射器接口文件设置基本的包路径,也就是DAO的包-->
  <property name="basePackage" value="com.szl.springorm.dao"></property>
  <!-- 一个数据源，可以不配置sqlSessionFactory -->
  <!-- <property name="sqlSessionFactory" ref="sqlSessionFactory" /> -->
</bean>	

```
