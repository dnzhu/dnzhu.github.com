---
layout: page
title: "Mongodb 查询条件及优化"
description: ""
---

## mongo 的数据库操作

### 创建数据库

```
use DATABASE_NAME
```
如果数据库不存在，则创建数据库，否则切换到指定数据库。

----

### 查看所有数据库

```
show dbs
```
demo:

```
> show dbs
local  0.078GB
test   0.078GB
> 
```
----

### 删除数据库

```
db.dropDatabase()
```

ps：先使用use 命令，切换到当前数据库，再执行dropDatabase命令，删除。

----

### 删除集合
```
db.collection.drop()    
```
----

## mongodb的查询条件

### AND 条件

MongoDB 的 find() 方法可以传入多个键(key)，每个键(key)以逗号隔开，及常规 SQL 的 AND 条件。

```
db.col.find({key1:value1, key2:value2}).pretty()
```
---

### OR 条件

MongoDB OR 条件语句使用了关键字 $or,语法格式如下：

```
db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

---

### AND 和 OR 联合使用

demo:

```
db.col.find({"nums": {$gt:50}, $or: [{"cate": "教程"},{"title": "MongoDB 教程"}]}).pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "cate" : "教程",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "nums" : 100
}

相当于：
sql :  "where nums>50 AND (cate = '教程' OR title = 'MongoDB 教程')";
```

---

### Limit() 方法

```
db.col.find().limit(number)
```

### Skip() 方法

```
db.col.find().limit(number).skip(number)
```

### sort()方法

**sort()方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而-1是用于降序排列。**

```
db.COLLECTION_NAME.find().sort({KEY:1})     asc
db.COLLECTION_NAME.find().sort({KEY:-1})    desc
```
---

## MongoDB 索引

### ensureIndex() 方法

```
db.COLLECTION_NAME.ensureIndex({KEY:1})
```

可以设置使用多个字段创建索引(关系型数据库中称作复合索引)：

```
db.col.ensureIndex({"title":1,"description":-1})
```
创建唯一索引：

```
db.col.ensureIndex({name:1}, {unique: true})
```
---

## MongoDB 聚合

### aggregate() 方法

MongoDB中聚合(aggregate)主要用于处理数据(诸如统计平均值,求和等)，并返回计算后的数据结果。

```
db.col.aggregate(aggregate_option)
```
demo:

```
db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
{
   "result" : [
      {
         "_id" : "w3cschool.cc",
         "num_tutorial" : 2
      },
      {
         "_id" : "Neo4j",
         "num_tutorial" : 1
      }
   ],
   "ok" : 1
}

类似sql语句：
 select by_user, count(*) from mycol group by by_user
```
下表展示了一些聚合的表达式:

|表达式 | 描述 |
| ----- | ---- |
| $sum  \||  计算总和。|
| $avg  \||  计算平均值。 |
| $min  \||  获取集合中所有文档对应值得最小值。|
| $max   \|| 获取集合中所有文档对应值得最大值。|
| $push  \|| 在结果文档中插入值到一个数组中。  |
| $addToSet \||  在结果文档中插入值到一个数组中，但不创建副本。|
| $first \|| 根据资源文档的排序获取第一个文档数据。|
| $last  \|| 根据资源文档的排序获取最后一个文档数据。|


**管道的概念**

Linux中一般用于将当前命令的输出结果作为下一个命令的输入参数。

MongoDB的聚合管道将MongoDB文档在一个管道处理完毕后将结果传递给下一个管道处理。管道操作是可以重复的。

聚合框架中常用的几个操作：

* $project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
* $match：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
* $limit：用来限制MongoDB聚合管道返回的文档数。
* $skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
* $unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
* $group：将集合中的文档分组，可用于统计结果。
* $sort：将输入文档排序后输出。
* $geoNear：输出接近某一地理位置的有序文档。

1、$project实例

```
db.article.aggregate(
{ $project : {
    _id : 0 ,
    title : 1 ,
    author : 1
}});
```

2.$match实例

```
db.articles.aggregate( [
    { $match : { score : { $gt : 70, $lte : 90 } } },
    { $group: { _id: null, count: { $sum: 1 } } }
] );
```
ps :$match用于获取分数大于70小于或等于90记录，然后将符合条件的记录送到下一阶段$group管道操作符进行处理。

3.$skip实例

```
db.article.aggregate(
    { $skip : 5 }
);
```
$skip管道操作符处理后，前五个文档被"过滤"掉。 

---
