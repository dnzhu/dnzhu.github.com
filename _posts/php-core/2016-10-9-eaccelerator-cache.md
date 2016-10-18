---
layout: post
title: "php缓存加速器之eaccelerator安装配置"
description: "eaccelerator安装配置"
category: php-core
tags: [cache]
---

### 安装环境
    centos 6.8 linux redhat2.6 x86_64 
    php5.5.26
    nginx1.6.3
    mysql5.5.32

### PHP缓存加速器的种类和选择

    常见的缓存加速器软件有：Xcache ， eAccelerator ， APC  ,ZendOpcache 等。
    上一篇中已经对这几种缓存加速器进行了说明。


### 安装前解决包依赖（如果没有安装）

解决部分加速器软件的perl编译问题

1).配置环境变量 LC_ALL.

    echo 'export LC_ALL=C' >> /etc/frofile
    tail -1 /etc/profile
    source /etc/profile
    echo $LC_ALL

解决安装eAccelerator 时，提示：setting locale failed .please check that your settings;

2).安装perl 相关的软件依赖

    yum -y install perl-devel ##需要提前安装perl相关的软件依赖包

### 安装eAccelerator 插件

**编译安装**

    cd /home/dnzhu/tools
    wget https://github.com/downloads/eaccelerator/eaccelerator/eaccelerator-0.9.6.1.tar.bz2
    ls eaccelerator-0.9.6.1.tar.bz2
    tar xvf eaccelerator-0.9.6.1.tar.bz2
    cd eaccelerator-0.9.6.1
    /opt/application/php/bin/phpize
    ./configure --enable-eaccelerator=shared --with-php-config=/opt/application/php/bin/php-config
    make
    make install
    ls /opt/application/php5.5.26/lib/php/extensions/no-debug-non-zts-20121212/

**注意：**
    
版本问题，php5.3.XX 可用eaccelerator-0.9.6版本，php5.2.XX 需要使用eaccelerator-0.9.5版本。
需要切换到eaccelerator-0.9.6.1目录下执行/opt/application/php/bin/phpize命令，否则会报错。 


**配置参数生效及优化**

(1).创建缓存数据目录。
    
    cd /opt/application/php/lib
    mkdir -p /tmp/eaccelerator
    chown -R nginx.nginx /tpm/eaccelerator
    ls -ld /tmp/eaccelerator/

(2).eaccelerator参数配置
    
    vim /opt/application/php/lib/php.ini
    [eaccelerator]
    extension=eaccelerator.so
    #共享内存为64M
    eaccelerator.shm_size="64"
    #磁盘缓存路径
    eaccelerator.cache_dir="/tmp/eaccelerator"
    #缓存生效开关
    eaccelerator.enable="1"
    #加速器激活开关
    eaccelerator.optimizer="1"
    #检查缓存修改时间
    eaccelerator.check_mtime="1"
    #是否开启调试
    eaccelerator.debug="0"
    #设定对象是否缓存规则，空表示不设定
    eaccelerator.filter=""
    #可以被缓存的最大值，0表示不限制
    eaccelerator.shm_max="0"
    #缓存文件的有效期
    eaccelerator.shm_ttl="3600"
    #共享内存不够时，从内存移除老数据的时间周期
    eaccelerator.shm_prune_period="3600"
    #是否运行数据缓存到磁盘，0为允许
    eaccelerator.shm_only="0"
    #是否开启压缩
    eaccelerator.compress="1"
    #压缩级别，9为最高
    eaccelerator.compress_level="9"
    #根据内容指定是否缓存到内存或者磁盘
    eaccelerator.keys="shm_and_disk"
    eaccelerator.sessions="shm_and_disk"
    eaccelerator.content="shm_and_disk"

(3).检查php配置文件

    /opt/application/php/bin/php -v

(4)重启php服务

    pkill php-fpm
    ps ef | grep php-fpm 
    lsof -i :9000
    /opt/application/php/sbin/php-fpm
    ps ef | grep php-fpm

**使用tmpfs优化eaccelerator缓存目录**

ps：tmpfs 是一种基于内存的文件系统，通过tmpfs作为数据临时存储，比本地磁盘存取要快，适用于临时使用的各类缓存场景。将/tmp/eaccelerator 挂载到tmpfs文件系统上，让访问数据更快一点。
    
    mount -t tmpfs -o size=16m tmpfs /tmp/eaccelerator
    df -h
    grep eaccele /proc/mounts  #检查挂载情况
    tail -1 /etc/fstab
    umount /tmp/eaccelerator
    grep eacce /proc/mounts
    mount -a
    grep eacce /proc/mounts









 


