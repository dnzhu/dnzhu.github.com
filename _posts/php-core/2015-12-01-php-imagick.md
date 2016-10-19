---
layout: post
title: "php扩展安装imagick"
description: "imagick安装配置"
category: php-core
tags: [lnmp]
---

### 概述

    imageMagick是一套功能强大，稳定且免费的图像处理工具，支持超过89中基本格式的图片文件。

### imageMagick常见功能：

    将图片从一个格式转换成另一个格式。
    可以改变图片尺寸，旋转，锐化，减色，设置图片特效。
    对图片设置各种尺寸缩略图。
    将图片设置为可以适用于web背景的透明图片
    将一组图片做成gif动画，直接convert。
    将几张图片合成一张组合图片。
    给图片加边框或者框架。
    取得图片的特征信息。

ps ：需要先安装imageMagick，再安装imagick。

### 安装imageMagick

    cd /home/dnzhu/tools/
    wget -q http://download.chinaunix.net/down.php?id=44041&ResourceID=95&site=1
    tar -xvf ImageMagick-6.7.9-9.tar.xz
    cd ImageMagick-6.7.9-9
    ./configure
    make
    make -k -i install  #以免报错
    cd ..

### 再安装imagick

    cd /home/dnzhu/tools/
    wget -q http://pecl.php.net/get/imagick-3.3.0.tgz
    tar xvf imagick-3.3.0.tgz
    cd imagick-3.3.0
    /opt/application/php/bin/phpize
    ./configure --with-php-config=/opt/application/php/bin/php-config
    make
    make install
    cd ..

### 配置生效
    插件安装完成以后，修改php.ini文件。
    修改：
    extension_dir =“/opt/application/php5.5.26/lib/php/extensions/no-debug-non-zts-20121212”
    增加：
    extension=imagick.so

### 重启php-fpm

    pkill php-fpm
    ps -ef | grep php-fpm
    /opt/application/php/sbin/php-fpm




