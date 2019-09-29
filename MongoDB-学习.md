title: MongoDB 学习
author: Elijah Zheng
tags:
  - Mongo DB
  - 数据库
categories: []
date: 2018-01-29 14:59:00
---
1.进入数据库
```shell
$ mongo
```
不进行任何操作的时候，默认使用的数据库为``test``

2.创建或切换数据库	
``` shell
> use learndb
switched to db learndb
```

<!--more-->

3.查看存在的数据库（只显示有数据的数据库）
``` shell
> show databases 
or
> show dbs
admin    0.000GB
config   0.000GB
```
当向数据库`learndb`中插入数据时，`show dbs`才会显示`learndb`。

4.查看当前使用的数据库
``` shell 
> db
learndb
```

5.删除当前的数据库
``` shell
db.dropDatabase()
{ "ok" : 1 }
```

6.创建数据库集合
``` shell
> db.createCollection('myCol')
{ "ok" : 1 }
```
创建时可以带参数`options`

options参数是可选的，因此只需要指定集合的名称。 以下是可以使用的选项列表：

字段	|类型	|描述
----|----|----
capped	| Boolean	|(可选)如果为true，则启用封闭的集合。上限集合是固定大小的集合，它在达到其最大大小时自动覆盖其最旧的条目。 如果指定true，则还需要指定size参数。
autoIndexId	|Boolean|(可选)如果为true，则在_id字段上自动创建索引。默认值为false。
size|数字|(可选)指定上限集合的最大大小(以字节为单位)。 如果capped为true，那么还需要指定此字段的值。
max|数字|(可选)指定上限集合中允许的最大文档数。

例如：
```shell
> db.createCollection("myCol2", {capped : true, autoIndexId : true, size : 6142800, max : 10000 })
{
	"note" : "the autoIndexId option is deprecated and will be removed in a future release",
	"ok" : 1
}
```

7.查看创建的集合
``` shell
> show collections
myCol
myCol2
```

8.删除指定的集合 ``db.collection.drop()``
```shell
> db.myCol2.drop()
true
```

9.插入文档 ``db.collection.insert({})``
```shell
> db.myCol.insert({
	title: 'learn mongodb', 
	url: 'https://blog.zhengxiangling.com',
    view: '99'
})
WriteResult({ "nInserted" : 1 })
```
其他插入方法
+ db.post.save(document)
+ db.collection.insertOne()
+ db.collection.insertMany()	

10.查询集合 ``db.collection.find()``
``` shell
> db.myCol.find()
{ "_id" : ObjectId("5a6eda148517fad4b57bd75e"), "title" : "learn mongodb", "url" : "https://blog.zhengxiangling.com", "view" : "99" }
```

结合``preey()``方法，格式化输出
```shell
> db.myCol.find().pretty()
{
	"_id" : ObjectId("5a6eda148517fad4b57bd75e"),
	"title" : "learn mongodb",
	"url" : "https://blog.zhengxiangling.com",
	"view" : "99"
}
```

加参数的查找

操作|语法|示例|RDBMS等效语句
----|----|----|----
相等|``{<key>:<value>}``|``db.mycol.find({"by":"yiibai"}).pretty()``|``where by = 'yiibai'``
小于|``{<key>:{$lt:<value>}}``|``db.mycol.find({"likes":{$lt:50}}).pretty()``|``where likes < 50``
小于等于|``{<key>:{$lte:<value>}}``|``db.mycol.find({"likes":{$lte:50}}).pretty()``|``where likes <= 50``
大于|``{<key>:{$gt:<value>}}``|``db.mycol.find({"likes":{$gt:50}}).pretty()``|``where likes > 50``
大于等于|``{<key>:{$gte:<value>}}``|``db.mycol.find({"likes":{$gte:50}}).pretty()``|``where likes >= 50``
不等于|``{<key>:{$ne:<value>}}``|``db.mycol.find({"likes":{$ne:50}}).pretty()``|``where likes != 50``

结合``and``或者``or``查找
```shell
>db.mycol.find(
   {
      $and: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()

>db.mycol.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

设置查找到的文档要显示出的字段，如只显示``_id``和``title``，``1``表示显示
```shell
db.mycol.find({}, {'_id':1, 'title':1})
```

11.更新文档 ``update``
```shell
> db.myCol.update({title: 'learn mongodb'}, {$set: {title: 'update title'}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```
默认只会修改匹配到的第一个文档，若要修改所有匹配到的文档，将参数``multi``设置为``true``
``` shell
> db.myCol.update({title: 'learn mongodb'}, {$set: {title: 'update title'}, {multi: true}})
```

12.删除文档 ``remove``

删除匹配到的所有文档
```shell
db.myCol.remove({view: '99'})
```

如果想只删除匹配到的第一条文档，加参数``1``
```shell
db.myCol.remove({view: '99'}, 1)
```

把集合下的所有文档都删除
```shell
db.myCol.remove()
```

13.限制记录数 ``limit()``,``skip()``	
比如查询到的记录有10条，只想要前5条，可以用``limit()``
```shell
db.myCol.find().limit(5)
```

比如查询出的数据，不想要前3条，可以调过前三条的文档，可以用``skip()``
```shell
db.myCol.find().skip(3)
```

14.排序记录 ``sort``
可以多查询出的文档记录进行排序,``1``代表升序，``-1``代表降序。
```shell
db.myCol.find().sort({'_id': 1})

db.myCol.find().sort({'_id': -1})
```