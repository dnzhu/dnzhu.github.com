---
layout: post
title: "mongdb linux下安装步骤"
description: ""
category: mongodb
tags: []
---

> MongoDB提供了linux平台上32位和64位的安装包,如果有需要的同学可以自己去[下载](http://www.mongodb.org/downloads)相应的版本。

### 1.下载安装包

地址：http://www.mongodb.org/downloads

---

### 2.解压拷贝到指定目录

```
tar -zxvf mongodb-linux-x86_64-ubuntu1604-3.2.8.tgz  #解压

mv ./ /usr/local/mongodb/   #从当前目录拷贝到usr目录

```

---

### 3.添加环境变量

MongoDB 的可执行文件位于 bin 目录下，所以可以将其添加到 PATH 路径中：

```
export PATH=\<mongodb-install-directory\>/bin:$PATH
```
\<mongodb-install-directory\> 为你 MongoDB 的安装路径。我的是在 /usr/local/mongodb 。

---

### 4.创建数据目录

MongoDB的数据存储在data目录的db目录下，但是这个目录在安装过程不会自动创建，所以你需要手动创建data目录，并在data目录中创建db目录。

**注意：/data/db 是 MongoDB 默认的启动的数据库路径(--dbpath)。**

```
mkdir -p /data/db
```

---

### 5.命令行运行mongod 服务

执行安装目录下有bin目录中的mongod命令，来运行mongodb服务。

```
./mongod
```
    如果你的数据库目录不是/data/db，可以通过 --dbpath 来指定。

```
>./mongod --dbpath=/var/data/db   #指定到var目录中

>jobs   #查看mongod 的进程号

> bg 进程号   #将mongod的进程切换到后台运行

```
---

### 6.运行mongo后台管理shell

执行安装目录下有bin目录中的mongo命令，来运行mongo后台管理shell。

```
./mongo

show dbs  #list database

use test  #创建或切换数据库

db.test.find()  #查询
……

```

---

### 7.mongodb web 用户界面

MongoDB 提供了简单的 HTTP 用户界面。 如果你想启用该功能，需要在启动的时候指定参数 --rest 。

```
./mongod --dbpath=/data/db --rest
```
    MongoDB 的 Web 界面访问端口比服务的端口多1000。

    如果你的MongoDB运行端口使用默认的27017，你可以在端口号为28017访问web用户界面，即地址为：http://localhost:28017。

---