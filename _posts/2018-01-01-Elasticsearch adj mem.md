---
layout: post
title: "Elascticsearch 节点优化：调整堆内存大小"
date: 2018-01-01
excerpt: "学习如何来优化基于Docker的ES的JVM性能"
tags: [Elasticsearch,Java,Docker]
slug: es-adj-mem
feature: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1522529189043&di=28588bffd119415e7f1efe842ec7dec6&imgtype=jpg&src=http%3A%2F%2Fimg4.imgtn.bdimg.com%2Fit%2Fu%3D2629733414%2C2046062133%26fm%3D214%26gp%3D0.jpg
---
**本文假设你已经熟悉JVM内存相关参数**

我的配置：
4台服务器组成的Elasticsearch集群，每台服务器只作为一个节点。
其中有两台数据节点
一台客户端节点
一台主节点
内存均为64G
启动命令：
{% highlight shell %}
docker run -d \
 --name=espn-50 \
-p 9200:9200 \ 
-p 9300:9300  \
-v \
 /var/espn/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \ 
-v \
/var/espn/data:/usr/share/elasticsearch/data elasticsearch:5.6.3
{% endhighlight %}
4台节点启动命令是一样的，暴露9200 9300端口，并挂载elasticsearch.yml及数据目录。
模拟了3个月数据，每天一个索引，索引数量90，每个索引有200W~300W条数据不等。

随着数据量增加，发现服务器CPU占用过多（没错，那两台数据节点）

查看这两台数据节点的elasticsearch日志：
```
 [gc][1779997] overhead, spent [45.2s] collecting in the last [45.8s] //这行日志若干行，就数字不一样
```

再去查询节点情况：
{% highlight shell %}
curl 'localhost:9200/_cat/nodes?v=pretty'
{% endhighlight %}
发现数据节点的 heap.percent 百分之99。
通过查询资料，我总结出来大概是这样的：

1. ES本身是Java项目，既然是Java项目，必然是运行在JVM上的，现在堆内存已满。
2. 堆内存已满，Elasticsearch正在尝试GC，然而每次GC又打不到释放的效果，因此不断的在GC，不断的使用CPU，因此CPU使用严重。
3. 索引数量比较多，而ES采用的倒排索引和segment这种机制，使得在比较多的索引而且索引数据必须要使用的情况下，必须调整JVM，也就是对JVM进行优化。
4. ES默认的Heap太小，一般用来做大数据存储是不够的，大概率是要手动调整的（2.X 版本默认1G，5.6.3是2G）

查了一圈网上的资料，包括官网，发现一个蛋疼的事实：
不同版本的ES调整JVM的方法都不太一样。
我使用的是 **Elasticsearch5.6.3**   ,再次强调一下这一点。

解决的方法很简单，在elasticsearch的config文件夹下，改写jvm.options文件即可。（5.6.3的docker容器中没有这个文件，我新写了一个文件挂载到容器内也是生效的）


jvm.options:
```
-Xms10g
-Xmx10g
```
不熟悉的话可以学习一下JVM的相关知识。

启动指令：
{% highlight shell %}
docker run -d --name=espn-50 -p 9200:9200 -p 9300:9300 --restart=always -v /var/espn/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v  /var/espn/config/jvm.options:/usr/share/elasticsearch/config/jvm.options -v /var/espn/data:/usr/share/elasticsearch/data elasticsearch:5.6.3
{% endhighlight %}
将jvm.options也挂载到docker容器内。注意，jvm.options的权限应该是可读可写可执行的。