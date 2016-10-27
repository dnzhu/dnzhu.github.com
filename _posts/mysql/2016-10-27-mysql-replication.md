---
layout: post
title: "mysql主从复制从原理到实践"
description: ""
category: mysql
tags: [mysql]
---

> mysql主从复制并不是数据库从磁盘上文件直接拷贝，而是通过mysql的binlog日志复制到要同步的服务器本地，然后在本地的mysql线程读取日志里面的sql语句，重新应用到mysql数据库中。

---

### 常见模式

**单向复制**

适用于 1主1从，或者1主多从场景。

**双向复制**

两台服务器互为主从。

**链式级联复制**

A->B->C的复制形式。

**环状复制**

环状级联单向多主同步，任意一太服务器都可以写入数据，此属于极端情况下的架构，一般情况慎用。

---

ps： 在生产环境中，mysql主从复制都是异步的，不是严格实时的数据同步，但是用户体验是实时的。

---

### 主从复制应用场景


* 1.从服务器作为主服务器的实时数据备份

    为了增强架构的健壮性，当主服务宕机时，自动切换到从服务器上。维持程序的稳定性。

* 2.主从复制实现读写分离，从服务器实现负载均衡

    主从服务器可以通过**程序**来实现主从读写分离，更新数据的业务在主库上进行操作，查询的请求去从库查询。

* 3.把多个从服务器根据业务重要性进行拆分

    比如，有为供外部访问的web服务器，也有内部DBA用来数据备份的从服务器，还有公司内部oa系统的服务器等，分离开后，降低单服务器的访问压力。也不至于影响整体的业务。

---

### 主从复制原理

mysql主从复制是一个异步的过程，数据从一台mysql服务器(称为master)，复制到另一太mysql服务器(称为slave)上，在master和slave之间实现整个主从复制的过程是由3个线程参与完成，其中2个线程(sql线程和I/O线程)在slave端，另一个线程(sql线程)在master端。

必须要在master端，开启binlog记录功能，然后再在slave端以相同的顺序执行binlog中所记录的各种sql操作。


**过程描述**

* 1）在slave端执行start slave 命令，开始主从复制。
* 2）slave 服务器I/O线程会通过在master上已经授权的用户权限请求连接master服务器，并请求从指定binlog日志指定位置之后开始发送binlog日志内容。
* 3）master上收到slave端的I/O请求后，会根据请求的信息分批读取指定的binlog日志，然后返回给slave端的I/O 线程。并记录读取过后新的位置。
* 4）当slave端收到数据以后，会将binlog日志一次写入在slave端自身的relay log 文件的最末端，并将新的binlog文件名和位置记录到master-info文件中，以便下次从新的位置开始读取。
* 5）slave端的sql进程会实时检测本地的relay log 的内容，然后把更新的内容解析成sql语句，并在自身slave服务器上按照顺序执行sql语句，并在relay-log.info中记录当前应用中继日志的文件名和位置。


## 主从复制实战

### 安装环境

centos6.8 mysql5.5.32 单机数据库多实例的安装配置。
        
    主库[master]  : ip [10.20.0.254] port [3306]
    主库[slave]  :  ip [10.20.0.254] port [3307]
    主库[slave]  :  ip [10.20.0.254] port [3308]

---

### master上操作配置

#### 设置server-id 并开启binlog功能

```
#修改配置
vim /data/3306/my.cnf 
[mysqld]
server-id = 1
log-bin = /data/3306/mysql-bin

#重启mysql服务3306端口
mysqladmin -u root -pmark123 -S /data/3306/mysql.sock shutdown  
mysqld_safe --defaults-file =/data/3306/my.cnf 2>&1 > /dev/null &

#登录数据库
mysql -u root -pmark123 -S /data/3306/mysql.sock 
show variables like 'server_id' ;
show variables like 'log_bin' ;
```

#### 在主库上建立主从复制账户

```
#登录数据库
mysql -u root -pmark123 -S /data/3306/mysql.sock 
grant replication slave on *.* to 'rep'@'10.20.0.%' identified by 'mark123';
flush privileges;

#检查rep账户
select suer,host from mysql.user;
show grants for rep@'10.20.0.%';

```

#### 实现对主库琐表只读

所表是为了停止主库写入，然后导出主库的数据，再解锁。

```
#锁表
flush table with read lock;

#查看master状态
show master status;

#新开ssh窗口，导出数据
mkdir -p /server/backup/
mysqldump -uroot -p'mark123' -S /data/3306/mysql.sock --events -A -B |gzip >/server/backup/mysql_bak.$(date +%F).sql.gz
ls -l /server/backup
show master status
unlock tables;
```

#### 主库数据迁移到从库服务器上

常用同步命令scp，rsync等，可以将备份数据同步到其它服务器上，我测试的demo就在一台机器上，不需要同步了。

---

### 在mysql从库上执行的操作

#### 设置server-id 并关闭binlog功能

```
vim /data/3307/my.cnf
[mysqld]
server-id = 3
#log-bin = /data/3307/mysql-bin

#重启服务
mysqladmin -u root -pmark123 -S /data/3307/mysql.sock shutdown  
mysqld_safe --defaults-file =/data/3307/my.cnf 2>&1 > /dev/null &

#登录数据库，查看参数
mysql -u root -pmark123 -S /data/3307/mysql.sock 
show variables like 'server_id' ;
show variables like 'log_bin' ;
```

#### 把从主库中备份的数据恢复到从库

```
cd /server/backup
gzip -d mysql_bak.2016-10-27.sql.gz
ll
mysql -uroot -p'mark123' -S /data/3307/mysql.sock < mysql_bak.2016-10-27.sql
```

#### 登录从库，配置主从复制参数

```
mysql -uroot -p 'mark123' -S /data/3307/mysql.sock <<EOF
CHANGE MASTER TO
MASTER_HOST='10.20.0.254',
MASTER_PORT=3306,
MASTER_USER='rep',
MASTER_PASSWORD='mark123',
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=1613;
EOF
```
PS:字符串单引号括起来，数值不用引号，内容前后不能有空格。
这一步操作主要是把用户密码等信息写入到从库的master.info文件中。

```
cat /data/3307/data/master.info
```
---

### 启动从库同步开关，测试主从复制配置

#### 启动同步开关

```
mysql -uroot -p'mark123' -S /data/3307/msyql.sock -e "start slave;"
mysql -uroot -p'mark123' -S /data/3307/msyql.sock -e "show slave status\G;"

```

#### 检查配置参数

```
mysql -uroot -p'mark123' -S /data/3307/msyql.sock -e "show slave status\G" | egrep "IO_Running|SQL_Running|Behind_Master"
```
**参数说明**

* Slave_IO_Running:Yes,这个是I/O线程状态，负责从主库读取binlog日志，并写入中继文件。
* Slave_SQL_Running:Yes,这个是sql线程状态，负责读取中继日志中的数据并转换成sql执行。
* Second_Behind_Master:0 ,这个是主从复制从库比主库延迟的秒数。

#### 测试主从复制

```
#master
    mysql -uroot -p'mark123' -S /data/3306/msyql.sock -e "create database dnzhu;"
#slave
    mysql -uroot -p'mark123' -S /data/3307/msyql.sock -e "show databases like 'dnzhu';"
```
---

### 实际生产中，轻松部署mysql主从复制

```
#凌晨定时备份主库数据
mysqldump -uroot -pmark123 -S /data/3306/mysql.sock -A --events -B -x --master-data=1|gzip >/opt/backup/$(date +%F).sql.gz

#白天在需要同步的从库上导入数据
gzip -d 2016-10-27.sql.gz
mysql -uroot -pmark123 -S /data/3308/mysql.sock <2016-10-27.sql
mysql -uroot -pmark123 -S /data/3306/mysql.sock <<EOF
CHANGE MASTER TO
MASTER_HOST='10.20.0.254',
MASTER_PORT=3306,
MASTER_USER='rep',
MASTER_PASSWORD='mark123',
EOF

#start
mysql -uroot -pmark123 -S /data/3306/mysql.sock -e "start slave;"
mysql -uroot -pmark123 -S /data/3306/mysql.sock -e "show slave status\G;"
```

