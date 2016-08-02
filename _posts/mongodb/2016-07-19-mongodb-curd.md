---
layout: post
title: "mongdb curd操作"
description: ""
category: mongodb
tags: []
---

### 插入操作

---
**创建或插入操作即向 collection 添加新的 documents.如果插入时集合不存在,插入操作会创建该集合。**

提供的方法：

* db.collection.insert()
* db.collection.insertOne()     New in version 3.2
* db.collection.insertMany()    New in version 3.2


#### 1.db.collection.insertOne()

```
demo :

    db.users.insertOne(
       {
          name: "sue",
          age: 19,
          status: "P"
       }
    )

返回值：

    {
       "acknowledged" : true,
       "insertedId" : ObjectId("5742045ecacf0ba0c3fa82b0")
    }

查询：

    db.users.find( { _id: ObjectId("5742045ecacf0ba0c3fa82b0") } )

```

#### 2.db.collection.insertMany()

ps：3.2新版的功能，批量插入操作。

```
demo ：

    db.users.insertMany(
       [
         { name: "bob", age: 42, status: "A", },
         { name: "ahn", age: 22, status: "A", },
         { name: "xi", age: 34, status: "D", }
       ]
    )

返回值：

    {
       "acknowledged" : true,
       "insertedIds" : [
          ObjectId("57420d48cacf0ba0c3fa82b1"),
          ObjectId("57420d48cacf0ba0c3fa82b2"),
          ObjectId("57420d48cacf0ba0c3fa82b3")
       ]
    }

查询：

    db.users.find(
       { _id:
          { $in:
             [
                ObjectId("57420d48cacf0ba0c3fa82b1"),
                ObjectId("57420d48cacf0ba0c3fa82b2"),
                ObjectId("57420d48cacf0ba0c3fa82b3")
             ]
          }
       }
    )
```

#### 3.db.collection.insert()

ps：该方法既支持单条数据插入，也支持批量插入，该方法没有版本要求。

单条插入操作：

```
demo :

    db.users.insert(
       {
          name: "sue",
          age: 19,
          status: "P"
       }
    )

返回值：

    WriteResult({ "nInserted" : 1 })
```
多条插入操作：

```
demo:

    db.users.insert(
       [
         { name: "bob", age: 42, status: "A", },
         { name: "ahn", age: 22, status: "A", },
         { name: "xi", age: 34, status: "D", }
       ]
    )

返回值：

    BulkWriteResult({
       "writeErrors" : [ ],
       "writeConcernErrors" : [ ],
       "nInserted" : 3,
       "nUpserted" : 0,
       "nMatched" : 0,
       "nModified" : 0,
       "nRemoved" : 0,
       "upserted" : [ ]
    })
```
---

### 更新操作

---
**更新操作修改 collection 中已经存在的 documents.MongoDB提供了以下方法去更新集合中的文档**

mongo提供的api：

* db.collection.update()
* db.collection.updateOne()  New in version 3.2
* db.collection.updateMany() New in version 3.2
* db.collection.replaceOne() New in version 3.2

<div><img src="{{BASE_PATH}}/assets/imgs/20160802150521.png" alt="update" /></div>

```
demo1:

    db.users.updateOne(
       { "favorites.artist": "Picasso" },
       {
         $set: { "favorites.food": "pie", type: 3 },
         $currentDate: { lastModified: true }
       }
    )

demo2:

    db.users.updateMany(
       { "favorites.artist": "Picasso" },
       {
         $set: { "favorites.artist": "Pisanello", type: 3 },
         $currentDate: { lastModified: true }
       }
    )
```
使用db.collection.update() 操作

```
demo1 :

    db.users.update(
       { "favorites.artist": "Pisanello" },
       {
         $set: { "favorites.food": "pizza", type: 0,  },
         $currentDate: { lastModified: true }
       }
    )

更新多条demo：

    db.users.update(
       { "favorites.artist": "Pisanello" },
       {
         $set: { "favorites.food": "pizza", type: 0,  },
         $currentDate: { lastModified: true }
       },
       { multi: true }
    )
```
---

### 删除操作

---

**删除操作即从集合中删除文档.MongoDB提供了如下方法删除集合中的文档:**

* db.collection.insert()
* db.collection.insertOne() New in version 3.2
* db.collection.deleteMany() New in version 3.2

在MongoDB中,删除作用于单个集合.MongoDB中所有的写操作在单个 document 层级上是 atomic.

#### 1.db.collection.deleteOne()

```
demo:
    
    db.users.deleteOne( { status: "D" } )

return :
    
    { "acknowledged" : true, "deletedCount" : 1 }

eq:

    db.users.remove( { status: "D" }, 1)

```

#### 2.db.collection.deleteMany()

```
demo:

    db.users.deleteMany({})

return :

    { "acknowledged" : true, "deletedCount" : 7 }

eq :

    db.users.remove({})

```
---

### 查询操作

---

基本用法：**db.collection.find( \<query filter\>, \<projection\> )**

```
demo:
    db.users.find(
       {
         status: "A",
         $or: [ { age: { $lt: 30 } }, { type: 1 } ]
       }
    )

Query on Arrays:

    db.users.find( { badges: [ "blue", "black" ] } )

单元素满足条件:

    db.users.find( { finished: { $elemMatch: { $gt: 15, $lt: 20 } } } )

```
