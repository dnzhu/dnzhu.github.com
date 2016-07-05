---
layout: post
title: "ubuntu和centos网络配置"
description: ""
category: linux
tags: []
---

## ubuntu网络设置

> Ubuntu系统进行网络配置涉及到几个配置文件1./etc/network/interfaces   2./etc/resolv.conf.

#### 1.编辑/etc/network/interfaces 文件
```
#默认设置
auto lo
iface lo inet loopback
动态获取的配置方法：
auto eth0
iface eth0 inet dhcp
静态分配的配置方法：
auto eth0
iface eth0 inet static
address 192.168.0.1
netmask  255.255.255.0
gateway  192.168.0.1
```

#### 2.编辑/etc/resolv.conf文件
```
#设置域名服务器
nameserver 114.114.114.114
```

#### 3.重启网卡
```
/etc/init.d/networking restart  #重启
或者：
ifdown eth0
ifup  eth0
```

## centos网络设置

> centos 系统进行网络配置涉及到几个配置文件1./etc/sysconfig/network-scripts/ifcfg-eth0  2./etc/resolv.conf .

#### 1.vim /etc/sysconfig/network-scripts/ifcfg-eth0
```
DEVICE=eth0
BOOTPROT=static
IPADDR=192.168.1.100
GATEWAY=192.168.1.1
NETMASK=255.255.255.0
ONBOOT=yes
```
```
DEVICE= 表示物理设备的名字
ONBOOT= yes表示系统启动时激活该设备，no表示不激活
BOOTPROTO= 取值可以是static(静态配置)、bootp(使用bootp协议)、dhcp(使用dhcp协议)
BROADCAST= 表示广播地址
IPADDR= 表示该网卡的IP地址
PREFIX= 子网掩码
GATEWAY=表示网关
DNS*=表示DNS
```

#### 2.vim /etc/resolv.conf
```
添加dns
nameserver 192.168.1.1
```

#### 3.重启网卡
```
service network restart
```

#### 4. 检查网络
```
ifconfig    #显示网络连接信息
ping 192.168.1.100  是否能ping通

```