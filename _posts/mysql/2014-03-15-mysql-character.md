---
layout: post
title: "mysql字符集"
description: "mysql常用的字符集和校正规则"
category: mysql
tags: [mysql]
---

### 常用字符集

    ASCII           > 定长   > 单字节7位编码    > 最早的字符集。
    ISO-8859/latin1 > 定长   > 单字节8位编码    > 西欧字符集。
    GB2312          > 定长   > 双字节编码       > 早期的中文编码
    GBK             > 定长   > 双字节编码       > 虽然不是国标，但是支持的系统很多。
    UTF-32          > 定长   > 4字节编码        > ucs-4原始编码，很少使用
    UTF-16          > 非定长 > 2字节或4字节编码 > java和window-xp/nt等内部使用。
    UTF-8           > 非定长 > 1-4位编码        > unicode字符集，广泛支持。

----

### mysql支持的字符集

mysql 字符集包括字符集(character)和校对规则(collation)两个概念。字符集决定了mysql存储字符串的方式，校对规则决定了字符串的比较方式。字符集和校对规则是一对多的关系，mysql支持30多种字符集和70多种校对规则。

#### 查看可用的字符集

```
show character set;
或者
desc information_schema.character.sets;

```

#### 查看校正规则

```
show collation like 'gbk%';
```
ps:校对规则命名约定，以_ci结尾，大小写不敏感，_cs结尾，大小写敏感，_bin结尾，二进制编码比较。

----

### mysql字符集设置

4个级别的默认设置：服务器级，数据库级，表级，字段级。

#### 服务器字符集和校对规则

设置：

```
在my.cnf中：
    [mysqld]
    character-set-server = gbk

在启动选项中指定
    mysqld --character-set-server=gbk

或者在编译时指定
    cmake . -default-charset=gbk

```
ps: 没有指定默认是latin1字符集。没有指定校对规则默认使用字符集对应的校对规则。

查看：

```
show variables like 'character_set_server';

show variables like 'collation_server';

```

#### 数据库字符集和校对规则

在创建数据库的时候指定字符集和校对规则。

数据库字符集规则如下：
    
    如果指定了字符集和校对规则，就使用指定的字符集和校对规则。
    如果指定了字符集没有指定校对规则，则使用指定字符集对应的校对规则。
    如果指定了校对规则没有指定字符集，那么使用校对规则关联的字符集。
    如果字符集和校对规则都没有指定，则使用服务器字符集和校对规则。

查看数据库字符集和校对规则：

```
show variables like 'character_set database';

show variables like 'collation_database';
```

#### 表字符集和校对规则

ps：如果表的字符集和校对规则都没指定，默认使用数据库的字符集和校对规则。

查看表的字符集和校对规则：

```
show create table test \G;

```

#### 列字符集

列字符集和校对规则很少自定义，默认使用表的字符集和校对规则。
 

#### 连接字符集和校对规则

以上4中设置方式，确定的是数据保存的字符集和校对方式，在实际应用中，还存在客户端和服务器端之间交互的字符集和校对规则设置。

mysql提供了3个参数：character_set_client, character_set_connection, character_set_results 。3个参数字符集一致才能保证保证数据的正确写入对出。

```
set names gbk | utf8 ……
```
或者：

```
在my.cnf文件中加入：
[mysql]
default-character-set=gbk
```

----

### 字符集修改步骤

场景：如果数据库已经存在原始数据，又不想丢弃这部分原始数据，如果修改数据库字符集和校对规则？

demo：模拟将 latin1 字符集的数据库修改为 gbk 字符集数据库的过程。

步骤：

1.导出表结构。

```
    mysqldump -uroot -p  --default-character-set=gbk -d databasename > creatab.sql
```

ps : --default-character-set=gbk 表示以什么字符集连接，-d 表示只导出表结构，不导出数据。


2.手动修改creatab.sql中表结构改为新的字符集和校对规则。


3.确保记录不再更新，导出所有记录。

```
mysqldump -uroot -p --quick --no-create-info --extented-insert --default-character-set=latin1 databasename > data.sql
```

4.打开data.sql，将set names latin1 修改为set names gbk。

5.使用新的字符集创建数据库。

```
create database databasename default charset gbk;
```

6.创建表，执行creatab.sql。

```
mysql -uroot -p databasename < creatab.sql
```

7.导入数据，执行data.sql。

```
mysql -uroot -p databasename < data.sql
```