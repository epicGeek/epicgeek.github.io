---
layout: post
title: "Elasticsearch Java API的基本使用"
date: 2017-12-15
excerpt: "学习使用Java来对ES进行增删改查等操作."
tags: [Elasticsearch,Java]
slug: es-java-api
feature: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1522529483188&di=d17440ed5565d3d0de700ddf2085d7e8&imgtype=0&src=http%3A%2F%2Fimg.colabug.com%2F2017%2F08%2Fdf101d9446223ef08e672f5e88c26253.png
---


## 说明
在明确了ES的基本概念和使用方法后，我们来学习如何使用ES的Java API.

## 客户端
你可以用Java客户端做很多事情：

* 执行标准的index,get,delete,update,search等操作。
* 在正在运行的集群上执行管理任务。

但是，通过官方文档可以得知，现在存在至少三种Java客户端。

1. Transport Client
1. Java High Level REST Client
1. Java Low Level Rest Client

造成这种混乱的原因是：
* 长久以来，ES并没有官方的Java客户端，并且Java自身是可以简单支持ES的API的，于是就先做成了TransportClient。但是TransportClient的缺点是显而易见的，它没有使用RESTful风格的接口，而是二进制的方式传输数据。
* 之后ES官方推出了Java Low Level REST Client,它支持RESTful，用起来也不错。但是缺点也很明显，因为TransportClient的使用者把代码迁移到Low Level REST Client的工作量比较大。官方文档专门为迁移代码出了一堆文档来提供参考。
* 现在ES官方推出Java High Level REST Client,它是基于Java Low Level REST Client的封装，并且API接收参数和返回值和TransportClient是一样的，使得代码迁移变得容易并且支持了RESTful的风格，兼容了这两种客户端的优点。当然缺点是存在的，就是版本的问题。ES的小版本更新非常频繁，在最理想的情况下，客户端的版本要和ES的版本一致（至少主版本号一致），次版本号不一致的话，基本操作也许可以，但是新API就不支持了。

* 强烈建议ES5及其以后的版本使用Java High Level REST Client。笔者这里使用的是ES5.6.3，下面的文章将基于JDK1.8+Spring Boot+ES5.6.3 Java High Level REST Client+Maven进行示例。

stackoverflow上的问答：

<https://stackoverflow.com/questions/47031840/elasticsearchhow-to-choose-java-client/47036028#47036028>

详细说明：

<https://www.elastic.co/blog/the-elasticsearch-java-high-level-rest-client-is-out>

参考资料：

<https://www.elastic.co/guide/en/elasticsearch/client/java-rest/5.6/java-rest-high.html>
## Java High Level REST Client 介绍
Java High Level REST Client 是基于Java Low Level REST Client的，每个方法都可以是同步或者异步的。同步方法返回响应对象，而异步方法名以“async”结尾，并需要传入一个监听参数，来确保提醒是否有错误发生。

Java High Level REST Client需要Java1.8版本和ES。并且ES的版本要和客户端版本一致。和TransportClient接收的参数和返回值是一样的。

以下实践均是基于5.6.3的ES集群和Java High Level REST Client的。

## Maven 依赖
{% highlight xml %}
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>5.6.3</version>
</dependency>
{% endhighlight %}

## 初始化


{% highlight java %}
        //Low Level Client init
		RestClient lowLevelRestClient = RestClient.builder(
		        new HttpHost("localhost", 9200, "http")).build(); 
		//High Level Client init
        RestHighLevelClient client =
			    new RestHighLevelClient(lowLevelRestClient);
{% endhighlight %}


High Level REST Client的初始化是依赖Low Level客户端的


## Index API
类似HTTP请求，Index API包括index request和index response
### Index request的构造
构造一条index request的例子：

{% highlight java %}
IndexRequest request = new IndexRequest(
        "posts", //index name 
        "doc",  // type
        "1");   // doc id
String jsonString = "{" +
        "\"user\":\"kimchy\"," +
        "\"postDate\":\"2013-01-30\"," +
        "\"message\":\"trying out Elasticsearch\"" +
        "}";
request.source(jsonString, XContentType.JSON);
{% endhighlight %}


注意到这里是使用的String 类型。
另一种构造的方法：

{% highlight java %}
Map<String, Object> jsonMap = new HashMap<>();
jsonMap.put("user", "kimchy");
jsonMap.put("postDate", new Date());
jsonMap.put("message", "trying out Elasticsearch");
IndexRequest indexRequest = new IndexRequest("posts", "doc", "1")
        .source(jsonMap); 
 //Map会自动转成JSON       

{% endhighlight %}
除了String和Map ,XContentBuilder 类型也是可以的：
{% highlight java %}
XContentBuilder builder = XContentFactory.jsonBuilder();
builder.startObject();
{
    builder.field("user", "kimchy");
    builder.field("postDate", new Date());
    builder.field("message", "trying out Elasticsearch");
}
builder.endObject();
IndexRequest indexRequest = new IndexRequest("posts", "doc", "1")
        .source(builder);  
{% endhighlight %}

更直接一点的，在实例化index request对象时，可以直接给出键值对:
{% highlight java %}
IndexRequest indexRequest = new IndexRequest("posts", "doc", "1")
        .source("user", "kimchy",
                "postDate", new Date(),
                "message", "trying out Elasticsearch"); 
{% endhighlight %}

### index response的获取
#### 同步执行
{% highlight java %}
IndexResponse indexResponse = client.index(request);
{% endhighlight %}
#### 异步执行
{% highlight java %}
client.indexAsync(request, new ActionListener<IndexResponse>() {
    @Override
    public void onResponse(IndexResponse indexResponse) {
        
    }

    @Override
    public void onFailure(Exception e) {
        
    }
});
{% endhighlight %}
需要注意的是，异步执行的方法名以Async结尾，并且多了一个Listener参数，并且需要重写回调方法。
在kibana控制台查询得到数据：
{% highlight json %}
{
  "_index": "posts",
  "_type": "doc",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "user": "kimchy",
    "postDate": "2017-11-01T05:48:26.648Z",
    "message": "trying out Elasticsearch"
  }
}
{% endhighlight %}
index request中的数据已经成功入库。
### index response的返回值操作
client.index()方法返回值类型为IndexResponse,我们可以用它来进行如下操作：
{% highlight java %}
String index = indexResponse.getIndex();  //index名称，类型等信息
String type = indexResponse.getType(); 
String id = indexResponse.getId();
long version = indexResponse.getVersion();
if (indexResponse.getResult() == DocWriteResponse.Result.CREATED) {
    
} else if (indexResponse.getResult() == DocWriteResponse.Result.UPDATED) {
    
}
ShardInfo shardInfo = indexResponse.getShardInfo();
//对分片使用的判断
if (shardInfo.getTotal() != shardInfo.getSuccessful()) {
    
}
if (shardInfo.getFailed() > 0) {
    for (ReplicationResponse.ShardInfo.Failure failure : shardInfo.getFailures()) {
        String reason = failure.reason(); 
    }
}
{% endhighlight %}

对version冲突的判断：
{% highlight java %}
IndexRequest request = new IndexRequest("posts", "doc", "1")
        .source("field", "value")
        .version(1);
try {
    IndexResponse response = client.index(request);
} catch(ElasticsearchException e) {
    if (e.status() == RestStatus.CONFLICT) {
        
    }
}
{% endhighlight %}
对index动作的判断：
{% highlight java %}
IndexRequest request = new IndexRequest("posts", "doc", "1")
        .source("field", "value")
        .opType(DocWriteRequest.OpType.CREATE);//create or update
try {
    IndexResponse response = client.index(request);
} catch(ElasticsearchException e) {
    if (e.status() == RestStatus.CONFLICT) {
        
    }
}
{% endhighlight %}

## GET API
### GET request
{% highlight java %}
GetRequest getRequest = new GetRequest(
        "posts",//index name 
        "doc",  //type
        "1");   //id
{% endhighlight %}

### GET response
同步方法：

{% highlight java %}
GetResponse getResponse = client.get(getRequest);
{% endhighlight %}

异步方法：
{% highlight java %}
client.getAsync(request, new ActionListener<GetResponse>() {
    @Override
    public void onResponse(GetResponse getResponse) {
        
    }

    @Override
    public void onFailure(Exception e) {
        
    }
});
{% endhighlight %}
对返回对象的操作：
{% highlight java %}
String index = getResponse.getIndex();
String type = getResponse.getType();
String id = getResponse.getId();
if (getResponse.isExists()) {
    long version = getResponse.getVersion();
    String sourceAsString = getResponse.getSourceAsString();        
    Map<String, Object> sourceAsMap = getResponse.getSourceAsMap(); 
    byte[] sourceAsBytes = getResponse.getSourceAsBytes();          
} else {
    //TODO
}
{% endhighlight %}
异常处理：
{% highlight java %}
GetRequest request = new GetRequest("does_not_exist", "doc", "1");
try {
    GetResponse getResponse = client.get(request);
} catch (ElasticsearchException e) {
    if (e.status() == RestStatus.NOT_FOUND) {
        
    }
    if (e.status() == RestStatus.CONFLICT) {
        
    }
}
{% endhighlight %}

## DELETE API
与Index API和 GET API及其相似
### DELETE request
{% highlight java %}
DeleteRequest request = new DeleteRequest(
        "posts",    
        "doc",     
        "1");      
{% endhighlight %}
### DELETE response
同步：
{% highlight java %}
DeleteResponse deleteResponse = client.delete(request);
{% endhighlight %}
异步:
{% highlight java %}
client.deleteAsync(request, new ActionListener<DeleteResponse>() {
    @Override
    public void onResponse(DeleteResponse deleteResponse) {
        
    }

    @Override
    public void onFailure(Exception e) {
        
    }
});
{% endhighlight %}

## Update API
### update request
{% highlight java %}
UpdateRequest updateRequest = new UpdateRequest(
        "posts", 
        "doc",  
        "1");   
{% endhighlight %}

update脚本：
在之前我们介绍了如何使用简单的脚本来更新数据
```
POST /posts/doc/1/_update?pretty
{
  "script" : "ctx._source.age += 5"
}
```
也可以写成:
```
POST /posts/doc/1/_update?pretty
{
  "script" : {
    "lang":"painless",
    "source":"ctx._source.age += 5"
  }
}
```
对应代码：
{% highlight java %}
        UpdateRequest updateRequest = new UpdateRequest("posts", "doc", "1");
		Map<String, Object> parameters = new HashMap<>();
		parameters.put("age", 4); 
		Script inline = new Script(ScriptType.INLINE, "painless", "ctx._source.age += params.age", parameters);  
		updateRequest.script(inline);
		try {
			UpdateResponse updateResponse = client.update(updateRequest);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
{% endhighlight %}
### 使用部分文档更新
1. String
{% highlight java %}
        String jsonString = "{" +
		        "\"updated\":\"2017-01-02\"," +
		        "\"reason\":\"easy update\"" +
		        "}";
		updateRequest.doc(jsonString, XContentType.JSON); 
		try {
			client.update(updateRequest);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
{% endhighlight %}

2.Map
{% highlight java %}
        Map<String, Object> jsonMap = new HashMap<>();
		jsonMap.put("updated", new Date());
		jsonMap.put("reason", "dailys update");
		UpdateRequest updateRequest = new UpdateRequest("posts", "doc", "1").doc(jsonMap);
		try {
			client.update(updateRequest);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
{% endhighlight %}

3.XContentBuilder 
{% highlight java %}
    try {
			XContentBuilder builder = XContentFactory.jsonBuilder();
			builder.startObject();
			{
			    builder.field("updated", new Date());
			    System.out.println(new Date());
			    builder.field("reason", "daily update");
			}
			builder.endObject();
			UpdateRequest request = new UpdateRequest("posts", "doc", "1")
			        .doc(builder);
			client.update(request);
		} catch (IOException e) {
			// TODO: handle exception
		}
{% endhighlight %}

4.键值对
{% highlight java %}
    try {
			UpdateRequest request = new UpdateRequest("posts", "doc", "1")
			        .doc("updated", new Date(),
			             "reason", "daily updatesss"); 
			client.update(request);
		} catch (IOException e) {
			// TODO: handle exception
		}
{% endhighlight %}
### upsert
如果文档不存在，可以使用upsert来生成这个文档。
{% highlight java %}
String jsonString = "{\"created\":\"2017-01-01\"}";
request.upsert(jsonString, XContentType.JSON);
{% endhighlight %}
同样地，upsert可以接Map,Xcontent，键值对参数。

### update response
同样地，update response可以是同步的，也可以是异步的

同步执行:
{% highlight java %}
UpdateResponse updateResponse = client.update(request);
{% endhighlight %}
异步执行：
{% highlight java %}
   client.updateAsync(request, new ActionListener<UpdateResponse>() {
    @Override
    public void onResponse(UpdateResponse updateResponse) {
        
    }

    @Override
    public void onFailure(Exception e) {
        
    }
});
{% endhighlight %}
与其他response类似，update response返回对象可以进行各种判断操作，这里不再赘述。


## Bulk API

### Bulk request
之前的文档说明过，bulk接口是批量index/update/delete操作
在API中，只需要一个bulk request就可以完成一批请求。
{% highlight java %}
BulkRequest request = new BulkRequest(); 
request.add(new IndexRequest("posts", "doc", "1")  
        .source(XContentType.JSON,"field", "foo"));
request.add(new IndexRequest("posts", "doc", "2")  
        .source(XContentType.JSON,"field", "bar"));
request.add(new IndexRequest("posts", "doc", "3")  
        .source(XContentType.JSON,"field", "baz"));
{% endhighlight %}
* 注意，Bulk API只接受JSON和SMILE格式.其他格式的数据将会报错。
* 不同类型的request可以写在同一个bulk request里。
{% highlight java %}
BulkRequest request = new BulkRequest();
request.add(new DeleteRequest("posts", "doc", "3")); 
request.add(new UpdateRequest("posts", "doc", "2") 
        .doc(XContentType.JSON,"other", "test"));
request.add(new IndexRequest("posts", "doc", "4")  
        .source(XContentType.JSON,"field", "baz"));
{% endhighlight %}

### bulk response
同步执行:
{% highlight java %}
BulkResponse bulkResponse = client.bulk(request);
{% endhighlight %}
异步执行：
{% highlight java %}
client.bulkAsync(request, new ActionListener<BulkResponse>() {
    @Override
    public void onResponse(BulkResponse bulkResponse) {
        
    }

    @Override
    public void onFailure(Exception e) {
        
    }
});
{% endhighlight %}
对response的处理与其他类型的response十分类似，在这不再赘述。

### bulk processor
BulkProcessor 简化bulk API的使用，并且使整个批量操作透明化。
BulkProcessor 的执行需要三部分组成：
1. RestHighLevelClient :执行bulk请求并拿到响应对象。
1. BulkProcessor.Listener：在执行bulk request之前、之后和当bulk response发生错误时调用。
1. ThreadPool：bulk request在这个线程池中执行操作，这使得每个请求不会被挡住，在其他请求正在执行时，也可以接收新的请求。

示例代码：
{% highlight java %}
		Settings settings = Settings.EMPTY; 
		ThreadPool threadPool = new ThreadPool(settings); //构建新的线程池
		BulkProcessor.Listener listener = new BulkProcessor.Listener() { 
            //构建bulk listener

		    @Override
		    public void beforeBulk(long executionId, BulkRequest request) {
                //重写beforeBulk,在每次bulk request发出前执行,在这个方法里面可以知道在本次批量操作中有多少操作数
		        int numberOfActions = request.numberOfActions(); 
		        logger.debug("Executing bulk [{}] with {} requests", executionId, numberOfActions);
		    }

		    @Override
		    public void afterBulk(long executionId, BulkRequest request, BulkResponse response) {
                //重写afterBulk方法，每次批量请求结束后执行，可以在这里知道是否有错误发生。
		        if (response.hasFailures()) { 
		            logger.warn("Bulk [{}] executed with failures", executionId);
		        } else {
		            logger.debug("Bulk [{}] completed in {} milliseconds", executionId, response.getTook().getMillis());
		        }
		    }

		    @Override
		    public void afterBulk(long executionId, BulkRequest request, Throwable failure) {
                //重写方法，如果发生错误就会调用。
		        logger.error("Failed to execute bulk", failure); 
		    }
			
		};
        BulkProcessor.Builder builder = new BulkProcessor.Builder(client::bulkAsync, listener, threadPool);//使用builder做批量操作的控制
		BulkProcessor bulkProcessor = builder.build();
        //在这里调用build()方法构造bulkProcessor,在底层实际上是用了bulk的异步操作

		builder.setBulkActions(500); //执行多少次动作后刷新bulk.默认1000，-1禁用
		builder.setBulkSize(new ByteSizeValue(1L, ByteSizeUnit.MB));//执行的动作大小超过多少时，刷新bulk。默认5M，-1禁用 
		builder.setConcurrentRequests(0);//最多允许多少请求同时执行。默认是1，0是只允许一个。 
		builder.setFlushInterval(TimeValue.timeValueSeconds(10L));//设置刷新bulk的时间间隔。默认是不刷新的。 
		builder.setBackoffPolicy(BackoffPolicy.constantBackoff(TimeValue.timeValueSeconds(1L), 3)); //设置补偿机制参数。由于资源限制（比如线程池满），批量操作可能会失败，在这定义批量操作的重试次数。

        //新建三个 index 请求
		IndexRequest one = new IndexRequest("posts", "doc", "1").
		        source(XContentType.JSON, "title", "In which order are my Elasticsearch queries executed?");
		IndexRequest two = new IndexRequest("posts", "doc", "2")
		        .source(XContentType.JSON, "title", "Current status and upcoming changes in Elasticsearch");
		IndexRequest three = new IndexRequest("posts", "doc", "3")
		        .source(XContentType.JSON, "title", "The Future of Federated Search in Elasticsearch");
        //新的三条index请求加入到上面配置好的bulkProcessor里面。
		bulkProcessor.add(one);
		bulkProcessor.add(two);
		bulkProcessor.add(three);
        // add many request here.
        //bulkProcess必须被关闭才能使上面添加的操作生效
        bulkProcessor.close(); //立即关闭
        //关闭bulkProcess的两种方法：
        try {
            //2.调用awaitClose.
            //简单来说，就是在规定的时间内，是否所有批量操作完成。全部完成，返回true,未完成返//回false
            
			boolean terminated = bulkProcessor.awaitClose(30L, TimeUnit.SECONDS);
            
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
{% endhighlight %}

## Search API

### Search request
Search API提供了对文档的查询和聚合的查询。
它的基本形式:
{% highlight java %}
SearchRequest searchRequest = new SearchRequest();  //构造search request .在这里无参，查询全部索引
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();//大多数查询参数要写在searchSourceBuilder里 
searchSourceBuilder.query(QueryBuilders.matchAllQuery());//增加match_all的条件。 
{% endhighlight %}

{% highlight java %}
SearchRequest searchRequest = new SearchRequest("posts"); //指定posts索引
searchRequest.types("doc"); //指定doc类型
{% endhighlight %}
### 使用SearchSourceBuilder
大多数的查询控制都可以使用SearchSourceBuilder实现。
举一个简单例子:
{% highlight java %}
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder(); //构造一个默认配置的对象
sourceBuilder.query(QueryBuilders.termQuery("user", "kimchy")); //设置查询
sourceBuilder.from(0); //设置从哪里开始
sourceBuilder.size(5); //每页5条
sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS)); //设置超时时间
{% endhighlight %}
配置好searchSourceBuilder后，将它传入searchRequest里：
{% highlight java %}
SearchRequest searchRequest = new SearchRequest();
searchRequest.source(sourceBuilder);
{% endhighlight %}
### 建立查询
在上面的例子，我们注意到，sourceBuilder构造查询条件时，使用QueryBuilders对象.
在所有ES查询中，它存在于所有ES支持的查询类型中。
使用它的构造体来创建:
{% highlight java %}
MatchQueryBuilder matchQueryBuilder = new MatchQueryBuilder("user", "kimchy");
{% endhighlight %}
这里的代码相当于：
{% highlight java %}
 "query": { "match": { "user": "kimchy" } }
{% endhighlight %}

相关设置：
{% highlight java %}
matchQueryBuilder.fuzziness(Fuzziness.AUTO);  //是否模糊查询
matchQueryBuilder.prefixLength(3); //设置前缀长度
matchQueryBuilder.maxExpansions(10);//设置最大膨胀系数 ？？？
{% endhighlight %}
QueryBuilder还可以使用 QueryBuilders工具类来创造，编程体验比较顺畅：
{% highlight java %}
QueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("user", "kimchy")
                                                .fuzziness(Fuzziness.AUTO)
                                                .prefixLength(3)
                                                .maxExpansions(10);
{% endhighlight %}
无论QueryBuilder对象是如何创建的，都要将它传入SearchSourceBuilder里面：
{% highlight java %}
searchSourceBuilder.query(matchQueryBuilder);
{% endhighlight %}
在之前导入的account数据中，使用match的示例代码:
{% highlight java %}
GET /bank/_search?pretty
{
  "query": {
    "match": {
      "firstname": "Virginia"  
   }
  }
}
{% endhighlight %}
JAVA:
{% highlight java %}
	@Test
	public void test2(){
		RestClient lowLevelRestClient = RestClient.builder(
		        new HttpHost("172.16.73.50", 9200, "http")).build();
		RestHighLevelClient client =
			    new RestHighLevelClient(lowLevelRestClient);
		SearchRequest searchRequest = new SearchRequest("bank");
		searchRequest.types("account");
		SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
		MatchQueryBuilder mqb = QueryBuilders.matchQuery("firstname", "Virginia");
		searchSourceBuilder.query(mqb);
		searchRequest.source(searchSourceBuilder);
		try {
			SearchResponse searchResponse = client.search(searchRequest);
			System.out.println(searchResponse.toString());
		} catch (IOException e) {
			e.printStackTrace();
		}
		
	}
{% endhighlight %}

### 排序
SearchSourceBuilder可以添加一种或多种SortBuilder。
有四种特殊的排序实现：
* field
* score
* GeoDistance
* scriptSortBuilder
{% highlight java %}
sourceBuilder.sort(new ScoreSortBuilder().order(SortOrder.DESC)); //按照score倒序排列
sourceBuilder.sort(new FieldSortBuilder("_uid").order(SortOrder.ASC));  //并且按照id正序排列
{% endhighlight %}

### 过滤
默认情况下，searchRequest返回文档内容，与REST API一样，这里你可以重写search行为。例如，你可以完全关闭"_source"检索。
{% highlight java %}
sourceBuilder.fetchSource(false);
{% endhighlight %}
该方法还接受一个或多个通配符模式的数组，以更细粒度地控制包含或排除哪些字段。
{% highlight java %}
String[] includeFields = new String[] {"title", "user", "innerObject.*"};
String[] excludeFields = new String[] {"_type"};
sourceBuilder.fetchSource(includeFields, excludeFields);
{% endhighlight %}

### 聚合请求
通过配置适当的 AggregationBuilder ，再将它传入SearchSourceBuilder里,就可以完成聚合请求了。
之前的文档里面，我们通过下面这条命令，导入了一千条account信息:
{% highlight shell %}
curl -H "Content-Type: application/json" -XPOST 'localhost:9200/bank/account/_bulk?pretty&refresh' --data-binary "@accounts.json"
{% endhighlight %}

随后，我们介绍了如何通过聚合请求进行分组:
{% highlight json %}
GET /bank/_search?pretty
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
{% endhighlight %}
我们将这一千条数据根据state字段分组，得到响应：
{% highlight json %}
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
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
{% endhighlight %}

Java实现:
{% highlight java %}
	@Test
	public void test2(){
		RestClient lowLevelRestClient = RestClient.builder(
		        new HttpHost("172.16.73.50", 9200, "http")).build();
		RestHighLevelClient client =
			    new RestHighLevelClient(lowLevelRestClient);
		SearchRequest searchRequest = new SearchRequest("bank");
		searchRequest.types("account");
		TermsAggregationBuilder aggregation = AggregationBuilders.terms("group_by_state")
		        .field("state.keyword");
		SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
		searchSourceBuilder.aggregation(aggregation);
		searchSourceBuilder.size(0);
		searchRequest.source(searchSourceBuilder);
		try {
			SearchResponse searchResponse = client.search(searchRequest);
			System.out.println(searchResponse.toString());
		} catch (IOException e) {
			e.printStackTrace();
		}
		
	}
{% endhighlight %}
输出:
{% highlight json %}
{"took":4,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":999,"max_score":0.0,"hits":[]},"aggregations":{"sterms#group_by_state":{"doc_count_error_upper_bound":20,"sum_other_doc_count":770,"buckets":[{"key":"ID","doc_count":27},{"key":"TX","doc_count":27},{"key":"AL","doc_count":25},{"key":"MD","doc_count":25},{"key":"TN","doc_count":23},{"key":"MA","doc_count":21},{"key":"NC","doc_count":21},{"key":"ND","doc_count":21},{"key":"MO","doc_count":20},{"key":"AK","doc_count":19}]}}}
{% endhighlight %}
### 同步执行
{% highlight java %}
SearchResponse searchResponse = client.search(searchRequest);
{% endhighlight %}
### 异步执行
{% highlight java %}
client.searchAsync(searchRequest, new ActionListener<SearchResponse>() {
    @Override
    public void onResponse(SearchResponse searchResponse) {
        
    }

    @Override
    public void onFailure(Exception e) {
        
    }
});
{% endhighlight %}

### Search response
Search response返回对象与其在API里的一样，返回一些元数据和文档数据。
首先，返回对象里的数据十分重要，因为这是查询的返回结果、使用分片情况、文档数据,HTTP状态码等

{% highlight java %}
RestStatus status = searchResponse.status();
TimeValue took = searchResponse.getTook();
Boolean terminatedEarly = searchResponse.isTerminatedEarly();
boolean timedOut = searchResponse.isTimedOut();
{% endhighlight %}
其次，返回对象里面包含关于分片的信息和分片失败的处理:
{% highlight java %}
int totalShards = searchResponse.getTotalShards();
int successfulShards = searchResponse.getSuccessfulShards();
int failedShards = searchResponse.getFailedShards();
for (ShardSearchFailure failure : searchResponse.getShardFailures()) {
    // failures should be handled here
}
{% endhighlight %}

### 取回searchHit
为了取回文档数据，我们要从search response的返回对象里先得到searchHit对象。
{% highlight java %}
SearchHits hits = searchResponse.getHits();
{% endhighlight %}
取回文档数据：
{% highlight java %}
	@Test
	public void test2(){
		RestClient lowLevelRestClient = RestClient.builder(
		        new HttpHost("172.16.73.50", 9200, "http")).build();
		RestHighLevelClient client =
			    new RestHighLevelClient(lowLevelRestClient);
		SearchRequest searchRequest = new SearchRequest("bank");
		searchRequest.types("account");
		SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
		searchRequest.source(searchSourceBuilder);
		try {
			SearchResponse searchResponse = client.search(searchRequest);
			SearchHits searchHits = searchResponse.getHits();
			SearchHit[] searchHit = searchHits.getHits();
			for (SearchHit hit : searchHit) {
				System.out.println(hit.getSourceAsString());
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
		
	}
{% endhighlight %}
根据需要，还可以转换成其他数据类型:
{% highlight java %}
String sourceAsString = hit.getSourceAsString();
Map<String, Object> sourceAsMap = hit.getSourceAsMap();
String documentTitle = (String) sourceAsMap.get("title");
List<Object> users = (List<Object>) sourceAsMap.get("user");
Map<String, Object> innerObject = (Map<String, Object>) sourceAsMap.get("innerObject");
{% endhighlight %}

### 取回聚合数据
聚合数据可以通过SearchResponse返回对象，取到它的根节点，然后再根据名称取到聚合数据。

{% highlight json %}
GET /bank/_search?pretty
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
{% endhighlight %}
响应:
{% highlight json %}
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
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
{% endhighlight %}
Java实现:
{% highlight java %}
    @Test
	public void test2(){
		RestClient lowLevelRestClient = RestClient.builder(
		        new HttpHost("172.16.73.50", 9200, "http")).build();
		RestHighLevelClient client =
			    new RestHighLevelClient(lowLevelRestClient);
		SearchRequest searchRequest = new SearchRequest("bank");
		searchRequest.types("account");
		TermsAggregationBuilder aggregation = AggregationBuilders.terms("group_by_state")
		        .field("state.keyword");
		SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
		searchSourceBuilder.aggregation(aggregation);
		searchSourceBuilder.size(0);
		searchRequest.source(searchSourceBuilder);
		try {
			SearchResponse searchResponse = client.search(searchRequest);
			Aggregations aggs = searchResponse.getAggregations();
			Terms byStateAggs = aggs.get("group_by_state");
			Terms.Bucket b = byStateAggs.getBucketByKey("ID"); //只取key是ID的bucket
			System.out.println(b.getKeyAsString()+","+b.getDocCount());
			System.out.println("!!!");
			List<? extends Bucket> aggList = byStateAggs.getBuckets();//获取bucket数组里所有数据
			for (Bucket bucket : aggList) {
				System.out.println("key:"+bucket.getKeyAsString()+",docCount:"+bucket.getDocCount());;
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
{% endhighlight %}

## Search Scroll API
search scroll API是用于处理search request里面的大量数据的。
* 使用ES做分页查询有两种方法。一是配置search request的from,size参数。二是使用scroll API。搜索结果建议使用scroll API，查询效率高。

为了使用scroll，按照下面给出的步骤执行：

### 初始化search scroll上下文
带有scroll参数的search请求必须被执行，来初始化scroll session。ES能检测到scroll参数的存在，保证搜索上下文在相应的时间间隔里存活
{% highlight java %}
SearchRequest searchRequest = new SearchRequest("account"); //从 account 索引中查询
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
searchSourceBuilder.query(matchQuery("first", "Virginia")); //match条件 
searchSourceBuilder.size(size); //一次取回多少数据
searchRequest.source(searchSourceBuilder);
searchRequest.scroll(TimeValue.timeValueMinutes(1L));//设置scroll间隔 
SearchResponse searchResponse = client.search(searchRequest);
String scrollId = searchResponse.getScrollId(); //取回这条响应的scroll id,在后续的scroll调用中会用到
SearchHit[] hits = searchResponse.getHits().getHits();//得到文档数组 
{% endhighlight %}
### 取回所有相关文档
第二步，得到的scroll id 和新的scroll间隔要设置到 SearchScrollRequest里，再调用searchScroll方法。
ES会返回一批带有新scroll id的查询结果。以此类推，新的scroll id可以用于子查询，来得到另一批新数据。这个过程应该在一个循环内，直到没有数据返回为止,这意味着scroll消耗殆尽，所有匹配上的数据都已经取回。

{% highlight java %}
SearchScrollRequest scrollRequest = new SearchScrollRequest(scrollId);  //传入scroll id并设置间隔。
scrollRequest.scroll(TimeValue.timeValueSeconds(30));
SearchResponse searchScrollResponse = client.searchScroll(scrollRequest);//执行scroll搜索
scrollId = searchScrollResponse.getScrollId();  //得到本次scroll id
hits = searchScrollResponse.getHits(); 
{% endhighlight %}

### 清理 scroll 上下文
使用Clear scroll API来检测到最后一个scroll id 来释放scroll上下文.虽然在scroll过期时，这个清理行为会最终自动触发，但是最好的实践是当scroll session结束时，马上释放它。

### 可选参数
{% highlight java %}
scrollRequest.scroll(TimeValue.timeValueSeconds(60L));  //设置60S的scroll存活时间
scrollRequest.scroll("60s"); //字符串参数
{% endhighlight %}
如果在scrollRequest不设置的话，会以searchRequest.scroll()设置的为准。

### 同步执行
{% highlight java %}
SearchResponse searchResponse = client.searchScroll(scrollRequest);
{% endhighlight %}

### 异步执行

{% highlight java %}
client.searchScrollAsync(scrollRequest, new ActionListener<SearchResponse>() {
    @Override
    public void onResponse(SearchResponse searchResponse) {
        
    }

    @Override
    public void onFailure(Exception e) {
        
    }
});
{% endhighlight %}

* 需要注意的是，search scroll API的请求响应返回值也是一个searchResponse对象。

### 完整示例
{% highlight java %}
    @Test
	public void test3(){
		RestClient lowLevelRestClient = RestClient.builder(
		        new HttpHost("172.16.73.50", 9200, "http")).build();
		RestHighLevelClient client =
			    new RestHighLevelClient(lowLevelRestClient);
		SearchRequest searchRequest = new SearchRequest("bank");
		SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
		MatchAllQueryBuilder mqb = QueryBuilders.matchAllQuery();
		searchSourceBuilder.query(mqb);
		searchSourceBuilder.size(10); 
		searchRequest.source(searchSourceBuilder);
		searchRequest.scroll(TimeValue.timeValueMinutes(1L)); 
		try {
			SearchResponse searchResponse = client.search(searchRequest);
			String scrollId = searchResponse.getScrollId(); 
			SearchHit[] hits = searchResponse.getHits().getHits();
			System.out.println("first scroll:");
			for (SearchHit searchHit : hits) {
				System.out.println(searchHit.getSourceAsString());
			}
			Scroll scroll = new Scroll(TimeValue.timeValueMinutes(1L));
			System.out.println("loop scroll:");
			while(hits != null && hits.length>0){
				SearchScrollRequest scrollRequest = new SearchScrollRequest(scrollId); 
			    scrollRequest.scroll(scroll);
			    searchResponse = client.searchScroll(scrollRequest);
			    scrollId = searchResponse.getScrollId();
			    hits = searchResponse.getHits().getHits();
				for (SearchHit searchHit : hits) {
					System.out.println(searchHit.getSourceAsString());
				}
			}
			ClearScrollRequest clearScrollRequest = new ClearScrollRequest(); 
			clearScrollRequest.addScrollId(scrollId);
			ClearScrollResponse clearScrollResponse = client.clearScroll(clearScrollRequest);
			boolean succeeded = clearScrollResponse.isSucceeded();
			System.out.println("cleared:"+succeeded);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} 
	}
{% endhighlight %}

## Info API

Info API 提供一些关于集群、节点相关的信息查询。

### request
{% highlight java %}
MainResponse response = client.info();
{% endhighlight %}

### response

{% highlight java %}
ClusterName clusterName = response.getClusterName(); 
String clusterUuid = response.getClusterUuid(); 
String nodeName = response.getNodeName(); 
Version version = response.getVersion(); 
Build build = response.getBuild(); 
{% endhighlight %}


{% highlight java %}
	@Test
	public void test4(){
		RestClient lowLevelRestClient = RestClient.builder(
		        new HttpHost("172.16.73.50", 9200, "http")).build();
		RestHighLevelClient client =
			    new RestHighLevelClient(lowLevelRestClient);
		try {
			MainResponse response = client.info();
			ClusterName clusterName = response.getClusterName(); 
			String clusterUuid = response.getClusterUuid(); 
			String nodeName = response.getNodeName(); 
			Version version = response.getVersion(); 
			Build build = response.getBuild(); 
			System.out.println("cluster name:"+clusterName);
			System.out.println("cluster uuid:"+clusterUuid);
			System.out.println("node name:"+nodeName);
			System.out.println("node version:"+version);
			System.out.println("node name:"+nodeName);
			System.out.println("build info:"+build);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
{% endhighlight %}

## 总结

关于Elasticsearch 的 Java High Level REST Client API的基本用法大概就是这些，一些进阶技巧、概念要随时查阅官方文档。

地址：

<https://www.elastic.co/guide/en/elasticsearch/client/java-rest/5.6/java-rest-high.html>