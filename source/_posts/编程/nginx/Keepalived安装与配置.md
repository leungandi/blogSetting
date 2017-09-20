---
title: Keepalived安装
date: 2017-09-19 11:50
categories: LVS + Keepalived + Nginx
tags: nginx
---
### Keepalived简介

> Keepalived的作用是检测服务器的状态，如果有一台web服务器宕机，或工作出现故障，Keepalived将检测到，并将有故障的服务器从系统中剔除，同时使用其他服务器代替该服务器的工作，当服务器工作正常后Keepalived自动将服务器加入到服务器群中，这些工作全部自动完成，不需要人工干涉，需要人工做的只是修复故障的服务器。

### Keepalived安装

#### 准备工作

##### 下载Keepalived

首先到官网下载Keepalived,点击[Keepalived官网](http://www.keepalived.org/download.html)

我把下载好的Keepalived及相关组件放到`/opt/soft/Keepalived/`目录下

```
[root@localhost nginx]# cd /opt/soft/Keepalived/
[root@localhost nginx]# ll
总用量 688
-rw-r--r--. 1 root root 702570 9月   6 06:26 keepalived-1.2.17.tar.gz
[root@localhost Keepalived]# 

```
##### 准备nginx

准备两台Nginx的主机，我这里两台的ip分别是:

- 192.168.1.21:80
- 192.168.1.23:80

也就是使用了上篇文章的nginx,hosts映射关系如下：

```
	192.168.1.23 nginx.test.com 
	192.168.1.21 nginx.test1.com

```


#### 开始安装

##### 安装Keepalived

分别在2台服务器独立安装Keepalived系统，一台是Master服务器(192.168.1.23)是主要的工作服务器，另一台是备份(slave)服务器(192.168.1.21)，在Master服务器出现问题后，由后者接替其工作,也就是在一台工作的Nginx崩溃的情况下，系统能够检测到，并自动将请求切换到另外一台备份的Nginx服务器上。


```
[root@localhost Keepalived]# tar -zxvf keepalived-1.2.17.tar.gz 
[root@localhost Keepalived]# cd keepalived-1.2.17
[root@localhost keepalived-1.2.17]# ./configure &&make && make install

```

##### 安装目录

安装时我们使用的默认路径安装,可以看到安装目录是`/usr/local/etc/keepalived`,`keepalived.conf`是keepalived的主要配置文件

```
[root@localhost keepalived]# whereis keepalived
keepalived: /usr/local/sbin/keepalived /usr/local/etc/keepalived
[root@localhost keepalived]# cd /usr/local/etc/keepalived
[root@localhost keepalived]# ls
keepalived.conf  samples
[root@localhost keepalived]# 

```
#### 开始配置

外网进行Nginx访问的浮动IP：192.168.1.222

我们将192.168.1.23这台服务器上运行的Nginx作为主要的Nginx，其上的keepalived服务我们设置成Master方式。

我们将192.168.1.21这台服务器上运行的Nginx作为备用的Nginx服务，其上的keepalived服务我们设置为Backup方式。

###### 配置Master

- master上的原始IP信息

```

[root@localhost keepalived]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:0A:2E:4A  
          inet addr:192.168.1.23  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe0a:2e4a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:25943 errors:0 dropped:0 overruns:0 frame:0
          TX packets:5586 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:8511108 (8.1 MiB)  TX bytes:915948 (894.4 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:1009 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1009 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:257366 (251.3 KiB)  TX bytes:257366 (251.3 KiB)

[root@localhost keepalived]# 

```


- MASTER keepalived.conf配置

```
! Configuration File for keepalived
#全局配置
global_defs {
   #指定keepalived的发生切换时需要发送email的对象，一行一个。    
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   #指定发件人
   notification_email_from Alexandre.Cassen@firewall.loc
   #指定smtp服务器地址
   smtp_server 192.168.200.1
   #指定smtp连接超时时间
   smtp_connect_timeout 30
   #运行keepalived机器的一个标识
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

#检查nginx状态
vrrp_script chknginx {
  script "/etc/keepalived/checkNginx.sh"
    #每5秒钟，检查一次
    interval 5
}

#keepalived实例设置，是最重要的设置信息
vrrp_instance VI_1 {
    #state状态MASTER表示是主要工作节点，备机为BACKUP。
    #一个keepalived组中，最多只有一个MASTER节点，当然也可以没有
    state MASTER
    #实例所绑定的网卡设备，我的网卡设备是eth0。
    interface eth0
    #同一个keepalived组，节点的设置必须一样，这样才会被识别
    virtual_router_id 51
    #节点优先级，BACKUP的优先级一定要比MASTER的优先级低
    priority 100
    #组播信息发送间隔，两个节点设置必须一样
    advert_int 1
    #实际的eth1上的固定ip地址
    mcast_src_ip=192.168.1.23
    #验证信息，只有验证信息相同，才能被加入到一个组中。
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    #虚拟地址和绑定的端口，如果有多个，就绑定多个
    #dev 是指定浮动IP要绑定的网卡设备号
    virtual_ipaddress {
        192.168.1.222 dev eth0
    }
    #设置的检查脚本
    #关联上方的“vrrp_script chknginx”
    track_script {
        chknginx
    } 
}

virtual_server 192.168.200.100 443 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 192.168.201.100 443 {
        weight 1
        SSL_GET {
            url {
              path /
              digest ff20ad2481f97b1754ef3e12ecd3a9cc
            }
            url {
              path /mrtg/
              digest 9b3a0c85a887a256d6939da88aabd8cd
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.2 1358 {
    delay_loop 6
    lb_algo rr 
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    sorry_server 192.168.200.200 1358

    real_server 192.168.200.2 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.3 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.3 1358 {
    delay_loop 3
    lb_algo rr 
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 192.168.200.4 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.5 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

```

###### 配置Backup 

- backup上的原始IP信息

```

[root@localhost keepalived-1.3.6]# ifconfig
eth1      Link encap:Ethernet  HWaddr 00:0C:29:A0:47:EC  
          inet addr:192.168.1.21  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fea0:47ec/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:28083 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4543 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:8609984 (8.2 MiB)  TX bytes:707367 (690.7 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:128 errors:0 dropped:0 overruns:0 frame:0
          TX packets:128 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:82519 (80.5 KiB)  TX bytes:82519 (80.5 KiB)

[root@localhost keepalived-1.3.6]# 


```

- BACKUP keepalived.conf配置

```
! Configuration File for keepalived
#全局配置
global_defs {
   #指定keepalived的发生切换时需要发送email的对象，一行一个。    
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   #指定发件人
   notification_email_from Alexandre.Cassen@firewall.loc
   #指定smtp服务器地址
   smtp_server 192.168.200.1
   #指定smtp连接超时时间
   smtp_connect_timeout 30
   #运行keepalived机器的一个标识
   router_id LVS_DEVEL_2
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
#keepalived实例设置，是最重要的设置信息
vrrp_instance VI_1 {
    #state状态MASTER表示是主要工作节点，备机为BACKUP。
    #一个keepalived组中，最多只有一个MASTER节点，当然也可以没有
    state BACKUP
    #实例所绑定的网卡设备，我的备机网卡设备是eth1。
    interface eth1
    #同一个keepalived组，节点的设置必须一样，这样才会被识别
    virtual_router_id 51
    #节点优先级，BACKUP的优先级一定要比MASTER的优先级低
    priority 99
    #组播信息发送间隔，两个节点设置必须一样
    advert_int 1
    #实际的eth1上的固定ip地址
    mcast_src_ip=192.168.1.21
    #验证信息，只有验证信息相同，才能被加入到一个组中。
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    #虚拟地址和绑定的端口，如果有多个，就绑定多个
    #dev 是指定浮动IP要绑定的网卡设备号
    virtual_ipaddress {
        192.168.1.222 dev eth1
    }
}

virtual_server 192.168.200.100 443 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 192.168.201.100 443 {
        weight 1
        SSL_GET {
            url {
              path /
              digest ff20ad2481f97b1754ef3e12ecd3a9cc
            }
            url {
              path /mrtg/
              digest 9b3a0c85a887a256d6939da88aabd8cd
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.2 1358 {
    delay_loop 6
    lb_algo rr 
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    sorry_server 192.168.200.200 1358

    real_server 192.168.200.2 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.3 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.3 1358 {
    delay_loop 3
    lb_algo rr 
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 192.168.200.4 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.5 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}


```

#### 启动keepalived

启动主节点和备用节点,keeplived启动关闭命令

```
默认的安装位置是：/usr/local/etc/keepalived/
[root@localhost init.d]# cp /usr/local/etc/rc.d/init.d/keepalived /etc/rc.d/init.d/ 
[root@localhost init.d]# cp /usr/local/etc/sysconfig/keepalived /etc/sysconfig/ 
[root@localhost init.d]# mkdir /etc/keepalived 
[root@localhost init.d]# cp /usr/local/etc/keepalived/keepalived.conf /etc/keepalived/ 
[root@localhost init.d]# cp /usr/local/sbin/keepalived /usr/sbin/ 

```
执行完以上几条命令后，我们就可以使用以下命令来启动、停止keepalived。

```
[root@localhost init.d]# service keepalived start
正在启动 keepalived：                                      [确定]
[root@localhost init.d]# 
[root@localhost init.d]# ps -aux |grep keepalived 
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
root       8934  0.0  0.0  40004   984 ?        Ss   08:45   0:00 keepalived -D
root       8936  0.2  0.1  40128  1896 ?        S    08:45   0:00 keepalived -D
root       8937  0.1  0.1  40004  1212 ?        S    08:45   0:00 keepalived -D
root       8944  0.0  0.0 103268   888 pts/0    S+   08:46   0:00 grep keepalived
[root@localhost init.d]# service keepalived stop
停止 keepalived：                                          [确定]
[root@localhost init.d]# 

```

- 主机启动状态

主机绑定了虚拟ip

```
[root@localhost keepalived]# service keepalived start
正在启动 keepalived：                                      [确定]
[root@localhost keepalived]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:0a:2e:4a brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.23/24 brd 192.168.1.255 scope global eth0
    inet 192.168.1.234/32 scope global eth0
    inet6 fe80::20c:29ff:fe0a:2e4a/64 scope link 
       valid_lft forever preferred_lft forever
[root@localhost keepalived]# 

```

- 备机启动状态

备机未绑定虚拟ip

```
[root@localhost log]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:a0:47:ec brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.21/24 brd 192.168.1.255 scope global eth1
    inet6 fe80::20c:29ff:fea0:47ec/64 scope link 
       valid_lft forever preferred_lft forever


```



##### 测试一下

1.访问192.168.1.234,进入主机的nginx
![image](/images/keepalived/master.jpg)

2.停掉192.168.1.23主机的keepalived服务,刷新页面,进入备机

![image](/images/keepalived/backup.jpg)

主机ip:

```
[root@localhost log]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:0a:2e:4a brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.23/24 brd 192.168.1.255 scope global eth0
    inet6 fe80::20c:29ff:fe0a:2e4a/64 scope link 
       valid_lft forever preferred_lft forever
[root@localhost log]# 

```

备机ip:

```
[root@localhost log]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:a0:47:ec brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.21/24 brd 192.168.1.255 scope global eth1
    inet 192.168.1.234/32 scope global eth1
    inet6 fe80::20c:29ff:fea0:47ec/64 scope link 
       valid_lft forever preferred_lft forever
[root@localhost log]# 


```

备机日志:

```

[root@localhost log]# tail -f messages
Sep  6 09:34:25 localhost Keepalived_healthcheckers[7025]: Remote SMTP server [192.168.200.1]:25 connected.
Sep  6 09:34:26 localhost Keepalived_healthcheckers[7025]: Timeout connect, timeout server [192.168.201.100]:443.
Sep  6 09:34:26 localhost Keepalived_healthcheckers[7025]: Removing service [192.168.201.100]:443 from VS [192.168.200.100]:443
Sep  6 09:34:26 localhost Keepalived_healthcheckers[7025]: Lost quorum 1-0=1 > 0 for VS [192.168.200.100]:443
Sep  6 09:34:26 localhost Keepalived_healthcheckers[7025]: Remote SMTP server [192.168.200.1]:25 connected.
Sep  6 09:34:53 localhost Keepalived_healthcheckers[7025]: Timeout reading data to remote SMTP server [192.168.200.1]:25.
Sep  6 09:34:53 localhost Keepalived_healthcheckers[7025]: Timeout reading data to remote SMTP server [192.168.200.1]:25.
Sep  6 09:34:53 localhost Keepalived_healthcheckers[7025]: Timeout reading data to remote SMTP server [192.168.200.1]:25.
Sep  6 09:34:55 localhost Keepalived_healthcheckers[7025]: Timeout reading data to remote SMTP server [192.168.200.1]:25.
Sep  6 09:34:56 localhost Keepalived_healthcheckers[7025]: Timeout reading data to remote SMTP server [192.168.200.1]:25.
Sep  6 09:40:15 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) Transition to MASTER STATE
Sep  6 09:40:15 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) Entering MASTER STATE
Sep  6 09:40:15 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) setting protocol VIPs.
Sep  6 09:40:15 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth1 for 192.168.1.234
Sep  6 09:40:15 localhost Keepalived_healthcheckers[7025]: Netlink reflector reports IP 192.168.1.234 added
Sep  6 09:40:20 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth1 for 192.168.1.234


```

3.重新启动192.168.1.24主机的keepalived服务,刷新页面,重新进入主机


备机日志:

```
[root@localhost log]# tail -f messages
Sep  6 09:40:15 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) Transition to MASTER STATE
Sep  6 09:40:15 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) Entering MASTER STATE
Sep  6 09:40:15 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) setting protocol VIPs.
Sep  6 09:40:15 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth1 for 192.168.1.234
Sep  6 09:40:15 localhost Keepalived_healthcheckers[7025]: Netlink reflector reports IP 192.168.1.234 added
Sep  6 09:40:20 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth1 for 192.168.1.234
Sep  6 09:42:37 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) Received higher prio advert
Sep  6 09:42:37 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) Entering BACKUP STATE
Sep  6 09:42:37 localhost Keepalived_vrrp[7026]: VRRP_Instance(VI_1) removing protocol VIPs.
Sep  6 09:42:37 localhost Keepalived_healthcheckers[7025]: Netlink reflector reports IP 192.168.1.234 removed

```

#### 检测nginx状态脚本

```
#!/bin/sh
if [ $(ps -C nginx | wc -l) -eq 1 ];then
  service keepalived stop
fi

```

- 注意字符集问题，否则脚本不能执行
- 注意脚本权限问题，否则脚本不能运行
























