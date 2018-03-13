---
title: RabbitMQ入门
date: 2018-02-27 13:50:14
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

## Java使用rabbitmq




## spring整合rabbitmq

















