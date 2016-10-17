---
layout: post
title: "php源码编译安装"
description: ""
category: linux
tags: [lnmp]

---

### 一.安装环境：

    centos 6.8 
    linux2.6 x86_64
    软件版本： php-5.5.26

### 二.准备所需要的安装包

#### 1.检查安装php所需要的lib库

    rpm -qa zlib-devel libxml2-devel libjpeg-devel libjpeg-turbo-devel libiconv-devel
    rpm -qa freetype-devel libpng-devel gd-devel libcurl-devel libxslt-devel

#### 2.yum安装这些类库

    yum -y install  zlib-devel libxml2-devel libjpeg-devel libjpeg-turbo-devel libiconv-devel
    yum -y install  freetype-devel libpng-devel gd-devel libcurl-devel libxslt-devel

#### 3.源码编译libiconv包

    ps ：默认的yum源里面没有libiconv包，所以需要手动安装。
    cd /home/dnzhu/tools
    wget http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz
    tar zxvf libiconv-1.14.tar.gz
    cd libiconv-1.14
    ./configure --prefix=/usr/local/libiconv
    make
    make install

#### 4.配置阿里云yum源

    wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
    ls -l /etc/yum.repos.d

#### 5.手动安装libmcrypt库

    ps：libmcrypt对于在程序运行时添加/移除算法时用到。在php的编译过程中，不是必须的安装包。默认的yum源里面也没有libmcrypt-devel 。需要配置第三方yum源。上步骤已经配置好了yum源。

    yum -y install libmcrypt-devel

#### 6.手动安装mhash加密扩展库
    
    yum -y install mhash

#### 7.手动安装mcrypt加密扩展库
    
    yum -y install mcrypt


### 三.编译安装php包

#### 1.准备php安装包

    wget http://cn2.php.net/get/php-5.5.26.tar.gz/from/this/mirror
    tar zxvf mirror
    cd php-5.5.26

#### 2.设置编译参数

    ./configure 
    --prefix=/opt/application/php5.5.26 
           --with-mysql=/opt/application/mysql/ 
    --with-pdo-mysql=mysqlnd 
    --with-iconv-dir=/usr/local/libiconv 
    --with-freetype-dir 
    --with-jpeg-dir 
    --with-png-dir 
    --with-zlib 
    --with-libxml-dir=/usr 
    --enable-xml 
    --disable-rpath 
    --enable-bcmath 
    --enable-shmop 
    --enable-sysvsem 
    --enable-inline-optimization 
    --with-curl 
    --enable-mbregex 
    --enable-fpm 
    --enable-mbstring 
    --with-mcrypt 
    --with-gd 
    --enable-gd-native-ttf 
    --with-openssl 
    --with-mhash 
    --enable-pcntl 
    --enable-sockets 
    --with-xmlrpc 
    --enable-soap 
    --enable-short-tags 
    --enable-static 
    --with-xsl 
    --with-fpm-user=nginx 
    --with-fpm-group=nginx 
    --enable-ftp 
    --enable-opcache=no

#### 3.make

    错误一： make ： *** [ext/phar/phar.php] error 127
    ln -s /opt/application/mysql/lib/libmysqlclient.so.18  /usr/lib64
    
    错误二： can not access `ext/phar/phar.phar`:No such file or directory
    touch ext/phar/phar.phar

#### 4.make install

#### 5.配置php.ini

    ln -s /opt/application/php5.5.26 /opt/application/php
    cp /opt/application/php5.5.26/php.ini-development /opt/application/php/lib/php.ini

#### 6.配置php-fpm.conf
    
    cd /opt/application/php/etc/
    ls
    cp php-fpm.conf.default php-fpm.conf

#### 7.启动php-fpm
    
    /opt/application/php/sbin/php-fpm
    ps -ef | grep php-fpm
    lsof -i :9000

### 四.配置nginx支持php程序访问。
    
    以blog配置文件为例。
    vim /opt/application/ngnix/nginx/conf/vhost/blog.conf

    location ~ .*\.(php|php5) ?${
        root html/blog;
        fastcgi_pass  127.0.0.1:9000;
        fastcgi_index index.php
        include fastcgi.conf;
    }

#### 重启nginx服务

    /opt/application/ngnix/nginx/sbin/nginx -t      --check status
    /opt/application/ngnix/nginx/sbin/nginx -s reload       --reload












