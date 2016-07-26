---
layout: post
title: "redis的数据结构之hash"
description: ""
category: redis
tags: []
---

### hash字典

---

    Type 和privdata属性是针对不同类型的键值对，为创建多态字典而设置。

    Type 属性是一个指向dictType结构的指针，每个dict-Type结构保存了一许多用于操作特定类型的键值对的函数，redis 会为不同字典设置不同的函数。

    dictType也是特定的结构体。

    Privdata属性保存了需要传给特定函数的参数。

    Ht属性包含了两个数组（ht[0], ht[1]）.字典只是用ht[0]，ht[1] 只会在ht[0] 进行refash的时候使用。

    Refashidx属性，记录refash的进度，因为refash是渐进式的refash，直到完成设置为-1.

---

下图是普通状态下的字典：

![字典结构](http://pic.xcar.com.cn/2016/07/26/3d3606126e8775ccfec1afa248013fe4.png)


### 哈希算法

---

    要将一个新的键值对，添加到字典里，需要根据键值对计算出哈希值和索引值。Redis的底层使用murmurhash2算法来实现。

    Redis哈希值的计算： hash = dict->type->hashFunction(key);

    根据哈希值和sizemask值：index = hash & dict->ht[x]->sizemask;

---

### 解决键冲突

---
    问题：当有两个或以上数量的键被分配到哈希表数组的同一个索引上时，就会发生键冲突。

    如何解决： redis的hash使用链表结构解决键冲突。

    假设：
    如果再加入一个键值对（k2,v2）,恰好分配到了索引2上面，那么hash表的结构就变成如下:

![hash](http://pic.xcar.com.cn/2016/07/26/c2a1e18b69e136a112bb12a015b2425d.png)

---

### Hash表的refash

---
   
    哈希表总会将负载因子维持在一个合理的范围之内，键值对太多或者太少时，程序需要对哈希表的大小进行扩展或收缩。

    哈希表已经使用的空间/哈希表的大小，就是哈希表的负载因子。

    Load_factor  =  ht[0].used/ht[0].size ; 
 
---

#### 什么情况下，哈希表会扩展和收缩？

---
    1.服务器目前没有执行BGSAVE或者BGREWRITEAOF 命令，且哈希表的负载因子大于等于1。
    
    2.服务器目前正在执行BGREWRITEAOF 命令，并且哈希表的负载因子大于等于5。
    
    3.哈希表的负载因子小于0.1时，程序自动开始执行收缩操作。

---

#### 如何扩展：

----

    1.为字典的ht[1]哈希表分配空间，分配空间的大小取决于ht[0].used的大小。

    Ps：（ht[1]的大小大于等于ht[0].used * 2 的2的n次方幂，如果是收缩：ht[1]的大小大于等于ht[0].used 的2的n次幂）。

    2.将保存在ht[0]中的所有键值对refash到ht[1]上。

    3.当ht[0] 包含的所有键值对都迁移到ht[1]之后，释放ht[0],将ht[1]设置为ht[0],并在ht[1]新创建一个哈希表，为下次refash做准备。

----
