---
title: RabbitMQ安装及简单使用
date: 2018-03-13 13:50:14
tags:
categories: 编程
---

## 安装RabbitMQ

**环境说明:**

1.操作系统:centos 7

2.RabbitMq:3.7.3

3.erlang:v20.2.3

4.centos已经关闭防火墙

```
<!--查看防火墙状态-->
systemctl status firewalld.service

<!--关闭防火墙-->
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动

```


### 下载erlang
[erlang rpm](https://github.com/rabbitmq/erlang-rpm/releases)

### 下载rabbitmq
[rabbitmq官网下载](http://www.rabbitmq.com/install-rpm.html)

### 开始安装
```
[root@localhost soft]# cd rabbitmq/
[root@localhost rabbitmq]# ll
总用量 29752
-rw-r--r--. 1 root root 18466680 2月  27 06:16 erlang-20.2.3-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root 11996604 2月  27 01:11 rabbitmq-server-3.7.3-1.el7.noarch.rpm
[root@localhost rabbitmq]# 

```
使用```yum install```分别安装erlang和rabbitmq-server


### 完成安装

#### erl

输入```erl```,出现以下界面,说明erlang安装成功

```
[root@localhost rabbitmq]# erl
Erlang/OTP 20 [erts-9.2.1] [source] [64-bit] [smp:1:1] [ds:1:1:10] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V9.2.1  (abort with ^G)
1> 

```

#### 启动rabbitmq

```
[root@localhost rabbitmq]# service rabbitmq-server start
Redirecting to /bin/systemctl start rabbitmq-server.service
[root@localhost rabbitmq]# 

```

##### 查看rabbitmq状态
```
[root@localhost rabbitmq]# service rabbitmq-server status
Redirecting to /bin/systemctl status rabbitmq-server.service
● rabbitmq-server.service - RabbitMQ broker
   Loaded: loaded (/usr/lib/systemd/system/rabbitmq-server.service; disabled; vendor preset: disabled)
   Active: active (running) since 三 2018-03-07 10:25:08 CST; 1min 14s ago
 Main PID: 1325 (beam.smp)
   Status: "Initialized"
   CGroup: /system.slice/rabbitmq-server.service
           ├─1325 /usr/lib64/erlang/erts-9.2.1/bin/beam.smp -W w -A 64 -P 1048576 -t 5000000 -stbt db -zdbbl 128000...
           ├─1507 /usr/lib64/erlang/erts-9.2.1/bin/epmd -daemon
           ├─1639 erl_child_setup 1024
           ├─1659 inet_gethost 4
           └─1660 inet_gethost 4

3月 07 10:25:04 localhost.localdomain rabbitmq-server[1325]: ##  ##
3月 07 10:25:04 localhost.localdomain rabbitmq-server[1325]: ##  ##      RabbitMQ 3.7.3. Copyright (C) 2007-201...nc.
3月 07 10:25:04 localhost.localdomain rabbitmq-server[1325]: ##########  Licensed under the MPL.  See http://ww...om/
3月 07 10:25:04 localhost.localdomain rabbitmq-server[1325]: ######  ##
<!--这里是log的位置-->
3月 07 10:25:04 localhost.localdomain rabbitmq-server[1325]: ##########  Logs: /var/log/rabbitmq/rabbit@localhost.log
3月 07 10:25:04 localhost.localdomain rabbitmq-server[1325]: /var/log/rabbitmq/rabbit@localhost_upgrade.log
3月 07 10:25:04 localhost.localdomain rabbitmq-server[1325]: Starting broker...
3月 07 10:25:08 localhost.localdomain rabbitmq-server[1325]: systemd unit for activation check: "rabbitmq-serve...ce"
3月 07 10:25:08 localhost.localdomain systemd[1]: Started RabbitMQ broker.
3月 07 10:25:10 localhost.localdomain rabbitmq-server[1325]: completed with 3 plugins.
Hint: Some lines were ellipsized, use -l to show in full.
[root@localhost rabbitmq]# 

```

##### 查看rabbitmq日志

```
[root@localhost rabbitmq]# cd /var/log/rabbitmq/
[root@localhost rabbitmq]# ls
erl_crash.dump  rabbit@localhost.log              rabbit@localhost_upgrade.log
log             rabbit@localhost.log-20180306.gz  rabbit@localhost_upgrade.log-20180306.gz

```

```
[root@localhost rabbitmq]# cat rabbit@localhost.log   
 Starting RabbitMQ 3.7.3 on Erlang 20.2.3
 Copyright (C) 2007-2018 Pivotal Software, Inc.
 Licensed under the MPL.  See http://www.rabbitmq.com/
 2018-03-07 10:25:04.157 [info] <0.247.0> 
 node           : rabbit@localhost
 home dir       : /var/lib/rabbitmq
 <!--这是里配置文件的路径,我已经配置好了 -->
 <!--第一次安装完成的时候,配置文件是不存在的,需要我们新建一个-->
 config file(s) : /etc/rabbitmq/rabbitmq.config
 cookie hash    : kdRZlH6EzW+h0o2u3onyUg==
 log(s)         : /var/log/rabbitmq/rabbit@localhost.log
                : /var/log/rabbitmq/rabbit@localhost_upgrade.log
 database dir   : /var/lib/rabbitmq/mnesia/rabbit@localhost
2018-03-07 10:25:08.222 [info] <0.255.0> Memory high watermark set to 390 MiB (409041305 bytes) of 975 MiB (1022603264 bytes) total

```

##### 新建rabbitmq config

进入`/etc/rabbitmq/`目录,新建rabbitmq.config文件

```
[root@localhost rabbitmq]# cd /etc/rabbitmq/
[root@localhost rabbitmq]# vim rabbitmq.config 

[
{rabbit, [{tcp_listeners, [5672]}, {loopback_users, ["admin"]}]}
].
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
~                                                                                                                     
"rabbitmq.config" 3L, 70C                                                                           3,1          全部


```

rabbitmq默认创建的用户guest，密码也是guest，这个用户默认只能是本机访问，localhost或者127.0.0.1，从外部访问需要添加上面的配置,保存配置后重启服务。

##### rabbitmq 命令
```
<!--查看RabbitMQ中用户命令-->
rabbitmqctl list_users

<!--创建用户命令-->
 rabbitmqctl add_user admin admin

<!--赋予用户权限命令-->
 rabbitmqctl  set_permissions -p "/" admin '.*' '.*' '.*'

<!--赋予用户角色命令-->
 rabbitmqctl set_user_tags admin administrator

<!--开启rabbitmq管理控制台命令-->
rabbitmq-plugins enable rabbitmq_management

```
##### rabbitmq管理界面
![image](http://p5in4o880.bkt.clouddn.com//loveyy/image/mq/admin_index.png)

## Java使用rabbitmq

### MAVEN依赖

```
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.1.2</version>
</dependency>

```
### 消息生产者代码

```
public class DirectProducer {

    public static final String QUEUE_NAME = "com.szl.direct.que";
    public static final String DIRECT_EXCHANGE_NAME = "com.szl.direct.exchange";
    public static final String ROUTING_KEY = "com.szl.direct.que.routing";

    public static void main(String[] args){

        // 创建连接工厂
        ConnectionFactory factory = RabbitMqBase.getConnectionFactoryInstance();
        Connection connection = null;
        Channel channel = null;

        // 设置rabbitmq
        factory.setHost("192.168.56.128");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("admin");
        factory.setPassword("admin");
        try {
            // 创建一个连接
            connection= RabbitMqBase.getConnection(factory);

            // 创建一个通道
            channel = RabbitMqBase.getChannel(connection);

            // 声明一个交换机,这里使用DIRECT
            channel.exchangeDeclare(DIRECT_EXCHANGE_NAME, BuiltinExchangeType.DIRECT,true);

            // 声明一个队列
            channel.queueDeclare(QUEUE_NAME, true, false, false, null);

            // 队列绑定到交换机
            channel.queueBind(QUEUE_NAME,DIRECT_EXCHANGE_NAME,ROUTING_KEY);

            // 发送消息到队列
            System.out.println("-----写入消息队列开始------");
            for (int i = 1; i < 11; i++) {
                String message = "Hello rabbitmq(" + i + ")";
                channel.basicPublish(DIRECT_EXCHANGE_NAME,ROUTING_KEY,null,message.getBytes());
            }
            System.out.println("-----写入消息队列完成------");
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
        finally {
            try {
                RabbitMqBase.close(connection,channel);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (TimeoutException e) {
                e.printStackTrace();
            }
        }

    }
}


```

### 运行消息提供者

运行消息提供者,我们可以在MQ管理控制台看到刚刚写入的对列消息

![image](http://p5in4o880.bkt.clouddn.com//loveyy/image/mq/mq_p.png)


### 消息消费者

```
public class DirectCustomer extends CustomerBase {

    public static void main(String[] args){
        // 创建连接工厂
        ConnectionFactory factory = RabbitMqBase.getConnectionFactoryInstance();

        // 设置rabbitmq
        factory.setHost("192.168.56.128");
        factory.setPort(5672);

        // 创建一个连接
        Connection connection = null;
        try {
            connection = RabbitMqBase.getConnection(factory);

            // 创建一个通道
            Channel channel = RabbitMqBase.getChannel(connection);

            // 声明一个交换机,这里使用DIRECT
            channel.exchangeDeclare(DirectProducer.DIRECT_EXCHANGE_NAME, BuiltinExchangeType.DIRECT,true);

            // 声明一个队列
            channel.queueDeclare(DirectProducer.QUEUE_NAME, true, false, false, null);
            // 监听一个通道
            Consumer consumer = defaultConsumer(channel);

            // 自动回复队列应答 -- RabbitMQ中的消息确认机制
            channel.basicConsume(DirectProducer.QUEUE_NAME, true, consumer);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
    }
}

```

### 运行消息消费者

运行消息消费者,我们可以在MQ管理控制台看到已经有了消费者,且在控制台输出了接收到的消息内容

- 控制台

![image](http://p5in4o880.bkt.clouddn.com//loveyy/image/mq/mq_c.png)

- 接收到的消息内容

![image](http://p5in4o880.bkt.clouddn.com//loveyy/image/mq/mq_c_idea.png)


++注意:以上代码使用的DIRECT模式,完整代码下载请[点我](https://github.com/leungandi/rabbitmq-demo)++

## spring整合rabbitmq

### MAVEN依赖

```
<!--rabbitmq依赖-->
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>2.0.2.RELEASE</version>
</dependency>

```

### rabbit.xml配置

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/rabbit http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">

    <!--RabbitMQ 服务器-->
    <rabbit:connection-factory id="connectionFactory" addresses="192.168.56.128:5672"
                               username="admin" password="admin" virtual-host="/"/>

    <rabbit:admin connection-factory="connectionFactory"/>

    <!--生产者-->
    <!--定义queue -->
    <rabbit:queue name="com.szl.direct.que" durable="true" auto-delete="false" exclusive="false"  />

    <!--定义exchange -->
    <rabbit:direct-exchange name="com.szl.direct.exchange" durable="true" auto-delete="false">
        <rabbit:bindings>
            <rabbit:binding queue="com.szl.direct.que" key="com.szl.direct.que.routing"/>
        </rabbit:bindings>
    </rabbit:direct-exchange>

    <!--声明一个 RabbitMQ Template-->
    <rabbit:template id="amqpTemplate"  connection-factory="connectionFactory" exchange="com.szl.direct.exchange" queue="com.szl.direct.que" routing-key="com.szl.direct.que.routing"/>

    <!--数据装换类-->
    <bean id="jackson2JsonMessageConverter"
          class="org.springframework.amqp.support.converter.Jackson2JsonMessageConverter"/>



    <!-- 消费者 -->
    <bean name="mqCustomer" class="com.szl.rabbit.customer.MqCustomer"/>
    <!-- 配置监听 -->
    <rabbit:listener-container connection-factory="connectionFactory" message-converter="jackson2JsonMessageConverter">
        <rabbit:listener ref="mqCustomer" queue-names="com.szl.direct.que" method="onMessage"/>
    </rabbit:listener-container>
</beans>

```

### aplicationContext.xml配置

```

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="
     http://www.springframework.org/schema/context
     http://www.springframework.org/schema/context/spring-context-3.0.xsd
     http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
     http://www.springframework.org/schema/util
     http://www.springframework.org/schema/util/spring-util-3.0.xsd
     http://www.springframework.org/schema/aop
     http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
     http://www.springframework.org/schema/tx
     http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
     http://www.springframework.org/schema/rabbit
     http://www.springframework.org/schema/rabbit/spring-rabbit-1.0.xsd">

    <import resource="classpath*:rabbitmq.xml"/>


    <context:component-scan base-package="com.szl"/>

    <context:annotation-config/>


</beans>

```

### 消息生产者

```
@Service("mqProvider")
public class MqProvider {

    @Resource(name="amqpTemplate")
    private AmqpTemplate amqpTemplate;


    public void sendMsg() throws UnsupportedEncodingException {
        System.out.println("spring:开始写入消息队列");
        for (int i = 0; i < 10 ; i++) {
            String msg = "Hello Spring-RabbitMq("+i+")";
            Message message = MessageBuilder.withBody(msg.getBytes("utf-8"))
                    .setMessageId(System.currentTimeMillis() + "")
                    .build();
            amqpTemplate.send(message);
        }
        System.out.println("spring:写入消息队列完成");
    }

}

```

### 消息消费者

```
public class MqCustomer implements MessageListener{


    public void onMessage(Message message) {
        System.out.println(new String(message.getBody()));
    }
}


```

### test

```
public class RabbitMqTest {

    private ApplicationContext applicationContext = null;

    @Before
    public void init(){
        applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
    }

    @Test
    public void mqTest() throws InterruptedException, UnsupportedEncodingException {
        MqProvider mqProvider = (MqProvider) applicationContext.getBean("mqProvider");
        mqProvider.sendMsg();

        // 暂停一下,让消费者去处理
        Thread.sleep(6000);
    }
}


```

### 测试结果

![image](http://p5in4o880.bkt.clouddn.com//loveyy/image/mq/mq_t.png)