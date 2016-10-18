---
layout: post
title: "php缓存加速器zendOpcache安装"
description: "安装配置"
category: php-core
tags: [cache]
---

### 安装环境
    centos 6.8 linux redhat2.6 x86_64 
    php5.5.26
    nginx1.6.3
    mysql5.5.32

### 安装前解决包依赖(如果已解决请忽略)
解决部分加速器软件的perl编译问题

1).配置环境变量 LC_ALL.

    echo 'export LC_ALL=C' >> /etc/frofile
    tail -1 /etc/profile
    source /etc/profile
    echo $LC_ALL

解决安装eAccelerator 时，提示：setting locale failed .please check that your settings;

2).安装perl 相关的软件依赖

    yum -y install perl-devel ##需要提前安装perl相关的软件依赖包



### 安装zendOpcache插件

**源码编译安装**

    cd /home/dnzhu/tools
    wget -q http://pecl.php.net/get/zendopcache-7.0.5.tgz
    ls zendopcache-7.0.5.tgz
    tar xvf zendopcache-7.0.5.tgz
    cd zendopcache-7.0.5
    /opt/application/php/bin/phpize
    ./configure --enable-opcache --with-php-config=/opt/application/php/bin/php-config
    make
    make install
    ls -l /opt/application/php5.5.26/lib/php/extensions/no-debug-non-zts--20121212/
    cd ..


**配置参数**

    vim /opt/application/php/lib/php.ini
    [opcache]
    zend_extension=/opt/application/php5.5.26/lib/extensions/no-debug-non-zts-20121212/opcache.so
    ;extension=opcache.so   #这种方式不起作用了
    #共享内存空间大小，默认是64m,常用256m
    opcache.memory_consumption=64
    #字符串存储大小默认是4m，常用8m
    opcache.interned_strings_buffer=4
    #散列表中key的最大数量
    opcache.max_accelerated_files=2000
    #检查文件时间戳的频率，默认2，常用60
    opcache.revalidate_freq=2
    #一个快速关闭队列，默认0 ，设置为1
    opcache.fast_shutdown=0
    #默认值0，激活php cli的opcache，用于测试。
    opcache.enable_cli=1

**检查，重启php-fpm**

    /opt/application/php/bin/php -v
    pkill php-fpm
    ps -ef | grep php-fpm
    /opt/application/php/sbin/php-fpm







 


