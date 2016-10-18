---
layout: post
title: "xcache安装配置"
description: ""
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

#### 比较：

**xcache ：**
    xcache效率高，速度快。
    开发社区活跃。
    支持高版本的php，如php5.5,5.6等。

**eAccelerator ：**
    安装及配置参数简单，加速效果不错。
    文档资料多，但社区不是很活跃。
    仅适合php版本5.4以下的程序。

**APC：**
    apc使用spinlocks自旋锁机制，缓存性能不错。
    提供apc.php脚本，能够监控和管理apc缓存。
    apc默认通过mmap匿名映射创建共享内存，缓存对象都存放在这内存区域里。

**zendOpcache：**
    是php官方推出的新一代缓存加速软件，现在的稳定性还有待考验，可以尝试。

#### 推荐使用：xcache -> eAccelerator -> apc -> zendOpcache

### 安装前解决包依赖

    解决部分加速器软件的perl编译问题

    1).配置环境变量 LC_ALL.
        echo 'export LC_ALL=C' >> /etc/frofile
        tail -1 /etc/profile
        source /etc/profile
        echo $LC_ALL
    解决安装eAccelerator 时，提示：setting locale failed .please check that your settings;

    2).安装perl 相关的软件依赖
    yum -y install perl-devel ##需要提前安装perl相关的软件依赖包



### 安装Xcache缓存加速器
**编译安装**

    cd /home/dnzhu/tools
    wget http://xcache.lighttpd.net/pub/Releases/3.2.0/xcache-3.2.0.tar.bz2
    ls  xcache-3.2.0.tar.bz2
    tar xvf xcache-3.2.0.tar.bz2
    cd xcache-3.2.0
    /opt/application/php/bin/phpize
    ./configure --enable-xcache --with-php-config=/opt/application/php/bin/php-config
    make
    make install
    ls -l /opt/application/php5.5.26/lib/php/extensions/no-debug-non-zts--20121212/
    cd ..

    注意：xcache3.2.XX版本，2014年底发布，全面支持php5.1~5.6。



**配置xcache参数**

    1).步骤
        extension=xcache.so
        #激活管理员认证
        [xcache.admin]
        xcache.admin.enable_auth=on
        xcache.admin.user="mOo"
        xcache.admin.pass="md5 encrypted password"
        [xcache]
        xcache.shm_scheme ="mmap"
        #共享内存块的大小，常设置为256m
        xcache.size=60M
        #指定cache切分成多少块，根据cpu的个数来定
        xcache.count=1
        #hash槽个数参考值
        xcache.slots=8k
        #设置数据的缓存周期，0 为永久，常用设置为86400
        xcache.ttl=0
        #回收内存数据间隔时间 ，常设置为3600
        xcache.gc_interval=0
        ;这几个值用于变量缓存，而不是opcode缓存
        xcache.var_size=4m
        xcache.var_count=1
        xcache.var_slots=8k
        xcache.var_ttl=0
        xcache.var_gc_interval=300
        #一般选择关闭，会影响性能
        xcache.readonly_protection=off
        #一般设置为/tmp/xcache这样的路径
        xcache.mmap_path="/dev/zero"
        #当xcache crash 后，把数据保存到指定路径
        xcache.coredump_directory=""
        #当xcache发生crash时，自动关闭xcache。
        xcache.disable_on_crash=off

        以上参数可以根据生产环境以及业务量的大小动态调整。

    2).将配置信息导入php.ini文件中：
        cd /opt/application/php/lib/
        cat /home/dnzhu/tools/xcache-3.2.0/xcache.ini >> php.ini
    
    3).重启php-fpm
        pkill php-fpm  #关闭
        ps -ef | grep php-fpm
        /opt/application/php/sbin/php-fpm  #重启

**内核优化**

    xcache和eaccelerator均使用系统共享内存作为存储空间，因此可以调整系统共享内存大小参数。
    tail /etc/sysctl.conf  内核参数文件，可以根据实际情况对参数进行调整。

    
**配置web界面查看xcache缓存加速信息**

    echo -n "123456" | md5sum
    生成密码：e10adc3949ba59abbe56e057f20f883e

    vim php.ini
    xcache.admin.enable_auth=on
    xcache.admin.user = "dnzhu"
    xcache.admin.pass = "e10adc3949ba59abbe56e057f20f883e"

    复制xcache解压目录的管理程序到php站点目录下：
    cd /home/dnzhu/tools/xcache-3.2.0
    cp -a htdocs/ /opt/application/nginx/html/blog/xadmin
    chown -R nginx.nginx /opt/application/nginx/ngnix/html/blog/xadmin
    pkill php-fpm
    /opt/application/php/sbin/php-fpm

访问地址：http://your-website/xadmin/index.php   根据自己配置的域名进行访问








 


