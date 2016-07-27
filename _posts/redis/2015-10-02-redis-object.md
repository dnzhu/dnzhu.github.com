---
layout: post
title: "redis数据对象"
description: ""
category: redis
tags: []
---

### Redis对象的结构：

| \| type \| | \| encoding \| | \| ptr(指针) \|| 

#### Type类型：

| 类型常量  |  对象名称 |
| --------- | --------- |
| REDIS_STRING  | \| 字符串对象|
| REDIS_LIST | \| 列表对象 |
| REDIS_HASH | \| 哈希对象 |
| REDIS_SET  | \| 集合对象 |
| REDIS_ZSET | \| 有序集合对象 |

#### encoding 

Encoding属性记录了对象所使用的编码，什么样的编码对应对象底层的数据结构。在redis中，encoding属性的值都都有对应的**常量**来表示。

#### ptr指针

对象的ptr指针指向对象的底层实现数据结构，而这些数据结构由对象的**encoding属性**决定。

**对象编码的数据结构**：


|编码常量  |  编码对应底层数据结构 |
| -------- | --------------------- |
| REDIS_ENCODING_INT | \| Long类型的整数 |
| REDIS_ENCODING_EMBSTR | \|  Embstr编码的简单动态字符串 |
| REDIS_ENCODING_RAW | \| 简单动态字符串 |
| REDIS_ENCODING_HT | \|  字典 |
| REDIS_ENCODING_LINKEDLIST | \|  双端链表 |
| REDIS_ENCODING_ZIPLIST | \| 压缩列表 |
| REDIS_ENCODING_INTSET | \|  整数集合 |
| REDIS_ENCODING_SKIPLIST | \| 跳跃表和字典 |

----

**Redis对象对应的编码**：

| 类型 | 编码 | 对象 |
| ---- | ---- | ---- |
| REDIS_STRING | \| REDIS_ENCODING_INT | \| 使用整数值实现的字符串对象 |
| REDIS_STRING | \| REDIS_ENCODING_EMBSTR | \| 使用embstr编码的字符串对象 |
|REDIS_STRING | \| REDIS_ENCODING_RAW | \| 使用简单动态字符串实现的字符串对象|
|REDIS_LIST | \| REDIS_ENCODING_ZIPLIST | \| 使用压缩列表实现的列表对象|
|REDIS_LIST | \| REDIS_ENCODING_LINKEDLIST | \| 使用双端列表实现的列表对象|
|REDIS_HASH | \| REDIS_ENCODING_HT | \|  使用字典实现的哈希对象|
|REDIS_HASH | \| REDIS_ENCODING_ZIPLIST | \| 使用压缩列表实现的哈希对象|
|REDIS_SET | \|  REDIS_ENCODING_INTSET | \|  使用整数集合编码实现的集合对象|
|REDIS_SET | \|  REDIS_ENCODING_HT  | \| 使用字典实现的集合对象|
|REDIS_ZSET | \| REDIS_ENCODING_ZIPLIST | \| 使用压缩列表实现的有序集合对象|
|REDIS_ZSET | \| REDIS_ENCODING_SKIPLIST| \| 使用跳跃比实现的有序集合对象|

----
----

### 1.字符串对象

**（1）.字符串对象的编码可以是int ，raw 或embstr。**

    如果字符串对象保存的值是一个整数值，并且这个整数值可以用long表示，那么redis的字符串对象的编码将使用REDIS_ENCODING_INT。

    如果字符串对象保存的值是一个字符串值，并且字符串的长度小于等于32个字节，那么redis字符串对象的编码将设置为REDIS_ENCODING_EMBSTR。

    如果字符串对象保存的值是一个字符串值，并且字符串的长度大于32个字节，那么redis字符串对象的编码将设置为REDIS_ENCODING_RAW。

    embstr编码是专门用于保存短字符串的一种优化编码方式，和raw一样，都是使用sdshdr结构来便是字符串对象，但是raw会两次调用内存分配函数分别创建redisObject和sdshdr结构，而embstr编码只会调用一次内存分配函数来分配一块连续的内存空间，依次保存redisObject和sdshdr。
---

**(2).使用embstr编码的好处是：**

    A.  相对于raw编码，只需要调用一次内存分配函数。
    
    B.  在释放embstr编码的字符串的时候，也只需要调用一次内存释放函数。
    
    C.  因为分配的是连续的内存块，相对于raw编码对象能够好的利用缓存。

---

    Ps ： long double类型的值在redis里面也是用字符串对象保存的，redis的底层会先将浮点数转换成字符串值，然后再保持在字符串对象中。
    
    在有必要的情况下，字符串值也会转换成浮点数值，执行操作完以后，再转换成字符串值保存在字符串对象中。

---

**(3) .编码的转换:**

    Int 编码和embstr编码的字符串对象在一定的情况下会转换成raw编码的字符串对象。

    Redis 没有为embstr编码的字符串编写任何相应的修改程序，所以embstr编码的字符串是只读的，如果对embstr字符串进行修改操作，就需要对embstr编码的字符串进行编码转换。

---

---

### 2. 列表对象    

**（1）.列表对象的编码可以是ziplist和linkedlist。**

---

**（2）.编码转换。**

    当列表对象可以满足两个条件时，列表对象使用ziplist编码：

    A.  列表对象保存的所有字符串元素的长度都小于64字节。

    B.  列表对象保存元素的数量小于512个。

---

Ps：这两种情况之外，列表对象都使用linkedlist编码。两个条件的上限值都是可以修改的，见配置文件中list-max-ziplist-value 和 list-max-ziplist-entries 选项的说明。

---  
---

### 3.  哈希对象

**（1）.哈希对象的编码可以是ziplist 或者hashtable。**

---

**（2）.编码转换。**

    当哈希对象可以同时满足以下两个条件时，使用ziplist编码：

    A . 哈希对象保存的所有键值对的键和值的字符串都小于64字节。
    
    B．哈希对象保存的键值对的数量小于512个。

---

Ps：以上两种情况不满足的话，就使用hashtable编码。键的长度和值得长度过长，键值对的数量过多，都会使用hashtable编码。

---
---

### 4.集合对象

**（1）.集合对象的编码可以是intset或hashtable。**

---

**（2）编码使用场景。**

    A．如果集合对象保存的所有元素都是整数值；

    B．集合对象保存的元素数量不超过512个。

---

Ps：不能满足这两个条件的集合对象都需要使用hashtable编码。第二个条件的上限值是可以在配置文件中set-max-inset-entries 中修改的。

---
---

### 5.有序集合对象


**（1）.有序集合的编码可以是ziplist或skiplist 。**

---

**（2）.编码转换。**


    使用ziplist的条件：

    A. 有序集合保存的元素数量小于128个。
    
    B．有序集合保存的所有元素成员的长度都小于64字节。

---

Ps ：不能满足条件的有序集将使用skiplist编码，两个条件的上限值可以修改，zset-max-ziplist-entries 和zset-max-ziplist-value 。

---
---

### Redis共享对象

    整数类型的值可以共享对象，共享对象需要两个步骤：

    1：将数据库的值指针指向一个现有的值对象。

    2：将被共享的值对象的引用计数加1 。

---

ps ：Redis会在初始化服务器时，创建1万个字符串对象，包含0~9999的所有整数值。

---

    为什么redis不共享包含字符串的对象？

    因为程序在判断一个共享对象时，需要判断是否跟目标对象完全相同，而一个共享对象保存的值越复杂，验证共享对象和目标对象是否相同的复杂度就越高，消耗的CPU时间也就越多。

---    



