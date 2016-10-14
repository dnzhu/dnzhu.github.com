---
layout: post
title: "nginx安装"
description: ""
category: linux
tags: []

---

### 安装环境：

centos 6.8 

查看版本：uname -r     uname -m

软件版本：nginx-1.6.3.tar.gz

---

### 步骤：

    1.设置好yum 源。

    2.先安装pcre和pcre-devel。

```
执行：yum install -y pcre pcre-devel
查看：yum -qa pcre pcre-devel

```
    3.安装openssl和openssl-devel。

```
执行：yum install -y openssl openssl-devel
查看：yum -qa openssl openssl-devel
```

    4.安装nginx。

```
mkdir -p /home/dnzhu/tools
cd /home/dnzhu/tools
wget http://nginx.org/download/nginx-1.6.3.tar.gz
ls -l nginx-1.6.3.tar.gz
useradd nginx -s /sbin/nologin -M      #设置用户和组
tar -xf nginx-1.6.3.tar.gz
cd nginx-1.6.3
./configure --user=nginx --group=nginx --prefix=/opt/application/nginx-1.6.3/ --with-http_stud_status_module --with-http_ssl_module
make
make install
ln -s /opt/application/nginx-1.6.3 /opt/application/nginx    #创建软链接

```

    5.启动并检查安装结果

    检查语法： /opt/application/nginx/sbin/nginx -t
    启动： /opt/application/nginx/sbin/nginx
    查看：(1).lsof -i:80   .(2).wget 127.0.0.1  (3).curl 127.0.0.1  (4).ps -ef |grep nginx
    重启：/opt/application/nginx/sbin/nginx   -s    reload 





