---
title: Nginx安装
date: 2017-09-15 14:25
categories: LVS + Keepalived + Nginx
tags: nginx
---
### Nginx简介

Nginx是一个web服务器也可以用来做负载均衡及反向代理使用，目前使用最多的就是负载均衡

### Nginx安装

#### 准备工作

##### 下载Nginx
首先到官网下载Nginx,点击[Nginx官网](http://nginx.org/en/download.html)

##### 下载模块依赖

下载模块依赖性Nginx需要依赖下面3个包
- gzip 模块需要 zlib 库 ( 下载: http://www.zlib.net/ )
- rewrite 模块需要 pcre 库 ( 下载: http://www.pcre.org/ )
- ssl 功能需要 openssl 库 ( 下载: http://www.openssl.org/ )


**如果没有安装c + +编译环境，请通过yum install gcc-c++完成安装**

#### 开始安装

我把下载好的Nginx及相关组件放到`/opt/soft/nginx/`目录下

```
[root@localhost nginx]# cd /opt/soft/nginx/
[root@localhost nginx]# ll
总用量 5040
-rw-r--r--. 1 root root  981093 9月   2 10:34 nginx-1.12.1.tar.gz
-rw-r--r--. 1 root root 1492654 9月   2 10:47 openssl-fips-2.0.16.tar.gz
-rw-r--r--. 1 root root 2068775 9月   2 10:47 pcre-8.41.tar.gz
-rw-r--r--. 1 root root  607698 9月   2 10:47 zlib-1.2.11.tar.gz
[root@localhost nginx]# 

```

依赖包安装顺序依次为:openssl、zlib、pcre, 然后安装Nginx包

##### 安装openssl

```
[root@localhost nginx]# tar openssl-fips-2.0.16.tar.gz 
[root@localhost nginx]# cd openssl-fips-2.0.16
[root@localhost openssl-fips-2.0.16]# ./config && make && make install

```
##### 安装zlib

```
[root@localhost nginx]# tar -zxvf zlib-1.2.11.tar.gz 
[root@localhost nginx]# cd zlib-1.2.11
[root@localhost nginx]# ./configure && make && make install

```

##### 安装pcre

```
[root@localhost nginx]# tar -zxvf pc
[root@localhost nginx]# cd pcre-8.41
[root@localhost nginx]# ./configure && make && make install

```

##### 安装nginx

```
[root@localhost nginx]# tar -zxvf nginx-1.12.1.tar.gz 
[root@localhost nginx]# cd nginx-1.12.1
[root@localhost nginx]# ./configure && make && make install
-- 查找ngnix安装目录
[root@localhost nginx-1.12.1]# whereis nginx
nginx: /usr/local/nginx


```

### 启动Ngnix

进入到安装目录启动nginx
```
[root@localhost sbin]# cd /usr/local/nginx/sbin/
[root@localhost sbin]# ./nginx 
./nginx: error while loading shared libraries: libpcre.so.1: cannot open shared object file: No such file or directory
[root@localhost sbin]# 

```


启动报错解决:
```
1.用whereis libpcre.so.1命令找到libpcre.so.1在哪里
2.用ln -s /usr/local/lib/libpcre.so.1 /lib64命令做个软连接就可以了
3.用sbin/nginx启动Nginx
[root@localhost nginx]# whereis libpcre.so.1
[root@localhost nginx]# ln -s /usr/local/lib/libpcre.so.1 /lib64
[root@localhost nginx]# sbin/nginx
```

查看启动状态
ps -aux | grep nginx
```
[root@localhost sbin]# ps -aux | grep nginx 
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
root      48399  0.0  0.0  20408   660 ?        Ss   11:57   0:00 nginx: master process ./nginx
nobody    48400  0.0  0.1  20852  1260 ?        S    11:57   0:00 nginx: worker process
root      48402  0.0  0.0 103268   884 pts/0    S+   11:58   0:00 grep nginx


```

启动后输入地址进入首页

![nginx首页](/images/nginx/ngnix.jpg)

### Nginx基本操作命令

```
启动
[root@localhost ~]# /usr/local/nginx/sbin/nginx
停止/重启
[root@localhost ~]# /usr/local/nginx/sbin/nginx -s stop(quit、reload)
命令帮助
[root@localhost ~]# /usr/local/nginx/sbin/nginx -h
验证配置文件
[root@localhost ~]# /usr/local/nginx/sbin/nginx -t
配置文件
[root@localhost ~]# vim /usr/local/nginx/conf/nginx.conf

```

### 简单配置Ngnix

配置nginx.conf

```
#================================以下是全局配置项
#指定运行nginx的用户和用户组，默认情况下该选项关闭（关闭的情况就是nobody）
#user  nobody;
#运行nginx的进程数量
worker_processes  1;
#nginx运行错误的日志存放位置。当然您还可以指定错误级别
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#指定主进程id文件的存放位置，虽然worker_processes != 1的情况下，会有很多进程，管理进程只有一个
#pid        logs/nginx.pid;


events {
 	#每一个进程可同时建立的连接数量
    worker_connections  1024;
}
#================================以上是全局配置项


http {


#================================以下是Nginx后端服务配置项
    upstream my_server{
        #nginx向后端服务器分配请求任务的方式，默认为轮询；如果指定了ip_hash，就是hash算法（上文介绍的算法内容）
        ip_hash;    
        #后端服务器 ip:port ，如果有多个服务节点，这里就配置多个
        server 192.168.1.23:8082; 
        #server 192.168.1.23:8080;    
        #backup表示，这个是一个备份节点，只有当所有节点失效后，nginx才会往这个节点分配请求任务
        #server 192.168.220.133:8080 backup;        
        #weight，固定权重，还记得我们上文提到的加权轮询方式吧。
        #server 192.168.220.134:8080 weight=100;    
    }
#================================以上是Nginx后端服务配置项

 #=================================================以下是 http 协议主配置
 	#安装nginx后，在conf目录下除了nginx.conf主配置文件以外，有很多模板配置文件，这里就是导入这些模板文件
    include       mime.types;
    #HTTP核心模块指令，这里设定默认类型为二进制流，也就是当文件类型未定义时使用这种方式
    default_type  application/octet-stream;
    #日志格式
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    #日志文件存放的位置
    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;
 	#指定一个连接的等待时间（单位秒），如果超过等待时间，连接就会断掉。注意一定要设置，否则高并发情况下会产生性能问题。
    #keepalive_timeout  0;
    keepalive_timeout  65;
	#开启数据压缩
    #gzip  on;
#=================================================以上是 http 协议主配置

#=================================================以下是一个服务实例的配置
    #每一个server就是一个虚拟主机，我们有一个当作web服务器来使用
    #listen 80;代表监听80端口
	#server_name xxx.com;代表外网访问的域名
	#location / {};代表一个过滤器，/匹配所有请求，我们还可以根据自己的情况定义不同的过滤，比如对静态文件js、css、image制定专属过滤
	#root html;代表站点根目录
	#index index.html;代表默认主页
    server {
        listen       80;
        server_name  nginx.test.com;

        #文字格式
        charset utf-8; 

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
            #反向代理
            proxy_pass   http://my_server;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

     server {
        listen       81;
        server_name  nginx.test1.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }




    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
#=================================================以上是一个服务实例的配置

}


```

配置[参考这里](http://blog.csdn.net/yinwenjie/article/details/46620711),感谢！！！







