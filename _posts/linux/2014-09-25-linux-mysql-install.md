---
layout: post
title: "mysql二进制安装"
description: ""
category: linux
tags: [lnmp]

---

### 安装环境：

    centos 6.8  
    x86_64   
    mysql-5.5.32-linux2.6-x86_64.tar.gz

### 步骤：

ps:软件下载目录：/home/dnzhu/tools/;  自定义安装目录在：/opt/application/mysql

#### 2.下载软件

    cd /home/dnzhu/tools
    wget http://dev.mysql.com/get/Downloads/MySQL-5.5/mysql-5.5.32-linux2.6-x86_64.tar.gz

#### 3.解压安装包并移动到安装目录

    tar xvf mysql-5.5.32-linux2.6-x86_64.tar.gz
    mv mysql-5.5.32-linux2.6-x86_64.tar.gz  /opt/application/mysql-5.5.32


#### 4.创建软链接

    ln -s /opt/application/mysql-5.5.32/  /opt/application/mysql
    ls -l /opt/application/

#### 5.初始化mysql配置文件my.cnf

    /bin/cp support-files/my-small.cnf   /etc/my.cnf

#### 6.创建数据文件目录

    mkdir -p /opt/application/mysql/data

#### 7.修改安装目录所属组和所有者

    chown -R mysql.mysql /opt/application/mysql

#### 8.初始化mysql数据库文件

    /opt/application/mysql/script/mysql_install_db --basedir=/opt/application/mysql --datadir=/opt/application/mysql/data --user=mysql
    ps: 可能会有错误提示，根据错误日志进行错误处理

#### 9.配置并启动mysql

    cp /opt/application/mysql/support-files/mysql.server  /etc/init.d/mysqld
    chmod +x /etc/init.d/mysqld

    #mysql默认安装路径是/usr/local/mysql  ,启动脚本的路径需要替换
    sed -i 's#/usr/local/mysql#/opt/application/mysql#g' /opt/application/mysql/bin/mysqld_safe /etc/init.d/mysqld

#### 10.启动/关闭mysql数据库

    /etc/init.d/mysqld start | stop 
    #后台启动mysql服务
    /opt/application/mysql/bin/mysqld_safe --user=mysql &   

#### 11.检查mysql的启动状态

    netstat -lntup |grep mysql

#### 12.设置mysql服务开机启动

    方法一：
    echo '/etc/init.d/mysql start' >> /etc/rc.local
    方法二：
    chkconfig --add mysqld
    chkconfig mysqld on
    chkconfig --list mysqld

#### 13.配置mysql命令为全局使用路径

    方法一：
    echo $PATH
    PATH=$PATH:/opt/application/mysql/bin
    方法二：
    echo 'export PATH=/opt/application/mysql/bin:$PATH' >> /etc/profile
    tail -1 /etc/profile
    export PATH=/opt/application/mysql/bin:$PATH
    source /etc/profile
    echo $PATH

#### 14.为mysql的root账户设置密码

    mysqladmin -u root password '123456'
    mysql -u root -p123456

#### 15.清除无用的mysql用户

    select user,host from mysql.user;
    delete from mysql.user where user='' or host='::1';












