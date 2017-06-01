---
title: 第五篇之Spring JDBC
date: 2017-05-31 14:28
categories: Spring的那点事
tags: Spring
---

## JDBC是什么?

### 概念
> JDBC（Java Data Base Connectivity,java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。JDBC提供了一种基准，据此可以构建更高级的工具和接口，使数据库开发人员能够编写数据库应用程序。-摘自百度百科

### 传统JDBC编程

- 代码示列

```
package com.szl.springjdbc.util;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;


public class JdbcUtil {
	
	private static String driver = "com.mysql.jdbc.Driver";
	private static String url = "jdbc:mysql://localhost:3306/test";
	private static String user = "root";
	private static String password = "123456";
	
	private Connection connection;
	private PreparedStatement prepareStatement;
	private ResultSet executeQuery;
	
	public void JdbcTest(){
		try {
			//加载驱动
			Class.forName(driver);
			//获取连接且开启事务
			connection = DriverManager.getConnection(url, user, password);
			connection.setAutoCommit(false);
			//预编译sql
			prepareStatement = connection.prepareStatement("select * from t_users");
			//执行sql
			ResultSet executeQuery = prepareStatement.executeQuery();
			int colNum = executeQuery.getMetaData().getColumnCount();
			//处理结果集
			while(executeQuery.next()){
				for(int i=0; i<colNum; i++){
					System.out.println(executeQuery.getString(i+1));
				}
				System.out.println("-----------------------------");
			}
			//提交事务
			connection.commit();
		} catch (ClassNotFoundException e) {
			try {
				//异常事务回滚
				connection.rollback();
			} catch (SQLException e1) {
				e1.printStackTrace();
			}
			e.printStackTrace();
		} catch (SQLException e) {
			try {
				connection.rollback();
			} catch (SQLException e1) {
				e1.printStackTrace();
			}
			e.printStackTrace();
		}
		finally {
			//关闭连接
			try {
				if(null != executeQuery)
					executeQuery.close();
				if(null != prepareStatement)
					prepareStatement.close();
				if(null != connection)
					connection.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
		
		
	}

}

```

- 总结
通过以上代码，我们可以看到，传统JDBC编程需要冗余、复杂的操作，为此Spring JDBC提供了一套JDBC抽象框架，用于简化JDBC开发。

## Spring JDBC
### 概念
Spring通过抽象JDBC访问并提供一致的API来简化JDBC编程的工作量。我们只需要声明SQL、调用合适的Spring JDBC框架API、处理结果集即可。事务由Spring管理，并将JDBC受查异常转换为Spring一致的非受查异常，从而简化开发。

### 框架
Spring主要提供JDBC模板方式、关系数据库对象化方式、SimpleJdbc方式简化JDBC编程。
本文以JDBC模板方式进行介绍

### JDBC模板
- JdbcTemplate
Spring里最基本的JDBC模板，利用JDBC和简单的索引参数查询提供对数据库的简单访问。

- NamedParameterJdbcTemplate
能够在执行查询时把值绑定到SQL里的命名参数，而不是使用索引参数。

- SimpleJdbcTemplate
利用Java 5的特性，比如自动装箱、通用（generic）和可变参数列表来简化JDBC模板的使用。

#### JdbcTemplate介绍

##### JdbcTemplate主要4类方法
- execute
用于执行任何SQL语句，一般用于执行DDL语句。

- update/batchUpdate
update方法用于执行新增、修改、删除等语句；batchUpdate方法用于执行批处理相关语句。

- query/queryForXXX
用于执行查询相关语句。

- call
用于执行存储过程、函数相关语句。

##### JdbcTemplate类支持的回调类
###### 预编译语句及存储过程创建回调
用于根据JdbcTemplate提供的连接创建相应的语句

- PreparedStatementCreator
通过回调获取JdbcTemplate提供的Connection，由用户使用该Conncetion创建相关的PreparedStatement

- CallableStatementCreator
通过回调获取JdbcTemplate提供的Connection，由用户使用该Conncetion创建相关的CallableStatement

###### 预编译语句设值回调
用于给预编译语句相应参数设值

- PreparedStatementSetter
通过回调获取JdbcTemplate提供的PreparedStatement，由用户来对相应的预编译语句相应参数设值

- BatchPreparedStatementSetter
类似于PreparedStatementSetter，但用于批处理，需要指定批处理大小

###### 自定义功能回调
提供给用户一个扩展点，用户可以在指定类型的扩展点执行任何数量需要的操作

- ConnectionCallback
通过回调获取JdbcTemplate提供的Connection，用户可在该Connection执行任何数量的操作

- StatementCallback
通过回调获取JdbcTemplate提供的Statement，用户可以在该Statement执行任何数量的操作

- PreparedStatementCallback
通过回调获取JdbcTemplate提供的PreparedStatement，用户可以在该PreparedStatement执行任何数量的操作

- CallableStatementCallback
通过回调获取JdbcTemplate提供的CallableStatement，用户可以在该CallableStatement执行任何数量的操作

###### 结果集处理回调
通过回调处理ResultSet或将ResultSet转换为需要的形式

- RowMapper
用于将结果集每行数据转换为需要的类型，用户需实现方法mapRow(ResultSet rs, int rowNum)来完成将每行数据转换为相应的类型

- RowCallbackHandler
用于处理ResultSet的每一行结果，用户需实现方法processRow(ResultSet rs)来完成处理，在该回调方法中无需执行rs.next()，该操作由JdbcTemplate来执行，用户只需按行获取数据然后处理即可

- ResultSetExtractor
用于结果集数据提取，用户需实现方法extractData(ResultSet rs)来处理结果集，用户必须处理整个结果集


#### JdbcTemplate测试

使用JdbcTemplate模板类时必须通过DataSource获取数据库连接，然后在使用JdbcTemplate模板对数据库进行操作。

##### 未使用回调

JdbcTemplate提供更简单的queryForXXX方法，来简化开发，参考一下代码。

```
/**
 * 测试spring JdbcTemplate
 */
@Test
public void testSpringJdbc() throws IOException {
	//读取配置文件的数据源配置
	Properties properties = new Properties();
	InputStream in = Thread.currentThread().getContextClassLoader().getResourceAsStream("database.properties");
	properties.load(in);
	String url = properties.getProperty("jdbc.url");
	String username = properties.getProperty("jdbc.username");
	String password = properties.getProperty("jdbc.password");
	String driver = properties.getProperty("jdbc.driver");
	//使用alibaba的DruidDataSource
	DruidDataSource dataSource = new DruidDataSource();
	dataSource.setPassword(password);
	dataSource.setUrl(url);
	dataSource.setUsername(username);
	dataSource.setDriverClassName(driver);
	dataSource.setInitialSize(2);
	dataSource.setMinIdle(2);
	dataSource.setMaxActive(10);
	dataSource.setMaxWait(60000);
	dataSource.setTimeBetweenEvictionRunsMillis(5000);
	dataSource.setMinEvictableIdleTimeMillis(120000);
	//new一个jdbctemplate
	JdbcTemplate jt = new JdbcTemplate(dataSource);
	//测试查询，直接返回List<Map<String,Object>>结果
	List<Map<String, Object>> queryForList = jt.queryForList("select * from t_users");
	System.out.println("查询结果："+queryForList.toString());
	//测试增加
//	int update = jt.update("INSERT INTO t_users(name,email)VALUES('王五','wangwu@163.com')");
//	System.out.println("增加结果："+update);
}

```
##### 结果集回调

```
/**
* 测试spring JdbcTemplate 结果集回调
*/
@Test
public void testSpringJdbcResult() throws IOException {
    //读取配置文件的数据源配置
    Properties properties = new Properties();
    InputStream in = Thread.currentThread().getContextClassLoader().getResourceAsStream("database.properties");
    properties.load(in);
    String url = properties.getProperty("jdbc.url");
    String username = properties.getProperty("jdbc.username");
    String password = properties.getProperty("jdbc.password");
    String driver = properties.getProperty("jdbc.driver");
    //使用alibaba的DruidDataSource
    DruidDataSource dataSource = new DruidDataSource();
    dataSource.setPassword(password);
    dataSource.setUrl(url);
    dataSource.setUsername(username);
    dataSource.setDriverClassName(driver);
    dataSource.setInitialSize(2);
    dataSource.setMinIdle(2);
    dataSource.setMaxActive(10);
    dataSource.setMaxWait(60000);
    dataSource.setTimeBetweenEvictionRunsMillis(5000);
    dataSource.setMinEvictableIdleTimeMillis(120000);
    //new一个jdbctemplate
    JdbcTemplate jt = new JdbcTemplate(dataSource);
    //测试RowMapper结果集回调
//		List<Map<String, Object>> query = jt.query("select * from t_users", new RowMapper<Map<String, Object>>() {
//			@Override
//			public Map<String, Object> mapRow(ResultSet rs, int rowNum) throws SQLException {
//				Map<String, Object> result = new HashMap<>();
//				result.put("id", rs.getInt("id"));
//				result.put("name", rs.getString("name"));
//				result.put("email", rs.getString("email"));
//				return result;
//			}
//		});
//		System.out.println("RowMapper结果："+query.toString());

    //测试RowCallbackHandler结果集回调
//		List<Map<String, Object>> query = new ArrayList<>();
//		jt.query("select * from t_users",new RowCallbackHandler() {
//			@Override
//			public void processRow(ResultSet rs) throws SQLException {
//				Map<String, Object> result = new HashMap<>();
//				result.put("id", rs.getInt("id"));
//				result.put("name", rs.getString("name"));
//				result.put("email", rs.getString("email"));
//				query.add(result);
//			}
//		});
//		System.out.println("RowCallbackHandler结果："+query.toString());

//测试ResultSetExtractor结果集回调,必须手动处理整个结果集
    List<Map<String, Object>> query = jt.query("select * from t_users", new ResultSetExtractor<List<Map<String, Object>>>() {
    	@Override
    	public List<Map<String, Object>> extractData(ResultSet rs) throws SQLException, DataAccessException {
    		List<Map<String, Object>> resultList = new ArrayList<>();
    		while(rs.next()){
    			Map<String, Object> result = new HashMap<>();
    			result.put("id", rs.getInt("id"));
    			result.put("name", rs.getString("name"));
    			result.put("email", rs.getString("email"));
    			resultList.add(result);
    		}
    		return resultList;
    	}
    });
    System.out.println("ResultSetExtractor结果："+query.toString());

}

```

##### 其他回调

请参考其他文档，使用方法大同小异。

#### JdbcTemplate实践  
Spring JDBC都是与IOC容器一起使用，通过配置方式使用Spring JDBC，一般都是使用JdbcTemplate类，Spring JDBC通过实现DaoSupport来支持一致的数据库访问。

##### Spring JDBC的DaoSupport实现

- JdbcDaoSupport：用于支持一致的JdbcTemplate访问
- NamedParameterJdbcDaoSupport:继承JdbcDaoSupport，同时提供NamedParameterJdbcTemplate访问
- SimpleJdbcDaoSupport：继承JdbcDaoSupport，同时提供SimpleJdbcTemplate访问

##### 定义表结构

```
DROP TABLE IF EXISTS `t_users`;
CREATE TABLE `t_users` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL,
  `email` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of t_users
-- ----------------------------
INSERT INTO `t_users` VALUES ('1', 'Andy', 'song@foxmail.com');

```

##### 定义DAO

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

##### 定义DAO的实现类

**ps:实现类继承JdbcDaoSupport**

```
public class UserDaoImpl extends JdbcDaoSupport implements UserDao{
	
	@Override
	public int addUser(User user) {
		
		return getJdbcTemplate().update("INSERT INTO t_users(name,email)VALUES(?,?)", user.getName(),user.getEmail());
	}

	@Override
	public List<Map<String, Object>> queryUser() {
		return getJdbcTemplate().queryForList("SELECT * FROM t_users");
	}

	@Override
	public int delUserById(Long id) {
		return getJdbcTemplate().update("delete from t_users where id=?",id);
	}

}

```

##### 定义service的实现类

```
public class UserServiceImpl implements UserService {

	private UserDao userDao;	
	
	@Override
	public int addUser(User user) {
		return userDao.addUser(user);
	}

	@Override
	public List<Map<String, Object>> queryUser() {
		return userDao.queryUser();
	}

	@Override
	public int delUserById(Long id) {
		return userDao. delUserById(id);
	}
	
	public UserDao getUserDao() {
		return userDao;
	}

	public void setUserDao(UserDao userDao) {
		this.userDao = userDao;
	}
}


```
##### xml文件配置

- 数据源
```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1/test?allowMultiQueries=true&zeroDateTimeBehavior=convertToNull&noAccessToProcedureBodies=true&characterEncoding=UTF-8
jdbc.username=root
jdbc.password=123456

```

- bean的配置
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
        <!-- 加载配置文件 -->
        <context:property-placeholder location="database.properties"/>
        
        <!-- 注入jdbcutil类，为了测试原生JDBC -->
        <bean id="jdbcTest" class="com.szl.springjdbc.util.JdbcUtil"></bean>
        
        <!-- 配置数据源 -->
        <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource"  
	    destroy-method="close">  
		    <property name="driverClassName" value="${jdbc.driver}" />  
		    <property name="url" value="${jdbc.url}" />  
		    <property name="username" value="${jdbc.username}" />  
		    <property name="password" value="${jdbc.password}" />  
		</bean>
 	 		 
        
        <!-- 定义抽象的abstractDao，其有一个dataSource属性，从而可以让继承的子类自动继承dataSource属性注入-->
        <bean id="abstractDao" abstract="true">
        	<property name="dataSource" ref="dataSource"></property>
        </bean>
        
        <!-- 注入UserDaoImpl类，为了测试使用jdbcTemplate, 继承abstractDao，从而继承dataSource注入-->
        <bean id="userDaoImpl" class="com.szl.springjdbc.dao.impl.UserDaoImpl" parent="abstractDao"></bean>
        
        <!-- 注入UserServiceImpl类，为了测试使用jdbcTemplate  -->
        <bean id="userServiceImpl" class="com.szl.springjdbc.service.impl.UserServiceImpl">
        	<property name="userDao" ref="userDaoImpl"></property>
        </bean>
</beans>

```

##### 测试类

```
public class JdbcTemplateTest {
	
	private ApplicationContext ac;
	
	@Before
	public void inti() {
		ac = new ClassPathXmlApplicationContext("jdbc_test.xml");
	}
	
	/**
	 * 测试jdbcTemplate编程_查询
	 * @author SongZhangLiang
	 */
	@Test
	public void testJdbcTemplateQuery() {
		UserServiceImpl userServiceImpl = ac.getBean("userServiceImpl",UserServiceImpl.class);
		List<Map<String, Object>> queryUser = userServiceImpl.queryUser();
		System.out.println("查询数据："+queryUser.toString());
	}
	/**
	 * 测试jdbcTemplate编程_添加
	 * @author SongZhangLiang
	 */
	@Test
	public void testJdbcTemplateAdd() {
		UserServiceImpl userServiceImpl = ac.getBean("userServiceImpl",UserServiceImpl.class);
		User user = new User();
		user.setName("李四");
		user.setEmail("lisi@foxmail.com");
		System.out.println("result:"+userServiceImpl.addUser(user));
	}
	
	/**
	 * 测试jdbcTemplate编程_删除
	 * @author SongZhangLiang
	 */
	@Test
	public void testJdbcTemplateDel() {
		UserServiceImpl userServiceImpl = ac.getBean("userServiceImpl",UserServiceImpl.class);
		System.out.println("result:"+userServiceImpl.delUserById(1L));
	}

}

```

[更多信息，请前往官网介绍](http://docs.spring.io/spring/docs/3.0.x/spring-framework-reference/html/jdbc.html)
