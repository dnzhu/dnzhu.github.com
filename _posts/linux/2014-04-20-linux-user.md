---
layout: post
title: "linux 账户管理"
description: ""
category: linux
tags: []
---

> 在linux系统中，对用户和组的管理是通过id来实现的。用户的ID为UID,组ID为GID。root账户的UID=0,1-499之间的id被系统所保留，我们创建的普通账户，ID从500开始。

> linux的组有基本组和附加组之分，一个账户只能加入一个基本组，但可以同时加入多个附加组，创建账户时，系统默认会创建 同名的基本组，并设置账户加入这个基本基本组。

### 创建账户

---

#### 格式 ： useradd [选项] 用户名称

```
选项：
    -c      设置账户的描述信息，一般为全称。
    -d      设置账户的家目录。
    -e      设置账户的失效时间。
    -g      设置账户的基本组。
    -G      设置账户的附加组。多个附加组使用逗号分隔。
    -u      指定账户的UID。
    -s      设置账户的登录shell。默认为bash。
    -M      不创建账户的家目录。

demo：
    useradd mark        //创建一个普通账户mark
    useradd -d  /home/dnzhu -g root -e 2013-12-29 -G bin,mark
    //创建一个dnzhu账户，所属root组，有效期到2013-12-29，附加组有bin，mark
    useradd -s /sbin/nologin -M nobody  
    //创建了一个不能登录的账户nobody
```
---

### 创建组

---

#### 格式：groupadd [选项] 组名

```
    -g      设置组的GID.

demo:
    groupadd mark               //创建mark组
    groupadd -g 1000 dnzhu      //创建dnzhu组，并设置组id为1000
```
---

### 修改密码

---

#### 格式：passwd [选项] 账户

```
选项：
    -a      报告所有账户的密码状态
    -d      情况账户密码。
    -l      锁定账户，root可用。
    -u      解锁账户。
    --stdin 从文件或管道读取密码。

demo：
    passwd          给当前账户设置新密码
    passwd root     给root设置密码。
    echo "dnzhu" | passwd --stdin dnzhu  从管道中读取密码给账户dnzhu。
    passwd -d dnzhu 清空账户密码 
```

### 修改账户信息

---

#### 格式：usermod [选项] 账户名

```
选项：
    -d      修改账户家目录。
    -e      修改账户过期时间。
    -g      修改账户所属基本组。
    -G      修改账户所属附加组。
    -u      修改账户的uid。

demo：
    usermod -d /home/mark dnzhu
    usermod -e 2013-10-01 mark
    usermod -s /bin/bash nobody -d /home/nobody
    usermod -u 1001 mark
```

### 删除账户，组

---

#### 格式：userdel [选项] 账户名

#### 格式：groupdel [选项] 组名

```
选项：
    -r      删除账户以及家目录下的文件。
    
demo：
    userdel  mark    
    userdel  -r  mark 

    groupdel dnzhu
```

### 显示账户和组信息

#### 格式：id 账户

```
id  dnzhu
id  root    
```

### 设置组密码

```
gpasswd  dnzhu      //设置dnzhu组的密码
gpasswd -A  dnzhu mark  //将dnzhu账户设置为mark组的管理员
```

### 账户信息文件位置
```
    /etc/passwd     保存账户信息
    /etc/shadow     保存账户密码的文件
    /etc/group      保存组信息
    /etc/gshadow    保存组密码的文件
```



{% include JB/setup %}