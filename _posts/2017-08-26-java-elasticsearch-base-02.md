---
layout: post
title:  "Elasticsearch 5.6.5 基础笔记(二) - Restfull API 和 分布式特性"
desc: "Elasticsearch 5.6.5 基础笔记(二) - Restfull API 和 分布式特性"
keywords: "Elasticsearch 5.6.5 基础笔记(二) - Restfull API 和 分布式特性,elasticsearch,es,kinglyjn"
date: 2017-08-26
categories: [Java]
tags: [java]
icon: fa-coffee
---



## Restfull API 

### 请求格式
```default
curl [-u xxx] [-I] -X[HEAD|POST|DELETE|PUT|GET] 'http://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' [-H 'Content-Type:application/json'] -d '<BODY>'

API基本格式：http://<ip>:<port>/<索引>/<类型>/<文档id>
常用HTTP动词：GET/PUT/POST/DELETE
索引名称：必须小写，不能有中划线

如何区分索引是结构化的还是非结构化的？
索引信息->mappings节点
当mappings节点后的内容为空时：非结构化索引
```
<br>

### 创建文档

当索引一个文档的时候，文档会被存储到一个主分片中。 Elasticsearch 如何知道一个文档应该存放到哪个分片中呢？当我们创建文档时，它如何决定这个文档应当被存储在分片1还是分片2中呢？首先这肯定不会是随机的，否则将来要获取文档的时候我们就不知道从何处寻找了。实际上，这个过程是根据下面这个公式决定的：
```default
shard = hash(routing) % number_of_primary_shards
```

routing 是一个可变值，默认是文档的 _id ，也可以设置成一个自定义的值。 routing 通过 hash 函数生成一个数字，然后这个数字再除以 number_of_primary_shards （主分片的数量）后得到 余数 。这个分布在 0 到 number_of_primary_shards-1 之间的余数，就是我们所寻求的文档所在分片的位置。这就解释了为什么我们要在创建索引的时候就确定好主分片的数量并且永远不会改变这个数量：因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了。你可能觉得由于ES主分片数量是固定的会使索引难以进行扩容。实际上当你需要时有很多技巧可以轻松实现扩容，详见 [点击](www.elastic.co/guide/cn/elasticsearch/guide/current/scale.html)。所有的文档 API（ get 、 index 、 delete 、 bulk 、 update 以及 mget ）都接受一个叫做 routing 的路由参数，通过这个参数我们可以自定义文档到分片的映射。一个自定义的路由参数可以用来确保所有相关的文档——例如所有属于同一个用户的文档——都被存储到同一个分片中。<br>
新建、索引和删除请求都是写操作，必须在主分片上面完成之后才能被复制到相关的副本分片。以下是在主副分片和任何副本分片上面 成功新建，索引和删除文档所需要的步骤顺序：
1. 客户端向 Node1 发送新建、索引或者删除请求;
2. 节点使用文档的_id确定文档属于分片0。请求会被转发到 Node 3`，因为分片 0 的主分片目前被分配在Node3上;
3. Node3在主分片上面执行请求。如果成功了，它将请求并行转发到 Node 1 和 Node 2 的副本分片上。一旦所有的副本分片都报告成功, Node 3 将向协调节点报告成功，协调节点向客户端报告成功。

```default
//创建的时候必须指定id；文档不存在则创建，存在则更新
PUT /{index}/{type}/{id}  
{
	"field": "value",
	...
}

//创建的时候必须指定id；文档不存在则创建，不存在则返回409状态码
PUT /{index}/{type}/{id}/_create  
{
	"field": "value",
	...
}

//创建的时候自动生成id
POST /{index}/{type}	
{
	"field": "value"
	...
}
```
<br>

### Bulk API 小批量导入或操作json数据

批量导入可以合并多个操作，比如index,delete,update,create等等。也可以帮助从一个索引导入到另一个索引。需要注意的是，每一条数据都由两行构成（delete除外），其他的命令比如index和create都是由元信息行和数据行组成，update比较特殊它的数据行可能是doc也可能是upsert或者script。<br>

bulk API 按如下步骤顺序执行：<br>

1. 客户端向 Node 1 发送 bulk 请求;
2. Node1 为每个节点创建一个批量请求，并将这些请求并行转发到每个包含主分片的节点主机。主分片一个接一个按顺序执行每个操作。当每个操作成功时，主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端。bulkAPI还可以在整个批量请求的最顶层使用consistency参数，以及在每个请求中的元数据中使用 routing 参数。
3. 由于每个子请求都是独立执行的，因此某个子请求的失败不会对其他子请求的成功与否造成影响。 如果其中任何子请求失败，返回的数据error字段标志被设置为 true ，并且在相应的请求报告出错误明细。这也意味着 bulk 请求不是原子的，不能用它来实现事务控制！

<br>

这种格式类似一个有效的单行 JSON 文档 流 ，它通过换行符(\n)连接到一起。注意以下几个要点：
* 每行一定要以换行符(\n)结尾， 包括最后一行 。这些换行符被用作一个标记，可以有效分隔行。
* 这些行不能包含未转义的换行符，因为他们将会对解析造成干扰。这意味着这个 JSON 不能使用 pretty 参数打印。
* 关于批量json文件的大小：编辑整个批量请求都需要由接收到请求的节点加载到内存中，因此该请求越大，其他请求所能获得的内存就越少。批量请求的大小有一个最佳值，大于这个值，性能将不再提升，甚至会下降。 但是最佳值不是一个固定的值。它完全取决于硬件、文档的大小和复杂度、索引和搜索的负载的整体情况。幸运的是，很容易找到这个最佳点，通过批量索引典型文档，并不断增加批量大小进行尝试。 当性能开始下降，那么你的批量大小就太大了。一个好的办法是开始时将 1,000 到 5,000 个文档作为一个批次, 如果你的文档非常大，那么就减少批量的文档个数。密切关注你的批量请求的物理大小往往非常有用，一千个 1KB 的文档是完全不同于一千个 1MB 文档所占的物理大小。一个好的批量大小在开始处理后所占用的物理大小约为 5-15 MB。<br>

数据示例一：data.json
```default
//相当于 PUT /test/type1/1/_create 操作
{ "create" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }      
{ "field1" : "value3" }

//相当于 PUT /test/type1/2 操作，不指定_id则会自动生成一个
{ "index" : { "_index" : "test", "_type" : "type1", "_id" : "2" } }		  
{ "field1" : "value1" }

//相当于 POST /test/type1/3/_update 操作
{ "update" : {"_index" : "test", "_type" : "type1", "_id" : "3" } }	  	  
{ "doc" : {"field2" : "value2"} }

//相当于 DELETE /test/type1/4 操作，谨记不要落下最后一个换行符！
{ "delete" : { "_index" : "test", "_type" : "type1", "_id" : "4" } }
	  
```
然后执行：curl -XPOST {hostname}:9200/_bulk --data-binary @data.json 即可 <br>

数据示例二：<br>
```default
在Url中设置默认的index和type，如果在路径中设置了index或者type，那么在JSON中就不需要设置了。
如果在JSON中设置，会覆盖掉路径中的配置。
```
然后执行：curl -XPOST {hostname}:9200/{index}/{type}/_bulk?pretty --data-binary @data.json <br>
<br>

### 简易搜索（query-string-search）

```default
//查看集群的状态，返回集群基本状态
GET /_cluster/health

//查看集群的状态，level 可取值 indices、shards 表示返回索引信息 或 每个索引的分片信息
GET _cluster/health?level=indices

//查看每个索引的基本情况
GET _cat/indices?v 

//查看某个索引类型的映射信息
GET /{index}/_mapping/{type}

//查看某个索引的映射信息	
GET /{index}/_mapping

//搜索文档，返回集群中所有索引下的所有文档，返回的hits表示查询结果的前10个文档，took表示请求毫秒数，_shards表示分片总数
GET /_search

//搜索文档，设置超时时间为10ms
GET /_search?timeout=10ms

//搜索文档，限定索引名称
GET /{index}/_search		

//搜索文档，使用通配符限定索引名称					 
GET /logstash*,mega*/_search

//搜索文档，限定索引名称、类型名称和文档id
GET /{index}/{type}/{id}/_search

//搜索文档，限定索引名称、类型名称 和字段条件(这里表示field1字段中包含xxx分词)
GET /{index}/{type}/_search?q=field1:xxx

//搜索文档，限定索引名称、类型名称 并且某个字段中含有xxx分词
GET /{index}/{type}/_search?q=xxx

//搜索文档，分页，注意在分布式系统中对结果排序的成本随分页的深度成指数上升，所以任何查询都最好不要返回超过1000个结果！	
//考虑到分页过深以及一次请求太多结果的情况，结果集在返回之前先进行排序。 但请记住一个请求经常跨越多个分片，每个分片都产生自己的排序结果，这些结果需要进行集中排序以保证整体顺序是正确的。
GET /_search?size=5&from=0	

//搜索文档，pretty参数可以使返回的json串更可读
GET /{index}/{type}/{id}?pretty

//搜索文档，仅返回部分字段信息		 
GET /{index}/{type}/{id}?_source=field1,field2   

//搜索文档，仅返回_source对象
GET /{index}/{type}/{id}/_source 

//搜索文档，指定返回的数列类型，可以是json或yaml
GET /megacorp/employee/1?format=json 

//搜索文档，指定版本，存在当前版本号的数据则正常返回，否则返回409冲突状态码
GET /{index}/{type}/{id}?version={_version}		 
```
<br>


### 请求体搜索（full-body-search）
对于一个查询请求，Elasticsearch 的工程师偏向于使用 GET 方式，因为他们觉得它比POST 能更好的描述信息检索（retrieving information）的行为。然而，因为带请求体的 GET 请求并不被广泛支持，所以 search API 同时支持 POST 请求：
```default
POST /_search
{
  "from": 30,
  "size": 10
}
```

#### mget搜索
以下是使用单个mget查询多个文档所需的步骤顺序：<br>
1. 客户端向 Node1 发送mget请求，Node1会创建一个大小为 from+size 的空优先队列;
2. Node1 为每个分片构建多文档获取请求，然后并行转发这些请求到托管在每个所需的主分片或者副本分片的节点上。一旦收到所有答复到Node1中的优先队列中，Node1构建响应并将其返回给客户端。可以对 docs 数组中每个文档设置 routing 参数。每个分片在本地执行查询请求并且创建一个长度为 from + size 的优先队列—也就是说，每个分片创建的结果集足够大，均可以满足全局的搜索请求。 分片返回一个轻量级的结果列表到协调节点，它仅包含文档 ID 集合以及任何排序需要用到的值，例如 _score。协调节点将这些分片级的结果合并到自己的有序优先队列里，它代表了全局排序结果集合。至此查询过程结束。

```default
//使用mget一次请求取回多个多个文档，返回的结果按请求的顺序排列，某个文档不存在则其对应的found字段为false
GET /_mget	 
{
	"docs": [
		{
			"_index": "indexname1",
			"_type": "type1",
			"_id": "id1",
		},
		{
			"_index": "indexname2",
			"_type": "type2",
			"_id": "id2",
			"_source": ["field1", "field2",...]
		},
		...
	]
}

//如果要搜索的所有文档都在同一个index（甚至可能type也相同），则可以在url中设置默认的index和type
GET /{index}/{type}/{id}/_mget		
{
	"docs": [
		{
			"_id": "id1"
		},
		{
			"_type": "type2",
			"_id": "id1"
		},
		...
	]
}

//如果要搜索的所有文档都在同一个index和type中则可以使用下面的mget方式发送请求
GET /{index}/{type}/{id}/_mget	
{
	"ids": ["id1", "id2", "id3"]
}
```

#### 深分页问题以及游标查询（scroll query）
查询阶段标识哪些文档满足 搜索请求，但是我们仍然需要取回这些文档。这是取回阶段的任务。分布式阶段由以下步骤构成：<br>
1. 协调节点辨别出哪些文档需要被取回并向相关的分片提交多个 GET 请求。
2. 每个分片加载并 丰富 文档，如果有需要的话，接着返回文档给协调节点。
3. 一旦所有的文档都被取回了，协调节点返回结果给客户端。

协调节点首先决定哪些文档 确实 需要被取回。例如，如果我们的查询指定了 { "from": 90, "size": 10 }，最初的90个结果会被丢弃，只有从第91个开始的10个结果需要被取回。这些文档可能来自和最初搜索请求有关的一个、多个甚至全部分片。先查后取的过程支持用 from 和 size 参数分页，但是这是 有限制的 。 要记住需要传递信息给协调节点的每个分片必须先创建一个 from + size 长度的队列，协调节点需要根据 number_of_shards * (from + size) 排序文档，来找到被包含在 size 里的文档。取决于你的文档的大小，分片的数量和你使用的硬件，给 10,000 到 50,000 的结果文档深分页（ 1,000 到 5,000 页）是完全可行的。但是使用足够大的 from 值，排序过程可能会变得非常沉重，使用大量的CPU、内存和带宽。因为这个原因，我们强烈建议你不要使用深分页。实际上，“深分页” 很少符合人的行为。当2到3页过去以后，人会停止翻页，并且改变搜索标准。会不知疲倦地一页一页的获取网页直到你的服务崩溃的罪魁祸首一般是机器人或者web spider。如果你确实需要从你的集群取回大量的文档，你可以通过用 scroll 查询禁用排序使这个取回行为更有效率。<br>

解决深度查询的内存消耗问题，使用`游标查询`（scroll query）：<br>
scroll查询 可以用来对 Elasticsearch 有效地执行大批量的文档查询，而又不用付出深度分页那种代价。游标查询允许我们先做查询初始化，然后再批量地拉取结果，这有点儿像传统数据库中的 cursor。游标查询会取某个时间点的快照数据。 查询初始化之后索引上的任何变化会被它忽略。它通过保存旧的数据文件来实现这个特性，结果就像保留初始化时的索引视图一样。深度分页的代价根源是结果集全局排序，如果去掉全局排序的特性的话查询结果的成本就会很低。 游标查询用字段 _doc 来排序。这个指令让 Elasticsearch 仅仅从还有结果的分片返回下一批结果。启用游标查询可以通过在查询的时候设置参数 scroll 的值为我们期望的游标查询的过期时间。游标查询的过期时间会在每次做查询的时候刷新，所以这个时间只需要足够处理当前批的结果就可以了，而不是处理查询结果的所有文档的所需时间。这个过期时间的参数很重要, 因为保持这个游标查询窗口需要消耗资源，所以我们期望如果不再需要维护这种资源就该早点儿释放掉。设置这个超时能够让 Elasticsearch在稍后空闲的时候自动释放这部分资源。<br>

```default
//保持游标查询窗口一分钟
//关键字 _doc 是最有效的排序顺序
GET /old_index/_search?scroll=1m 				
{
    "query": { "match_all": {} },
    "sort" : [ "_doc" ], 						
    "size":  1000
}
```
这个查询的返回结果包括一个字段 _scroll_id`，它是一个base64编码的长字符串 ((("scroll_id")))，现在我们能传递字段`_scroll_id 到 _search/scroll 查询接口获取下一批结果。尽管我们指定字段 size 的值为1000，我们有可能取到超过这个值数量的文档。当查询的时候，字段 size 作用于单个分片，所以每个批次实际返回的文档数量最大为 size * number_of_primary_shards。当没有更多的结果返回的时候，我们就处理完所有匹配的文档了。

```default
//注意再次设置游标查询过期时间为一分钟
GET /_search/scroll
{
    "scroll": "1m", 
    "scroll_id" : "cXVlcnlUaGVuRmV0Y2g7NTsxM...mJHYWM4U2E2eUIzVkMxalZidFE7MDs="
}
```
<br>

#### match_all搜索

```default
//相当于 GET /_search
GET /_search									
{
	"match_all": {}
}
```
<br>

#### match搜索

```default
//相当于 GET /_search?q=xxx
GET /_search									
{
	"query": {
		"match": "xxx"
	}
}

//相当于 GET /_search?q=tweet:xxx
//match是 精确匹配还是全文匹配 取决于match的字段是 text类型的 还是非text类型的[keyword、integer、date、bool等]
GET /_search	
{												
	"query": {
		"match": { "tweet": "xxx" }
	}
}
```
<br>

#### multi_match搜索

```default
//multi_match 查询可以在多个字段上执行相同的 match 查询
GET /_search									
{
	"multi_match": {
		"query": "full text search",
		"fields": [ "title", "body" ]
	}
}
```
<br>

#### term搜索：

```default	
//term搜索被用于精确值 匹配，这些精确值可能是数字、时间、布尔或者那些 not_analyzed 的字符串		
//term 搜索对于输入的文本不 分析 ，所以它将给定的值进行精确查询	
GET /_search									
{
	"term": { 
		"date": "2017-10-10" 
	}	
}
```
<br>

#### terms搜索

```default
//terms 查询和 term 查询一样，但它允许你指定多值进行匹配
//如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件
GET /_search									
{
	"terms": {
		"tag": [ "xxx", "xxx", "xxx" ]
	}
}
```
<br>

#### range搜索

```default
//range搜索找出那些落在指定区间内的数字或者时间
GET /_search	
{
	"range": {
		"age": { "gte": 20, "lt": 30 }
	}
}
```
<br>

#### exists搜索和missing搜索

```default
//exists搜索和missing搜索被用于查找那些指定字段中有值 (exists) 或无值 (missing) 的文档
GET /_search
{
	"exists": {
		"field": "title"
	},
	"missing": {
		"field": "content"
	}
}
```
<br>

#### 组合多查询

现实的查询需求从来都没有那么简单；它们需要在多个字段上查询多种多样的文本，并且根据一系列的标准来过滤。为了构建类似的高级查询，你需要一种能够将多查询组合成单一查询的查询方法。你可以用 bool 查询来实现你的需求。这种查询将多查询组合在一起，成为用户自己想要的布尔查询。它接收以下参数：
```default
must：		文档必须匹配这些条件才能被包含进来。
must_not：	文档必须不匹配这些条件才能被包含进来。
should：	如果满足这些语句中的任意语句，将增加 _score ，否则，无任何影响。它们主要用于修正每个文档的相关性得分。
filter：	必须 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。
```
由于这是我们看到的第一个包含多个查询的查询，所以有必要讨论一下相关性得分是如何组合的。每一个子查询都独自地计算文档的相关性得分。一旦他们的得分被计算出来， bool 查询就将这些得分进行合并并且返回一个代表整个布尔操作的得分。下面的查询用于查找 title 字段匹配 how to make millions 并且不被标识为 spam 的文档。那些被标识为 starred 或在2014之后的文档，将比另外那些文档拥有更高的排名。如果 _两者_ 都满足，那么它排名将更高：

```default
{
   "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
}
```
如果我们不想因为文档的时间而影响得分，可以用 filter 语句来重写前面的例子：
```default
{
   "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "range": { "date": { "gte": "2014-01-01" }} 
        }
    }
}
```
通过将 range 查询移到 filter 语句中，我们将它转成不评分的查询，将不再影响文档的相关性排名。由于它现在是一个不评分的查询，可以使用各种对 filter 查询有效的优化手段来提升性能。所有查询都可以借鉴这种方式。将查询移到 bool 查询的 filter 语句中，这样它就自动的转成一个不评分的 filter 了。如果你需要通过多个不同的标准来过滤你的文档，bool 查询本身也可以被用做不评分的查询。简单地将它放置到 filter 语句中并在内部构建布尔逻辑：
```default
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "bool": { 
              "must": [
                  { "range": { "date": { "gte": "2014-01-01" }}},
                  { "range": { "price": { "lte": 29.99 }}}
              ],
              "must_not": [
                  { "term": { "category": "ebooks" }}
              ]
          }
        }
    }
}
```
通过混合布尔查询，我们可以在我们的查询请求中灵活地编写 scoring 和 filtering 查询逻辑。对于constant_score查询，尽管没有bool 查询使用这么频繁，constant_score 查询也是你工具箱里有用的查询工具。它将一个不变的常量评分应用于所有匹配的文档。它被经常用于你只需要执行一个 filter 而没有其它查询（例如，评分查询）的情况下。可以使用它来取代只有 filter 语句的 bool 查询。在性能上是完全相同的，但对于提高查询简洁性和清晰度有很大帮助。
```default
{
   "constant_score":   {
        "filter": {
            "term": { "category": "ebooks" } 
        }
    }
}
```
<br>

#### 对搜索结果排序

为了按照相关性来排序，需要将相关性表示为一个数值。在 Elasticsearch 中， 相关性得分 由一个浮点数进行表示，并在搜索结果中通过 _score 参数返回，默认排序是 _score 降序。有时，相关性评分对你来说并没有意义。例如，下面的查询返回所有 user_id 字段包含 1 的结果：

```default
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : {
                "term" : {
                    "user_id" : 1
                }
            }
        }
    }
}
```
这里没有一个有意义的分数：因为我们使用的是 filter （过滤），这表明我们只希望获取匹配 user_id: 1 的文档，并没有试图确定这些文档的相关性。 实际上文档将按照随机顺序返回，并且每个文档都会评为零分。你可以使用 constant_score 查询进行替代，这将让所有文档应用一个恒定分数（默认为 1 ）。它将执行与前述查询相同的查询，并且所有的文档将像之前一样随机返回，这些文档只是有了一个分数而不是零分：
```default
GET /_search
{
   "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "user_id" : 1
                }
            }
        }
    }
}
```
在下面案例中，通过时间来对 tweets 进行排序是有意义的，最新的 tweets 排在最前。 我们可以使用 sort 参数进行实现，返回结果中我们发现，第一：_score的值为null, 计算 _score 的花销巨大，通常仅用于排序，我们并不根据相关性排序，所以记录_score是没有意义的，如果无论如何你都要计算 _score，你可以将 track_scores 参数设置为 true；第二，返回的字段中包含date排序字段，它以时间戳的形式进行返回。
```default
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : { "term" : { "user_id" : 1 }}
        }
    },
    "sort": { "date": { "order": "desc" }}
}
```
<br>

#### 多级排序

假定我们想要结合使用 date 和 _score 进行查询，并且匹配的结果首先按照日期排序，然后按照相关性排序。排序条件的顺序是很重要的。结果首先按第一个条件排序，仅当结果集的第一个 sort 值完全相同时才会按照第二个条件进行排序，以此类推。多级排序并不一定包含 _score 。你可以根据一些不同的字段进行排序， 如地理距离或是脚本计算的特定值。

```default
GET /_search
{
    "query" : {
        "bool" : {
            "must":   { "match": { "tweet": "manage text search" }},
            "filter" : { "term" : { "user_id" : 2 }}
        }
    },
    "sort": [
        { "date":   { "order": "desc" }},
        { "_score": { "order": "desc" }}
    ]
}
```
<br>

#### 字段多值的排序

一种情形是字段有多个值的排序， 需要记住这些值并没有固有的顺序；一个多值的字段仅仅是多个值的包装，这时应该选择哪个进行排序呢？对于数字或日期，你可以将多值字段减为单值，这可以通过使用 min 、 max、avg 或是 sum 排序模式。例如你可以按照每个 date 字段中的最早日期进行排序，通过以下方法：
```default
GET /_search
{
    "query" : {
        "bool" : {
            "must":   { "match": { "tweet": "manage text search" }},
            "filter" : { "term" : { "user_id" : 2 }}
        }
    },
    "sort": {
    	"dates": {
    		"order": "asc",
    		"mode": "min"
    	}
    }
}
```
<br>

#### 字符串的排序

为了以字符串字段进行排序，这个字段应仅包含一项：整个 not_analyzed 字符串（即字段为keyword类型）。但是我们仍需要analyzed 字段，这样才能以全文进行查询。一个简单的方法是用两种方式对同一个字符串进行索引，这将在文档中包括两个字段：analyzed 用于搜索，not_analyzed 用于排序。但是保存相同的字符串两次在 _source 字段是浪费空间的。 我们真正想要做的是传递一个 单字段 但是却用两种方式索引它。所有的 _core_field 类型 (text, keyword、numbers, Booleans, dates) 接收一个 fields 参数，该参数允许你转化一个简单的映射如：
```default
//转化
"tweet": {
   "type":     "text",
    "analyzer": "english"
}

//为一个多字段映射如：
"tweet": { 
    "type":     "text",
    "analyzer": "english",
    "fields": {
        "raw": { 
            "type":  "text",
            "index": "not_analyzed"
        }
    }
}
```
tweet 主字段与之前的一样: 是一个 analyzed 全文字段。新的 tweet.raw 子字段是 not_analyzed。现在，至少只要我们重新索引了我们的数据，使用 tweet 字段用于搜索，tweet.raw 字段用于排序。现在，至少只要我们重新索引了我们的数据，使用 tweet 字段用于搜索，tweet.raw 字段用于排序。注意：以全文 analyzed 字段排序会消耗大量的内存。
```default
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    },
    "sort": "tweet.raw"
}
```
<br>

#### 检查文档在检查的时刻是否存在

```default
//返回200状态码表示文档存在，404状态码表示不存在
HEAD /{index}/{type}/{id}						 
```
<br>


### 更新文档

#### 更新整个文档

```default
//返回的_version表示更新后文档的版本，created表示文档是否是新创建的
//在内部，Elasticsearch 已将旧文档标记为已删除，并增加一个全新的文档。 
//尽管你不能再对旧版本的文档进行访问，但它并不会立即消失。    
//当继续索引更多的数据，Elasticsearch 会在后台清理这些已删除文档。
PUT /{index}/{type}/{id}						 
{
  "field": "value",   
  ... 	
}

//更新整个文档，如果当前待更新的版本号不正确与当前数据版本号不同则返回409冲突状态码，常用于乐观锁并发控制
PUT /{index}/{type}/{id}?version={_version}   	 
{
	"field": "value",
	...
}

//更新整个文档，如果当前待更新的版本号比当前版本号小则返回409冲突状态码，常用于同步RDBS数据
PUT /{index}/{type}/{id}?version={_version}&version_type=external  
{
	"field": "value",
	...
}
```
<br>

#### 更新部分文档
在更新整个文档 , 我们已经介绍过 更新一个文档的方法是检索并修改它，然后重新索引整个文档，这的确如此。然而，使用 update API 我们还可以部分更新文档，例如在某个请求时对计数器进行累加。我们也介绍过文档是不可变的：他们不能被修改，只能被替换。update API 必须遵循同样的规则。从外部来看，我们在一个文档的某个位置进行部分更新。然而在内部， update API 简单使用与之前描述相同的 检索-修改-重建索引 的处理过程。 区别在于这个过程发生在分片内部，这样就避免了多次请求的网络开销。通过减少检索和重建索引步骤之间的时间，我们也减少了其他进程的变更带来冲突的可能性。update 请求最简单的一种形式是接收文档的一部分作为 doc 的参数，它只是与现有的文档进行合并。对象被合并到一起，覆盖现有的字段，增加新的字段。 update API 还接受routing、replication、consistency 和 timeout 等参数。以下是部分更新一个文档的步骤：

1. 客户端向Node1发送更新请求;
2. Node1将请求转发到主分片所在的 Node3;
3. Node3从主分片检索文档，修改 _source 字段中的 JSON ，并且尝试重新索引主分片的文档。如果文档已经被另一个进程修改，它会重试步骤3，超过 retry_on_conflict 次后放弃;
4. 如果Node3成功地更新文档，它将新版本的文档并行转发到Node1和Node2上的副本分片，重新建立索引。一旦所有副本分片都返回成功，Node3向协调节点也返回成功，协调节点向客户端返回成功。注意当主分片把更改转发到副本分片时， 它不会转发更新请求。相反，它转发完整文档的新版本。请记住，这些更改将会异步转发到副本分片，并且不能保证它们以发送它们相同的顺序到达。

```default
//文档的部分数据更新之后，数据版本_version也会增加1，如果待更新的文档不存在则返回404
POST /{index}/{type}/{id}/_update				 
{
   "doc" : {
      "field1" : "value1",
      ...
   }
}

//部分更新文档，若更新不成功将会试图重试3次
POST /megacorp/employee/3/_update?retry_on_conflict=3  
{
   "doc" : {
      "field1" : "value1",
      ...
   }
}
```
<br>

#### 使用脚本部分更新文档

对于那些 API 不能满足需求的情况，Elasticsearch 允许你使用脚本编写自定义的逻辑。 许多API都支持脚本的使用，包括搜索、排序、聚合和文档更新。 脚本可以作为请求的一部分被传递，从特殊的 .scripts 索引中检索，或者从磁盘加载脚本。默认的脚本语言 是 Groovy，一种快速表达的脚本语言，在语法上与 JavaScript 类似。 它在 Elasticsearch V1.3.0 版本首次引入并运行在 沙盒 中，然而 Groovy 脚本引擎存在漏洞， 允许攻击者通过构建 Groovy 脚本，在 Elasticsearch Java VM 运行时脱离沙盒并执行 shell 命令。因此，在版本 v1.3.8 、 1.4.3 和 V1.5.0 及更高的版本中，它已经被默认禁用。 此外，您可以通过设置集群中的所有节点的 config/elasticsearch.yml 文件来禁用动态 Groovy 脚本: <br>
```yaml
script.groovy.sandbox.enabled: false 
```
这将关闭 Groovy 沙盒，从而防止动态 Groovy 脚本作为请求的一部分被接受， 或者从特殊的 .scripts 索引中被检索。当然，你仍然可以使用存储在每个节点的 config/scripts/ 目录下的 Groovy 脚本。
```default
//增加博客文章中views的数量
POST /website/blog/1/_update 			
{
   "script" : "ctx._source.views+=1"
}

//增加博客文章中views的数量，如果views字段不存在先创建它
POST /website/pageviews/1/_update	
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 1
   }
}

//给tags数组添加一个新的标签，这里我们为了提高代码的可重用性，指定了新的标签作为参数，而不是硬编码到脚本内部
POST /website/blog/1/_update
{
   "script" : "ctx._source.tags+=new_tag",
   "params" : {
      "new_tag" : "search"
   }
}

//通过设置ctx.op为delete来删除基于其内容的文档
POST /website/blog/1/_update			
{
   "script" : "ctx.op = ctx._source.views == count ? 'delete' : 'none'",
    "params" : {
        "count": 1
    }
}
```
<br>

#### 并发情况下的更新操作

```default
方法一：悲观锁，安全性高，但效率低

方法二：乐观锁，保证数据安全的情况下，兼顾效率，推荐使用。操作如下：

//更新前先获取带有版本号的数据
GET /{index}/{type}/{id}

//更新时指定版本号，版本号与当前GET到的版本号不相等则更新失败（返回409状态码）                     
PUT /{index}/{type}/{id}?version={_version}      

//更新时自定版本号，版本号与当前GET到的版本号小则更新失败（返回409状态码）
//这种版本控制方式通常用于外部RDBS中已经有一个版本字段，ES的版本同步RDBS中的版本
PUT /{index}/{type}/{id}?version={_version}&version_type=external 																			  
```
<br><br>

## ES分布式特性

### 概述

Elasticsearch 尽可能地屏蔽了分布式系统的复杂性。这里列举了一些在后台自动执行的操作:

* 分配文档到不同的容器 或 分片中，文档可以储存在一个或多个节点中
* 按集群节点来均衡分配这些分片，从而对索引和搜索过程进行负载均衡
* 复制每个分片以支持数据冗余，从而防止硬件故障导致的数据丢失
* 将集群中任一节点的请求路由到存有相关数据的节点
* 集群扩容时无缝整合新节点，重新分配分片以便从离群节点恢复

> 参考
> [集群内的原理](https://www.elastic.co/guide/cn/elasticsearch/guide/current/distributed-cluster.html)
> [应对文档存储(分布式文档存储)](https://www.elastic.co/guide/cn/elasticsearch/guide/current/distributed-docs.html) 
> [执行分布式搜索(执行分布式检索)](https://www.elastic.co/guide/cn/elasticsearch/guide/current/distributed-search.html) 
> [分区（shard）及其工作原理(分片内部原理)](https://www.elastic.co/guide/cn/elasticsearch/guide/current/inside-a-shard.html)

<br>

### 集群内的原理

#### 概念简介

**cluster** <br>
一个或者多个拥有相同 cluster.name 配置的节点组成，它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。<br>

**node** <br>
一个运行中的 Elasticsearch 实例称为一个节点。<br>

**shard** <br>
一个 分片 是一个底层的工作单元 ，它仅保存了全部数据中的一部分。在分片内部机制中，我们将详细介绍分片是如何工作的，而现在我们只需知道一个分片是一个 Lucene 的实例，以及它本身就是一个完整的搜索引擎。 我们的文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互。Elasticsearch 是利用分片将数据分发到集群内各处的。分片是数据的容器，文档保存在分片内，分片又被分配到集群内的各个节点里。 当你的集群规模扩大或者缩小时， Elasticsearch 会自动的在各节点中迁移分片，使得数据仍然均匀分布在集群里。<br>
主分片和副分片：一个分片可以是 主 分片或者 副本 分片。 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。技术上来说，一个主分片最大能够存储 Integer.MAX_VALUE - 128 个文档，但是实际最大值还需要参考你的使用场景：包括你使用的硬件，文档的大小和复杂程度，索引和查询文档的方式以及你期望的响应时长。一个副本分片只是一个主分片的拷贝。 副本分片作为硬件故障时保护数据不丢失的冗余备份，并为搜索和返回文档等读操作提供服务。在索引建立的时候就已经确定了主分片数，但是副本分片数可以随时修改。索引在默认情况下会被分配5个主分片。<br>

**master** <br>
当一个节点被选举成为 主 节点时， 它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。 而主节点并不需要涉及到文档级别的变更和搜索等操作，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。 任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。作为用户，我们可以将请求发送到 集群中的任何节点 ，包括主节点。 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。 无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。 Elasticsearch 对这一切的管理都是透明的。<br>

#### 集群的健康

```default
GET /_cluster/health
GET /_cluster/health?level=[indices|shards]

//在一个不包含任何索引的空集群中，它将会有一个类似于如下所示的返回内容：
//status：值为green、yellow、red 中的其中一个。green表示所有的主分片和副本分片都正常运行；
//yellow表示所有的主分片都正常运行，但不是所有的副本分片都正常运行；red表示有主分片没能正常运行
{
   "cluster_name":          "elasticsearch",
   "status":                "green", 
   "timed_out":             false,
   "number_of_nodes":       1,
   "number_of_data_nodes":  1,
   "active_primary_shards": 0,
   "active_shards":         0,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```
<br>

#### 创建索引时设置分片和副本个数

```default
PUT /blogs
{
	"settings": {
		"number_of_shards": 3,
		"number_of_replicas": 1
	}
}
```
<br>

#### 索引数据的分布查看 和 人为干预

现在我们的ES是单节点的，blogs主分片的索引的数据分布情况如下：
```default
cluster
	|--node1-master
	|  p0  p1  p2
```
查看集群健康状况如下：
```default
{
	"cluster_name": "elasticsearch",
	 "status": "yellow", 
	 ...
	 "unassigned_shards": 3, 
	 ...
}
```
可以看到【"status": "yellow", "unassigned_shards": 3】，即集群的状态为yellow，没有被分配到任何节点的副本数为3个。集群的健康状况为 yellow 则表示全部 主 分片都正常运行（集群可以正常服务所有请求），但是 副本 分片没有全部处在正常状态。实际上，所有3个副本分片都是 unassigned —— 它们都没有被分配到任何节点。在同一个节点上既保存原始数据又保存副本是没有意义的，因为一旦失去了那个节点，我们也将丢失该节点上的所有副本数据。当前我们的集群是正常运行的，但是在硬件故障时有丢失数据的风险。<br>

**添加故障转移，启动第二个节点：** <br>
为了测试第二个节点启动后的情况，你可以在同一个目录内，完全依照启动第一个节点的方式来启动一个新节点（参考安装并运行 Elasticsearch）。多个节点可以共享同一个目录。当你在同一台机器上启动了第二个节点时，只要它和第一个节点有同样的 cluster.name 配置，它就会自动发现集群并加入到其中。 但是在不同机器上启动节点的时候，为了加入到同一集群，你需要配置一个可连接到的单播主机列表。 详细信息请查看最好使用单播代替组播拥有两个节点的集群——所有主分片和副本分片都已被分配：
```default
cluster
	|--node1-master
	|  p0  p1  p2
	|
	|--node2-data
	|  r0 r1 r2
	|
```
当第二个节点加入到集群后，3个 副本分片 将会分配到这个节点上——每个主分片对应一个副本分片。 这意味着当集群内任何一个节点出现问题时，我们的数据都完好无损。所有新近被索引的文档都将会保存在主分片上，然后被并行的复制到对应的副本分片上。这就保证了我们既可以从主分片又可以从副本分片上获得文档。查看集群健康状况如下：
```default
{
  "cluster_name": "elasticsearch",
  "status": "green", 
  ...
  "unassigned_shards": 0, 
  ...
}
```
集群现在展示的状态为green，这表示所有6个分片（包括3个主分片和3个副本分片）都在正常运行。我们的集群现在不仅仅是正常运行的，并且还处于 始终可用 的状态。<br>

**再次水平扩容，使集群中有三个节点：** <br>
增加集群的节点数为3，集群的数据分布如下：<br>

```default
cluster
	|--node1-master
	|  p1   p2
	|
	|--node2-data
	|  r0  r1
	|
	|--node3-data
	|  p0  r2
	|
```
Node 1 和 Node 2 上各有一个分片被迁移到了新的 Node 3 节点，现在每个节点上都拥有2个分片，而不是之前的3个。 这表示每个节点的硬件资源（CPU, RAM, I/O）将被更少的分片所共享，每个分片的性能将会得到提升。分片是一个功能完整的搜索引擎，它拥有使用一个节点上的所有资源的能力。我们这个拥有6个分片（3个主分片和3个副本分片）的索引可以最大扩容到6个节点，每个节点上存在一个分片，并且每个分片拥有所在节点的全部资源。<br>

**假如继续扩容超过六个节点怎么办呢？** <br>
主分片的数目在索引创建时 就已经确定了下来。实际上，这个数目定义了这个索引能够 存储 的最大数据量。（实际大小取决于你的数据、硬件和使用场景。） 但是，读操作——搜索和返回数据——可以同时被主分片 或 副本分片所处理，所以当你拥有越多的副本分片时，也将拥有越高的吞吐量。在运行中的集群上是可以动态调整副本分片数目的 ，我们可以按需伸缩集群。让我们把副本数从默认的 1 增加到 2 ：
```default
PUT /blogs/_settings
{
   "number_of_replicas" : 2
}
```
blogs 索引现在拥有9个分片：3个主分片和6个副本分片。 这意味着我们可以将集群扩容到9个节点，每个节点上一个分片。相比原来3个节点时，集群搜索性能可以提升 3 倍。集群的数据现在分布如下:
```default
cluster
	|--node1-master
	|  p1   p2  r0
		|
	|--node2-data
	|  r0  r1  r2
	|
	|--node3-data
	|  p0  r1  r2
	|
```
当然，如果只是在相同节点数目的集群上增加更多的副本分片并不能提高性能，因为每个分片从节点上获得的资源会变少。你需要增加更多的硬件资源来提升吞吐量。但是更多的副本分片数提高了数据冗余量：按照上面的节点配置，我们可以在失去2个节点的情况下不丢失任何数据。<br>


