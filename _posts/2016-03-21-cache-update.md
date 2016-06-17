---
layout: post
title: "更新缓存的几种方式"
date: 2016-03-21 12:06
category: php-basic
tags: []
---

>ps:该文摘自于忽然之间的技术博客

> 总结最常用到的几个缓存更新的方法， 使用伪代码来说明一下。 

### 1)周期时长之后缓存过期，触发更新。
最常规的一种方式，在缓存层一般要这样做。 

```
if  (false === cache::get(key)) {
        cache::set(key, val, ttl);
}
```

### 2) 主动事件更新 
比如：商品的价格或文章的标题更新了， 我们如何更新缓存呢？

一般由 事件 => 动作 来完成缓存更新。

事件：比如后台对id =1 的数据进行了更新

```
sql = "update tbl set column= 'val' WHERE id='1' ";
```

动作：

```
if (update) {
    updateCache(tbl, 1);    //主动执行更新函数
}

function updateCache(tbl, id) 
{
     val = db::get(tbl, 1);
     cache::set(key, val, ttl);
}
```

这种方式对一些单表字段比较有效。
至于updateCache 可以是如上，也可以发送到消息队列，交由专门的服务器程序完成。过程化还是异步 ，根据自己的应用自行设计。 


### 3）缓存依赖

比如我们按最新上架查询商品，如果商品上架后要在及时更新到最新列表里， 这个缓存如何更新呢？

我们来使用一种条件缓存的方式，是第2种方式的一种扩展。建立依赖条件，条件对比来触发更新。

可以设置一个标记，来表明我们的业务端有了新的操作，需要删除缓存或更新缓存。 

```
update_at = time();

update_at = cache::get('news_list_update');    //缓存依赖这个条件。
key = 'news_list';

if (false === (result = cache::get(key)) || (update_at && (result['update_at'] < update_at))) {
    //方法1 或方法2 更新缓存
    // result
    result = db::getlist();    //最新的ID
    result['update_at'] = time();    //我们把当前时间戳作为一个条件。请结合自己的业务来考虑。
}
```

> ps:还有一些方式比如后端主动写缓存、或使用nosql直接取热数据等就不详细展开了。


