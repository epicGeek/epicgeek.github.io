---
layout: post
title: "Elasticsearch 入门学习"
date: 2017-12-03
excerpt: "Elasticsearch的入门学习，包括基础概念和基本DSL书写和操作."
tags: [Elasticsearch]
slug: es-basic
---



## 基本概念
ES有些核心基本概念需要先理解，对后续的学习有帮助

* 1.Near Realtime(NRT)：
ES是近似实时的查询平台。这意味着一条数据从开始索引到它变成可查询的状态是有轻微的时延的（通常是一秒）

* 2.集群(Cluster) ：集群是一个或多个节点（服务器）的集合，它们共同保存全部数据，并在所有节点上提供联合索引和搜索功能。一个集群由一个唯一的名称默认为“Elasticsearch”。这个名称很重要，因为如果节点被设置为通过名称加入集群，则节点只能是集群的一部分。确保不要在不同的环境中重用相同的集群名称，否则最终会导致节点加入错误的集群。注意，有一个只有一个节点的集群是有效和非常好的。此外，还可以拥有多个独立集群，每个集群都有自己独特的集群名称。

* 3.节点(Node): 节点是单个服务器，是集群的一部分，存储数据，并参与集群的索引和搜索功能。就像一个集群节点的名称默认为一个随机的通用唯一标识符（UUID），确定在启动时分配给该节点。如果不想要默认值，可以定义所需的任意节点名。节点可以通过配置加入一个指定的集群。默认配置是，每个节点都会加入到名为"elasticsearch"的集群，这意味着，如果你启动了一些节点（假设它们能互相发现）它们最终都将自动加入名为"elasticsearch"的集群。在单个集群内，你可以加入多个任意节点.此外，如果没有其他正在运行的ES节点，新启动的节点将会被命名为elasticsearch.

* 4.索引(Index):索引是一些含有相似特点的文档(document)的集合。例如，你有一个“客户数据”的索引，另一个索引是“产品分类”，还有一个是“订单”.索引由名称（必须为小写）标识，在对其文档执行索引、搜索、更新和删除操作时，该名称用于引用索引。在单个集群中，你可以定义任意多个索引。

* 5.类型(Type):在一个索引中，可以定义一个或者多个类型。类型是索引的逻辑类/分区，其语义完全自定义。一般来说，拥有相同字段的文档定义为一种类型。假设你有一个博客平台，在一个单一的索引里面存储全部数据.在这个索引里面，你定义一个“用户数据”类型，一个“博客数据”类型，一个“评论数据”类型。

* 6.文档(Document):文档是可以被检索到的基本信息单元。例如，你可以写一个单个用户的文档数据，另一个文档存储产品数据,还可以存一份订单数据。文档使用JSON格式。需要注意的是，尽管文档在物理层面上是驻留在一个索引里的，但是必须要给文档指定一种类型。

* 7.分片和副本(shard&replica): 索引可以存储大量的数据，这些数据可能超过单个节点的硬件限制。例如，一个索引占据1TB空间，可能会超过硬件限制，也有可能会导致查询速度慢。为了解决这种问题，ES提供"分片"的解决方案：细分这种索引为多片索引。当你创建一个新的索引时，你可以简单的配置一下分片数目。每个分片自身是功能完整的，并且是独立索引的，在集群上的任意节点都可以托管。分片之所以重要是因为： 1.允许水平分割内容卷。 2.允许分片之间的分发和并行（可能在多个节点上）从而增加吞吐量。分片的分配和将它自身的文档数据聚合到查询请求的行为是由ES控制的，并且对用户透明。在一个可能发生失败的网络或者云的环境下，建议使用失效备援机制以防在某些情况下分片或者节点离线或者丢失。为此，ES允许制作索引或者分片的拷贝，称为"副本（replica）"。副本的两点重要性：1.在索引或分片不可用的情况下，副本是非常有用的。需要注意的是，分片的副本不应该和这个分片分配到同一个节点上。2.它允许扩大搜索量/吞吐量，因为搜索可以并行地在所有副本上执行。总结起来，每个索引可以被分为多个分片，每个索引可以没有或者有多个副本。一旦索引被分片，索引将有主分片和副本分片。分片和副本的数量在索引被创建之后就定义出。索引创建后，你可以在任意时刻改变副本数量，但是不能改变分片的数量了。在默认条件下，索引有5个主分片和1个副本，也就是说，如果你的集群中有至少两个节点的话，你的索引将会有5个主分片和5个副本分片(1个完整副本),每个索引有10个分片。

## 安装和运行
使用Docker来安装并运行ES：

*下载最新镜像:

$ docker pull elasticsearch

* 启动elasticsearch:

$ docker run -d --name=espn -p 9200:9200 -p 9300:9300 -v /data/espn:/data/espn elasticsearch:5.6.3

* 验证运行：

$ curl localhost:9200

* 在浏览器中访问：
%espn-host-ip%:9200
在服务器上，挂载目录/var/espn下生成 config文件夹和data文件夹。
## 关于Kibana
Kibana是ES的可视化管理工具，建议安装。
$ docker pull kibana
启动kibana:
$ docker run --name=kibana --link espn:elasticsearch -p 5601:5601 -d kibana
在浏览器访问:$kibana_host$:5601，如果页面可以访问，说明kibana已安装成功。
## 探索集群
RESTful风格API

通过ES提供的API，可以：

* 查看节点、集群、索引健康、状态、数据
* 管理节点、集群、索引数据和元数据
* 进行增删改查操作、索引查询操作
* 执行进阶的查询操作：分页、排序、过滤、脚本、聚合等

## 集群健康
kibana console:GET /_cat/health?v

$curl -XGET 'localhost:9200/_cat/health?v&pretty'

响应：
```
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign
1475247709 17:01:49  elasticsearch green           1         1      0   0    0    0        0        

```

可以看到目前的集群名称、健康状况、节点个数、节点数据、分片等信息。
健康状态的说明：
green:集群状况好。
yellow：所有数据都可用，但是有些副本没有分配。
red:某些数据由于一些原因不可用。
注意：当集群状态为red，它将继续从可用的分片里提供查询请求的服务，有未分配的分片的话，你要尽快的修复它。

从上面的响应来看，我们看到总共有一个节点，0个分片，因为暂时没有数据。注意到由于使用了默认的集群名称，并且ES默认使用单播网络来在同一台机器上查找其他节点，那么就有可能意外地发生多个节点自动加入一个集群的情况。在这种情形下，有可能看到多于一个节点的响应。

查看节点信息：
GET /_cat/nodes?v

$curl -XGET 'localhost:9200/_cat/nodes?v&pretty'

这样就查看到集群上的节点了.（这里的pretty是指如果有json数据，就以较为美观的形式打印出来）

## 查看全部索引
GET /_cat/indices?v

$curl -XGET 'localhost:9200/_cat/indices?v&pretty'

## 建立索引
PUT /customer?pretty
GET /_cat/indices?v

curl -XPUT 'localhost:9200/customer?pretty&pretty'
curl -XGET 'localhost:9200/_cat/indices?v&pretty'

响应：
```
{
  "acknowledged" : true,
  "shards_acknowledged" : true
}
```
```
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer 95SQ4TSUT7mWBT7VNHH67A   5   1          0            0       260b           260b
```
现在健康状况是yellow，因为有一个副本尚未被分配。原因是ES默认为新的索引建立一份副本。现在我们只运行了一个节点，这份副本还没有分配出去，在后面的时间点里可能会加入其它节点到集群里面，这时状态就会变成green了。

## 索引并查询文档

向指定的索引添加文档时，必须指定这个文档的类型。向customer索引添加external类型数据：
```
PUT /customer/external/1?pretty
{
  "name": "John Doe"
}
```
```
$curl -XPUT 'localhost:9200/customer/external/1?pretty&pretty' -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'
```
响应:
```
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "created" : true
}
```
从响应数据能看，新的数据已经添加到customer索引，类型为external。并且每条数据都有一条内部的id.
注意：添加数据时，如果索引不存在，会自动建立索引。

查询刚刚输入的数据：
```
GET /customer/external/1?pretty
```
```
curl -XGET 'localhost:9200/customer/external/1?pretty&pretty'
```
响应：
```
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : { "name": "John Doe" }
}
```
除了一些描述性质的数据之外在没什么了。仅仅返回JSON文档数据而已。

## 删除索引

```
DELETE /customer?pretty
GET /_cat/indices?v
```
```
curl -XDELETE 'localhost:9200/customer?pretty&pretty'
curl -XGET 'localhost:9200/_cat/indices?v&pretty'

```

到目前为止，已经学习了如何增删查索引。总结起来，URL是遵循这样的格式的：
```
<REST Verb> /<Index>/<Type>/<ID>
```
这种RESTful风格的请求非常普遍、好记，为掌握ES奠定基础。

## 修改数据
ES有着近乎实时的查询数据、修改数据的能力。默认情况下，从对数据进行操作到显示操作结果大概有一秒的延迟。这与其他平台（如SQL）有很大区别。SQL的事务一旦完成，数据就马上可用。

### 索引/替换文档
之前已经输入一条数据
```
PUT /customer/external/1?pretty
{
  "name": "John Doe"
}
```
```
curl -XPUT 'localhost:9200/customer/external/1?pretty&pretty' -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'

```

这条数据ID为1，那么，如果再次PUT一条ID相同的数据，会覆盖原来的数据。

在添加数据时，是否带ID是可选的。不指定ID的请求方法：
```
POST /customer/external?pretty
{
  "name": "Jane Doe"
}
```
```
curl -XPOST 'localhost:9200/customer/external?pretty&pretty' -H 'Content-Type: application/json' -d'
{
  "name": "Jane Doe"
}
'

```

响应:
```
{
  "_index": "customer",
  "_type": "external",
  "_id": "AV9stFetBUc7shyhx1ET",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "created": true
}
```
但是这样的话，文档的ID为一串UUID了。

## 更新文档

update操作同样是ES支持的。需要注意的是，ES实际上并没有在后台就地更新数据。
而是在一次更新操作中，删除旧数据并写入新数据。

```
POST /customer/external/1/_update?pretty
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
```
```
curl -XPOST 'localhost:9200/customer/external/1/_update?pretty&pretty' -H 'Content-Type: application/json' -d'
{
  "doc": { "name": "Jane Doe" , "age":20 }
}
'

```
注意：Update时，键值对的变化

还可以在update时，使用简单的脚本

```
POST /customer/external/1/_update?pretty
{
  "script" : "ctx._source.age += 5"
}
```

在上面的例子中，ctx._source指的是当前源文档。
需要注意的是，这种以上几种写法的添加、修改数据每次只能影响到一条文档数据。在将来，ES可能会提供像SQL里面的WHERE条件来进行批量的更新。

## 删除文档

```
DELETE /customer/external/2?pretty
```

## 批量过程

除了增删改查单个独立文档之外，ES还提供了批量上述操作。_bulk API的批量操作上效率非常高，而且尽可能的少使用网络切换。

```
POST /customer/external/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
```
上面的请求把ID为1和2的数据name字段改变了。

```
POST /customer/external/_bulk?pretty
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
```
更新文档1的，再删除文档2

Bulk API不会因为操作中的某一步失败而失败。如果其中一步的操作失败了，它将继续后续的操作。Bulk的返回值将包含每一步的状态（根据发送的顺序），由此可以查看特定的步骤是否成功执行。


## 探索你的数据

### 样本数据集

```
{
    "account_number": 0,
    "balance": 16623,
    "firstname": "Bradshaw",
    "lastname": "Mckenzie",
    "age": 29,
    "gender": "F",
    "address": "244 Columbus Place",
    "employer": "Euron",
    "email": "bradshawmckenzie@euron.com",
    "city": "Hobucken",
    "state": "CO"
}
```
这是一条比较实际的数据。现在尝试向ES导入一千条数据。新建文件"account.json"
文件内容:
```
{"index":{"_id":"1"}}
{"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
```
### 载入数据文件

将数据文件LOAD进集群：
上传account.json至 $ES$/data下 (本文档为/var/espn/data)
```
curl -H "Content-Type: application/json" -XPOST 'localhost:9200/bank/account/_bulk?pretty&refresh' --data-binary "@accounts.json"
curl 'localhost:9200/_cat/indices?v'
```

通过响应可以看见，bank索引成功建立，account类型成功建立，数据已经LOAD至集群中。

## 查询API
查询方式有两种：一种是通过在REST URL传入参数查询，一种是传输resquest body。
使用request body可以使查询的可读性增强。这里先举例使用URL传参来查询，在后面的文档中会提到使用request body.

在URL中，索引的后面加入关键字"_search".这里举例来查询所有的bank下面的文档数据。
```
curl -XGET 'localhost:9200/bank/_search?q=*&sort=account_number:asc&pretty&pretty'

```
我们先来看一下URL的含义：
_search?q=* 意思是查询bank索引下全部数据. sort=account_number:asc 意思是以account_number:asc字段的正序排序。

响应：
```
{
  "took" : 63,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : null,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "0",
      "sort": [0],
      "_score" : null,
      "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
    }, {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "1",
      "sort": [1],
      "_score" : null,
      "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, ...
    ]
  }
}
```
从响应上来看：
* took :查询用了多少毫秒
* timed_out:查询是否超时
* _shards:告诉我们这个查询用了多少分片，以及具体情况。
* hits:查询结果
* hits.total : 符合查询条件的结果数量。
* hits.hits: 实际数据。默认显示10条。
* hits.sort: 排序关键字。如果没有的话，是按照score排序的。
* hits._score , max_score : 暂且不提，稍后再议。

下面介绍用request body来实现上面的查询的写法：

```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
  {
    "query": { "match_all": {} },
    "sort": [
      { "account_number": "asc" }
    ]
  }
'

```
区别是：写request body的方法是直接向search API发送JSON格式数据，而不是在url里传输参数。我们稍后讨论JSON查询的方法。


## 执行搜索
前面我们说明了基本的查询，现在我们来深入学习。默认情况下，查询结果返回值中，文档数据只是一部分，称它为"source"(因为数据在_source字段里)。如果不想看见文档的全部字段，我们可以指定一些字段。
```
GET /bank/_search
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
```
这个例子的返回结果中，文档数据只包含account_number和balance字段。
如果你有SQL背景的话，这种查询可以理解为 select field1,field2 from table 这种查询。

上面的例子中，我们的查询结果都是使用match_all,下面介绍match的用法。
下面这个例子，查询account_number为20的文档：
```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "account_number": 20 } }
}
'
```
查询address：
```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill" } }
}
'
```
响应：
```
{
  "took": 11,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": 4.3100996,
    "hits": [
      {
        "_index": "bank",
        "_type": "account",
        "_id": "472",
        "_score": 4.3100996,
        "_source": {
          "account_number": 472,
          "balance": 25571,
          "firstname": "Lee",
          "lastname": "Long",
          "age": 32,
          "gender": "F",
          "address": "288 Mill Street",
          "employer": "Comverges",
          "email": "leelong@comverges.com",
          "city": "Movico",
          "state": "MT"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "136",
        "_score": 4.2662063,
        "_source": {
          "account_number": 136,
          "balance": 45801,
          "firstname": "Winnie",
          "lastname": "Holland",
          "age": 38,
          "gender": "M",
          "address": "198 Mill Lane",
          "employer": "Neteria",
          "email": "winnieholland@neteria.com",
          "city": "Urie",
          "state": "IL"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "970",
        "_score": 3.861861,
        "_source": {
          "account_number": 970,
          "balance": 19648,
          "firstname": "Forbes",
          "lastname": "Wallace",
          "age": 28,
          "gender": "M",
          "address": "990 Mill Road",
          "employer": "Pheast",
          "email": "forbeswallace@pheast.com",
          "city": "Lopezo",
          "state": "AK"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "345",
        "_score": 3.861861,
        "_source": {
          "account_number": 345,
          "balance": 9812,
          "firstname": "Parker",
          "lastname": "Hines",
          "age": 38,
          "gender": "M",
          "address": "715 Mill Avenue",
          "employer": "Baluba",
          "email": "parkerhines@baluba.com",
          "city": "Blackgum",
          "state": "KY"
        }
      }
    ]
  }
}
```
注意，match不是“等于”的概念，而是包括的概念。

看这个例子：
```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill lane" } }
}
'
```

响应：
```
{
  "took": 21,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 19,
    "max_score": 7.3900023,
    "hits": [
      {
        "_index": "bank",
        "_type": "account",
        "_id": "136",
        "_score": 7.3900023,
        "_source": {
          "account_number": 136,
          "balance": 45801,
          "firstname": "Winnie",
          "lastname": "Holland",
          "age": 38,
          "gender": "M",
          "address": "198 Mill Lane",
          "employer": "Neteria",
          "email": "winnieholland@neteria.com",
          "city": "Urie",
          "state": "IL"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "472",
        "_score": 4.3100996,
        "_source": {
          "account_number": 472,
          "balance": 25571,
          "firstname": "Lee",
          "lastname": "Long",
          "age": 32,
          "gender": "F",
          "address": "288 Mill Street",
          "employer": "Comverges",
          "email": "leelong@comverges.com",
          "city": "Movico",
          "state": "MT"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "556",
        "_score": 3.9074605,
        "_source": {
          "account_number": 556,
          "balance": 36420,
          "firstname": "Collier",
          "lastname": "Odonnell",
          "age": 35,
          "gender": "M",
          "address": "591 Nolans Lane",
          "employer": "Sultraxin",
          "email": "collierodonnell@sultraxin.com",
          "city": "Fulford",
          "state": "MD"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "934",
        "_score": 3.9074605,
        "_source": {
          "account_number": 934,
          "balance": 43987,
          "firstname": "Freida",
          "lastname": "Daniels",
          "age": 34,
          "gender": "M",
          "address": "448 Cove Lane",
          "employer": "Vurbo",
          "email": "freidadaniels@vurbo.com",
          "city": "Snelling",
          "state": "NJ"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "970",
        "_score": 3.861861,
        "_source": {
          "account_number": 970,
          "balance": 19648,
          "firstname": "Forbes",
          "lastname": "Wallace",
          "age": 28,
          "gender": "M",
          "address": "990 Mill Road",
          "employer": "Pheast",
          "email": "forbeswallace@pheast.com",
          "city": "Lopezo",
          "state": "AK"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "345",
        "_score": 3.861861,
        "_source": {
          "account_number": 345,
          "balance": 9812,
          "firstname": "Parker",
          "lastname": "Hines",
          "age": 38,
          "gender": "M",
          "address": "715 Mill Avenue",
          "employer": "Baluba",
          "email": "parkerhines@baluba.com",
          "city": "Blackgum",
          "state": "KY"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "908",
        "_score": 3.861861,
        "_source": {
          "account_number": 908,
          "balance": 45975,
          "firstname": "Mosley",
          "lastname": "Holloway",
          "age": 31,
          "gender": "M",
          "address": "929 Eldert Lane",
          "employer": "Anivet",
          "email": "mosleyholloway@anivet.com",
          "city": "Biehle",
          "state": "MS"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "637",
        "_score": 3.861861,
        "_source": {
          "account_number": 637,
          "balance": 3169,
          "firstname": "Kathy",
          "lastname": "Carter",
          "age": 27,
          "gender": "F",
          "address": "410 Jamison Lane",
          "employer": "Limage",
          "email": "kathycarter@limage.com",
          "city": "Ernstville",
          "state": "WA"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "449",
        "_score": 3.8428833,
        "_source": {
          "account_number": 449,
          "balance": 41950,
          "firstname": "Barnett",
          "lastname": "Cantrell",
          "age": 39,
          "gender": "F",
          "address": "945 Bedell Lane",
          "employer": "Zentility",
          "email": "barnettcantrell@zentility.com",
          "city": "Swartzville",
          "state": "ND"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "742",
        "_score": 3.8428833,
        "_source": {
          "account_number": 742,
          "balance": 24765,
          "firstname": "Merle",
          "lastname": "Wooten",
          "age": 26,
          "gender": "M",
          "address": "317 Pooles Lane",
          "employer": "Tropolis",
          "email": "merlewooten@tropolis.com",
          "city": "Bentley",
          "state": "ND"
        }
      }
    ]
  }
}
```
从结果可以看出，响应数据中，address字段里要么包含"mill"，要么包含"lane"。
现在介绍match_phrase的用法：
```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
'
```
响应：
```
{
  "took": 18,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 7.3900023,
    "hits": [
      {
        "_index": "bank",
        "_type": "account",
        "_id": "136",
        "_score": 7.3900023,
        "_source": {
          "account_number": 136,
          "balance": 45801,
          "firstname": "Winnie",
          "lastname": "Holland",
          "age": 38,
          "gender": "M",
          "address": "198 Mill Lane",
          "employer": "Neteria",
          "email": "winnieholland@neteria.com",
          "city": "Urie",
          "state": "IL"
        }
      }
    ]
  }
}
```
可以看出，查询的结果是address字段包含"mill lane"的。
总结：match是包含的意思，而不是等于的意思。使用"match"时，多个词语以空格隔开，表示OR的逻辑关系。

下面介绍bool查询。
bool查询提供查询条件之间的逻辑关系，下面举例说明：

```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
```
响应：
```
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 7.3900023,
    "hits": [
      {
        "_index": "bank",
        "_type": "account",
        "_id": "136",
        "_score": 7.3900023,
        "_source": {
          "account_number": 136,
          "balance": 45801,
          "firstname": "Winnie",
          "lastname": "Holland",
          "age": 38,
          "gender": "M",
          "address": "198 Mill Lane",
          "employer": "Neteria",
          "email": "winnieholland@neteria.com",
          "city": "Urie",
          "state": "IL"
        }
      }
    ]
  }
}
```
从查询请求上看，查询条件服从这种逻辑关系：
条件1：address 包含 mill
条件2：address 包含 lane
条件1和条件2属于“must”数组，也就是说，查询的数据要同时满足条件1和条件2。

再看一下should关键词的例子：
```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
```
响应:
```
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 19,
    "max_score": 7.3900023,
    "hits": [
      {
        "_index": "bank",
        "_type": "account",
        "_id": "136",
        "_score": 7.3900023,
        "_source": {
          "account_number": 136,
          "balance": 45801,
          "firstname": "Winnie",
          "lastname": "Holland",
          "age": 38,
          "gender": "M",
          "address": "198 Mill Lane",
          "employer": "Neteria",
          "email": "winnieholland@neteria.com",
          "city": "Urie",
          "state": "IL"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "472",
        "_score": 4.3100996,
        "_source": {
          "account_number": 472,
          "balance": 25571,
          "firstname": "Lee",
          "lastname": "Long",
          "age": 32,
          "gender": "F",
          "address": "288 Mill Street",
          "employer": "Comverges",
          "email": "leelong@comverges.com",
          "city": "Movico",
          "state": "MT"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "556",
        "_score": 3.9074605,
        "_source": {
          "account_number": 556,
          "balance": 36420,
          "firstname": "Collier",
          "lastname": "Odonnell",
          "age": 35,
          "gender": "M",
          "address": "591 Nolans Lane",
          "employer": "Sultraxin",
          "email": "collierodonnell@sultraxin.com",
          "city": "Fulford",
          "state": "MD"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "934",
        "_score": 3.9074605,
        "_source": {
          "account_number": 934,
          "balance": 43987,
          "firstname": "Freida",
          "lastname": "Daniels",
          "age": 34,
          "gender": "M",
          "address": "448 Cove Lane",
          "employer": "Vurbo",
          "email": "freidadaniels@vurbo.com",
          "city": "Snelling",
          "state": "NJ"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "970",
        "_score": 3.861861,
        "_source": {
          "account_number": 970,
          "balance": 19648,
          "firstname": "Forbes",
          "lastname": "Wallace",
          "age": 28,
          "gender": "M",
          "address": "990 Mill Road",
          "employer": "Pheast",
          "email": "forbeswallace@pheast.com",
          "city": "Lopezo",
          "state": "AK"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "345",
        "_score": 3.861861,
        "_source": {
          "account_number": 345,
          "balance": 9812,
          "firstname": "Parker",
          "lastname": "Hines",
          "age": 38,
          "gender": "M",
          "address": "715 Mill Avenue",
          "employer": "Baluba",
          "email": "parkerhines@baluba.com",
          "city": "Blackgum",
          "state": "KY"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "908",
        "_score": 3.861861,
        "_source": {
          "account_number": 908,
          "balance": 45975,
          "firstname": "Mosley",
          "lastname": "Holloway",
          "age": 31,
          "gender": "M",
          "address": "929 Eldert Lane",
          "employer": "Anivet",
          "email": "mosleyholloway@anivet.com",
          "city": "Biehle",
          "state": "MS"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "637",
        "_score": 3.861861,
        "_source": {
          "account_number": 637,
          "balance": 3169,
          "firstname": "Kathy",
          "lastname": "Carter",
          "age": 27,
          "gender": "F",
          "address": "410 Jamison Lane",
          "employer": "Limage",
          "email": "kathycarter@limage.com",
          "city": "Ernstville",
          "state": "WA"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "449",
        "_score": 3.8428833,
        "_source": {
          "account_number": 449,
          "balance": 41950,
          "firstname": "Barnett",
          "lastname": "Cantrell",
          "age": 39,
          "gender": "F",
          "address": "945 Bedell Lane",
          "employer": "Zentility",
          "email": "barnettcantrell@zentility.com",
          "city": "Swartzville",
          "state": "ND"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "742",
        "_score": 3.8428833,
        "_source": {
          "account_number": 742,
          "balance": 24765,
          "firstname": "Merle",
          "lastname": "Wooten",
          "age": 26,
          "gender": "M",
          "address": "317 Pooles Lane",
          "employer": "Tropolis",
          "email": "merlewooten@tropolis.com",
          "city": "Bentley",
          "state": "ND"
        }
      }
    ]
  }
}
```
从响应可以看出，should数组里面包含的条件，满足其一即可。也就是说，should里面的条件是or的逻辑。
再来看一下must_not关键词的例子：
```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
```
响应：
```
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 980,
    "max_score": 1,
    "hits": [
      {
        "_index": "bank",
        "_type": "account",
        "_id": "25",
        "_score": 1,
        "_source": {
          "account_number": 25,
          "balance": 40540,
          "firstname": "Virginia",
          "lastname": "Ayala",
          "age": 39,
          "gender": "F",
          "address": "171 Putnam Avenue",
          "employer": "Filodyne",
          "email": "virginiaayala@filodyne.com",
          "city": "Nicholson",
          "state": "PA"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "44",
        "_score": 1,
        "_source": {
          "account_number": 44,
          "balance": 34487,
          "firstname": "Aurelia",
          "lastname": "Harding",
          "age": 37,
          "gender": "M",
          "address": "502 Baycliff Terrace",
          "employer": "Orbalix",
          "email": "aureliaharding@orbalix.com",
          "city": "Yardville",
          "state": "DE"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "99",
        "_score": 1,
        "_source": {
          "account_number": 99,
          "balance": 47159,
          "firstname": "Ratliff",
          "lastname": "Heath",
          "age": 39,
          "gender": "F",
          "address": "806 Rockwell Place",
          "employer": "Zappix",
          "email": "ratliffheath@zappix.com",
          "city": "Shaft",
          "state": "ND"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "119",
        "_score": 1,
        "_source": {
          "account_number": 119,
          "balance": 49222,
          "firstname": "Laverne",
          "lastname": "Johnson",
          "age": 28,
          "gender": "F",
          "address": "302 Howard Place",
          "employer": "Senmei",
          "email": "lavernejohnson@senmei.com",
          "city": "Herlong",
          "state": "DC"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "126",
        "_score": 1,
        "_source": {
          "account_number": 126,
          "balance": 3607,
          "firstname": "Effie",
          "lastname": "Gates",
          "age": 39,
          "gender": "F",
          "address": "620 National Drive",
          "employer": "Digitalus",
          "email": "effiegates@digitalus.com",
          "city": "Blodgett",
          "state": "MD"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "145",
        "_score": 1,
        "_source": {
          "account_number": 145,
          "balance": 47406,
          "firstname": "Rowena",
          "lastname": "Wilkinson",
          "age": 32,
          "gender": "M",
          "address": "891 Elton Street",
          "employer": "Asimiline",
          "email": "rowenawilkinson@asimiline.com",
          "city": "Ripley",
          "state": "NH"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "183",
        "_score": 1,
        "_source": {
          "account_number": 183,
          "balance": 14223,
          "firstname": "Hudson",
          "lastname": "English",
          "age": 26,
          "gender": "F",
          "address": "823 Herkimer Place",
          "employer": "Xinware",
          "email": "hudsonenglish@xinware.com",
          "city": "Robbins",
          "state": "ND"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "190",
        "_score": 1,
        "_source": {
          "account_number": 190,
          "balance": 3150,
          "firstname": "Blake",
          "lastname": "Davidson",
          "age": 30,
          "gender": "F",
          "address": "636 Diamond Street",
          "employer": "Quantasis",
          "email": "blakedavidson@quantasis.com",
          "city": "Crumpler",
          "state": "KY"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "208",
        "_score": 1,
        "_source": {
          "account_number": 208,
          "balance": 40760,
          "firstname": "Garcia",
          "lastname": "Hess",
          "age": 26,
          "gender": "F",
          "address": "810 Nostrand Avenue",
          "employer": "Quiltigen",
          "email": "garciahess@quiltigen.com",
          "city": "Brooktrails",
          "state": "GA"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "222",
        "_score": 1,
        "_source": {
          "account_number": 222,
          "balance": 14764,
          "firstname": "Rachelle",
          "lastname": "Rice",
          "age": 36,
          "gender": "M",
          "address": "333 Narrows Avenue",
          "employer": "Enaut",
          "email": "rachellerice@enaut.com",
          "city": "Wright",
          "state": "AZ"
        }
      }
    ]
  }
}
```
从查询结果能看出来，返回数据既不满足must_not里的条件1,也不满足条件2。
下面举一个组合关键词的例子：
```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
'
```
响应：
```
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 43,
    "max_score": 1,
    "hits": [
      {
        "_index": "bank",
        "_type": "account",
        "_id": "948",
        "_score": 1,
        "_source": {
          "account_number": 948,
          "balance": 37074,
          "firstname": "Sargent",
          "lastname": "Powers",
          "age": 40,
          "gender": "M",
          "address": "532 Fiske Place",
          "employer": "Accuprint",
          "email": "sargentpowers@accuprint.com",
          "city": "Umapine",
          "state": "AK"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "40",
        "_score": 1,
        "_source": {
          "account_number": 40,
          "balance": 33882,
          "firstname": "Pace",
          "lastname": "Molina",
          "age": 40,
          "gender": "M",
          "address": "263 Ovington Court",
          "employer": "Cytrak",
          "email": "pacemolina@cytrak.com",
          "city": "Silkworth",
          "state": "OR"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "468",
        "_score": 1,
        "_source": {
          "account_number": 468,
          "balance": 18400,
          "firstname": "Foreman",
          "lastname": "Fowler",
          "age": 40,
          "gender": "M",
          "address": "443 Jackson Court",
          "employer": "Zillactic",
          "email": "foremanfowler@zillactic.com",
          "city": "Wakarusa",
          "state": "WA"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "792",
        "_score": 1,
        "_source": {
          "account_number": 792,
          "balance": 13109,
          "firstname": "Becky",
          "lastname": "Jimenez",
          "age": 40,
          "gender": "F",
          "address": "539 Front Street",
          "employer": "Isologia",
          "email": "beckyjimenez@isologia.com",
          "city": "Summertown",
          "state": "MI"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "302",
        "_score": 1,
        "_source": {
          "account_number": 302,
          "balance": 11298,
          "firstname": "Isabella",
          "lastname": "Hewitt",
          "age": 40,
          "gender": "M",
          "address": "455 Bedford Avenue",
          "employer": "Cincyr",
          "email": "isabellahewitt@cincyr.com",
          "city": "Blanford",
          "state": "IN"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "340",
        "_score": 1,
        "_source": {
          "account_number": 340,
          "balance": 42072,
          "firstname": "Juarez",
          "lastname": "Gutierrez",
          "age": 40,
          "gender": "F",
          "address": "802 Seba Avenue",
          "employer": "Billmed",
          "email": "juarezgutierrez@billmed.com",
          "city": "Malott",
          "state": "OH"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "485",
        "_score": 1,
        "_source": {
          "account_number": 485,
          "balance": 44235,
          "firstname": "Albert",
          "lastname": "Roberts",
          "age": 40,
          "gender": "M",
          "address": "385 Harman Street",
          "employer": "Stralum",
          "email": "albertroberts@stralum.com",
          "city": "Watrous",
          "state": "NM"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "666",
        "_score": 1,
        "_source": {
          "account_number": 666,
          "balance": 13880,
          "firstname": "Mcguire",
          "lastname": "Lloyd",
          "age": 40,
          "gender": "F",
          "address": "658 Just Court",
          "employer": "Centrexin",
          "email": "mcguirelloyd@centrexin.com",
          "city": "Warren",
          "state": "MT"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "998",
        "_score": 1,
        "_source": {
          "account_number": 998,
          "balance": 16869,
          "firstname": "Letha",
          "lastname": "Baker",
          "age": 40,
          "gender": "F",
          "address": "206 Llama Court",
          "employer": "Dognosis",
          "email": "lethabaker@dognosis.com",
          "city": "Dunlo",
          "state": "WV"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "432",
        "_score": 1,
        "_source": {
          "account_number": 432,
          "balance": 28969,
          "firstname": "Preston",
          "lastname": "Ferguson",
          "age": 40,
          "gender": "F",
          "address": "239 Greenwood Avenue",
          "employer": "Bitendrex",
          "email": "prestonferguson@bitendrex.com",
          "city": "Idledale",
          "state": "ND"
        }
      }
    ]
  }
}
```
从响应数据我们知道，一定满足must里面的年龄为40的条件，也满足state不为ID的条件。
使用bool查询，可以完成复杂的逻辑查询。

## 执行过滤

在前面的说明中，我们暂时跳过"score"字段。score描述的是查询结果与查询请求的匹配度量。score越大，表示这条数据与查询越相关。但是查询并不一定一直要求产生分数，尤其是需要对查询结果进行过滤的时候。ES检测到这种情形并自动调整查询，以免生成无用的score.

之前介绍的bool查询是支持过滤的。下面举个例子来查询balance在20000到30000之间的数据。
```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
'

```
响应：
```
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 209,
    "max_score": 1,
    "hits": [
      {
        "_index": "bank",
        "_type": "account",
        "_id": "400",
        "_score": 1,
        "_source": {
          "account_number": 400,
          "balance": 20685,
          "firstname": "Kane",
          "lastname": "King",
          "age": 21,
          "gender": "F",
          "address": "405 Cornelia Street",
          "employer": "Tri@Tribalog",
          "email": "kaneking@tri@tribalog.com",
          "city": "Gulf",
          "state": "VT"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "520",
        "_score": 1,
        "_source": {
          "account_number": 520,
          "balance": 27987,
          "firstname": "Brandy",
          "lastname": "Calhoun",
          "age": 32,
          "gender": "M",
          "address": "818 Harden Street",
          "employer": "Maxemia",
          "email": "brandycalhoun@maxemia.com",
          "city": "Sidman",
          "state": "OR"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "645",
        "_score": 1,
        "_source": {
          "account_number": 645,
          "balance": 29362,
          "firstname": "Edwina",
          "lastname": "Hutchinson",
          "age": 26,
          "gender": "F",
          "address": "892 Pacific Street",
          "employer": "Essensia",
          "email": "edwinahutchinson@essensia.com",
          "city": "Dowling",
          "state": "NE"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "734",
        "_score": 1,
        "_source": {
          "account_number": 734,
          "balance": 20325,
          "firstname": "Keri",
          "lastname": "Kinney",
          "age": 23,
          "gender": "M",
          "address": "490 Balfour Place",
          "employer": "Retrotex",
          "email": "kerikinney@retrotex.com",
          "city": "Salunga",
          "state": "PA"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "784",
        "_score": 1,
        "_source": {
          "account_number": 784,
          "balance": 25291,
          "firstname": "Mabel",
          "lastname": "Thornton",
          "age": 21,
          "gender": "M",
          "address": "124 Louisiana Avenue",
          "employer": "Zolavo",
          "email": "mabelthornton@zolavo.com",
          "city": "Lynn",
          "state": "AL"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "880",
        "_score": 1,
        "_source": {
          "account_number": 880,
          "balance": 22575,
          "firstname": "Christian",
          "lastname": "Myers",
          "age": 35,
          "gender": "M",
          "address": "737 Crown Street",
          "employer": "Combogen",
          "email": "christianmyers@combogen.com",
          "city": "Abrams",
          "state": "OK"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "14",
        "_score": 1,
        "_source": {
          "account_number": 14,
          "balance": 20480,
          "firstname": "Erma",
          "lastname": "Kane",
          "age": 39,
          "gender": "F",
          "address": "661 Vista Place",
          "employer": "Stockpost",
          "email": "ermakane@stockpost.com",
          "city": "Chamizal",
          "state": "NY"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "19",
        "_score": 1,
        "_source": {
          "account_number": 19,
          "balance": 27894,
          "firstname": "Schwartz",
          "lastname": "Buchanan",
          "age": 28,
          "gender": "F",
          "address": "449 Mersereau Court",
          "employer": "Sybixtex",
          "email": "schwartzbuchanan@sybixtex.com",
          "city": "Greenwich",
          "state": "KS"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "204",
        "_score": 1,
        "_source": {
          "account_number": 204,
          "balance": 27714,
          "firstname": "Mavis",
          "lastname": "Deleon",
          "age": 39,
          "gender": "F",
          "address": "400 Waldane Court",
          "employer": "Lotron",
          "email": "mavisdeleon@lotron.com",
          "city": "Stollings",
          "state": "LA"
        }
      },
      {
        "_index": "bank",
        "_type": "account",
        "_id": "300",
        "_score": 1,
        "_source": {
          "account_number": 300,
          "balance": 25654,
          "firstname": "Lane",
          "lastname": "Tate",
          "age": 26,
          "gender": "F",
          "address": "632 Kay Court",
          "employer": "Genesynk",
          "email": "lanetate@genesynk.com",
          "city": "Lowell",
          "state": "MO"
        }
      }
    ]
  }
}
```
此bool查询包括一个match_all关键字，和range关键字。我们可以替换查询体重任意一部分来实现过滤查询。除了match_all,match,bool,range之外，还有很多种查询类型。暂时在这里不深入介绍。

## 执行聚合

ES提供的聚合(aggregation)功能可以用来计算一些统计数据。可以简单的类比成SQL里面的Group by这一类的聚合函数。但是ES的聚合更强大的是，在一条查询请求中可以同时得到文档数据和统计数据。
下面举例说明：
```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
'
```
响应：
```
{
  "took": 47,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 999,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "group_by_state": {
      "doc_count_error_upper_bound": 20,
      "sum_other_doc_count": 770,
      "buckets": [
        {
          "key": "ID",
          "doc_count": 27
        },
        {
          "key": "TX",
          "doc_count": 27
        },
        {
          "key": "AL",
          "doc_count": 25
        },
        {
          "key": "MD",
          "doc_count": 25
        },
        {
          "key": "TN",
          "doc_count": 23
        },
        {
          "key": "MA",
          "doc_count": 21
        },
        {
          "key": "NC",
          "doc_count": 21
        },
        {
          "key": "ND",
          "doc_count": 21
        },
        {
          "key": "MO",
          "doc_count": 20
        },
        {
          "key": "AK",
          "doc_count": 19
        }
      ]
    }
  }
}
```
从SQL的角度来看，这条聚合查询类似于:
```
SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC;
```
从响应数据来看，可以看到有27个户主居住在ID(爱达荷州),紧跟着是27个户主居住于TX（德克萨斯）。
需要注意的是，请求中的size=0意思是，我们不想看到查询到的数据，而是只看聚合数据即可。

在上个查询基础上，我们可以查询平均每个州的账户的平均余额(默认显示前十条并倒序排列）。

```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'

```
响应:
```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'

```
需要注意到的是，我们在group_by_state的聚合体里面又嵌套了一层average_balance的聚合体。这是一种很常见的做法，所有的聚合体都应该遵守这样的样式。通过层层嵌套最终得到想要查询的数据。

在上一个聚合查询基础上，我们对average_balance做降序排列。
```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'
```
响应:
```
{
  "took": 17,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 999,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "group_by_state": {
      "doc_count_error_upper_bound": -1,
      "sum_other_doc_count": 917,
      "buckets": [
        {
          "key": "AL",
          "doc_count": 6,
          "average_balance": {
            "value": 41418.166666666664
          }
        },
        {
          "key": "SC",
          "doc_count": 1,
          "average_balance": {
            "value": 40019
          }
        },
        {
          "key": "AZ",
          "doc_count": 10,
          "average_balance": {
            "value": 36847.4
          }
        },
        {
          "key": "VA",
          "doc_count": 13,
          "average_balance": {
            "value": 35418.846153846156
          }
        },
        {
          "key": "DE",
          "doc_count": 8,
          "average_balance": {
            "value": 35135.375
          }
        },
        {
          "key": "WA",
          "doc_count": 7,
          "average_balance": {
            "value": 34787.142857142855
          }
        },
        {
          "key": "ME",
          "doc_count": 3,
          "average_balance": {
            "value": 34539.666666666664
          }
        },
        {
          "key": "OK",
          "doc_count": 9,
          "average_balance": {
            "value": 34529.77777777778
          }
        },
        {
          "key": "CO",
          "doc_count": 13,
          "average_balance": {
            "value": 33379.769230769234
          }
        },
        {
          "key": "MI",
          "doc_count": 12,
          "average_balance": {
            "value": 32905.916666666664
          }
        }
      ]
    }
  }
}
```

下面举一个复杂的例子：先按照年龄段统计( 20-29, 30-39, 40-49)，然后按照性别分组，最后计算每个年龄段、各个性别的平均余额。
```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
'
```
响应:
```
{
  "took": 11,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 999,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "group_by_age": {
      "buckets": [
        {
          "key": "20.0-30.0",
          "from": 20,
          "to": 30,
          "doc_count": 450,
          "group_by_gender": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "M",
                "doc_count": 231,
                "average_balance": {
                  "value": 27400.982683982686
                }
              },
              {
                "key": "F",
                "doc_count": 219,
                "average_balance": {
                  "value": 25341.260273972603
                }
              }
            ]
          }
        },
        {
          "key": "30.0-40.0",
          "from": 30,
          "to": 40,
          "doc_count": 504,
          "group_by_gender": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "F",
                "doc_count": 253,
                "average_balance": {
                  "value": 25670.869565217392
                }
              },
              {
                "key": "M",
                "doc_count": 251,
                "average_balance": {
                  "value": 24288.239043824702
                }
              }
            ]
          }
        },
        {
          "key": "40.0-50.0",
          "from": 40,
          "to": 50,
          "doc_count": 45,
          "group_by_gender": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "M",
                "doc_count": 24,
                "average_balance": {
                  "value": 26474.958333333332
                }
              },
              {
                "key": "F",
                "doc_count": 21,
                "average_balance": {
                  "value": 27992.571428571428
                }
              }
            ]
          }
        }
      ]
    }
  }
}
```
还有许多其他的聚合方法，在这不再赘述。其他聚合查询可参考官方文档：

https://www.elastic.co/guide/en/elasticsearch/reference/5.6/search-aggregations.html

## 总结
ES是一款既简单又复杂的产品。目前我们已经学习了ES的基本用法，也清楚了ES是什么,它的内部是怎样的，如何使用它提供的RESTful API进行操作。

官方文档：
https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html

