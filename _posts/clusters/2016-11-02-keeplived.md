---
layout: post
title: "keepalived 高可用集群搭建"
description: ""
category: cluster
tags: []
---

> keepalived 软件起初是专为LVS负载均衡设计的，用来管理并监控LVS集群系统中各个节点状态，后来又实现了VRRP协议。keepalived主要通过VRRP协议实现高可用功能，vrrp是virtual router redundancy protocol（虚拟路由器冗余协议）的缩写。vrrp协议是为了解决静态路由单点故障问题，能够保证个别节点宕机时，整个网络可以不间断运行。

---

### keepalived 软件的三个重要功能

#### 1.管理lvs负载均衡软件

    keepalived可以通过读取自身的配置文件，实现通过底层的接口直接管理lvs的配置以及控制服务的启动，停止等功能。

#### 2.实现对lvs集群节点的健康检查功能

    当lvs集群中的某个或多个节点发生故障时，keepalived服务会自动将失效的节点服务器从lvs的正常转发队列中清除，并将请求调度到正常的服务器上，从而保证最终用户不受影响。当故障服务器修复以后，keepalived服务又能自动的把服务器加入到正常转发队列中，并对其转发请求。

#### 3.作为系统网络服务的高可用功能

    keepalived可以实现任意两台主机之间，master和backup之间的故障转移和自动切换，这个主机可以用普通的不能停机的业务服务器，也可以是lvs负载均衡，nginx反向代理服务器等。

---

### keepalived 高可用故障切换原理

主要是通过vrrp协议实现的，当keepalived正常工作时，master服务器会不断的向备份服务器发送心跳消息，来告送备份服务器自己还活着，当master服务器发生故障时，就无法发送心跳消息，当备份服务器无法继续检测到master心跳消息时，就会主动接管程序，接管主机的ip资源以及服务。当master节点恢复时，备份服务器又会释放接管的ip资源和服务，恢复到备份角色。

### vrrp协议

* 1.VRRP，全称virtual router redundancy protocol，中文名虚拟路由冗余协议，vrrp是为了解决静态路由的单点故障而出现的。
* 2.VRRP 是通过一种竞选协议机制来将路由任务交给某台VRRP路由器的。
* 3.VRRP 用IP 多播的方式（默认多播地址224.0.0.18）实现高可用对之间通信。
* 4.工作时，主节点发包，备节点接包，当备节点接收不到主节点发的数据包时，就启动接管程序接管主节点的资源。备节点可以有多个，通过优先级竞选，但一般是一主一备。
* 5.VRRP 使用了加密协议加密数据，但keepalived官方还是推荐用明文方式配置认证类型和密码。

---

### 安装keepalived

ps：需要准备2台服务器或者2台虚拟主机。并且两台机器上都要安装keepalived。

```
#yum安装
yum install keepalived -y
rpm -qa keepalived

#启动/停止服务
/etc/init.d/keepalived start
ps -ef | grep keep | grep -v grep
ip add | grep 192.168
/etc/init.d/keepalived stop
```

### keepalived 配置文件说明

#### 全局定义部分

```
! Configuration File for keepalived   # 注释

global_defs {
   notification_email {
    #定义服务故障时，报警地址，可以是一个或多个
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc  #指定发件人地址，可选配置
   smtp_server 192.168.200.1   # 指定发送邮件的smtp服务器
   smtp_connect_timeout 30     #smtp超时时间
   router_id LVS_DEVEL         # keepalied服务器的路由标示，在一个局域网内是唯一的。
}
```

#### VRRP 实例定义区块部分

这部分定义了具体服务实例，包括keepalived主备状态，接口，优先级，认证方式和ip信息等。

```
vrrp_instance VI_1 {    #VI_1 实例名称
    state MASTER        #当前角色
    interface eth0      #网络通信接口
    virtual_router_id 51  #虚拟路由id标示，tip:master和backup中相同实例这个值要保持一致
    priority 100        #优先级，数越大优先级越高，实际应用中master的优先级高
    advert_int 1        #同步通知间隔
    authentication {    #权限认证配置
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress { #虚拟ip地址
        192.168.200.16
        192.168.200.17
        192.168.200.18
    }
}
```
----

### keepalived高可用服务实战

#### 配置keepalived实现单实例单ip自动漂移接管(一主一备)

实现vip自动漂移切换，仅适合两台服务器提供的服务均保持开启应用场景，这也是工作中常用的高可用解决方案。

**master服务器中的keepalived.conf配置：**

```
! Configuration File for keepalived

global_defs {
   notification_email {
     422298140@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id lb01      
}

vrrp_instance VI_1 {
    state MASTER    
    interface eth0
    virtual_router_id 55
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.20.0.249/24 dev eth0 label eth0:1
    }
}
```

**backup服务器中的keepalived.conf配置：**

```
! Configuration File for keepalived

global_defs {
   notification_email {
     422298140@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id lb02      #主的参数是：lb01
}

vrrp_instance VI_1 {
    state BACKUP    #主的参数是master
    interface eth0
    virtual_router_id 55
    priority 100    #主的参数是150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.20.0.249/24 dev eth0 label eth0:1
    }
}
```

#### 配置keepalived实现双实例双主模式(互为主备)

**什么是双实例双主模式？**

例如A业务在lb01上是主模式，在lb02上是备模式，而B业务在lb01上是备模式，在lb02上是主模式。

**lb01 上的keepalived.conf配置**

```
! Configuration File for keepalived

global_defs {
   notification_email {
     422298140@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id lb01      
}

vrrp_instance VI_1 {
    state MASTER    
    interface eth0
    virtual_router_id 55
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.20.0.249/24 dev eth0 label eth0:1
    }
}

vrrp_instance VI_2 {
    state BACKUP    
    interface eth0
    virtual_router_id 56
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.20.0.248/24 dev eth0 label eth0:2
    }
}
```

**lb02上的keepalived.conf配置**

```
! Configuration File for keepalived

global_defs {
   notification_email {
     422298140@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id lb02       #区别于lb01      
}

vrrp_instance VI_1 {
    state BACKUP    #区别于lb01
    interface eth0
    virtual_router_id 55
    priority 100    #区别于lb01
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.20.0.249/24 dev eth0 label eth0:1
    }
}

vrrp_instance VI_2 {
    state MASTER       #区别于lb01
    interface eth0
    virtual_router_id 56
    priority 150       #区别于lb01
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.20.0.248/24 dev eth0 label eth0:2
    }
}
```
---

### nginx负载均衡配合keepalived高可用架构

**架构图：**

---
![keepalived](/imgs/keepalived.png)

---

**lb01和lb02上nginx配置**

```
worker_processes  1;
pid  logs/nginx.pid;
error_log  logs/error.log;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server_tokens off;
    upstream www_server_pools {
        server 10.20.0.251:80 weight=1;     #web01的ip
        server 10.20.0.252:80 weight=1;     #web02的ip
    }
    server {
        listen       10.20.0.249:80;    #指定监听的ip
        server_name  www.dnzhu-blog.com;
        location / {
            proxy_pass http://www_server_pools;
            proxy_set_header_Host $host;
            proxy_set_header X-Forwarded-For $remote_addr;
        }
    }
}

```
**lb01上的keeepalived配置**

```
! Configuration File for keepalived

global_defs {
   notification_email {
     422298140@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id lb01
}

vrrp_instance VI_1 {
    state MASTER    
    interface eth0
    virtual_router_id 55
    priority 150    
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.20.0.249/24 dev eth0 label eth0:1
    }
}
```

**lb02上keeepalived配置**

```
! Configuration File for keepalived

global_defs {
   notification_email {
     422298140@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id lb02
}

vrrp_instance VI_1 {
    state BACKUP    
    interface eth0
    virtual_router_id 55
    priority 100    
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.20.0.249/24 dev eth0 label eth0:1
    }
}
```
ps：在测试中，客服端需要把域名对应的ip绑定在hosts文件中，正式场景下需要通过dns解析。