---
layout: post
title: "mysql的分区"
description: ""
category: mysql
tags: [mysql]
---

### mysql分区的优点

    1.能够存储更大的数据。
    2.优化查询。where子句中包含分区条件，可以只扫描必要的一个或多个分区提高查询效率；同时设计到集合函数查询时，可以在每个分区上并行处理，然后结果汇总。
    3.对于已经过期或者不需要保存的数据，可以通过删除相关分区，来快速的删除。
    4.跨多个磁盘分散查询数据，提高了磁盘I/O吞吐量。

----

### 检查是否支持分区

```
show variables like '%partition%';
```
查看mysql当前安装了哪些插件:

```
SHOW PLUGINS;
```

PS: MYSQL大部分存储引擎(MyISAM,InnoDB,Memory等)支持创建分区表，但像merge，cvs 等存储引擎不支持分区。

----

### 分区类型

    1. range 分区，给定一个连续的范围区间，把数据分配到不同的分区。
    2. list分区，类似range分区，枚举出不同的值，分布到不同分区。
    3. hash分区，基于给定的分区个数，把数据分配到不同的分区。
    4. key分区，类似hash分区。


无论是哪种分区类型，要么分区表上没有主键/唯一键，要么分区表的主键/唯一键都必须包含分区键，也就是说，不能使用主键或唯一键之外的字段进行分区。

----

### range分区

利用取值范围将数据分成分区,分区要连续并且不能相互重叠。

创建range分区

```
create table emp (id int not null ,ename varchar(30) not null default '', separated DATE not null default '9999-12-31',job varchar(30) not null ,store_id int not null)
ENGINE=INNODB
PARTITION BY RANGE(store_id)(
    PARTITION p0 values less than (10),
    PARTITION p1 values less than (20),
    PARTITION p2 values less than MAXVALUE
);
```

MYSQL5.5 改进了range分区功能，提供了range columns分区支持非整数分区。

```
create table emp (id int not null ,ename varchar(30) not null default '', separated DATE not null default '9999-12-31',job varchar(30) not null ,store_id int not null)
ENGINE=INNODB
PARTITION BY RANGE COLUMNS (separated) (
    PARTITION p0 values less than ('2000-01-01'),
    PARTITION p1 values less than ('2008-01-01'),
    PARTITION p2 values less than ('2016-01-01')
);

```
# range 分区应用场景：

    * 需要删除某部分历史数据的时候，可以直接删除分区就ok。
    * 包含分区键的查询，可以很快的定位到记录。

----

list 分区

list 分区是一个枚举列表值的聚合，range分区是一个连续值的聚合。

创建list分区


```
create table books (
    add_date DATE NOT NULL,
    category INT,
    amount decimal(10,3)
)PARTITION BY LIST (category) (
    PARTITION P0 VALUES IN (3,5),
    PARTITION P1 VALUES IN (1,10),
    PARTITION P2 VALUES IN (4,9),
    PARTITION P3 VALUES IN (2),
    PARTITION P4 VALUES IN (6)
);

```

* mysql5.5 之前的版本，LIST分区只能匹配整数列表。5.5之后的版本支持非整数列表。

----

hash 分区

mysql支持两种hash分区，常规hash分区和线性hash分区。常规hash分区就是使用取模算法。线性hash分区是使用2的幂运算法则。

----

mysql分区处理NULL值的方式，range分区。null值当作最小值处理。list分区null值必须在枚举列表中。否则将不被接受。hash/key分区中，null值会被当作0来处理。

----


