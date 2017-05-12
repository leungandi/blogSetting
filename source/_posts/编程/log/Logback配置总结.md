---
title: Logback配置总结
date: 2017-05-11 15:22
categroies: log
---


### 什么是Logback?  
Logback是由log4j创始人设计的又一个开源日志组件,[官方网站](http://logback.qos.ch)。

### Logback配置
1. 根节点configuration  
- 主要包含以下三个子节点
    - appender
    - logger
    - root
    ```
    graph TD
    A[configuration]
    A-->B[appender]
    A-->C[logger]
    A-->D[root]
    
    ```

- 根节点的属性  
  - scan:默认为true,当配置文件改变时,是否自动加载;
  - scanPreiod:默认为1分钟(60 seconds),设置监测配置文件是否有修改的时间间隔,当scan为true时,此属性才会生效;
  - debug:默认为false,设置为true时,会打印Logback的内部日志,查看Logback的运行状态;

    ```
    <configuration scan="true" scanPeriod="1800 seconds" debug="false">
        <!-- 省略中间的配置 -->
    </configuration>

    ```
2. 全局设置  
    2.1. 设置变量
    - 用来自定义用户的变量,使用<proerty>来进行定义,property有2个属性值,我们可以使用"${USER_HOME}"来获取变量的值.
        - name:变量的名称  
        - value:变量的值  

    ```
    <configuration scan="true" scanPeriod="1800 seconds" debug="false">
        <property name="USER_HOME" value="/opt/logs" />
        <property scope="context" name="FILE_NAME" value="test_log" />
    </configuration>
    
    ```
    2.2. 获取时间戳字符串
    - 获取时间戳字符串,使用<timestamp>来进行定义,timestamp也有2个属性值,获取值得方法同样为"${}"
        - key:名称
        - datePattern:时间解析为字符串的格式
    ```
    <configuration scan="true" scanPeriod="1800 seconds" debug="false">
        <property name="USER_HOME" value="/opt/logs" />
        <property scope="context" name="FILE_NAME" value="test_log" />
        <timestamp key="byDay" datePattern="yyyy-MM-dd" />
    </configuration>
    
    ```
3. 子节点appender  
    - appender是负责写日志的组件,有2个属性值：
        - name:appender的名称
        - class:指定appender的全限定名
    
    ```
    <!-- demo 1-->
    <configuration scan="true" scanPeriod="1800 seconds" debug="false">
        <property name="USER_HOME" value="/opt/logs" />
        <property scope="context" name="FILE_NAME" value="test_log" />
        <timestamp key="byDay" datePattern="yyyy-MM-dd" />
        
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        	<encoder>
    		    <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        	</encoder>
        </appender>
    </configuration>
        
    ```
    - appender的clas常用的有三种
    
    ```
    graph TD
    A[appender class]
    A-->B[ch.qos.logback.core.ConsoleAppender]
    A-->C[ch.qos.logback.core.rolling.RollingFileAppender]
    A-->D[ch.qos.logback.core.FileAppender]
    
    ```
3.1. ConsoleAppender:  
    日志输出到控制台,配置参考[demo 1]主要有2个子节点：  
    <encoder>:对日志进行格式化;  
    <target>:字符串System.out或者 System.err,默认 System.out;  
    
3.2. RollingFileAppender:  
    滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件,主要有4个节点:  
    <file>:写入的文件名,相对/绝对路径都可以,如果目录不存在,Logback会创建;  
    <append>:日志是否追加到文件结尾,可设置的参数为true或者false  
    <encoder>:对日志进行格式化;    
    <rollingPolicy>:滚动策略,有TimeBasedRollingPolicy（按时间制定策略,常用这种）和FixedWindowRollingPolicy（按固定窗口算法）
    <triggeringPolicy >: 触发器策略。
    <prudent>：当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，1不支持也不允许文件压缩，2不能设置file属性，必须留空。  
    
    
```
graph TD
A[rollingPolicy]
A-->B[ch.qos.logback.core.rolling.FixedWindowRollingPolicy]
A-->C[ch.qos.logback.core.rolling.TimeBasedRollingPolicy]

```  
    
```
    
<!-- demo 2-->
<configuration scan="true" scanPeriod="1800 seconds" debug="false">
    <property name="USER_HOME" value="/opt/logs" />
    <property scope="context" name="FILE_NAME" value="test_log" />
    <timestamp key="byDay" datePattern="yyyy-MM-dd" />
    
    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${USER_HOME}/${FILE_NAME}.log</>
        <!-- 按日志大小切分日志文件 -->
    	<!-- 
		<rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
			<fileNamePattern>${USER_HOME}/${byDay}/${FILE_NAME}-${byDay}-%i.log.zip</fileNamePattern>
			<minIndex>1</minIndex>
			<maxIndex>10</maxIndex>
		</rollingPolicy>
        <!-- 
        当SizeBasedTriggeringPolicy触发时（即文件大小达到5MB），则启 动FixedWindowsRollingPolicy对日志文件进行滚动。MinIndex和MaxIndex分别表示最小计数和最大计数。 MaxFileSize则表示日志文件达到多少的时候进行滚动
        -->
		<triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
			<maxFileSize>5MB</maxFileSize>
		</triggeringPolicy>
 		-->
 		<!-- 按照日期切分日志文件，每天生成一个日志文件 -->
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${USER_HOME}/${FILE_NAME}-%d{yyyy-MM-dd}.log.tar.gz</fileNamePattern>  
			<!-- 保存?天的日志文件，默认永久保留 -->
			<!-- <maxHistory>30</maxHistory>  -->
		</rollingPolicy>
    	<encoder>
		    <pattern>%-4relative %d - [%thread] %-5level %logger{35} - %msg%n </pattern>
    	</encoder>
    </appender>
</configuration>

```
        
3.3. FileAppender:  
    日志添加到文件  
    <file>:写入的文件名,相对/绝对路径都可以,如果目录不存在,Logback会创建;  
    <append>:日志是否追加到文件结尾,可设置的参数为true或者false  
    <encoder>:对日志进行格式化;    
    <prudent>:是否安全写入文件,默认false,设置为ture会影响效率;   
    
```
<!-- demo 3-->
<configuration>  
    <property name="USER_HOME" value="/opt/logs" />
    <property scope="context" name="FILE_NAME" value="test_log" />
    <timestamp key="byDay" datePattern="yyyy-MM-dd" />
    
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">  
        <file>${USER_HOME}/${FILE_NAME}.log</>
        <append>true</append>  
        <encoder>  
          <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>  
        </encoder>  
    </appender>  
</configuration> 

```
4. 子节点logger
    - 用来设置某一个包或者具体的某一个类的日志打印级别、以及指定<appender>,<loger>仅有一个name属性,一个可选的level和一个可选的addtivity属性
        - name:用来指定受此loger约束的某一个包或者具体的某一个类;
        - level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF,还有一个特俗值INHERITED或者同义词NULL,代表强制执行上级的级别;
        - addtivity:是否向上级loger传递打印信息,默认是true;
    - <loger>可以包含零个或多个<appender-ref>元素,标识这个appender将会添加到这个loger;
```
<configuration>  
    <property name="USER_HOME" value="/opt/logs" />
    <property scope="context" name="FILE_NAME" value="test_log" />
    <timestamp key="byDay" datePattern="yyyy-MM-dd" />
    
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    	<encoder>
		    <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    	</encoder>
    </appender>
    
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">  
        <file>${USER_HOME}/${FILE_NAME}.log</>
        <append>true</append>  
        <encoder>  
          <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>  
        </encoder>  
    </appender> 
    
    <logger name="com.diandian" level="debug" additivity="false">
		<appender-ref ref="FILE" />
		<appender-ref ref="STDOUT" />
	</logger>
</configuration> 

```

5. 子节点root
    - root其实是一个根logger,name的名称已经被命名为"root",level属性的值同logger;

6. <encoder>  
    - 负责两件事 
        - 一是把日志信息转换成字节数组;
        - 二是把字节数组写入到输出流;
    - 它有一个<pattern>子节点,用来设置日志的输出格式,使用“%”加“转换符”方式，如果要输出“%”，则必须用“\”对“\%”进行转义
```
<encoder>  
  <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>  
</encoder> 

```
    - <pattern>格式介绍:
    
转换符| 作用
---|---
c {length }  lo {length }  logger {length }  |  输出日志的logger名
d {pattern } date {pattern }     |  输出日志的打印日志,模式语法与java.text.SimpleDateFormat 兼容   
m msg message    |  输出应用程序提供的信息 
n | 输出平台相关的分行符“\n”或者“\r\n”
p  le  level | 输出日志级别
r relative | 输出从程序启动到创建日志记录的时间,单位是毫秒
t thread | 输出线程名
replace(p ){r, t} | p 为日志内容,r 是正则表达式,将p 中符合r 的内容替换为t,例如, "%replace(%msg){'\s', ''}"

格式修饰符 | 作用
--- | ---
-(减号) | 左对齐
4(十进制数表示) | 最小宽度,如果字符小于最小宽度,则左填充或右填充，默认是左填充（即右对齐）,填充符为空格,如果字符大于最小宽度,字符永远不会被截断 
.4(点加十进制数表示) | 最大宽度,如果字符大于最大宽度,则从前面截断;点符号“.”后面加减号“-”在加数字,表示从尾部截断 

**重要**:格式修饰符位于“%”和转换符之间

```
<!--完整DEMO-->
<?xml version="1.0" encoding="UTF-8" ?>
<configuration scan="true" scanPeriod="1800 seconds"
	debug="false">

	<property name="USER_HOME" value="/opt/logs" />
	<property scope="context" name="FILE_NAME" value="test_log" />

	<timestamp key="byDay" datePattern="yyyy-MM-dd" />

	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
		</encoder>
	</appender>

	<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${USER_HOME}/${FILE_NAME}.log</file>
 		<!-- 按照日期切分日志文件，每天生成一个日志文件 -->
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${USER_HOME}/${FILE_NAME}-%d{yyyy-MM-dd}.log.tar.gz</fileNamePattern>  
		</rollingPolicy>
		<!-- 日志输出格式 -->
		<encoder>
			<pattern>%-4relative %d - [%thread] %-5level %logger{35} - %msg%n </pattern>
		</encoder>
	</appender>
	<logger name="com.demo" level="debug" additivity="false">
		<appender-ref ref="FILE" />
		<appender-ref ref="STDOUT" />
	</logger>
	<root level="INFO">
		<appender-ref ref="FILE" />
		<appender-ref ref="STDOUT" />
	</root>
</configuration>


```



