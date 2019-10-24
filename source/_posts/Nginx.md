---
title: Nginx
date: 2019-10-6 15:31:52
categories: [Java,stuNotes,web]
tags: nginx
---

> 倏忽国庆70周年，回顾历史，外交风云，随风入画

**Nginx**是一款高性能的http 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。c语言开发，可支撑5万并发链接，tomcat 500。资源消耗低，稳定开源。  
使用nginx进行配置，达到只使用域名访问的目的

<!--more-->

**应用场景**
1、http服务器。Nginx是一个http服务可以独立提供http服务。做网页静态服务器。
2、虚拟主机。在一台服务器虚拟出多个网站。例如个人网站使用的虚拟主机。
3、反向代理，负载均衡。当网站的访问量达到一定程度后，单台服务器不能满足用户的请求时，需要用多台服务器集群可以使用nginx做反向代理。并且多台服务器可以平均分担负载，不会因为某台服务器负载高宕机而某台服务器闲置的情况。
**安装要求的环境**
1、gcc环境。yum -y install gcc-c++
2、第三方的开发包

yum install -y pcre pcre-devel  解析正则

yum install -y zlib zlib-devel 压缩和解压缩

yum install -y openssl openssl-devel 密码库
**安装步骤**
第一步：把nginx的源码包上传到linux系统
第二步：解压缩
[root@localhost ~]# tar zxf nginx-1.8.0.tar.gz
第三步：
目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目录:
mkdir /var/temp/nginx/client -p
第四步：使用configure命令创建一makeFile文件。
./configure
–prefix=/usr/local/nginx
–pid-path=/var/run/nginx/nginx.pid
–lock-path=/var/lock/nginx.lock
–error-log-path=/var/log/nginx/error.log
–http-log-path=/var/log/nginx/access.log
–with-http_gzip_static_module
–http-client-body-temp-path=/var/temp/nginx/client
–http-proxy-temp-path=/var/temp/nginx/proxy
–http-fastcgi-temp-path=/var/temp/nginx/fastcgi
–http-uwsgi-temp-path=/var/temp/nginx/uwsgi
–http-scgi-temp-path=/var/temp/nginx/scgi

第四步：编译,使用：make
第五步：make install
sbin目录，启动命令 ./nginx
关闭命令 ./nginx -s stop 和 ./nginx -s quit
刷新配置文件 ./nginx -s reload

##### 通过端口区分不同的虚拟的主机

```
server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
    }
    server {
        listen       81;
        server_name  localhost;
        location / {
            root   html-81;
            index  index.html index.htm;
        }
    }
}
```

##### 域名区分

```
server {
        listen       80;
        server_name  www.taobao.com;
        location / {
            root   html-taobao;
            index  index.html index.htm;
        }
    }
    server {
        listen       80;
        server_name  www.baidu.com;
        location / {
            root   html-baidu;
            index  index.html index.htm;
        }
    }
}
```

##### 反向代理

> 反向代理服务器决定哪台服务器提供服务，只是请求的转发。

```
    upstream tomcat1 {
	    server 192.168.25.148:8080;
    }
    server {
        listen       80;
        server_name  www.sina.com.cn;
        location / {
            proxy_pass   http://tomcat1;
            index  index.html index.htm;
        }
    }
    upstream tomcat2 {
	    server 192.168.25.148:8081;
    }
    server {
        listen       80;
        server_name  www.sohu.com;
        location / {
            proxy_pass   http://tomcat2;
            index  index.html index.htm;
        }
    }
```

##### 负载均衡

默认的负载均衡的策略就是轮询的方式，可以根据服务器的实际情况调整服务器权重。权重越高分配的请求越多，权重越低，请求越少。默认是都是1

```
 upstream tomcat2 {
	server 192.168.25.148:8081;
	server 192.168.25.148:8082 weight=2;
 }
```

`其他的负载均衡的策略`：1.通过IP地址的hash值 做映射。2.通过URL的方式计算出Hash值 。3.随机策略。4.最少并发量。

##### 负载均衡高可用

建立一个备份机,主服务器和备份机上都运行高可用（High Availability）监控程序

**keepalived+nginx实现主备**

keepalived是集群管理中保证集群高可用的一个服务软件，用来防止单点故障。以VRRP协议为实现基础的-------检测web服务器的状态

keepalived主要有三个模块，分别是core、check和VRRP。core模块为keepalived的核心，负责主进程的启动、维护以及全局配置文件的加载和解析。check负责健康检查，包括常见的各种检查方式。VRRP模块是来实现VRRP协议的。 

`SSO`英文全称Single Sign On，单点登录。SSO是在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统。其实就是redis模拟Session。
