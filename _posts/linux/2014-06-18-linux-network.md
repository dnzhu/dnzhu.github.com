---
layout: post
title: "linux性能监控和网络配置"
description: ""
category: linux
tags: []
---

> linux 服务器的运行状况常常是我们所担心的，现在有很多监控软件，能够检测服务器的运行情况，我们作为后端开发人员，也要熟练监控服务器的常用命令，保证服务器稳定才能保证程序的正常运行。


### (1).uptime 监控cpu的使用情况

---
打印当前时间，系统已经运行了多长时间，当前登录系统的用户数以及系统最近1分钟，5分钟，15分钟的负载情况。

---

### (2).free 监控内存以及交换分区使用情况

---

有如下几个显示信息：

* total       系统内存总容量
* userd       系统已使用的内存容量(分为buffer和cache)
* free        剩余内存容量
* shared      共享的内存容量
* buffers     缓冲区
* cached      缓存区

tips: linux在开机后会预先提取一部分内存，并划分为buffer和cache以便随时提供给进程使用。

---

### (3).df 监控磁盘使用情况

```
参数：
    -h  人性化的显示容量信息。
    -i  显示磁盘的inode使用信息。
    -T  显示文件系统类型。

demo ：
    df -hT      #显示容量的详细信息。

    df -i       #显示容量的inode使用情况。
```

### (4).ps 查看当前进程

```
标准语法格式：
    ps  -e      #查看所有的进程信息
    ps  -ef     #全格式显示进程信息
```

### (5).top 动态的查看进程

```
选项：
    -d      刷新间隔，默认3秒。
    -p      查看指定的PID进程信息。
demo ：
    top -d 1      #1秒刷新一次。
    top -d 1 -p 1,2     #显示PID为1,2的进程信息。     
```

### (6).ifconfig 显示或设置网卡的接口信息

格式：

#### ifconfig interface 选项

```
demo ：
    ifconfig eth0 192.168.10.1 netmask 255.255.255.0
    #设置网卡的ip地址和子网掩码
    ifconfig eth0       #显示eth0网卡信息
    ifconfig eth0 down  #关闭eth0网卡
    ifconfig eth0 up    #开启eth0网卡
```

### (7).netstat 显示网络连接信息

格式：

#### netstat [选项]

```
选项：
    -s      显示协议数据统计信息
    -n      使用数字形式的IP，端口号的等信息。
    -p      显示进程名称以及对应的进程ID号。
    -l      仅显示正在监听的sockets接口信息。
    -u      查看udp协议的连接信息。
    -t      查看tcp协议的连接信息。
demo：
    netstat -untlp    #显示网络详情
```

### (8).route 显示或设置静态IP路由表

格式：

#### route [选项]

```
命令：
    add   添加路由表记录
    del   删除路由表记录
    flush 刷新路由表
选项：
    -v    显示版本信息。
    -n    以数字形式显示ip
    -e    显示详情
demo：
    route       #显示路由表
    route -n    #以数字形式显示ip的路由表
    #添加默认网关
    route add default gw 192.168.0.254  
    #添加指定网段内的网关
    route add -net 182.18.0.0/16 gw 192.168.0.254  
    #添加路由记录
    route add -net 192.26.75.0 network 255.255.255.0 dev eth0
    #删除默认网关
    route del default gw 192.168.0.254 
```

### (9).hostname 查看或设置主机名

格式：

#### hostname [选项]

```
demo：
    hostname    #查看主机名
    hostname dnzhu-mark #设置主机名
    hostname -i     #查看本机ip 
```

### (10).traceroute 跟踪数据包的路由过程

```
参数：
    -I      使用ICMP封装跟踪包，默认是UDP封装。

demo：
    traceroute -I  www.sina.com.cn

more： 
nslookup www.sina.com.cn    检查本地dns服务器是否正常。
dig www.sina.com.cn         更多的dns查询信息。
   
```