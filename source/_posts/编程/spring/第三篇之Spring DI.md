---
title: 第三篇之Spring DI
date: 2017-05-27 11:49
categories: Spring的那点事
tags: Spring
---

### Spring IoC容器注入依赖资源主要有以下两种基本实现方式
- 构造器注入
- setter注入

我们在介绍注入之前，先建立一个User实体类，且生成无参、有参构造及setter，重写`toString()`方法以便输出信息。

```
package com.szl.springioc.model;

public class User {
	
	private String name;
	private int age;
	private String email;

	/**
	 * 无参构造(当我们生成有参构造时，默认的无参构造要自行添加)
	 */
	public User() {
		super();
	}
	
	/**
	 * 有参构造
	 * @param name
	 * @param age
	 * @param email
	 */
	public User(String name, int age, String email) {
		super();
		this.name = name;
		this.age = age;
		this.email = email;
	}


	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}

	
	@Override
	public String toString() {
		return "User [name=" + name + ", age=" + age + ", email=" + email + "]";
	}
}

```

#### 构造器注入
##### 无参构造
- xml配置

```
<bean id="user" class="com.szl.springioc.model.User" />

```
- test

```
@Test
public void testDI() {
	User user = ac.getBean("user",User.class);
	System.out.println(user.toString());
}

```
- 输出

```
User [name=null, age=0, email=null]

```
- 结论
xml文件中的配置相当于我们代码中写`User user = new User();`。

- 注意
当我们在类中自己生成有参构造时，默认的无参构造要自行添加。

##### 有参构造
- xml配置

```
<!-- 有参构造 -->
<bean id="user1" class="com.szl.springioc.model.User">
	<constructor-arg value="张三"></constructor-arg>
	<constructor-arg value="18"></constructor-arg>
	<constructor-arg value="zhangsan@qq.com"></constructor-arg>
</bean>

```
- test

```

@Test
public void testDI() {
	//测试有参构造注入
	User user1 = ac.getBean("user1",User.class);
	System.out.println(user1.toString());

}

```

- 输出

```
User [name=张三, age=18, email=zhangsan@qq.com]
```
- 结论
xml文件中的配置相当于我们代码中写`User user = new User("张三","18","zhangsan@qq.com");`。

- 注意
参数的类型和数量必须一致，`constructor-arg`标签属性中可以使用index指定参数的位置、使用type指定参数的类型及使用name指定参数的名称。

#### setter注入
- xml配置

```
<!-- setter注入 -->
<bean id="user2" class="com.szl.springioc.model.User">
	<property name="name" value="李四"></property>
</bean>
```
- test

```
@Test
public void testDI() {
	//测试setter注入
	User user2 = ac.getBean("user2",User.class);
	System.out.println(user2.toString());

}

```
- 输出

```
User [name=李四, age=0, email=null]

```
- 结论

xml文件中的配置相当于我们代码中写`User user = new User(); user.setName("李四")`。
- 注意

setter注入的方法名要遵循“JavaBean getter/setter 方法命名约定”，`<property>`标签中name表示类中setter的名字，value表示要注入的参数值。

### 其他注入测试

我们建下面的一个实体类，我们来测试setter注入boolean，List,Map和Set类型。
```
package com.szl.springioc.model;

import java.util.List;
import java.util.Map;
import java.util.Set;

public class Collections {
	
	private boolean flag;
	
	private Map<String, Object> mapParams;
	
	private List<String> listParmas;
	
	private Set<String> setParams;

	public boolean isFlag() {
		return flag;
	}

	public void setFlag(boolean flag) {
		this.flag = flag;
	}

	public Map<String, Object> getMapParams() {
		return mapParams;
	}

	public void setMapParams(Map<String, Object> mapParams) {
		this.mapParams = mapParams;
	}

	public List<String> getListParmas() {
		return listParmas;
	}

	public void setListParmas(List<String> listParmas) {
		this.listParmas = listParmas;
	}

	public Set<String> getSetParams() {
		return setParams;
	}

	public void setSetParams(Set<String> setParams) {
		this.setParams = setParams;
	}
	
	@Override
	public String toString() {
		return "Collections [flag=" + flag + ", mapParams=" + mapParams + ", listParmas=" + listParmas + ", setParams="
				+ setParams + "]";
	}

}


```
#### 注入Boolean类型
- xml配置

```
<!-- setter注入 boolean类型 -->
<bean id="collectionsBoolean" class="com.szl.springioc.model.Collections">
	<property name="flag" value="true"></property>
</bean>

```
- test

```
@Test
public void testDIForCollections() {
	//测试setter注入
	Collections collectionsBoolean = ac.getBean("collectionsBoolean",Collections.class);
	System.out.println(collectionsBoolean.toString());
	
}

```
- 输出

```
Collections [flag=true, mapParams=null, listParmas=null, setParams=null]

```

- 结论

setter注入Boolean类型，`<property>`标签中的value值可以是以下几种来代表“真假”

NO.|支持的参数值
---|---
1|true/false
2|1/0
3|on/off
4|yes/no

#### 注List类型
- xml文件配置

```
<!-- setter注入 List类型 -->
<bean id="collectionsList" class="com.szl.springioc.model.Collections">
<property name="listParmas">
	<list>
		<value>test</value>
		<value>test1</value>
	</list>
</property>
</bean>

```
- test

```
@Test
public void testDIForCollections() {
    //测试List注入
    Collections collectionsList = ac.getBean("collectionsList",Collections.class);
    System.out.println(collectionsList.toString());

}

```
- 输出

```
Collections [flag=false, mapParams=null, listParmas=[test, test1], setParams=null]
```
- 结论
注入List需要使用`<list>`标签，该标签中有可选的value-type属性，用来指定数据类型，比如`value-type=java.lang.String`。


#### 注Map类型
- xml文件配置

```
<!-- setter注入 Map类型 -->
<bean id="collectionsMap" class="com.szl.springioc.model.Collections">
	<property name="mapParams">
		<map>
			<entry key="key1" value="你好"></entry>
			<entry key="key2" value="世界"></entry>
		</map>
	</property>
</bean>

```
- test

```
@Test
public void testDIForCollections() {
	//测试Map注入
	Collections collectionsMap = ac.getBean("collectionsMap",Collections.class);
	System.out.println(collectionsMap.toString());
	
}

```
- 输出

```
Collections [flag=false, mapParams={key1=你好, key2=世界}, listParmas=null, setParams=null]
```
- 结论

注入List需要使用`<map>`标签，该标签中有可选的key-type、value-type属性，用来指定数据类型。

#### 注入Set类型

注入Set需要使用`<set>`标签，同注入List一样，该标签中有可选的value-type属性，用来指定数据类型。

### p命名空间简化setter注入

- xml文件配置

```
<!-- p命名空间 -->
<bean id="collectionsP" class="com.szl.springioc.model.Collections" 
	p:flag="1"
/>

```

- test

```
@Test
public void testDIForCollections() {
	Collections collectionsP = ac.getBean("collectionsP",Collections.class);
	System.out.println(collectionsP.toString());
	
}
```

- 输出

```
Collections [flag=true, mapParams=null, listParmas=null, setParams=null]
```
- 结论

`p:flag="1"`等价于`<property name="flag" value="true"></property>`，可以混合使用哦。











