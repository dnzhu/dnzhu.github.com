---
layout: post
title: "Linux系统软件管理工具"
description: ""
category: linux
tags: []

---

    redhat /fedora /centos  使用的是rpm包管理工具。

    ubuntu / debian         使用的是 dpkg（deb）。

    无论是dpkg还是rpm 管理的都是二进制的包 。他们的最大的缺点就是无法自动解决包的依赖关系。

    相对应的就是源代码包，源代码包先要经过编译，然后才能安装。

    相对应的各个系统度提供了前端程序，rpm 对应的yum ，dpkg 对应的 apt。

    无论是yum还是apt ，之所以能够找到包的依赖关系，首先他们先读取仓库文件，仓库文件中记录包之间的依赖关系。

    Ubuntu系统本身就是基于debian研发的。

---

### dpkg 包管理常用命令：

---

    安装软件： dpkg  -i  packagename.deb

    删除软件： dpkg  -r  packagename

    查询软件包信息：
        dpkg     –info   packagename.deb
        dpkg     –status   packagename

    查询软件包所含文件

        dpkg  --listfiles   packagename
        dpkg  --contents    packagename.deb

    查询文件归属： dpkg  --search filename

    查询系统中安装的包：  dpkg -l

---

### Apt 是dpkg的前端程序，常用的命令有：

---

    安装软件 ： apt-get  install package

    删除软件 ：apt-get   remove  package

    查询包信息：apt-cache  show package

    查询所包含的文件：apt-file list  package

    查询文件归属：apt-file  search package

    查询系统中安装包： apt-cache  pkgnames

---

    Apt仓库源地址 :  /etc/apt/source.list   

    下载的软件包所在位置：
        /var/lib/apt/lists
        /var/cache/apt/archives

---

### Rmp 基础命令：

---

    安装软件 ： rpm  -i  filename.rpm

    卸载软件 ： rpm   -e  filename

    升级形式安装 ： rpm  -U  new-filename.rpm

    Rpm 支持通过htttp，ftp 协议进行安装软件。

    rpm  -ivh  http://www.linux-resource.com/aaa.rpm 

    -v  显示版本信息

    -h  显示进度条

    rpm  -K  filename.rpm   验证rpm包是否完整。


#### Rpm 查询命令：

    rpm  –qa    列出所有的安装的rpm软件

    rpm  -qf      filename  查询指定文件属于哪个rpm包

    rpm  -qi      software  查询指定已安装rpm软件的信息

    rpm  -ql   software  查询指定已安装软件所包含文件

    rpm  -qip  software.rpm   查询rpm文件信息

    rpm  -qlp  software.rpm   查询rpm文件包含的文件

---

> yum可以解决rpm包之间的依赖关系，所以使用yum安装比较便捷。

### yum安装步骤：

#### 1.设置yum源。

    yum 所在位置： /etc/yum.repos.d/目录下。
    注意：yum源的配置文件都是以 .repo 结尾，该目录下可以存在多个配置文件。

#### 2.yum 安装常用命令。

**安装：**

    yum  install  全部安装
    yum -y install  全部安装(自动完成)
    yum  install  package1   安装指定的包 package1
    yum  groupinstall  group1  安装指定的组 group1

**更新：**

    yum  update  全部更新(命令慎用，linux内核也会升级，升级后的linux内核是需要配置)。
    yum -y update 全部更新(自动完成)
    yum  update  package1  更新指定的包 package1 (推荐使用升级指定的软件包)
    yum  check-update  检查可更新的程序
    yum  upgrade  package1  升级软件
    yum  groupupdate  group1  升级程序组 

**查找和显示：**

    yum  info  package1   显示安装包信息
    yum  list  显示所有的安装包
    yum  list  package1  显示指定包的安装情况
    yum  list  installed   显示已安装软件包
    yum search package     查询包以及包的依赖关系

**删除：**

    慎用yum卸载，因为都是自动完成，卸载一个软件，依赖该软件的包都会卸载，又可能还有其他软件有依赖关系。这时就有可能导致问题出现。

    yum  remove or erase  package1   删除程序包 package1
    yum  groupremove  group1    删除程序组 group1
    yum  deplist  package1    查看程序package1依赖情况



