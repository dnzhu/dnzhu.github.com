---
layout: post
title: "php扩展安装memcache"
description: "memcache安装配置"
category: php-core
tags: [cache,lnmp]
---

### 概述

    memcached是一个开源，高性能，高并发，分布式的内存缓存服务软件。
    memcache分为服务器端和客户端，服务器端软件名字如memcached-1.4.13.tar.gz,客户端软件名字如：memcache-2.2.7.tar.gz

### 安装memcache客服端程序

**插件安装参数：**

    cd /home/dnzhu/tools/
    wget -q http://pecl.php.net/get/memcache-2.2.7.tgz
    tar zxvf memcache-2.2.7.tgz
    cd memcache-2.2.7
    /opt/application/php/bin/phpize
    ./configure --enable-memcache --with-php-config=/opt/application/php/bin/php-config
    make 
    make install
    cd ..
    ls -l /opt/application/php5.5.26/lib/php/extensions/no-debug-non-zts-2012121

**配置生效**

    插件安装完成以后，修改php.ini文件。
    修改：extension_dir =“/opt/application/php5.5.26/lib/php/extensions/no-debug-non-zts-20121212”
    增加：
    extension=memcache.so

**重启php-fpm**
    
    pkill php-fpm
    ps -ef | grep php-fpm
    /opt/application/php/sbin/php-fpm




