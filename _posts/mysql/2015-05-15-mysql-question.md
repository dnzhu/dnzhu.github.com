---
layout: post
title: "性能优化之MySQL优化"
description: ""
category: mysql
tags: []
---


## SQL语句优化

### 1. MySQL慢查日志的开启方式和存储格式 

- 查看mysql是否开启慢查询日志

        show variables like 'slow_query_log';
    
    - 查看所有log的参数
      
            show variables like '%log%';

- 设置没有索引的记录到慢查询日志

        set global log_queries_not_using_indexes=on;

- 查看超过多长时间的sql进行记录到慢查询日志

        show variables like 'long_query_time';
    
    - 设置记录的延迟时间
    
            set global long_query_time=1;

- 开启慢查询日志

        set global slow_query_log=on;
      
- 查看日志记录位置

        show variables like 'slow_query_log_file';

- 查看日志

        tail -50 /usr/local/mysql/var/localhost-slow.log
        
        
### 2. MySQL慢查日志分析工具之mysqldumpslow

- 用mysql官方提供的日志分析工具查看TOP3慢日志

        mysqldumpslow -t 3 /usr/local/mysql/var/localhost-slow.log | more
        
### 3. MySQL慢查日志分析工具之pt-query-digest


- 用pt-query-digest分析慢日志

        pt-query-digest /usr/local/mysql/var/localhost-slow.log | more
        
### 4. 如何通过慢查日志发现有问题的SQL

1. 查询次数多且每次查询占用时间长的SQL

        通常为pt-query-digest分析的前几个查询

1. IO大的SQL
    
        注意pt-query-digest分析的Rows examine项

1. 未命中索引的SQL
        
        注意pt-query-digest分析的Rows examine（扫描行数） 和 Rows Send（发送行数）的对比

### 5. 通过explain查询和分析SQL的执行计划

`explain`返回各项的含义：

-  table: 显示这一行的数据是关于哪一张表
-  type: 这是重要的列, 显示连接使用了何种类型。从最好到最差的连接类型为: `const`, `eq_reg`, `ref`, `range`, `index` 和 `ALL`。
    - index 基于索引的扫描
    - ALL 表扫描
-  possible_keys: 显示可能应用在这张表里的索引，如果为空则表示没用可能的索引。
-  key: 实际使用的索引, 如果为空表示没有使用索引。
-  key_len: 使用索引的长度，在不损失精确性的前提下，长度越短越好。
-  ref: 显示索引的哪一列被使用了，如果可能的话，是一个常数。
-  rows: MYSQL认为必须检查的用来返回请求数据的行数（表扫描的行数）。


### 6. Count()和Max()的优化

- 优化`count()`函数 - 在一条SQL中同时查出2006年和2007年电影的数量

    SELECT 
    COUNT(release_year='2006' OR NULL) AS '2006年的电影数量',
    COUNT(release_year='2007' OR NULL) AS '2007年的电影数量' 
    FROM file;

- 优化`max()`函数 - 查询最后支付时间 

    select max(payment_date) from payment;
    
    - 优化：给payment_date字段建立索引
    
        create index idx_paydate on payment(payment_date);
        
### 7. 子查询的优化

通常情况下，需要把子查询优化为join查询，但在优化时需要注意关联键是否有一对多的关系，要注意重复数据。

### 8. group by的优化

- 优化前
    
        Explain SELECT actor.first_name,actor.last_name,COUNT(*)
        FROM sakila.film_actor
        INNER JOIN sakila.actor USING(actor_id)
        GROUP BY film_actor.actor_id;

- 优化后

        Explain SELECT actor.first_name,actor.last_name,c.cnt
        FROM sakila.actor INNER JOIN (
            SELECT actor_id,COUNT(*) AS cnt FROM sakila.film_actor
            GROUP BY actor_id
        ) AS c USING(actor_id)
        
### 9. Limit查询的优化

limit常用与分页处理，时常会伴随着order by 从句使用，因此大多时候会使用filesorts,这样会造成大量的IO问题

        SELECT film_id,description FROM sakila.film ORDER BY title LIMIT 50,5

- 优化步骤1：使用有索引的列或者主键进行Oder by操作

        SELECT film_id,description FROM sakila.film ORDER BY film_id LIMIT 50,5; 

- 优化步骤2：记录上一次返回的主键，在下次查询时使用主键过滤

        * 避免了数据量大时扫描过多的记录
        
        SELECT film_id,description FROM sakila.film 
            WHERE file_id > 55 AND film_id <= 60 ORDER BY film_id LIMIT 1,5;

---

## 索引优化

### 1. 如何选择合适的列建立索引

- 在where从句，group by从句，order by从句，on从句中出现的列
- 索引字段越小越好
- 离散度大的列放在联合索引前面

        SELECT * FROM payment WHERE staff_id=2 AND customer_id=584;
        
        *由于customer_id的离散度更大，应该建立索引 index(customer_id,staff_id)，即把离散程度高的列放在联合索引的前面

    - 判断列的离散程度（唯一值越多，离散度越高，可选择性越高）
    
            SELECT count(distinct customer_id),count(distinct staff_id) FROM payment;
            
### 2. 索引优化SQL的方法

1. 索引的维护及优化 - 重复及冗余索引

重复索引是只相同的列以相同的顺序简历同类型的索引，如下表中`primary key` 和`ID`列上的索引就是重复索引

        CREATE table test(
            id init not null primary key,
            name varchar(10) not null,
            title varchar(50) not null,
            unique(id)
        )engine=innodb;
        
 2. 索引的维护及优化 - 查找重复及冗余索引 

        SELECT a.TABLE_SCHEMA AS '数据库名'
            ,a.table_name as '表名'
            ,a.index_name AS '索引1'
            ,b.INDEX_NAME AS '索引2'
            ,a.COLUMN_NAME AS '重复列名' 
        FROM STATISTICS a JOIN STATISTICS b 
        ON a.TABLE_SCHEMA=b.TABLE_SCHEMA AND 
           a.TABLE_NAME=b.TABLE_NAME AND 
           a.SEQ_IN_INDEX=b.SEQ_IN_INDEX AND 
           a.COLUMN_NAME=b.COLUMN_NAME 
        WHERE a.SEQ_IN_INDEX = 1 AND a.INDEX_NAME <> b.INDEX_NAME;
 
3. 索引的维护及优化 - 查找重复及冗余索引工具

    使用`pt-duplicate-key-checker`工具检查重复及冗余索引
    
        pt-duplicate-key-checker \
        -uroot \
        -p '123456'  \
        -h 127.0.0.1
        
### 3. 索引维护的方法

索引的维护及优化 - 删除不用的索引

    pt-index-usage \
    -uroot \
    -p 'tjw199022' \
    -h 127.0.0.1 \
    /usr/local/mysql/var/localhost-slow.log

---
    
## 3. 数据库结构优化
    
### 1. 选择合适的数据类型

数据类型的选择，重点在于 `合适` 二字，如何确定选择的数据类型是否合适？

- 使用可以存下你的数据的最小的数据类型
- 使用简单的数据类型。int要比varchar类型在mysql处理上简单
- 尽可能的使用not null定义字段
- 尽量少使用text类型，非用不可时最好考虑分表

**例1**

使用`int`来存储日期时间，利用`FROM_UNIXTIME()`,`UNIX_TIMESTAMP()`两个函数来进行转换 

    CREATE TABLE test(
        id INT AUTO_INCREMENT NOT NULL,
        timestr INT,
        PRIMARY KEY(id)
    );
    
    INSERT INTO test(timestr) VALUES(UNIX_TIMESTAMP('2014-06-01 13：12：00'));
    
    SELECT FROM_UNIXTIME(timestr) FROM test;
    
**例2**

使用`bigint`来存储IP地址，利用`INET_ATON()`,`INET_NTOA()`两个函数来进行转换

    CREATE TABLE sessions(
        id INT AUTO_INCREMENT NOT NULL,
        ipaddress BIGINT,
        PRIMARY KEY(id)
    ); 
    
    INSERT INTO sessions(ipaddress) VALUES(INET_ATON(192.168.0.1));
    
    SELECT INET_NTOA(ipaddress) FROM sessions;
    
### 2. 表的范式化优化

### 3. 表的反范式化优化

### 4. 表的垂直拆分（解决宽度问题）

所谓垂直拆分，就是把原来一个有很多列的表拆分成多个表，这解决了表的宽度问题。通常垂直拆分可以按以下原则进行：

- 把不常用的字段单独存放到一个表
- 把大字段独立存放到一个表中
- 把经常一起使用的字段存放在一起

### 5. 表的水平拆分 （解决数据量问题）

表的水平拆分是为了解决单表的数据量过大的问题，水平拆分的表每一个表的结构都是完全一致的。

常用的水平拆分方法为：

1. 对`customer_id`进行`hash`运算，如果要拆分成5个表则使用`mod(customer_id,5)`取出0-4个值（对5取模）
2. 针对不同的`hashID`把数据存到不同的表中

挑战：

1. 跨分区表进行数据查询
2. 统计及后台报表操作

---

## 4. 系统配置优化

### 1. 数据库系统配置优化

网络方面的配置，要修改/etc/sysctl.conf

        #增加tcp支持的队列数
        net.ipv4.tcp_max_syn_backlog = 65535
        #减少断开连接时，资源回收
        net.ipv4.tcp_max_tw_buckets = 8000
        net.ipv4.tcp_tw_reuse = 1
        net.ipv4.tcp_tw_recycle = 1
        net.ipv4.tcp_fin_timeout = 10
        
打开文件数的限制，可以使用ulimit -a查看目录的各位限制，可以修改/etc/security/limits.conf文件，增加以下内容以修改打开文件数量的限制

        soft nofile 65535
        hard nofile 65535

### 2. MySQL配置文件优化

        innodb_buffer_pool-size：非常重要的一个参数，用于配置Innodb的缓冲池，如果数据库中个只有Innodb表，则推荐配置量为总内存的75%.
        
        innodb_buffer_pool_instances:Mysql5.5中新增加参数，可以控制缓冲池的个数，默认情况下只有一个缓冲池
        
        Innodb_log_buffer_size:innodb log 缓冲的大小，由于日志最长每秒钟就会刷新所以一般不用太大
        
        innodb_flush_log_at_trx_commit:关键参数，对innodb的IO效率影响很大。默认值为1,可以取0,1,2三个值，一般建议设为2，但如果数据安全性要求比较高则使用默认值1.
        
        innodb_read_io_threads,innodb_write_io_threads：决定了Innodb读写的IO进程数，默认为4
        
        innodb_file_per_table:关键参数，控制Innodb每一个表使用独立的表空间，默认为off，也就是所有表都会建立在共享表空间中
        
        innodb_stats_on_metadata:决定了Mysql在什么情况下会刷新innodb表的统计信息

    
### 3. 第三方工具 Percon Configuration Wizard
 
 
        

