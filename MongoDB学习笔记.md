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



### 2. 创建数据目录

MongoDB 启动时将使用一个数据目录存放所有数据文件。我们将为3个复制集节点创建各自的数据目录。

Linux/MacOS：

```
mkdir -p /data/db{1,2,3}
```

Windows：

```
md d:\data\db1
md d:\data\db2
md d:\data\db3
```



### 3. 准备配置文件

复制集的每个 mongod 进程应该位于不同的服务器。我们现在在一台机器上运行3个进程，因此要为他们各自配置：

- 不同的端口。实例中将使用28017/28018/28019,

- 不同的数据目录。示例中将使用：

  ```
  /data/db1或c:\data\db1
  /data/db2或c:\data\db2
  /data/db3或c:\data\db3
  ```

- 不同的日志文件路径。示例中将使用：

  ```
  data/db1/mongod.log或c:\data\db1\mongod.log
  data/db2/mongod.log或c:\data\db2\mongod.log
  data/db3/mongod.log或c:\data\db3\mongod.log
  ```

  

#### 配置文件

Linux/MacOS：

```yaml
# /data/db1/mongod.conf
systemLog:
  destination: file
  path: /data/db1/mongo.log # log path
  logAppend: true
storage:
  dbPath: /data/db1 # data directory
net:
  bindIp: 0.0.0.0
  port: 28017 # port
replication:
  replSetName: rs0
processManagement:
  fork: true
```





Windows：

```yaml
# c:\data\db1\mongod.conf
systemLog:
  destination: file
  path: c:\data1\mongod.log # 日志文件路径
  logAppend: true
storage:
  dbPath: c:\data1 # 数据目录
net:
  bindIp: 0.0.0.0
  port: 28017 # 端口
replication:
  replSetName: rs0
```



### 4. 启动 MongoDB 进程

Linux/MacOS：

```
mongod -f db1/mongod.conf
mongod -f db2/mongod.conf
mongod -f db3/mongod.conf
```

注意：如果启用了SELInux，可能阻止上述进程启动。简单起见请关闭SELinux。

Windows：

```
mongod -f c:\data1\mongod.conf
mongod -f c:\data2\mongod.conf
mongod -f c:\data3\mongod.conf
```

因为 Windows 不支持 fork，以上命令需要在3个不同的窗口执行，执行后不可关闭，否则进程将直接结束。



### 5. 配置复制集

方法1

```
# mongo --port 28017
rs.initiate()
rs.add("HOSTNAME:28018")
rs.add("HOSTNAME:28019")
```

注意：此方式 hostname 需要能被解析

方法2：

```

# mongo -- port 28017
rs.initiate({
    _id: "rs0",
    members: [{
        _id: 0,
        host: "localhost:28017"
    }, {
        _id: 1,
        host: "localhost:28018"
    }, {
        _id: 2,
        host: "localhost:28019"
    }]
})
```





###  6. 验证

MongoDB 主节点进行写入

```sql
# mongo localhost:28017
db.test.insert({ a: 1})

db.test.insert({ a: 1})
```



MongoDB 从节点进行读

```sql
# mongo localhost:28018
rs.slaveOk()
db.test.find()
```



![image-20230102115632611](D:/程序/Typora图片存放地/images/image-20230102115632611.png)



# MongoDB 全家桶



| 软件模块                  | 描述                                              |
| ------------------------- | ------------------------------------------------- |
| mongod                    | MongoDB 数据库软件                                |
| mongo                     | MongoDB 命令行工具，管理 MongoDB 数据库           |
| mongos                    | MongoDB 路由进程，分片环境下使用                  |
| mongodump / mongorestore  | 命令行数据库备份与恢复工具                        |
| mongoexport / mongoimport | CSV / JSON 导入与导出，主要用于不同系统同数据迁移 |
| Compass                   | MongoDB GUI 管理工具                              |
| Ops Manager(企业版)       | MongoDB 集群管理软件                              |
| BI Connector(企业版)      | SQL 解释器 / BI 套接件                            |
| MongoDB Charts(企业版)    | MongoDB 可视化软件                                |
| Atlas(付费及免费)         | MongoDB 云托管服务，包括永久免费云数据库          |



## mongodump . mongorestore

- 类似于 MySQL 的 dump/restore 工具
- 可以完成全库 dump: 不加条件
- 也可以根据条件 dump 部分数据： -q 参数
- Dump 的同时跟踪数据就更：--oplog
- Restore 是反操作，把 mongodump 的输出导入到 mongodb

```sql
mongodump -h 127.0.0.1:27017 -d test -c test

mongorestore -h 127.0.0.1:27017 -d test -c test xxx.bson
```





## Atlas - MongoDB 公有云托管服务

![image-20230102122120368](D:/程序/Typora图片存放地/images/image-20230102122120368.png)



## MongoDB BI Connector

![image-20230102122503258](D:/程序/Typora图片存放地/images/image-20230102122503258.png)

端口和 MySQL 一样。

**不支持写入，它的目的不是用 SQL 取代 MQL，这不是它的初衷**



## MongoDB Ops Manager - 集群管理平台

![image-20230102122711605](D:/程序/Typora图片存放地/images/image-20230102122711605.png)



分片集群的备份只能用这个软件，用Java写的。



# 从熟练到精通

## 数据模型

什么是数据模型？

数据模型是一组由符号、文字组成的集合，用以准确表达信息，达到有效交流、沟通的目的。

Steve Hoberman 霍伯曼.数据建模经典教程



### 数据模型设计的元素

**实体 Entity**

- 描述业务的主要数据集合
- 谁，什么，何时，何地，为何，如何

**属性 Attribute**

- 描述实体里面的单个信息

**关系 Relationship**

- 描述实体与实体之间的数据规则
- 结构规则：1~N，N~1，N~N
- 引用规则：电话号码不能单独存在

![image-20230102161021033](D:/程序/Typora图片存放地/images/image-20230102161021033.png)





### 传统模型设计：从概念到逻辑到物理





|            | 概念模型 CDM                                       | 逻辑模型 LDM                                     | 物理模型 PDM                                                 |
| ---------- | -------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| 目的       | 描述业务系统要管理的对象                           | 基于概念模型，详细列出所有实体、实体的属性及关系 | 根据逻辑模型，结合数据库的物理结构，设计具体的表结构，字段列表及主外键 |
| 特点       | 用概念名词来描述现实中的实体及业务规则，如"联系人" | 基于业务的描述和数据库无关                       | 技术实现细节和具体的数据库类型相关                           |
| 主要使用这 | 用户需求分析师                                     | 需求分析师架构师及开发者                         | 开发者DBA                                                    |



**从开发者的视角：概念模型**

![image-20230102162312487](D:/程序/Typora图片存放地/images/image-20230102162312487.png)





**从开发者的视角：逻辑模型**

![image-20230102162353010](D:/程序/Typora图片存放地/images/image-20230102162353010.png)



**从开发者的视角：第三范式下的物理模型**

![image-20230102163252598](D:/程序/Typora图片存放地/images/image-20230102163252598.png)



## MongoDB 文档模型设计的三个误区

1. **不需要模型设计**
2. **MongoDB 应该用一个超级大文档来组织所有数据**
3. **MongoDB 不支持关联或事务**

**上述陈述都不正确！！！**



### 关于文档 JSON 文档模型设计

文档模型设计处于是物理模型设计阶段(PDM)

JSON 文档模型通过内嵌数组或引用字段来表示关系

文档模型设计不遵从第三范式，允许冗余。

![image-20230102163753166](D:/程序/Typora图片存放地/images/image-20230102163753166.png)



### 为什么人们都说 MongoDB 是无模式？

严格来说，MongoDB 同样需要概念 / 逻辑建模

文档模型设计的物理层结构可以和逻辑层类似

**MongoDB 无模式由来：**

- **可以省略物理建模的具体过程**

![image-20230102164102754](D:/程序/Typora图片存放地/images/image-20230102164102754.png)



### 逻辑模型 - JSON 模型

![image-20230102164350132](D:/程序/Typora图片存放地/images/image-20230102164350132.png)



### 文档模型的设计原则：性能和易用

![image-20230102164537499](D:/程序/Typora图片存放地/images/image-20230102164537499.png)



## 关系模型 vs 文档模型



|              | 关系数据库                       | JSON 文档模型        |
| ------------ | -------------------------------- | -------------------- |
| 模型设计层次 | 概念模型<br>逻辑模型<br>物理模型 | 概念模型<br>逻辑模型 |
| 模型实体     | 表                               | 集合                 |
| 模型属性     | 列                               | 字段                 |
| 模型关系     | 关联关系，主外键                 | 内嵌数组，引用字段   |





## MongoDB 文档模型设计三部曲

![image-20230102165226708](D:/程序/Typora图片存放地/images/image-20230102165226708.png)



### 第一步：建立基础文档模型

1. 根据概念模型或者业务需求推到出逻辑模型 - 找到对象
2. 列出实体之间的关系(及基数) - 明确关系
3. 套用逻辑设计原则来决定内嵌方式 - 机型建模
4. 完成基础模型构建

![image-20230102165542103](D:/程序/Typora图片存放地/images/image-20230102165542103.png)



#### 一个联系人管理应用的例子

1. **找到对象**
   - Contacts
   - Groups
   - Address
   - Portraits

2. **确定关系**
   - 一个联系人由一个头像 **(1-1)**
   - 一个联系人可以有多个地址 **(1-N)**
   - 一个联系人可以属于多个组，一个组可以有多个联系人 **(N-N)**

![image-20230102170058295](D:/程序/Typora图片存放地/images/image-20230102170058295.png)



**1-1 关系建模：portraits**

**基本原则：**

- 一对一关系以内嵌为主
- 作为子文档形式或者直接在顶级不涉及到数据冗余

**例外情况：**

- 如果内嵌后导致文档大小超过16MB

**一个文档最多有16MB**

**MongoDB 支持二进制文件内容直接放入到 JSON 文档中，不需要任何处理**

![image-20230102171004413](D:/程序/Typora图片存放地/images/image-20230102171004413.png)

**1-N 关系建模：Addresses**

**基本原则：**

- 一对多关系同样以内嵌为主
- 用数据来表示一对多不涉及到数据冗余

**例外情况：**

- 内嵌后导致文档大小超过16MB
- 数组长度太大(数万或更多)
- 数组长度不确定

![image-20230102171023870](D:/程序/Typora图片存放地/images/image-20230102171023870.png)

**N-N 关系建模：内嵌数组模式**

**基本原则：**

- 不需要映射表
- 一遍用内嵌数组表示一对多
- 通过冗余来实现N-N

**例外情况：**

- 内嵌后导致文档大小超过16MB
- 数组长度太大(数万或更多)
- 数组长度不确定

![image-20230102171419415](D:/程序/Typora图片存放地/images/image-20230102171419415.png)



### 第二补：根据读写工况细化

![image-20230102172024808](D:/程序/Typora图片存放地/images/image-20230102172024808.png)

- 最频繁的数据查询模式
- 最常用的查询参数
- 最频繁的数据写入模式
- 读写操作的比例
- 数据量的大小



> **基于内嵌的文档模型<br>根据业务需求，<br>	使用引用来避免性能瓶颈<br>	使用冗余来优化访问性能**



#### 联系人管理应用的分组需求

1. 用于客户营销
2. 有千万级联系人
3. 需要频繁变动分组(group)<br>的信息，如增加分组及修改名称<br>及描述以及营销状态
4. 一个分组可以有百万级联系人

![image-20230102173000649](D:/程序/Typora图片存放地/images/image-20230102173000649.png)



**解决方案：Group 使用单独的集合**

1. 类似于关系型设计
2. 用 id 或者唯一键关联
3. 使用 $lookup 来提供一次查询多表<br>的能力(类似关联)

![image-20230102173223818](D:/程序/Typora图片存放地/images/image-20230102173223818.png)



**引用模式下的关联查询**

![image-20230102173402638](D:/程序/Typora图片存放地/images/image-20230102173402638.png)





#### 联系人的头像：引用模式

1. 头像使用高保真，大小在5MB-10MB
2. 头像一旦上传，一个月不可更换
3. 基础信息查询(不含头像)和头像查询的比例为9：1
4. 建议：使用引用方式，把头像数据<br>放到另一个集合，可以显著提升 90% 的查询效率

![image-20230104163029482](D:/程序/Typora图片存放地/images/image-20230104163029482.png)



**什么时候该使用引用方式？**

- 内嵌文档太大，数 MB 或者超过 16MB
- 内嵌文档或数组元素会频繁修改
- 内嵌数组会持续增长并且没有封顶



#### MongoDB 引用设计的限制

- MongoDB 对使用引用的集合之间并无主外键检查
- MongoDB 使用聚合框架的 $lookup 来模仿关联查询
- $lookup 只支持 left outer join
- $lookup 的关联莫表(from)不能是分片表

![image-20230104164004649](D:/程序/Typora图片存放地/images/image-20230104164004649.png)



### 第三步：套用设计模式

文档模型：无范式，无思维定式，充分发挥想象力

设计模式：实战过屡试不爽的设计技巧，快速引用

举例：一个 loT 场景的分桶设计模式，可以帮助存储空间吉降低10倍并且查询效率提升数10倍.

![image-20230104164517704](D:/程序/Typora图片存放地/images/image-20230104164517704.png)



#### 问题：物联网场景下的海量数据处理 - 飞机监控数据

![image-20230104164738474](D:/程序/Typora图片存放地/images/image-20230104164738474.png)



**520亿条，10TB - 海量数据**

- 10 万架飞机
- 1 年的数据
- 每分钟一条





|                      | 每分钟1条   |      |
| -------------------- | ----------- | ---- |
| 文档条数             | **52.6B**   |      |
|                      |             |      |
| 索引大小             | **6364 GB** |      |
| _id index            | 1468 GB     |      |
| {ts: 1, deviceld: 1} | 4895 GB     |      |
|                      |             |      |
| 文档平均大小         | 92 Bytes    |      |
| 数据大小             | **4503 GB** |      |



**解决方案：分桶设计**

![image-20230104165437901](D:/程序/Typora图片存放地/images/image-20230104165437901.png)





可视化表现24小时的飞行数据

1440 次读



|                      | 每分钟1条   | 每小时一个文档 |
| -------------------- | ----------- | -------------- |
| 文档条数             | **52.6 B**  | **876 M**      |
|                      |             |                |
| 索引大小             | **6364 GB** | **106 GB**     |
| _id index            | 1468 GB     | 24.5 GB        |
| {ts: 1, deviceld: 1} | 4895 GB     | 81.6 GB        |
|                      |             |                |
| 文档平均大小         | 92 Bytes    | 758 Bytes      |
| 数据大小             | **4503 GB** | **618 GB**     |



模式小结：分桶

| 场景                                       | 痛点                       | 设计模式的方案及优点                                         |
| ------------------------------------------ | -------------------------- | ------------------------------------------------------------ |
| 时序数据<br>物联网<br>智慧城市<br>智慧交通 | 数据点采集频繁，数据量太多 | 利用文档内嵌数组，将一个时<br>间段的数据聚合到一个文档里。<br><br>大量减少文档数量<br><br>大量减少索引占用空间 |



一个好的设计模式可以显著地：

- 提升数据读写的效率
- 降低资源的需求



**更多的 MongoDB 设计模式：**

| 表现形式类 | 数据访问类 | 组织结构类 |
| ---------- | ---------- | ---------- |
| 列转行     | 子集       | 预聚合     |
| 文档版本   | 近似处理   | 分桶       |



## 设计模式集锦

### 问题：大文档，很多字段，很多索引

![image-20230104170917090](D:/程序/Typora图片存放地/images/image-20230104170917090.png)



#### 解决方案：列转行

![image-20230105215052224](D:/程序/Typora图片存放地/images/image-20230105215052224.png)



**列转行**

| 场景                                                         | 痛点                                                         | 设计模式方案及优点                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------- |
| 产品属性 'color', 'size',<br>'dimensions',...<br><br>多语言(多国家)属性 | 文档中有很多类似的字段<br><br>会用于组合查询，需要建很多索引 | 转化为数组，一个索引解决所<br>有查询问题 |



### 问题：模式灵活了，如何管理文档不同版本？

![image-20230105215709307](D:/程序/Typora图片存放地/images/image-20230105215709307.png)



#### 解决方案：增加一个版本字段

![image-20230105215849826](D:/程序/Typora图片存放地/images/image-20230105215849826.png)



**版本字段**

| 场景                   | 痛点                                                         | 设计模式方案及优点                                           |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 任何有版本衍变的数据库 | 文档模式格式多，无法知道合理性<br><br>升级时候需要更新太多文档 | 增加一个版本号字段<br><br>快速过滤掉不需要升级的文档<br><br>升级时候对不同版本的文档做不同的处理 |



### 问题：统计网页点击流量

![image-20230105220404063](D:/程序/Typora图片存放地/images/image-20230105220404063.png)



#### 解决方案：用近似计算

![image-20230105220720566](D:/程序/Typora图片存放地/images/image-20230105220720566.png)

使用随机数写10次

**近似计算**

| 场景                                     | 痛点                     | 设计模式方案及优点                                  |
| ---------------------------------------- | ------------------------ | --------------------------------------------------- |
| 网页计数<br><br>各种结果不需要准确的排名 | 写入太频繁，消耗系统资源 | 间隔写入，每个10次或者100次<br><br>大量减少写入需求 |



### 问题：业绩排名，游戏排名，商业统计等精确统计

热销榜：某个商品今天买了多少，这个星期卖了多少，这个月卖了多少？

电影排行：观影者，场次统计

传统解决方案：通过聚合计算

痛点：消耗资源多，聚合计算时间长



#### 解决方案：用预聚合字段

![image-20230105221455696](D:/程序/Typora图片存放地/images/image-20230105221455696.png)

**预聚合**

| 场景               | 痛点                     | 设计模式方案及优点                                           |
| ------------------ | ------------------------ | ------------------------------------------------------------ |
| 准确排名<br>排行榜 | 统计计算耗时，计算时间长 | 模型中直接增加统计字段<br><br>每次更新数据时候同时更新统计值 |



## 事务开发

### 写操作事务

#### 什么是 writeConcern ?

writeConcern 决定一个写操作落到多少个节点上才算成功。writeConcern 的取值包括：

- 0：发起写操作，不关心是否成功;
- 1~集群最大数据节点数：写操作需要被复制到指定节点数才算成功;
- majority：写操作需要被复制到大多数节点上才算成功。

发起写操作的程序将阻塞到写操作到达指定的节点为止



#### 默认行为

3节点复制集不做任何特别设定（默认值）：

![image-20230106221650891](D:/程序/Typora图片存放地/images/image-20230106221650891.png)



#### w："majority"

**大多数节点确认模式**

![image-20230106222108352](D:/程序/Typora图片存放地/images/image-20230106222108352.png)



#### w："all"

**全部节点确认模式**

![image-20230106222352399](D:/程序/Typora图片存放地/images/image-20230106222352399.png)

**会出现一个问题：如果一个节点出现问题，写操作就进行不成了，会一直等待**



#### j:true

作用是数据库宕机的时候，可以快速恢复刚才的写操作。

一般数据库会先写日志文件再写数据文件，MongoDB 同样也有日志文件。默认情况下，不会等写日志就返回了。如果写操作非常重要，可以强制。

writeConcern 可以决定写操作到达多少节点才算成功，journal 则定义如何才算成功。取值包括：

- true：写操作落到 journal 文件中才算成功;
- false：写操作到达内存即算作成功

![image-20230106222815622](D:/程序/Typora图片存放地/images/image-20230106222815622.png)

**这个是事务中增强持久性的一个手段**



#### writeConcern 的意义

对于5个基点的复制集来说，写操作落到多少个节点上才算安全？

- 1
- 2
- 3✔
- 4✔
- 5✔
- majority✔



#### writeConcer  实验

**在复制集测试 writeConcern 参数**<br>

```java
db.test.insert({count: 1}, {writeCOncern: {w: "majority"}})
db.test.insert({count: 1}, {writeCOncern: {w: 3}})
db.test.insert({count: 1}, {writeCOncern: {w: 4}})
```

![image-20230107224412302](D:/程序/Typora图片存放地/images/image-20230107224412302.png)



**配置延迟节点，模拟网络延迟（复制延迟）**<br>

```
conf=rs.conf()
conf.members[2].slaveDelay=10
conf.members[2].priority=0
rs.reconfig(conf)
```

**观察复制延迟下的写入，以及timeout参数**<br>

```
db,test,insert({count: 1}, {writeConcern: {w: 3}})
db.test.insert({count: 1}, {writeConcern: {w: 3, wtimeout: 3000}})
```

![image-20230107231107589](D:/程序/Typora图片存放地/images/image-20230107231107589.png)

这个时候数据已经写进去了，但没有完全写到第三个节点，需要报警处理



#### 注意事项

- 虽然多于半数的 writeConcern 都是安全的，但通常只会设置 majority，因为这是等待写入延迟时间的最短选择;
- 不要设置 writeConcern 等于总节点数，因为一旦有一个节点故障，所有写操作都将失败;
- writeConcern 虽然会增加写操作延迟时间，但并不会显著增加集群压力，因此无论是否等待，写操作最终都会复制到所有节点上。设置 writeConcern 知识让写操作等待复制后在返回而已;
- 应对重要数据应用{w: "majority"}，普通数据可以应用{w:1}以确保最佳性能



### 读操作事务

**综述**

在读取数据的过程中我们需要关注一下两个问题：

- 从哪里读？关注数据节点 位置
- 什么样的数据可以读？关注数据的隔离性

第一个问题是是由 readPreference 来解决

第二个问题则是由 readConcern 来解决



#### 什么是 readPreference？

readPreference 决定使用哪一个节点来满足正在发起的读请求。可选值包括：

- primary：只选择主节点**(默认)**;
- primaryPreferred：优先选择朱姐带你，如果不可用则选择从节点;
- secondary：只选择节点;
- secondaryPreferred：优先选择从节点，如果从节点不可用则选择主节点;
- nearest：选择最近的节点;

![image-20230110213043340](D:/程序/Typora图片存放地/images/image-20230110213043340.png)





#### readPreference 场景举例

- 用户下订单后马上将用户转到订单详情页——primary/primaryPreferred。因为此时从节点可能还没有复制到新订单;
- 用户从查询自己下过的订单——secondary/secondaryPreferred。查询历史订单对时效性通常没有太高需求;
- 生成报表——secondary。报表对时效性要求不高，但资源需求大，可以在从节点单独处理，避免对线上用户造成影响;
- 将用户上传的图片分发到全世界，让各地用户能够就近读取——nearest。每个地区的应用选择最近的节点读取数据。



#### readPreference 与 Tag

readPreference 只能控制使用一类节点。Tag 则可以将节点选择控制到一个或几个节点。考虑一下场景：

- 一个 5 个节点的复制集;
- 3 个节点硬件较好，专用于服务线上客户;
- 2 个节点硬件较差，专用于生成报表;

可以使用 Tag 来达到这样的控制目的：

- 为 3 个较好的节点打上 {purpose: "online"};
- 为 2 个交叉的节点打上 {purpose: "analyse"};
- 在线应用读取时指定 online，报表读取时指定 reporting。

更多信息请参考文档：

![image-20230110214728768](D:/程序/Typora图片存放地/images/image-20230110214728768.png)



#### readPreference 配置

**通过 MongoDB 的连接串参数：**

- mongodb://host1:27017,host2:27017,host3:27017/?replicaSet=rs&readPreference=secondary



**通过 MongoDB 驱动程序 API：**

- MongoCollection.withReadPreference(ReadPreference readPref)



**Mongo Shell：**

- db.collection.find({}).readPref("secondary")

















































































































































































































































































