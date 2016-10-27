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

配置文件的格式参照（/opt/application/mysql5.5.32/support-files/my-small.cnf）
vim /data/3306/my.cnf

```
[client]
port        = 3306
socket      =/data/3306/mysql.sock #这个要变

[mysql]
no-auto-rehash

[mysqld]
user        =mysql
port        = 3306
socket      =/data/3306/mysql.sock #
basedir     =/opt/application/mysql #
datadir     =/data/3306/data #
open_files_limit= 1024
back_log    = 600
max_connections = 800
max_connect_errors = 3000
table_cache = 64
external-locking = FALSE
max_allowed_packet =8M
sort_buffer_size = 1M
join_buffer_size = 1M
thread_cache_size = 100
thread_concurrency = 2
query_cache_size = 2M
query_cache_limit = 1M
query_cache_min_res_unit = 2k
thread_stack = 192K
tmp_table_size = 2M
max_heap_table_size = 2M
long_query_time = 1
pid-file    =/data/3306/mysql.pid #
relay-log   =/data/3306/relay-bin #
relay-log-info-file = /data/3306/relay-log.info #
binlog_cache_size = 1M
max_binlog_cache_size = 1M
max_binlog_size = 2M
max_allowed_packet = 1M
key_buffer_size = 16M
read_buffer_size = 1M
read_rnd_buffer_size = 1M
bulk_insert_buffer_size = 1M
lower_case_table_names = 1
skip-name-resolve
slave-skip-errors =1032,1062
replicate-ignore-db=mysql
server-id   = 1
innodb_additional_mem_pool_size = 4M
innodb_buffer_pool_size = 32M
innodb_data_file_path = ibdata1:128M:autoextend
innodb_file_io_threads  =4
innodb_thread_concurrency = 8
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_log_file_size = 4M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout =120
innodb_file_per_table = 0

[mysqldump]
quick
max_allowed_packet = 16M


[mysqld_safe]
log-error =/data/3306/mysql_dnzhu.err #
pid-file=/data/3306/mysqld.pid #

```
这是3306端口服务的配置文件，3307端口的配置文件一样，只需要把所有的**3306替换成3307**,server-id换一个值。

```
vim /data/3307/my.cnf 
:% s/3306/3307/g

```

#### 5.启动/停止mysql服务进程

    启动3306：mysqld_safe --defaults-file=/data/3306/my.cnf 2>&1 > /dev/null &
    启动3307：mysqld_safe --defaults-file=/data/3307/my.cnf 2>&1 > /dev/null &
    停止3306：mysqladmin -uroot -S /data/3306/mysql.sock shutdown
    停止3307：mysqladmin -uroot -S /data/3307/mysql.sock shutdown

#### 查看mysql进程

```
netstat -lntup | grep 330
```

#### 5.设置文件权限

    授权mysql用户和组管理mysql多实例的根目录 /data
    chown -R mysql.mysql /data
    find /data -name mysql | xargs ls -l

    修改启动文件的权限为700.启动文件中有管理员账号，密码，所以权限最小化。
    find /data -name mysql | xargs chmod 700
    find /data -name mysql -exec ls -l {} \;

#### 6.设置mysql命令的全路径

```
PATH=/opt/application/mysql/bin:$PATH
echo $PATH
```
ps:因mysql环境变量配置顺序导致的错误，务必将mysql的命令路径放在其他路径前面。

#### 7.初始化数据库文件

```
cd /opt/application/mysql/scripts
./mysql_install_db --basedir=/opt/application/mysql --datadir=/data/3306/data --user=mysql
./mysql_install_db --basedir=/opt/application/mysql --datadir=/data/3307/data --user=mysql

```

#### 8.配置mysql进程快速启动脚本
vim /data/3306/mysql

```
#!/bin/sh
#mysql multi config
###############################
#init
port=3306
mysql_user = "root"    #实际连接mysql数据库的账户，密码
mysql_pwd = ""
bindir = "/opt/application/mysql/bin"
mysql_sock = "/data/${port}/mysql.sock"

#start up function
function_start_mysql()
{
  if [ ! -e "$mysql_sock" ];then
    printf "starting MYSQL...\n"
    /bin/sh ${bindir}/mysql_safe --defaults-file=/data/${port}/my.cnf 2>&1 > /dev/null &
  else
    printf "mysql is running ...\n"
    exit
  fi
}

#mysql stop function
function_stop_mysql()
{
  if [ ! -e "$mysql_sock" ];then
    printf "mysql is stopped ..."
    exit
  else
    printf "stoping mysql ..."
    ${bindir}/mysqladmin -u ${musql_user} -p${mysql_pwd} -S /data/${port}/mysql.sock shutdown
  fi
}

## restart mysql function
function_restart_mysql()
{
  printf "restarting mysql ...\n"
  function_stop_mysql
  sleep 2
  function_start_mysql

}

case $1 in 
start)
  function_start_mysql
;;
stop)
  function_stop_mysql
;;
restart)
  function_restart_mysql
;;
*)
    printf "usage : /data/${port}/mysql{start|stop|restart}\n"
esac

```
快捷启动命令：/data/3306/mysql start | stop | restart

ps ： 如果需要设置开启开机启动，将启动命令写入"/etc/rc.local"文件中。

####　给root账户设置密码

默认安装的mysql的root账户没有设置密码。

```
mysql -S /data/3306/mysql.sock #无密码
mysql -S /data/3307/mysql.sock #直接登录
```
**设置密码**

```
mysqladmin -u root -S /data/3306/mysql.sock password 'mark123'
mysql -u root -pmark123 -S /data/3306/mysql.sock
```

### 新增一个mysql进程

```
mkdir -p /data/3308/data
cp /data/3306/my.cnf /data/3308/
cp /data/3306/mysql /data/3308/

#修改配置文件
sed -i 's/3306/3308/g' /data/3308/my.cnf
sed -i 's/server-id = 1/server-id = 8/g' /data/3308/my.cnf
#sed -i 's/3306/3308/g' /data/3308/mysql

#修改文件的所有者和所属组以及权限最小化
chown -R mysql:mysql /data/3308
chmod 700 /data/3308/mysql

#初始化数据库文件
cd /opt/application/mysql/scripts
./mysql_install_db --basedir=/opt/application/mysql --datadir=/data/3308/data --user=mysql
chown -R mysql:mysql /data/3308

#启动mysql服务
mysqld_safe --defaults-file=/data/3308/my.cnf 2>&1 > /dev/null &
#或者快捷方式
/data/3308/mysql start  #关键是mysql启动文件设置ok

#查看
netstat -lnupt
```

