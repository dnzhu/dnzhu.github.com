---
layout: post
title: "centos6.8自定义安装"
description: ""
category: linux
tags: [lnmp]

---

### 安装软件版本：
    centos 6.8

### 安装准备
    1.下载安装镜像文件
    http://www.centos.org   ->downloads->mirrors  
    http://mirrors.aliyun.com/centos/6.8/isos/x86_64/
    http://mirrors.aliyun.com/centos/6.8/isos/i386/
    主要下载Centos-6.8-x86_64-bin-DVD1.iso和Centos-6.8-x86_64-bin-DVD2.iso

### 安装centos6.8

#### 1.选择系统引导方式
    选择 install or upgrade an existing system

#### 2. 检查安装光盘介质
    选择：skip

#### 3. 选择安装过程语言
    选择：english

#### 4. 选择键盘布局
    选择：U.S.English

#### 5.选择合适的物理设备
    选择：basic storage devices

#### 6.初始化硬盘提示
    选择：yes ,discard and data

#### 7.初始化主机名以及网络配置
    （1）.为系统设置主机名  主机名为：dnzhu
    （2）.配置网卡及连接网络（可选）

#### 8.系统时钟及时区
    选择：Asia/Shanghai
    取消：system clock uses UTC  
    然后：next

#### 9.设置root口令

### 磁盘分区类型选择与磁盘分区配置过程

#### 1.选择系统安装磁盘空间类型
    选择：create custom layout

#### 2.进入“create custom layout”分区界面
    可以create (创建),update(修改) ,delete(删除)等操作。

#### 3.按企业生产标准定制磁盘分区
    选择：standard partition 
    1）.创建引导分区，/boot分区
    mount point:/boot
    file system type:ext4
    size:200 

    2）.创建swap交换分区
    mount point :<not applicable>
    file system type:swap
    size:1024 (物理内存的1-2倍)
    addtion size options : fixed size
    force to be a primary partition

    3).创建( / )根分区
    mount point :/
    file system type : ext4
    size : 剩余
    addtion size options : fill to maximum allowable size  (根分区是最后一个分区，所以把剩余的空间都分配给根分区)
    force to be a primary partition 

    4).格式化警告
    选择： format

### 系统安装包的选择与配置

#### 1.启动引导设备的配置
    系统默认使用GRUB作为启动加载器，引导程序默认在MBR下：
    install boot loader on /dev/sda ->change device ->master boot record -/dev/sda

#### 2.系统安装类型选择及自定义额外包组
    系统默认是desktop ，但是这里选择minimal。
    自定义安装包选择：customsize now 
    base system :
        base ,compatibility libraries  , debugging tools
    development : 
        development tools
    然后：next

### 开始安装->安装完成->reboot

### 系统安装后的配置

#### 1.更新系统，打补丁到最新
    修改更新yum源：
    cp  /etc /yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.ori
    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirros.163.com/.help/CentOS 6-Base-163.repo
    ll /etc/pki/rpm-gpg/
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY*
    yum update -y
    
    ps:一般在首次安装时执行yum update -y ,如果是在实际生产环境中，切记使用，以免导致异常。

#### 2.安装额外的软件包
    yum install tree telnet dos2unix sysstat lrzsz nc nmap -y
    yum grouplist    #查看包组列表
    yum frouplist "development Tools"






