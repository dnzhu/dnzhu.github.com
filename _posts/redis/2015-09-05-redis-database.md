---
layout: post
title: "redis-数据库"
description: ""
category: redis
tags: []
---

### Redis数据库
 
    1. redis 服务器默认会创建16个数据库。
    2. 通过select 命令来切换不同的数据库。默认数据库为0。

----

### 数据库键空间
 
    Redis服务器中的每一个数据库都由一个redisDb结构来表示，其中redisDb结构的dict字典保存了数据库中的所有键值对。这个字典称为键空间(key space)。
 
    判断一个键是否过期，调用is_expired()函数。

----

### 过期键删除策略
 
1. 定时删除：
    在设置键的过期时间的同时，创建一个定时器，让定时器在过期时间来临时，立即执行删除键的操作。

2. 惰性删除：
    每次从键空间获取键时，先检查该键是否过期，过期就删除，没过期就返回。

3. 定期删除：
    每隔一段时间，程序对数据库进行一次检查，删除里面的过期键。至于要删多少，以及要检查多少个数据库，由具体算法决定。
 
Ps：1 和3 是主动删除，2 是被动删除。
 

定时删除的缺点：
    
    对CPU时间不是很友好。如果过期键比较多的情况下，删除过期键可能占用大量的CPU时间，影响服务器的吞吐量。
 
惰性删除的缺点：
    
    对内存不是很友好，内存中会存在大量过期键，如果这些过期键没有访问到的话，或许永远不会删除（除非手动执行flushdb）。
 
定期删除策略是前两种的折合：
 
    定期删除策略是每隔一段时间，执行一次删除过期键操作，并通过限制删除操作执行时长和频率来减少删除操作对CPU时间的影响。服务器必须根据实际情况，合理的设定删除操作的执行时长和执行频率。

----

### AOF , RDB 数据对过期键的处理
 
**RDB**

1.生成RDB文件

在执行save 命令或BGSAVE 

命令创建一个RDB文件时，程序会对数据库中的键进行检查，过期键不会被持久化到RDB文件中。
 
2.载入RDB文件

启动redis服务器时，如果服务器启动了RDB功能，那么服务器会对RDB文件进行载入。

如果redis服务器以主服务器模式运行，程序会过滤掉过期键。

如果redis服务器以从服务器模式运行，程序不会过滤掉过期键。
 

ps : RDB生成的快照，有可能存在几分钟或者几秒钟内的数据丢失的风险，但是速度比AOF快。
 

**AOF**

当AOF持久化时，会记录服务器执行的所有写操作命令。

在redis服务器启动时，会通过执行AOF文件中的这些命令去还原数据库。

当服务器的某个键已经过期，但没有被惰性删除，那么AOF文件不会产生影响。

当过期键被删除以后，程序会向AOF 文件中追加一条DEL命令，来显示记录已删除该键。

----
 
### redis主从复制
 

当redis配置了主从复制，主服务器中执行删除操作，会显示的向所有的从服务器发送一条DEL命令。所有的从服务器会执行删除操作。
 
如果从服务器没有收到del命令，在客服端访问从服务器过期的键的时候，也会返回相应的值。
 
由主服务器统一控制所有的从服务器删除过期键，就可以保证主从服务器的数据一致性。