---
layout: post
title: "linux系统调优"
description: ""
category: linux
tags: []

---

### 关闭selinux

```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

#修改配置文件后，需重启系统生效。
```
或者执行命令：

```
setenforce 0    #临时关闭
setenforce 1    #开启
getenforce      #查看状态
```

---

### 设置系统的运行级别

设置系统的运行级别为3，即文本命令行模式。

```
grep 3:initdefault /etc/inittab     #编辑该文件可以设置其他的运行级别

runlevel    #查看运行级别

init 5      #切换运行级别为图形界面(安装了图形界面才能执行该命令)
```
---

### 精简系统开机自启动

根据经验，系统需要开机自启动的服务有，sshd，rsyslog，network，crond，sysstat(系统性能检测工具)。

#### 如何设置开机自启动

* 执行ntsysv命令，然后弹窗进行设置。
* 执行setup命令->system service, 然后在弹窗中进行设置。
* 通过脚本设置。

```
chkconfig --list| grep 3:on | grep -vE "crond|sshd|network|rsyslog|sysstat" | awk '{print "chkconfig " $1 " off"}'|bash
```
---

### linux系统安全最小化原则

* 安装linux系统最小化，yum安装软件包也要最小化。
* 开机自启动最小化，用不到的服务不要开启。
* 操作命令最小化。比如能用“rm -f test.txt”就不要用“rm -rf test.txt”
* 登录用户最小化。
* 普通用户的权限最小化。
* linux系统文件及目录的权限最小化。
* 程序服务运行最小化。即程序尽量不要用root身份运行。

---

### 更改ssh服务器端远程登录配置

```
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.ori    #先备份

vim /etc/ssh/sshd_config
Port 52113      #修改默认端口
PermitRootLogin no      #是否允许root登录
PermitEmptyPasswords no     #是否运行密码为空登录
UseDNS no       # 指定sshd是否应该对远程主机进行反向解析。
GSSAPIAuthentication no   #优化设置
ListenAddress   10.20.0.254:52113  #监听内网ip

/etc/init.d/sshd reload     #平滑重启
/etc/init.d/sshd restart    #重启
```

#### 更高安全的ssh策略
设置sshd监听内网ip ，然后再让内网ip穿透防火墙。

```
iptables -I INPUT -p tcp --dport 52113 -s 10.20.0.0/24 -j ACCEPT    #内网ip穿透防火墙
```
---

### 利用sudo控制用户对系统命令的使用权限

为了安全以及管理的方便，可以将需要root权限的普通用户加入sudo管理。

```
visudo 或者vim /etc/sudoers
dnzhu ALL=(ALL)     NOPASSWORD:ALL    #给dnzhu赋予系统管理员权限
dnzhu ALL=(ALL)     /usr/sbin/useradd,/usr/sbin/userdel     #给dnzhu用户赋予useradd和userdel权限
%dnzhu ALL=(ALL)    /user/sbin/useradd      #给dnzhu组的所有用户赋予useradd权限
sudo -l     #查看当前用户被赋予的权限集合
```
---

### linux系统中文显示

```
cat /etc/sysconfig/i18n

echo 'LANG="zh_CN.UTF-8"' >/etc/sysconfig/i18n
source /etc/sysconfig/i18n  #使文件修改生效
echo $LANG  #查看
```

### 设置linux服务器时间同步
linux系统的时间同步服务为ntp服务。

```
/usr/sbin/ntpdate time.nist.gov     #time.nist.gov为时间服务器
```
网上的时间服务器有：ntp.sjtu.edu.cn , ntp1.aliyun.com，
可以写定时任务定时执行“/usr/sbin/ntpdate time.nist.gov ”脚本，同步时间。如果是大规模的集群，就需要搭建时间服务器。

---

### 调整linux系统文件描述符数量

#### 什么是文件描述符？
文件描述符就是文件创建时的一个标示，是一个无符号整型的文件句柄。文件描述符的范围是0-OPEN_MAX。默认是1024。

```
ulimit -n      #默认1024
echo '*     -    nofile    65535' >> /etc/security/limits.conf  #改为65535
ulimit -n      #查看为65535
```
---

### 定时清理邮件服务器临时目录垃圾文件

    centos5默认安装sendMail服务，临时文件存放路径：/var/spool/clientmqueue/
    centos6默认安装postfix服务，临时文件路径为：/var/spool/postfix/maildrop/

这两个目录很容易被垃圾文件填满，导致系统的inode数量不够用，从而无法存放文件。使用定时脚本清理垃圾文件。

```
find /var/spool/postfix/maildrop/ -type f |xargs rm -f
find /var/spool/clientmqueue/ -type f |xargs rm -f
```

---

### 锁定关键系统文件，防止被修改

加锁：

```
chattr +i /etc/passwd /etc/shadow /etc/group /etc/gshadow /etc/inittab
```
解锁：

```
chattr -i /etc/passwd /etc/shadow /etc/group /etc/gshadow /etc/inittab
```
还可以将chattr命令改名，防止被利用:

```
mv /usr/bin/chattr /usr/bin/chattr1     #改名为chattr1
```
---

### 为grub菜单加密

防止用户以单用户模式启动进行破解root密码等操作。

加密步骤：

* 利用/sbin/grub-md5-crypt产生一个MD5的密码串。假如是（sdfsagdgsafhsdkfsdk）
* 修改/etc/grub.conf。在hiddenmenu和title之间插入password --md5 sdfsagdgsafhsdkfsdk   

ps : 实际的秘钥不是这个样子。

---

### 禁止linux系统被ping通(非必须)

禁用：

```
echo "net.ipv4.icmp_echo_ignore_all=1" >> /etc/sysctl.conf
sysctl -p
```
启用：

```
删除/etc/sysctl.conf文件中"net.ipv4.icmp_echo_ignore_all=1"
echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_all
```
---

### 升级软件版本

常见需要升级的软件openssl，openssh，bash等。

```
rpm -qa openssl openssh bash    #查看
yum install openssl openssh bash -y     #升级
```




