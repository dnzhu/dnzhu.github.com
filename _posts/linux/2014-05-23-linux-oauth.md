---
layout: post
title: "linux权限控制"
description: "文件及目录权限"
category: linux
tags: []
---

---

linux 权限主要分为 ：读(r)，写(w)，执行(x)。
一个文件或目录又分为 ：文档所有者(user),所属组(group),其他账户(other)，分别对应字母：u，g，o。

使用 ls-l命令可以清楚的查看文件或目录所有的权限。 ls命令显示的详细信息就不展示了。

---

对于权限的表示，除了使用rwx字母表示以外，还可以使用数字表示：

|   数字   |    字符    |      文件     |               目录                |
|:--------:|:----------:|:-------------:|:----------------------------------|
|   4      |     R      |查看文件内容。 |查看目录下的文件与目录名称。       |
|   2      |     W      |修改文件内容。 |在目录下增，删，改文件与目录名称。 |
|   1      |     X      |文件可执行 。  |可以用cd命令进入目录。             |

### 修改文档权限

#### chmod [选项] 权限  文件或目录

```
option :
    --reference=RFILE   #根据参考文档设置权限
    -R                  #递归的修改权限

    u   代表所属账户
    g   代表组
    o   其他账户
    a   代表所有人
demo :
    chmod u=rwx,g=rwx,o=rwx /usr/software/      #给software目录加读写执行权限
    chmod 777 /usr/software                     #同上一个意思
    chmod a=rx /usr/software/                   #给software目录加上所有账户都有读和执行的权限
    chmod g-x,o-wx /usr/software/               #给组用户去掉执行权限，其他账户去掉写和执行的权限
    chmod g+rwx /usr/software/                  #给组用户增加读写执行权限。
    chmod --reference=system.log install.log    #以system.log的标准修改install.log的权限
```

### 修改文件或目录的所有者与所属组

#### chown [选项] [所有者]:[所属组] 文件或目录

```
demo :
    chown dnzhu:mark hello.html     #修改文件的所有者为dnzhu,所属组mark
    chown :root hello.html          #修改文件所属组为root
    chown root hello.html           #修改文件所有者为root
```

### ACL访问控制权限

> ACL权限控制主要目的是提供传统的owner,group,other的read,wirte,execute权限之外的具体权限设置，可以针对单一用户或组来设置特定的权限。

#### ACL启动

---

要使用ACL必须要有文件系统支持才行，目前绝大多数的文件系统都会支持，EXT3文件系统默认启动ACL的。

---

#### 查看ACL权限

```
格式：getfacl 文件或目录

demo ： 
    getfacl software/       查看software目录的acl
    getfacl hello.html      查看hello.html文件的acl    
```

#### 设置ACL权限

```
格式：setfacl [选项][{-m | -x} acl条目] 文件或目录
选项：
    -b      删除所有附加的acl条目
    -k      删除默认的acl
    -m      添加acl条目
    -x      删除指定的acl条目

ps:假设mark账户没有hello.html目录的权限。
demo：  
    setfacl -m u:mark:rw hello.html     #让mark账户可以有读写hello.html的权限
    setfacl -m g:dnzhu:rw hello.html    #让dnzhu组对hello.html有读写权限
    setfacl -x g:dnzhu  hello.html      #删除dnzhu对hello的读写权限
    setfacl -b hello                    #删除所有的附加acl条目
```
{% include JB/setup %}

