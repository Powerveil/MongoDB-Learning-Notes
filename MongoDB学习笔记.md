MongoDB学习笔记



# 基础语法

## 使用 insert 完成插入操作

操作格式：

- db.<集合>.insertOne(<JSON对象>)
- db.<集合>.insertMany([<JSON 1>,<JSON 2>,...<JSON n>])

```sql
示例：
db.fruit.insertOne({
  "name": "apple"
})
db.fruit.insertMany([
  {"name": "apple"},
  {"name": "pear"},
  {"name": "orange"}
])
```



## 使用 find 查询文档

关于 fiind:

- find 是 MongoDB 中查询数据的基本指令，相当于 SQL 中的 SELECT
- find 返回的是游标

```sql
示例：
db.movies.find({"year": 1975}) // 单条件查询
db.movies.find({"year": 1989,"title": "Batman"}) // 多条件and查询
db.movies.find({$and:[{"title": "Batman"},{"category": "action"}]}) // and的另一种形式
db.movies.find({$or:[{"year": 1989},{"title": "Batman"}]}) // 多条件or查询
db.movies.find({"title": /^B/}) // 按正则表达式查找
```



### 查询逻辑运算符

- $lt: 存在
- $lte: 存在并且小于等于
- $gt: 存在并大于
- $gte: 存在并大于等于
- $ne: 不存在或存在但不等于
- $in: 存在并在指定数组中
- $nin: 不存在或不在指定数组中
- $or: 匹配两个或多个条件中的一个
- $and: 匹配全部条件



### 使用 find 搜索子文档

- find 支持使用 "field.sub_field" 的形式查询子文档。假设有一个文档：

```sql
db.fruit.insertOne({
  "name": "apple",
  "from": {
    "country": "China",
    "province": "Guangdon"
  }
})
```



### 考虑一下的查询的意义

```sql
db.fruit.find({"from.country": "China"}) // 查询from.country字段为 China的文档
db.fruit.find({"from": {"country": "China"}}) // 查询from字段为"country": "China"的文档
```



### 使用 find 搜索数组中的对象

- 考虑一下文档，在其中搜索

```sql
db.movies.insertOnd({
  "title": "Raiders of the Lost Ark",
  "filming_locations": [
    {
      "city": "Los Angeles",
      "state": "CA",
      "country": "USA"
    },
    {
      "city": "Rome",
      "state": "Lazio",
      "country": "Italy"
    },
    {
      "city": "Florence",
      "state": "SC",
      "country": "USA"
    }
  ]
})
```



```sql
// 查找城市是 Rome 的记录
db.movies.find({"filming_locations.city": "Rome"})
```



### 在数组中搜索子对象的多个字段时，如果使用 $elemMatch，它表示必须是同一个子对象满足多个条件。考虑一下两个查询：

```sql
db.getCollection('movies').find({
  "filming_locations.city": "Rome",
  "filming_locations.country": "USA"
})
// 使用elemMatch
db.getCollection('movies').find({
  "filming_locations": {
    "$elemMatch": {
      "city": "Rome",
      "country": "USA"
    }
  }
})
```



### 控制 find 返回的字段

- find 可以指定只返回指定的字段;

- _id字段必须明确指明不返回，否则默认返回;

- 在 MongoDB 中我们称这个为投影(projection);

- ```sql
  db.movies.find({"category": "action"},{"_id": 0, "title": 1})
  //                                     不返回_id   返回title
  ```



## 使用 remove 删除文档

- remove 命令血药配合查询条件使用;
- 匹配查询条件的文档会被删除;
- 指定一个空文档条件会删除全部文档;

```sql
db.testcol.remove({a: 1}) // 删除a等于1的记录
db.testcol.remove({a: {$lt: 5}}) // 删除a小于5的记录
db.testcol.remove({}) // 删除所有记录
db.testcol.remove() // 报错
```



## 使用 update 更新数组

- update 操作执行格式：db.<集合>.update(<查询条件>,<更新字段>)

```sql
db.fruit.updateOne({"name": "apple"},{$set: {from: "China"}})
```

![image-20221230224816753](D:/程序/Typora图片存放地/images/image-20221230224816753.png)



- 使用 updateOne 表示无论条件匹配多少条记录，始终只更新**第一条**;

- 使用 updateMany 表示条件匹配多少条就更新多少条;

- updateOne/updateMany 方法要求更新条件部分必须具有一下之一，否则报错：

  - $set/$unset
  - $push/$pushAll/$pop
  - $pull/$pullAll
  - $addToSet

- ```sql
  // 报错
  db.fruit.updateOne({"name": "apple"}, {"from": "China"})
  ```



- $push: 增加一个对象到数组底部
- $pushAll: 增加多个对象到数组底部
- $pop: 从数组底部删除一个对象
- $pull: 如果匹配指定的值，从数组中删除对应的对象
- $pullAll: 如果匹配任意的值，从数据中删除相应的对象
- $addToSet: 如果不存在则增加一个值到数组



## 使用 drop 删除集合

- 使用db.<集合>.drop()来删除一个集合
- 集合中的全部文档都会被删除
- 集合相关的索引也会被删除

```sql
db.colToBeDropped.drop()
```



## 使用 dropDatabase 删除数据库

- 使用 db.dropDatabase() 来删除数据库
- 数据库相应文件也会被删除，磁盘空间被释放

```sql
use tempDB
db.dropDatabase()
show collections // No collections
show abs // The db is gone
```



# 开发第一个Hello World 程序

## 1. 安装 Python MongoDB 驱动程序

**linux 默认安装 Python，我的默认安装2.7.5版本**

### 安装 MongoDB 驱动

在 Python 中使用 MongoDB 之前必须先安装用于访问数据库的驱动程序

```sql
pip install pymongo
```

![image-20221231112337985](D:/程序/Typora图片存放地/images/image-20221231112337985.png)

### 检查驱动程序

在 Python 交互模式导入 pymongo，检查驱动是否已正确安装：

```sql
import pymongo
pymongo.version
```

![image-20221231112318455](D:/程序/Typora图片存放地/images/image-20221231112318455.png)

## 2. 创建连接

## 确定 MongoDB 字符串

使用驱动连接到 MongoDB 集群只需要指定 MongoDB 连接字符串即可。其基本格式可以参考文档: [Connection String URI Format](www.baidu.con "看看官方文档")(这个链接没有补充)

```sql
mongodb://数据库服务器主机地址:端口号

mongodb://127.0.0.1:27017
```

## 初始化数据库连接

```sql
from pymongo import MongoClient
uri = "mongodb://127.0.0.1:27017"
client = MongoClient(uri)
print client
```

![image-20221231113415056](D:/程序/Typora图片存放地/images/image-20221231113415056.png)



## 3. 数据库操作：插入用户

### 初始化数据库和集合

```sql
db = client["eshop"]
user_coll = db["users"]
```

![image-20221231114104975](D:/程序/Typora图片存放地/images/image-20221231114104975.png)

我们拿到了数据库和集合，但是，MongoDB 数据库中还没有创建该数据库和集合。

![image-20221231114145494](D:/程序/Typora图片存放地/images/image-20221231114145494.png)

### 插入一条新的用户数据

```sql
new_user = {"username": "power", "password": "111111", "email": "1968103220@qq.com"}
result = user_coll.insert_one(new_user)
print result
```

**注意：我们并没有提前创建数据库和表/集合**

![image-20221231114407798](D:/程序/Typora图片存放地/images/image-20221231114407798.png)

此时数据库表都已经插入。

![image-20221231114517343](D:/程序/Typora图片存放地/images/image-20221231114517343.png)



**_id 是 Python 驱动程序自动创建的，如果不提供该字段，MongoDB 会新增一个 ObjectId 类型的数据，保证数据的唯一性。这个是 MongoDB 的主键。**

**我们通过程序就自动创建数据库和表，所以 MongoDB 被成为 "无模式"(不需要预先创建模式这种模式来操作)**





## 4. 更新用户

### 需求变化，要求修改用户属性，增加字段 phone

```sql
result = user_coll.update_one({"username": "power"},{"$set": {"phone": "123456789"}})
print result
```

**注意：我们并没有去数据库修改表结构**

**动态特性**







# MongoDB 聚合查询

## 什么是 MongoDB 聚合框架

- MongoDB 聚合框架 (Aggregation Framework) 是一个计算框架，它可以：
  - 作用在一个或几个集合上;
  - 对集合中的数据进行的一系列运算;
  - 将这些数据转化为期望的形式;

- 从效果而言，聚合框架相当于 SQL 查询中的：
  - GROUP BY
  - LEFT OUTER JOIN
  - AS等





## MongoDB 中的聚合

### 管道 (Pipeline) 和步骤 (Stage)

- 整个聚合运算过程成为管道 (Pipeline)，它是由多个步骤 (Stage) 组成的，

  每个管道：

  - 接受一系列文档 (原始数据);
  - 每个步骤对这些文档进行一系列运算;
  - 结果文档输出给下一个步骤;

![image-20221231150116061](D:/程序/Typora图片存放地/images/image-20221231150116061.png)



## 聚合运算的基本格式

```sql
pipeline = {$stage1, $stage2, ...$stageN}

db.<COLLECTION>.aggregate(
	pipeline,
	{ options }
)
```



### 常见步骤

| 步骤         | 作用     | SQL等价运算符   |
| ------------ | -------- | --------------- |
| $match       | 过滤     | WHERE           |
| $project     | 投影     | AS              |
| $sort        | 排序     | ORDER BY        |
| $group       | 分组     | GROUP BY        |
| $skip/$limit | 结果限制 | SKIP/LIMIT      |
| $lookup      | 左外连接 | LEFT OUTER JOIN |
|              |          |                 |

![image-20230101104136317](D:/程序/Typora图片存放地/images/image-20230101104136317.png)



![image-20230101104253768](D:/程序/Typora图片存放地/images/image-20230101104253768.png)





## 聚合运算的使用场景

- 聚合查询可以用于OLAP和OLTP场景。例如：

  | OLAP | OLTP                                                         |
  | ---- | ------------------------------------------------------------ |
  | 计算 | 分析一段时间内的销售总额、均值<br/>计算一段时间内的净利润<br/>分析购买人的年龄分布<br/>分析学生成绩分布<br/>统计员工<br/> |

**数据处理一般可以分成两大类：联机分析处理OLAP（OnLine Analytical Processing）和联机事务处理OLTP（OnLine Transaction Processing）。**

- OLAP为使用多维结构为分析提供对数据的快速访问的技术，OLAP 的源数据通常存储在关系数据库的数据仓库中。OLAP是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。
- OLTP也称为面向交易的处理系统，其基本特征是顾客的原始数据可以立即传送到计算中心进行处理，并在很短的时间内给出处理结果。OLTP是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易。





### MQL 常用步骤与 SQL 对比

![image-20230101110306381](D:/程序/Typora图片存放地/images/image-20230101110306381.png)



![image-20230101110727702](D:/程序/Typora图片存放地/images/image-20230101110727702.png)

### MQL 特有步骤 $unwind

**将数组展开，做一些报表的时候使用。**

![image-20230101111228311](D:/程序/Typora图片存放地/images/image-20230101111228311.png)



### MQL 特有步骤 $bucket

电商业务或产品目录非常有用。

![image-20230101111530424](D:/程序/Typora图片存放地/images/image-20230101111530424.png)

### MQL 特有步骤 $facet

**产品展现和目录展现非常有用。**

![image-20230101111857031](D:/程序/Typora图片存放地/images/image-20230101111857031.png)







```sql
db.orders.aggregate([
    { $group:
    	{
    	_id: null,
    	total: { $sum: "$to"}
    }
    }
])
```



# 复制集机制及原理

## 复制集的作用

- MongoDB 复制集的主要意义在于实现服务高可用
- 它的现实依赖于两个方面的功能：
  - 数据写入时将数据迅速复制到两一个独立节点上
  - 在接受写入的节点发生故障时自动选举出一个新的替代节点



- 在实现高可用的同时，复制集实现了其他几个附加作用：
  - 数据分发：将数据从一个区域复制到另一个区域，减少另一个区域的读延迟
  - 读写分离：不同类型的压力分别在不同的节点上执行
  - 异地容灾：在数据中心故障时候快速切换到异地



## 典型复制集结构

- 一个典型的复制集由3个以上具有投票权的节点组成，包括：
  - 一个主节点(PRIMARY)：接受写入操作和选举时投票
  - 两个(或多个)从节点(SECONDARY)：复制主节点上的数据和选举时投票
  - 不推荐使用 Arbiter(投票节点)

![image-20230101161941205](D:/程序/Typora图片存放地/images/image-20230101161941205.png)



## 数据是如何复制的？

- 当一个修改操作，无论是插入，更新或删除，到达主节点时，它对数据的操作将被记录下来(经过一些必要的转换)，这些记录成为 oplog。
- 从节点通过在主节点上打开一个 tailable 游标不断获取新进入主节点的 oplog，并在自己的数据上回放，以此保持跟主节点的数据一致

![image-20230101162712040](D:/程序/Typora图片存放地/images/image-20230101162712040.png)



## 通过选举完成故障修复

- 具有投票权的节点之间相互发送心跳;
- 当5次心跳未受到时判断为节点失联;
- 如果失联的是主节点，从节点会发起选举，选举新的主节点;
- 如果失联的是从节点则不会产生新的选举;
- 选举基于RAFT一致性算法实现，选举成功的必要条件是大多数投票节点存活;
- 复制集中最多可以有50个节点，但具有投票权的节点最多7个。

![image-20230101163918870](D:/程序/Typora图片存放地/images/image-20230101163918870.png)



### 影响选举的因素

- 整个集群必须有大多数节点存活;
- 被选举为主节点的节点必须：
  - 能够与多数节点建立连接
  - 具有较新的 oplog
  - 具有较高的优先级 (如果有配置)

### 常见选项

- 复制集节点有以下常见的选配项：
  - 是否具有投票权(v 参数)：有则参与投票;
  - 优先级(priority 参数)：优先级越高的节点越优先成为主节点。优先级为0的节点无法成为主节点;
  - 隐藏(hidden 参数)：复制数据，但对应用不可见。隐藏节点可以具有投票权，但优先级必须为0;
  - 延迟(slaveDelay 参数)：复制 n 秒之前的数据，保持与主节点的时间差

![image-20230101164706557](D:/程序/Typora图片存放地/images/image-20230101164706557.png)



### 复制集注意事项

- 关于硬件：
  - 因为正常的复制集节点都有可能成为主节点，它们的地位是一样的，因此硬件配置上必须一致;
  - 为了保证节点不会同时宕机，各节点使用的硬件必须具有独立性。

- 关于软件：
  - 复制集个节点软件版本必须一致，以避免出现不可预知的问题。

- 增加节点不会增加系统写性能！

## 搭建一个复制集

- 本实验中，我们通过在一台机器上运行3个实例来搭建一个最简单的复制集。通过实验，我们将学会：
  - 如何启动一个 MongoDB 实例
  - 如何将3个 MongoDB 实例搭建成一个复制集
  - 如何对复制集直接进行参数做一些常规调整



### 1. 准备

- 安装最新的 MongoDB 版本
- Windows 系统请事先配置好 Mongo 可执行文件的环境变量
- Linux 和 Mac 系统请配置 PATH 变量
- 确保有 10GB 以上的硬盘空间



































































































