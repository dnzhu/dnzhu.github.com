---
layout: post
title: "mysql多实例安装配置"
description: ""
category: mysql
tags: [mysql]
---

### 什么是mysql多实例？

    mysql多实例就是在一台服务器上同时开启多个不同的服务器端口，同时运行多个mysql进程，这些进程通过不同的socket监听不同的服务器端口来提供服务。

### mysql多实例的优与劣？

    有效的利用服务器资源，能够最大限度的使用服务器性能。
    节约资源，如果公司资金紧张，但是数据库有需要各自尽量独立的提供服务，就能使用mysql多实例。
    物极必反，单台服务器的mysql进程不能太多，不然会消耗服务器性能，导致整体效率下降。
    一般情况下，单机2-4个mysql进程是合适的。也可以根据服务器的性能，进行调整。

### mysql多实例的配置方案


#### 1.安装mysql包依赖

```
yum install ncurses-devel libaio-devel -y
rpm -qa ncurses-devel libaio-devel
```

#### 2.安装cmake

```
cd /home/dnzhu/tools
wget  https://cmake.org/files/v2.8/cmake-2.8.8.tar.gz
tar xvf cmake-2.8.8.tar.gz
cd cmake-2.8.8
./configure
gmake
gmake install
which cmake
```

#### 3.源码编译安装mysql

ps : 源码包->**mysql-5.5.32.tar.gz**，二进制包->**mysql-5.5.32-linux2.6-x86_64.tar.gz**

**先创建mysql用户和组**

    useradd -s /sbin/nologin -M mysql
    id mysql # 查看是否创建成功

**编译安装**

```
cd /home/dnzhu/tools
wget http://dev.mysql.com/get/Downloads/MySQL-5.5/mysql-5.5.32.tar.gz
tar xvf mysql-5.5.32.tar.gz
cd mysql-5.5..32
cmake . -DCMAKE_INSTALL_PREFIX=/opt/application/mysql-5.5.32 \
-DMYSQL_DATAADIR=/opt/application/mysql-5.5.32/data \
-DMYSQL_UNIX_ADDR=/opt/application/mysql-5.5.32/tmp/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DEXTRA_CHARSETS=gbk,gb2312,utf8,ascii \
-DENABLED_LOCAL_INFILE=ON \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITHOUT_EXAMPLE_STORAGE_ENGINE=1 \
-DWITHOUT_PARTITION_STORAGE_ENGINE=1 \
-DWITH_FAST_MUTEXES=1 \
-DWITH_ZLIB=bundled \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_READLINE=1 \
-DWITH_EMBEDDED_SERVER=1 \
-DWITH_DEBUG=0 \
make
make install

ln -s /opt/application/mysql-5.5.32/  /opt/application/mysql #创建软连接文件
ls -l /opt/application/
```

#### 4.创建多实例数据文件及目录

    mkdir -p /data/{3306,3307}/data
    tree /data/

**创建配置文件**

```
cp /opt/application/mysql5.5.32/support-files/my-innodb-heavy-4G.cnf /data/3306/my.cnf
cp /opt/application/mysql5.5.32/support-files/my-innodb-heavy-4G.cnf /data/3307/my.cnf
vim /data/3307/my.cnf   #将所有的3306替换成3307
```

**创建启动文件**
mysql 启动文件内容，参考/opt/application/mysql/support-files/mysql.server

```
vim /data/3306/mysql
vim /data/3307/mysql
```

#### 5.启动/停止mysql服务进程

    启动3306：mysqld_safe --defaults-file=/data/3306/my.cnf 2>&1 > /dev/null &
    启动3307：mysqld_safe --defaults-file=/data/3307/my.cnf 2>&1 > /dev/null &
    停止3306：mysqladmin -uroot -pmark123 -S /data/3306/mysql.sock shutdown
    停止3307：mysqladmin -uroot -pmark123 -S /data/3307/mysql.sock shutdown

#### 5.设置文件权限

    授权mysql用户和组管理mysql多实例的根目录 /data
    chown -R mysql.mysql /data
    find /data -name mysql | xargs ls -l

    修改启动文件的权限为700.启动文件中有管理员账号，密码，所以权限最小化。
    find /data -name mysql | xargs chmod 700

#### 6.设置mysql命令的全路径

```
PATH=/opt/application/mysql/bin:$PATH
echo $PATH
```
ps:因mysql环境变量配置顺序导致的错误，务必将mysql的命令路径放在其他路径前面。

#### 7.初始化数据库

```
cd /opt/application/mysql/scripts
./mysql_install_db --basedir=/opt/application/mysql --datadir=/data/3306/data --user=mysql
./mysql_install_db --basedir=/opt/application/mysql --datadir=/data/3307/data --user=mysql

```

#### 8.启动多实例mysql

```
/data/3306/mysql start
/data/3307/mysql start

netstat -lntup | grep 330
```


