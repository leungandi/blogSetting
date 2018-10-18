---
title: Redis5.0集群的简单搭建
date: 2018-10-18 17:19:00
tags:
categories: 编程
---

## 安装Redis

**环境说明:**

1.操作系统:centos 7

2.redis:5.0

```
<!--查看防火墙状态-->
systemctl status firewalld.service

<!--关闭防火墙-->
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动

```


### Redis官网

[Redis官网下载](https://redis.io/download)

### 开始下载安装

```
$ wget http://download.redis.io/releases/redis-5.0.0.tar.gz
$ tar xzf redis-5.0.0.tar.gz
$ cd redis-5.0.0

[root@localhost redis-5.0.0]# make install PREFIX=/usr/local/redis

```

### 完成安装

进入到安装目录,拷贝`redis.conf`配置文件到bin目录

```

[root@localhost redis-5.0.0]# cd /usr//local/redis/bin/
[root@localhost bin]# ll
总用量 32700
-rwxr-xr-x. 1 root root 4365928 10月 14 10:52 redis-benchmark
-rwxr-xr-x. 1 root root 8087400 10月 14 10:52 redis-check-aof
-rwxr-xr-x. 1 root root 8087400 10月 14 10:52 redis-check-rdb
-rwxr-xr-x. 1 root root 4782960 10月 14 10:52 redis-cli
-rw-r--r--. 1 root root   62155 10月 14 10:53 redis.conf
lrwxrwxrwx. 1 root root      12 10月 14 10:52 redis-sentinel -> redis-server
-rwxr-xr-x. 1 root root 8087400 10月 14 10:52 redis-server


```

## 修改配置

### 修改目录名

- 把bin文件名称修改为7001

```
[root@localhost redis]# mv bin/ 7001
[root@localhost redis]# ll
总用量 0
drwxr-xr-x. 2 root root 152 10月 14 10:58 7001


```

- 复制6个同样的文件


```

[root@localhost redis]# cp -r 7001/ 7002
[root@localhost redis]# cp -r 7001/ 7003
[root@localhost redis]# cp -r 7001/ 7004
[root@localhost redis]# cp -r 7001/ 7005
[root@localhost redis]# cp -r 7001/ 7006
[root@localhost redis]# ll
总用量 0
drwxr-xr-x. 2 root root 152 10月 14 10:58 7001
drwxr-xr-x. 2 root root 152 10月 14 11:00 7002
drwxr-xr-x. 2 root root 152 10月 14 11:01 7003
drwxr-xr-x. 2 root root 152 10月 14 11:01 7004
drwxr-xr-x. 2 root root 152 10月 14 11:01 7005
drwxr-xr-x. 2 root root 152 10月 14 11:01 7006


```

### 修改redis.conf


以下是最小的Redis群集配置文件

```
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes

```

### 启动redis

可写一个脚本,启动6台redis

```
cd 7001
./redis-server redis.conf
cd ../7002
./redis-server redis.conf
cd ../7003
./redis-server redis.conf
cd ../7004
./redis-server redis.conf
cd ../7005
./redis-server redis.conf
cd ../7006
./redis-server redis.conf


```

### 配置集群

使用命名`./redis-cli --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 --cluster-replicas 1`配置集群,配置成功如下：

```

[root@localhost 7001]# ./redis-cli --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
Adding replica 127.0.0.1:7006 to 127.0.0.1:7003
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 4ad5a426a7a912fa27f9afb77a919f99312a86c3 127.0.0.1:7001
   slots:[0-5460] (5461 slots) master
M: 1b280d5e72a30650f93c89cfadc431e0fb5d0f5c 127.0.0.1:7002
   slots:[5461-10922] (5462 slots) master
M: 2aac2917fad5bb2f8598631473fa1b5f1d461963 127.0.0.1:7003
   slots:[10923-16383] (5461 slots) master
S: ed82bf6d0a0a81090a731a75bc0558260babd0d0 127.0.0.1:7004
   replicates 4ad5a426a7a912fa27f9afb77a919f99312a86c3
S: 21e3c518644901931baa8f8464734f8bec0d6633 127.0.0.1:7005
   replicates 1b280d5e72a30650f93c89cfadc431e0fb5d0f5c
S: 69dc59c483da498c8b4c311c2e2ab3eaef8812f1 127.0.0.1:7006
   replicates 2aac2917fad5bb2f8598631473fa1b5f1d461963
==Can I set the above configuration? (type 'yes' to accept): yes==
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
..
>>> Performing Cluster Check (using node 127.0.0.1:7001)
M: 4ad5a426a7a912fa27f9afb77a919f99312a86c3 127.0.0.1:7001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 69dc59c483da498c8b4c311c2e2ab3eaef8812f1 127.0.0.1:7006
   slots: (0 slots) slave
   replicates 2aac2917fad5bb2f8598631473fa1b5f1d461963
M: 2aac2917fad5bb2f8598631473fa1b5f1d461963 127.0.0.1:7003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: 1b280d5e72a30650f93c89cfadc431e0fb5d0f5c 127.0.0.1:7002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 21e3c518644901931baa8f8464734f8bec0d6633 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 1b280d5e72a30650f93c89cfadc431e0fb5d0f5c
S: ed82bf6d0a0a81090a731a75bc0558260babd0d0 127.0.0.1:7004
   slots: (0 slots) slave
   replicates 4ad5a426a7a912fa27f9afb77a919f99312a86c3
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.


```


## 测试访问

 - 使用命令登录`redis-cli -c -p 7001`,set hello

```
[root@localhost 7001]# ./redis-cli -c -p 7001
127.0.0.1:7001> set hello world
OK
127.0.0.1:7001> get hello
"world"
127.0.0.1:7001> 

```
- 重新登录7002端口的机器,get hello,可以看到返回world,且是在7001端口的机器上

```
[root@localhost 7001]# ./redis-cli -c -p 7002
127.0.0.1:7002> get hello
-> Redirected to slot [866] located at 127.0.0.1:7001
"world"
127.0.0.1:7001> 


```

