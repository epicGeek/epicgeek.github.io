---
layout: post
title: "基于Docker的Elasticsearch集群部署"
date: 2018-02-01
excerpt: "学习如何来部署基于docker的Elastic集群"
tags: [Elasticsearch,Docker]
slug: es-cluster-deploy
---



**本文是基于ES5.6.3和Docker的集群部署、配置说明，如有错误或更好的建议请指正**

## 节点类型

### Master节点 （主节点）
```
node.master: true 
node.data: false
```
这样配置的节点为master节点。主节点的主要职责是和集群操作相关的内容，如创建或删除索引，跟踪哪些节点是群集的一部分，并决定哪些分片分配给相关的节点。稳定的主节点对集群的健康是非常重要的。

为了防止数据丢失，配置discovery.zen.minimum_master_nodes设置是至关重要的

（默认为1），每个主节点应该知道形成一个集群的最小数量的主资格节点的数量。

解释如下：

​ 假设我们有一个集群。有3个主资格节点，当网络发生故障的时候，有可能其中一个节点不能和其他节点进行通信了。这个时候，当discovery.zen.minimum_master_nodes设置为1的时候，就会分成两个小的独立集群，当网络好的时候，就会出现数据错误或者丢失数据的情况。当discovery.zen.minimum_master_nodes设置为2的时候，一个网络中有两个主资格节点，可以继续工作，另一部分，由于只有一个主资格节点，则不会形成一个独立的集群，这个时候当网络回复的时候，节点又会从新加入集群。

设置这个值的原则是：

（master_eligible_nodes / 2）+ 1

### Data节点（数据节点）

```
node.master: false 
node.data: true
```
数据节点主要是存储索引数据的节点，主要对文档进行增删改查操作，聚合操作等。数据节点对cpu，内存，io要求较高，在优化的时候需要监控数据节点的状态，当资源不够的时候，需要在集群中添加新的节点。


### Client节点 (客户端节点)
当主节点和数据节点配置都设置为false的时候，该节点只能处理路由请求，处理搜索，分发索引操作等，从本质上来说该客户节点表现为智能负载平衡器。独立的客户端节点在一个比较大的集群中是非常有用的，他协调主节点和数据节点，客户端节点加入集群可以得到集群的状态，根据集群的状态可以直接路由请求。 
警告：添加太多的客户端节点对集群是一种负担，因为主节点必须等待每一个节点集群状态的更新确认！客户节点的作用不应被夸大，数据节点也可以起到类似的作用。配置如下：
```
node.master: false 
node.data: false
```

**在配置ES集群的时候，要根据现场情况进行配置**

下面举例来进行集群配置。
## 准备工作
 - Elasticsearch5.6.3镜像
 - 两台服务器
 - 两份elasticsearch.yml配置文件

## 操作

首先我这里有两台服务器，172.16.73.49 和 172.16.73.50 
计划把172.16.73.50作为master节点，172.16.73.49作为data节点。

### 1.准备配置文件

在用docker启动master节点前，我们需要先写好master节点的elasticsearch.yml文件。我准备好的配置文件内容如下：
```
cluster.name: "boss-es-cluster"
node.name: node-50
node.master: true
node.data: true
network.host: 0.0.0.0
network.publish_host: 172.16.73.50
discovery.zen.ping.unicast.hosts: ["172.16.73.49"]
discovery.zen.minimum_master_nodes: 1
```

解释一下内容：

```
cluster.name:  //集群名称。如果想让多个节点加入一个集群，那么需要使集群名称一致。
node.name: //节点名，为这个节点起一个独一无二的名字
node.master: //该节点是否担任master角色
node.data: //该节点是否可以担任data节点的角色
network.host: //设置为0.0.0.0 ，意思是任何IP都可以访问
network.publish_host: //本节点在外部的IP
discovery.zen.minimum_master_nodes: //自动发现master节点的最小数，如果这个集群中配置进来的master节点少于这个数目，es的日志会一直报master节点数目不足。
discovery.zen.ping.unicast.hosts: // 按照我的理解，这里配置的host ip才是可以ping通的，因此在这里加上49的ip。因为本文是只有两个节点的es集群，所以只写对方的ip即可。如果是大于2个以上的节点的es集群，那么我想应该是在这里写上所有集群的ip
在这个配置中，注意到这个节点既是主节点又是数据节点，实际上对这个节点的压力是挺大的，在资源比较充裕的条件下不建议这样做。

```

对比一下 49 这个data节点的配置文件:
```
cluster.name: "boss-es-cluster"
node.name: node-49
node.master: false
node.data: true
network.host: 0.0.0.0
network.publish_host: 172.16.73.49
discovery.zen.ping.unicast.hosts: ["172.16.73.50"]
```

### 2.准备目录
在172.16.73.49和172.16.73.50上，都准备如下目录结构：

- /var/espn/config
- /var/espn/data

config是挂载elasticsearch.yml的目录
data是挂载数据的目录


### 3.启动master节点
在172.16.73.50上 ，执行：
```
docker run -d --name=espn-50 -p 9200:9200 -p 9300:9300  -v /var/espn/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /var/espn/data:/usr/share/elasticsearch/data elasticsearch:5.6.3
```
将master节点容器命名为espn-50,开放9200 和 9300端口，并挂载config目录下的elasticsearch.yml和data目录


### 4.启动data节点

在172.16.73.49上 ，执行：
```
docker run -d --name=espn-49 -p 9200:9200 -p 9300:9300  -v /var/espn/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /var/espn/data:/usr/share/elasticsearch/data elasticsearch:5.6.3
```

至此为止，如果docker启动无误，我们就可以来看一下各个es节点状态和集群状态了。


### 5.确认集群

在172.16.73.50上：
```
$ curl 'localhost:9200'
```
response:
```
{
  "name" : "node-50",
  "cluster_name" : "boss-es-cluster",
  "cluster_uuid" : "kNr1ejDGQ2GZCN3UYc_WGA",
  "version" : {
    "number" : "5.6.3",
    "build_hash" : "1a2f265",
    "build_date" : "2017-10-06T20:33:39.012Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```
节点启动正常！

查看节点健康度：
```
$ curl 'localhost:9200/_cat/health?v=pretty' 
```
```
epoch      timestamp cluster         status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1515988034 03:47:14  boss-es-cluster green           2         2      0   0    0    0        0             0                  -                100.0%
```

从结果可以看到，集群健康为绿色，有两个数据节点在集群中。

查看集群状况
```
$ curl 'localhost:9200/_cat/nodes?v=pretty' 
```
```
ip           heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.16.73.49           21          50   7    0.47    0.52     0.66 di        -      node-49
172.16.73.50           26         100   1    0.09    0.11     0.22 mdi       *      node-50
```
可以看见 ，基本的节点情况已经很清楚的看到集群的情况了。

