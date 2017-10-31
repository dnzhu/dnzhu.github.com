---
layout: post
title: "mysql忘记密码"
description: ""
category: mysql
tags: [mysql]
---

### 问题是？

    一般情况下，在本地环境搭建的mysql账号密码，一段时间以后总会忘记，如果忘记了mysql的密码，是不是就需要重新安装呢？答案当然是“不需要”。那么如何绕过验证，去登录mysql呢 ？

----

### 1.修改mysql的配置文件

```
 
vi /etc/my.cnf
[mysqld] 
datadir=/var/lib/mysql 
socket=/var/lib/mysql/mysql.sock 
skip-grant-tables   #增加选项

```

ps ： 在windows环境下，找到mysql.ini文件，在[mysqld] 后面增加 “skip-grant-tables” 选项。

----

### 2.重启mysql。

```
/etc/init.d/mysqld restart   #重启     （start-启动，stop-停止）

```

----

### 3.登录并修改MySQL的root密码

```
/usr/bin/mysql 
use mysql;
show tables;
UPDATE user SET Password = password ( 'new-password' ) WHERE User = 'root' ;
flush privileges ;

```

----


### 4.恢复mysql配置文件，并重启。

----




