---
layout: post
title: "nginx反向代理与负载均衡应用"
description: ""
category: cluster
tags: []
---

### 什么是集群？

集群是若干个相互独立的计算机，利用高速网络通信而组成的一份较大的计算机服务系统。

---

### 集群的优势

* 1.高性能。对高并发请求而言，承受能力加倍。
* 2.价格有效性。相对于大型计算机，使用集群的成本还是相对比较低的。
* 3.可伸缩性。当服务负载，压力增长时，针对集群系统进行简单的扩展即可满足需求。
* 4.高可用性。集群可以有容灾备份的服务器，当出现单台服务器宕机的情况下，还能继续提供服务。
* 5.透明性。对于用户或者客户端程序来讲，集群是透明的，相当于访问单台机器一样。
* 6.可管理性。软硬件模块的增减，能做到即插即用。

---

### 集群分类

按照功能结构划分为：

 * 1.负载均衡集群(load balancing clusters),简称LB。
 * 2.高可用集群(high-availability clusters),简称HAC。
 * 3.高性能计算集群(high-performance clusters),简称HPC。
 * 4.网格计算集群。

---

### 常用集群软硬件介绍

    企业常用的开源集群软件有：nginx,LVS,haproxy ,keepalived ,heartbeat。
    企业常用的商业集群硬件有： F5,netscaler，redware， A10等。

    ps：小型企业使用：nginx / haproxy + keepalived 比较多，大企业一般使用：LVS+keepalived。
    如果是数据库或存储的负载均衡和高可用，建议使用LVS + heartbeat，当然技术力量薄弱些的公司也可以使用硬件去做负载均衡。 

---

### nginx反向代理与负载均衡区别

    普通的负载均衡(lvs)是转发用户请求的数据包，而nginx反向代理是接收用户请求然后重新发起请求去请求后面的节点。

---

## 快速搭建nginx负载均衡环境

### 软硬件准备

系统: centos6.8 x86_64
软件：nginx-1.6.3.tar.gz

准备4台虚拟机，2台做负载均衡，两台做web服务器。

| hostname      |    IP         | description |
| ------------- |:-------------:| -----------:|
| lb01          | 10.20.0.251   |  nginx主负载均衡 |
| lb02          | 10.20.0.252   |  nginx副负载均衡 |
| web01         | 10.20.0.253   |  web1服务器  |
| web02         | 10.20.0.254   |  web2服务器  |


#### 虚拟机环境准备

设置桥接网卡，配置如下：

```
HWADDR=08:00:27:98:FD:06   #mac地址修改
TYPE=Ethernet
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=static
IPADDR=10.20.0.253      #地址修改  
NETMASK=255.255.252.0
GATEWAY=10.20.0.1
DNS=10.20.26.12
DNS1=10.20.26.13
```

#### 安装nginx软件

```
#安装依赖
yum install -y openssl openssl-devel pcre pcre-devel 
rpm -aq openssl openssl-devel pcre pcre-devel

#安装nginx
mkdir -p /home/dnzhu/tools
cd /home/dnzhu/tools
wget http://nginx.org/download/nginx-1.6.3.tar.gz
useradd nginx -s /sbin/nologin -M
tar xvf nginx-1.6.3.tar.gz
cd nginx-1.6.3
./configure --user=nginx --group=nginx --prefix=/opt/application/nginx-1.6.3 --with-http_stub_status_module --with-http_ssl_module
make
make install
ln -s /opt/application/nginx-1.6.3 /opt/application/nginx

#修改配置文件nginx.conf

#创建虚拟主机目录
mkdir /opt/application/nginx/html/{www,bbs}
#创建测试文件
vim /opt/application/nginx/html/www/index.html 
vim /opt/application/nginx/html/bbs/index.html 
#修改hosts
vim /etc/hosts
```

### lb01负载均衡nginx的配置

```
worker_processes  1;
pid  logs/nginx.pid;
error_log  logs/error.log;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    sendfile        on;
    keepalive_timeout  65;
    server_tokens off;
    #定义连接池
    upstream www_server_pools {
        server 10.20.0.254:80 weight=1;
        server 10.20.0.253:80 weight=1;
    }
    server {
        listen      80;
        server_name www.dnzhu.com;
        location / {
            pro_pass http://www_server_pools; #定义负载均衡虚拟主机
            proxy_set_header Host $host; #代理转发请求的时候带上host字段信息
            proxy_set_header X-Forwarded-For $remote_addr; #在转发请求中加上X-Forwarded-For请求头
        }
    
    }
}

```

### 把nginx反向代理参数独立出来

```
vim proxy.conf

proxy_set_header Host $host;
proxy_set_header X-Forwarded-For $remote_addr;
proxy_connect_timeout 60;
proxy_send_timeout 60;
proxy_read_timeout 60;
proxy_buffer_size 4k;
proxy_buffers 4 32k;
proxy_busy_buffers_size 64k;
proxy_temp_file_size 64k;

```
include proxy.conf 到nginx.conf配置文件中。








