---
layout: post
title: "redis数据结构之跳跃表与压缩表"
description: ""
category: redis
tags: []
---

## 跳跃表结构

| Header       |
| Tail         |
| Level  (2)   | 
| Length (4)   | 

---
- **Header**：指向跳跃表的表头节点。
- **Tail** ： 指向跳跃表的表尾结点。
- **Level** ：记录跳跃表内，层数最大的那个节点的层数（表头节点不计算在内）。
- **Length** ： 记录跳跃表的长度（包含的节点的数量）。

---

跳跃表结构图：

![跳跃表](http://pic.xcar.com.cn/2016/07/26/3e021264968c79b43c1dc14dc722ce32.png)

---

**层**：      每层有两个属性，前进指针和跨度。
    
**后退指针**：指向当前节点的前一个节点。
    
**分值**：    在跳跃表中，节点按各自所保存的分值从小到大排列。
    
**成员对象**：代表当前的节点对象。

---

## 压缩列表

---

压缩列表是列表键和哈希键的底层实现之一。

压缩列表是由一系列特殊编码的连续内存块组成的顺序型数据结构。

---

| \|  Zlbytes \|  |  Zltail \| |   Zllen \| |  Entry1 \| |  Entry2 \| |   … \| |  Entry \| |  n \| |   zlend \| |

---

- **Zlbytes** : 表示压缩列表的总长有多少个字节。

- **Zltail** ： 假设zltail的值为60，如果有一个指针指向压缩列表的起始地址，那么只要用指针p加上偏移量60，就能得到表尾节点地址。

- **Zllen** ： 表示压缩列表含有几个节点。

---

#### 压缩列表节点的构成

每个压缩列表的节点可以保存一个字节的数组，或者一个整数值。

**字节数组的长度**：

```
    长度小于等于63（26 -1）字节的字节数组。
    长度小于等于16383（214 -1）字节的字节数组。
    长度小于等于429497295（232 -1）字节的字节数组。
```

**整数值长度**：

```
    Int16_t 类型整数。
    Int32_t 类型整数。
    Int64_t 类型整数。
```
**节点结构**：

|   \| Previous_entry_length \|   |    Encoding   \|   |    content \|   |

---

*1.Previous_entry_length属性*：记录列表前一个节点的长度。

---
**注意**：

    (1).  如果列表前一个节点的长度小于254 个字节，那么Previous_entry_length的属性值为1个字节，前一字节长度就保存在这一字节里面。

    (2).  如果前一字节的长度大于254，那么Previous_entry_length的长度为5个字节，第一个字节会被设置为0Xfe(十进制254)，之后的4个字节保存前一字节的长度。

利用Previous_entry_length这一属性，当知道当前字节的指针以后就能计算出前一字节的位置。压缩表从表尾向表头遍历的时候就是这样进行的。

---

*2.Encoding属性*：记录节点的content属性所保存的数据的类型和长度。

*3.Content属性*：保存节点的值，节点的值可以是字节数组或者整数。

---

### 连锁更新

**什么是连锁更新？**

---
就是一连串的地址保存的数据的字节数都是在254以下的，但是突然插入一个数据字节数大于254个字节，就需要对zllist进行连锁更新。

---

## 集合set类型结构

set集合的结构图：

![set](http://pic.xcar.com.cn/2016/07/26/379940c0638dd5183d10e49a7f222d3b.png)


Ps: encoding 的编码方式决定了contents中数据的取值范围。

#### 集合存储类型的升级

---
    每当我们要将一个新元素添加到证书集合里面，并且新元素的类型比现有所有元素类型都要长时，整数集合先要升级，然后再将新元素添加到整数集合里。

---

#### 降级

---
    整数集合不支持降级操作，一旦对数组进行了升级，编码就会一直保持升级后的状态。

    整数集合的底层实现为数组，这个数组以有序，无重复的方式保存集合元素，有需要时，程序会添加新元素，改变这个数组的类型。

---