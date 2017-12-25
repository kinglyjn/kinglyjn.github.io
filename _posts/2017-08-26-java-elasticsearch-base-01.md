---
layout: post
title:  "Elasticsearch 5.6.5 基础笔记(一) - 概念和安装"
desc: "Elasticsearch 5.6.5 基础笔记(一) - 概念和安装"
keywords: "Elasticsearch 5.6.5 基础笔记(一) - 概念和安装,elasticsearch,es,kinglyjn"
date: 2017-08-26
categories: [Java]
tags: [java]
icon: fa-coffee
---



## 概念

### Elasticsearch

* 分布式、可扩展、实时的搜索与数据分析引擎
* 建立在全文搜索引擎库 Apache Lucene 基础之上
* 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据
* 将所有的功能打包成一个单独的服务，这样你可以通过程序与它提供的简单的 RESTful API 进行通信
* 全文搜索/结构化数据的实时

<br>



### ES索引和Lucene索引比较

一个 Lucene 索引 我们在 Elasticsearch 称作 分片 。 一个 Elasticsearch 索引 是分片的集合。 当 Elasticsearch 在索引中搜索的时候， 他发送查询到每一个属于索引的分片(Lucene 索引)，然后像 执行分布式检索 提到的那样，合并每个分片的结果到一个全局的结果集。<br>



### 索引

n.代表一个索引库，相当于关系型数据库的schema概念。索引是保存相关数据的地方，实际上是指向一个或者多个物理分片的逻辑命名空间。索引名称必须小写，不能以下划线开头，不能包含逗号！<br>
v.存储数据到ES的行为。<br>



**创建一个索引** <br>
我们可以通过索引一个文档创建一个新的索引，这个索引采用的是默认的配置，新的字段通过动态映射的方式被添加到类型映射。现在我们需要对这个建立索引的过程做更多的控制：<br>
我们想要确保这个索引有数量适中的主分片，并且在我们索引任何数据 之前 ，分析器和映射已经被建立好。为了达到这个目的，我们需要手动创建索引，在请求体里面传入设置或类型映射，如下所示。如果你想禁止自动创建索引，你 可以通过在 config/elasticsearch.yml 的每个节点下添加下面的配置：action.auto_create_index: false
```json
PUT /my_index
{
    "settings": { ... any settings ... },
    "mappings": {
        "type_one": { ... any mappings ... },
        "type_two": { ... any mappings ... },
        ...
    }
}
```
<br>



**删除一个索引** <br>

```json
DELETE /my_index
DELETE /index_one,index_two
DELETE /index_*
```
对一些人来说，能够用单个命令来删除所有数据可能会导致可怕的后果。如果你想要避免意外的大量删除, 你可以在你的 elasticsearch.yml 做如下配置：action.destructive_requires_name: true。这个设置使删除只限于特定名称指向的数据, 而不允许通过指定 _all 或通配符来删除指定索引库。<br>



**索引的一些设置** <br>
Elasticsearch 提供了优化好的默认配置。 除非你理解这些配置的作用并且知道为什么要去修改，否则不要随意修改。下面是两个最重要的设置：<br>
```default
number_of_shards：  每个索引的主分片数，默认值是 5。这个配置在索引创建后不能修改。
number_of_replicas：每个主分片的副本数，默认值是 1 。对于活动的索引库，这个配置可以随时修改。
```

例如，我们可以创建只有 一个主分片，没有副本的小索引：<br>
```json
PUT /my_temp_index
{
    "settings": {
        "number_of_shards" : 1,
        "number_of_replicas" : 0
    }
}

//然后，我们可以用 update-index-settings API 动态修改副本数：
PUT /my_temp_index/_settings
{
    "number_of_replicas": 1
}
```
<br>



**重新索引你的数据**
尽管可以增加新的类型到索引中，或者增加新的字段到类型中，但是不能添加新的分析器或者对现有的字段做改动。 如果你那么做的话，结果就是那些已经被索引的数据就不正确， 搜索也不能正常工作。对现有数据的这类改变最简单的办法就是重新索引：用新的设置创建新的索引并把文档从旧的索引复制到新的索引。字段_source 的一个优点是在Elasticsearch中已经有整个文档，你不必从源数据中重建索引，而且那样通常比较慢。为了有效的重新索引所有在旧的索引中的文档，用 scroll 从旧的索引检索批量文档 ， 然后用 bulk API 把文档推送到新的索引中。从Elasticsearch v2.3.0开始，Reindex API 被引入。详见：[点击](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex)
```json
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "internal"
  }
}
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "external"
  }
}
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "op_type": "create"
  },
  "conflicts": "proceed"
}
POST _reindex
{
  "source": {
    "index": "twitter",
    "type": "tweet",
    "query": {
      "term": {
        "user": "kimchy"
      }
    }
  },
  "dest": {
    "index": "new_twitter"
  }
}
```
<br>



**索引别名和零停机** <br>

在前面提到的，重建索引的问题是必须更新应用中的索引名称。 索引别名就是用来解决这个问题的！索引别名就像一个快捷方式或软连接，可以指向一个或多个索引，也可以给任何一个需要索引名的API来使用。别名 带给我们极大的灵活性，允许我们做下面这些：
a.在运行的集群中可以无缝的从一个索引切换到另一个索引<br>
b.给多个索引分组 (例如，last_three_months)<br>
c.给索引的一个子集创建 视图在后面我们会讨论更多关于别名的使用。<br>
现在，我们将解释怎样使用别名在零停机下从旧索引切换到新索引。有两种方式管理别名： 
```default
_alias 用于单个操作， _aliases 用于执行多个原子级操作。
```
在这里，我们假设你的应用有一个叫 my_index 的索引。事实上， my_index 是一个指向当前真实索引的别名。真实索引包含一个版本号： my_index_v1，my_index_v2 等等。首先，创建索引 my_index_v1 ，然后用别名 my_index 指向它：<br>

```json
PUT /my_index_v1 
PUT /my_index_v1/_alias/my_index
```
你可以检测这个别名指向哪一个索引, 或哪些别名指向这个索引：
```json
GET /*/_alias/my_index
GET /my_index_v1/_alias/*
```
然后，我们决定修改索引中一个字段的映射。当然，我们不能修改现存的映射，所以我们必须重新索引数据。首先，我们用新映射创建索引my_index_v2：
```json
PUT /my_index_v2
{
    "mappings": {
        "my_type": {
            "properties": {
                "tags": {
                    "type":   "text",
                    "index":  "not_analyzed"
                }
            }
        }
    }
}
```
然后我们将数据从 my_index_v1 索引到 my_index_v2，一旦我们确定文档已经被正确地重索引了，我们就将别名指向新的索引。一个别名可以指向多个索引，所以我们在添加别名到新索引的同时必须从旧的索引中删除它。这个操作需要原子化，这意味着我们需要使用 _aliases 操作：
```json
POST /_aliases
{
    "actions": [
        { "remove": { "index": "my_index_v1", "alias": "my_index" }},
        { "add":    { "index": "my_index_v2", "alias": "my_index" }}
    ]
}
```
现在，你的应用已经在零停机的情况下从旧索引迁移到新索引了。使用别名而不是索引名，然后你就可以在任何时候重建索引。别名的开销很小，应该广泛使用。<br>



**Refresh API** <br>
在 Elasticsearch 中，写入和打开一个新段的轻量的过程叫做 refresh 。 默认情况下每个分片会每秒自动刷新一次。这就是为什么我们说 Elasticsearch 是近实时搜索: 文档的变化并不是立即对搜索可见，但会在一秒之内变为可见。这些行为可能会对新用户造成困惑: 他们索引了一个文档然后尝试搜索它，但却没有搜到。这个问题的解决办法是用 refresh API 执行一次手动刷新:<br>
```json
POST /_refresh 
POST /blogs/_refresh
```
尽管刷新是比提交轻量很多的操作，它还是会有性能开销。 当写测试的时候， 手动刷新很有用，但是不要在生产环境下每次索引一个文档都去手动刷新。 相反，你的应用需要意识到 Elasticsearch 的近实时的性质，并接受它的不足。并不是所有的情况都需要每秒刷新。可能你正在使用 Elasticsearch 索引大量的日志文件， 你可能想优化索引速度而不是近实时搜索， 可以通过设置 refresh_interval ， 降低每个索引的刷新频率：<br>
```json
PUT /my_logs
{
  "settings": {
    "refresh_interval": "30s" 
  }
}
```
refresh_interval 可以在既存索引上进行动态更新。 在生产环境中，当你正在建立一个大的新索引时，可以先关闭自动刷新，待开始使用该索引时，再把它们调回来。
refresh_interval 需要一个 持续时间 值， 例如 1s （1 秒） 或 2m （2 分钟）。 一个绝对值 1 表示的是 1毫秒，这无疑会使你的集群陷入瘫痪。
```json
PUT /my_logs/_settings
{ "refresh_interval": -1 } 

PUT /my_logs/_settings
{ "refresh_interval": "1s" } 
```
<br>



**Flush API** <br>
这个执行一个提交并且截断 translog 的行为在 Elasticsearch 被称作一次 flush 。 分片每30分钟被自动刷新（flush），或者在 translog 太大的时候也会刷新。请查看 translog 文档 来设置，它可以用来 控制这些阈值，flush API 可以 被用来执行一个手工的刷新（flush），你很少需要自己手动执行一个的 flush 操作；通常情况下，自动刷新就足够了。
```json
POST /blogs/_flush 
POST /_flush?wait_for_ongoing
```
<br>

### 类型
索引一个数据之前，需要确定将数据存储在哪里。
```default
Elasticsearch集群 --> 多个索引[index] --> 多个类型[type] --> 多个文档[document]
```
一个_type命名可以是大写或者小写，但是不能以下划线或者句号开头，不应该包含逗号，并且长度限制为256个字符！
<br>



### 文档
代表一条被索引的记录。_id是一个字符串， 当它和_index 以及 _type 组合就可以唯一确定 Elasticsearch 中的一个文档。 当你创建一个新的文档，要么提供自己的 _id ，要么让 Elasticsearch 帮你生成。
<br>



### 映射（Mapping）

**描述数据在每个字段内如何存储** <br>
注意：ES不允许同一个索引库中的两个不同的类型下具有同名但是类型不同的字段，因为同一个索引库所有类型下字段的映射是共享的！为索引设置映射示例：
```json
PUT /{index}/_mapping/{type}
{
  "properties": {
    "field1": {
      "type": "text",
      "analyzer": "my_analyzer"
    },
    "field2": {
      "type": "text"
    },
    ...
  }
}
```
<br>



**查看索引的映射** <br>
```json
GET /{index}/_mapping/{type}
```
<br>



**动态映射** <br>
当 Elasticsearch 遇到文档中以前 未遇到的字段，它用 dynamic mapping 来确定字段的数据类型并自动把新的字段添加到类型映射。 有时这是想要的行为有时又不希望这样。通常没有人知道以后会有什么新字段加到文档，但是又希望这些字段被自动的索引。也许你只想忽略它们。 如果Elasticsearch是作为重要的数据存储，可能就会期望遇到新字段就会抛出异常，这样能及时发现问题。幸运的是可以用 dynamic 配置来控制这种行为，可接受的选项如下：
```json
true：动态添加新的字段--缺省
false：忽略新的字段
strict：如果遇到新字段抛出异常
```
配置参数 dynamic 可以用在根 object 或任何 object 类型的字段上。你可以将 dynamic 的默认值设置为 strict , 而只在指定的内部对象中开启它, 例如：
```json
PUT /my_index
{
    "mappings": {
        "my_type": {
            "dynamic": "strict", 
            "properties": {
                "title":  { "type": "text"},
                "stash":  {
                    "type": "object",
                    "dynamic": true 
                }
            }
        }
    }
}
```
<br>



**自动检测日期映射** <br>
当 Elasticsearch 遇到一个新的字符串字段时，它会检测这个字段是否包含一个可识别的日期，比如 2014-01-01。如果它像日期，这个字段就会被作为 date 类型添加。否则，它会被作为 text 类型添加。有些时候这个行为可能导致一些问题。想象下，你有如下这样的一个文档:
```json
{ "note": "2014-01-01" }
```
假设这是第一次识别 note 字段，它会被添加为 date 字段。但是如果下一个文档像这样：
```json
{ "note": "Logged out" }
```
这显然不是一个日期，但为时已晚。这个字段已经是一个日期类型，这个 不合法的日期 将会造成一个异常。日期检测可以通过在根对象上设置 date_detection 为 false 来关闭，使用下面设置 字符串将始终作为 string 类型。如果你需要一个 date 字段，你必须手动添加。Elasticsearch 判断字符串为日期的规则可以通过 dynamic_date_formats setting 来设置。<br>
```json
PUT /my_index
{
    "mappings": {
        "my_type": {
            "date_detection": false
        }
    }
}
PUT my_index
{
  "mappings": {
    "my_type": {
      "dynamic_date_formats": ["yyyy-MM-dd HH:mm:ss||yyyy/MM/dd HH:mm:ss"]
    }
  }
}
```
<br>



**动态模板映射** <br>
使用 dynamic_templates，你可以完全控制 新检测生成字段的映射。你甚至可以通过字段名称或数据类型来应用不同的映射。每个模板都有一个名称，你可以用来描述这个模板的用途，一个 mapping 来指定映射应该怎样使用，以及至少一个参数 (如 match) 来定义这个模板适用于哪个字段。模板按照顺序来检测；第一个匹配的模板会被启用。例如，我们给 string 类型字段定义两个模板：
```json
es ：以 _es 结尾的字段名需要使用 spanish 分词器。
en ：所有其他字段使用 english 分词器
```
我们将 es 模板放在第一位，因为它比匹配所有字符串字段的 en 模板更特殊。match_mapping_type 允许你应用模板到特定类型的字段上，就像有标准动态映射规则检测的一样，例如 text 或 long；match 参数只匹配字段名称； path_match 参数匹配字段在对象上的完整路径，例如 address.*.name。
```json
PUT /my_index
{
    "mappings": {
        "my_type": {
            "dynamic_templates": [
                { "es": {
                      "match":              "*_es", 
                      "match_mapping_type": "text",
                      "mapping": {
                          "type":           "text",
                          "analyzer":       "spanish"
                      }
                }},
                { "en": {
                      "match":              "*", 
                      "match_mapping_type": "text",
                      "mapping": {
                          "type":           "text",
                          "analyzer":       "english"
                      }
                }}
            ]
}}}
```


**缺省映射** <br>
通常，一个索引中的所有类型共享相同的字段和设置。 _default_ 映射更加方便地指定通用设置，而不是每次创建新类型时都要重复设置。 _default_  映射是新类型的模板。在设置 _default_ 映射之后创建的所有类型都将应用这些缺省的设置，除非类型在自己的映射中明确覆盖这些设置。例如，我们可以使用 _default_ 映射为所有的类型禁用 _all 字段， 而只在 blog 类型启用。_default_映射也是一个指定索引动态模板的好方法。

```json
PUT /my_index
{
    "mappings": {
        "_default_": {
            "_all": { "enabled":  false }
        },
        "blog": {
            "_all": { "enabled":  true  }
        }
    }
}
```
<br>



### 映射数据类型

```default
精确匹配类型:
精确匹配的意思是 “要么匹配，要么不匹配”
binary、bate、integer、float、double、boolan、date、keyword、geo_point、geo_shape、nasted、object


全文匹配类型：
text
该类型域会被认为包含全文，就是说，它们的值在索引前，会通过一个分析器，针对于这个域的查询在搜索前也会经过一个分析器。注意全文匹配的意思是 当前文档与给定查询的相关性如何。text域映射的最重要的属性是 analyzer，默认的分析器是标准分析器（standard）。
我们很少对全文类型的域做精确匹配。相反，我们希望在文本类型的域中搜索。不仅如此，我们还希望搜索能够理解我们的意图，例如：
搜索 UK ，会返回包含 United Kindom 的文档。
搜索 jump ，会匹配 jumped ， jumps ， jumping ，甚至是 leap 。
搜索 johnny walker 会匹配 Johnnie Walker ， johnnie depp 应该匹配 Johnny Depp 。
fox news hunting 应该返回福克斯新闻（ Foxs News ）中关于狩猎的故事，同时， fox hunting news 应该返回关于猎狐的故事。	


包含内部数组的字段：
对于数组，没有特殊的映射需求。任何域都可以包含0、1或者多个值，就像全文域分析得到多个词条。这暗示数组中所有的值必须是相同数据类型的 。你不能将日期和字符串混在一起。如果你通过索引数组来创建新的域，ES会用数组中第一个值的数据类型作为这个域的类型。
假设我们有个followers数组：
{
    "followers": [
        { "age": 35, "name": "Mary White"},
        { "age": 26, "name": "Alex Jones"},
        { "age": 19, "name": "Lisa Smith"}
    ]
}
这个文档会像我们之前描述的那样被扁平化处理，结果如下所示：
{
  "followers.age":    [19, 26, 35],
    "followers.name":   [alex, jones, lisa, smith, mary, white]
}
{age: 35} 和 {name: Mary White} 之间的相关性已经丢失了，因为每个多值域只是一包无序的值，而不是有序数组。这足以让我们问，“有一个26岁的追随者？” 但是我们不能得到一个准确的答案：“是否有一个26岁 名字叫 Alex Jones 的追随者？” 下面的内部对象被称为 nested 对象，可以回答上面的查询。


包含内部对象的字段：
映射含有内部对象的字段示例：
{
  "myindex": {
    "myrootobj": { 
      "properties": {
        "tweet":            { "type": "text" },
        "user": { 
          "type":             "object",
          "properties": {
            "id":           { "type": "text" },
            "gender":       { "type": "text" },
            "age":          { "type": "long"   },
            "name":   { 
              "type":         "object",
              "properties": {
                "full":     { "type": "text" },
                "first":    { "type": "text" },
                "last":     { "type": "text" }
              }
            }
          }
        }
      }
    }
  }
}


包含内部对象的字段：
Lucene 不理解内部对象。 Lucene 文档是由一组键值对列表组成的。为了能让ES有效地索引内部类，它把我们的文档转化成这样：
{
   "tweet":            [elasticsearch, flexible, very],
    "user.id":          [@johnsmith],
    "user.gender":      [male],
    "user.age":         [26],
    "user.name.full":   [john, smith],
    "user.name.first":  [john],
    "user.name.last":   [smith]
}
内部域 可以通过名称引用（例如， first）。为了区分同名的两个域，我们可以使用全路径 
例如， user.name.first  或 type名加路径 tweet.user.name.first 


包含空域的字段：
下面三种域被认为是空的，它们将不会被索引：
"null_value":               null,
"empty_array":              [],
"array_with_null_value":    [ null ]
```
<br>



### 倒排索引

使用一种称为倒排索引的结构，它适用于快速的全文搜索。一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。详见：[点击](https://www.elastic.co/guide/cn/elasticsearch/guide/current/inverted-index.html)
```default
倒排序索引结构举例：
Term      Doc_1   Doc_2   Doc_3  
-------------------------------  
brown   |   X   |   X   |  
dog     |   X   |       |   X  
dogs    |       |   X   |   X  
fox     |   X   |       |   X  
foxes   |       |   X   |  
in      |       |   X   |  
jumped  |   X   |       |   X  
lazy    |   X   |   X   |  
leap    |       |   X   |  
over    |   X   |   X   |   X  
quick   |   X   |   X   |   X  
summer  |       |   X   |  
the     |   X   |       |   X  
-------------------------------
```
倒排索引被写入磁盘后是 不可改变 的:它永远不会修改。 不变性有重要的价值：
1. 不需要锁。如果你从来不更新索引，你就不需要担心多进程同时修改数据的问题。
2. 一旦索引被读入内核的文件系统缓存，便会留在哪里，由于其不变性。只要文件系统缓存中还有足够的空间，那么大部分读请求会直接请求内存，而不会命中磁盘。这提供了很大的性能提升。
3. 其它缓存(像filter缓存)，在索引的生命周期内始终有效。它们不需要在每次数据改变时被重建，因为数据不会变化。
4. 写入单个大的倒排索引允许数据被压缩，减少磁盘 I/O 和 需要被缓存到内存的索引的使用量。
   当然，一个不变的索引也有不好的地方。主要事实是它是不可变的! 你不能修改它。如果你需要让一个新的文档 可被搜索，你需要重建整个索引。这要么对一个索引所能包含的数据量造成了很大的限制，要么对索引可被更新的频率造成了很大的限制。

<br>



### doc_values

前面讲到的doc values给我们几个印象：快速访问、高效、基于硬盘。现在我们来看看doc values到底是如何工作的？doc values是在“索引期“随着倒排索引一起生成的，也就是说doc values是基于每个索引段生成且是不可改变的（immutable），和倒排索引一样，doc values也会被序列化到磁盘上，这使得它具有了高效性和可扩展性。通过序列化一个数据结构到磁盘上，我们可以依赖操作系统的 file system cache 替代JVM的堆内存，当我们的“工作集”小于OS可用内存时，操作系统会自然的加载这些doc values到内存。这时doc values的性能和在JVM 堆内存中表现是一样的。但是当工作集大于操作系统可用内存时，操作系统将会按需加载doc values，这种情况下的访问速度会明显的慢于全量加载doc values的时候。但这种操作使得我们的服务器内存利用率远超过服务器最大内存限制。试想一下，如果全量加载到doc values到内存中势必会造成 ES OutOfMemery。注意，由于doc values不受JVM堆内存管理，所以我们可以把ES对内存设置得小一点，把更多的内存留给操作系统来换出（doc-values），同时这也可以使JVM的GC工作在更小的堆内存上，更快更高效的执行GC。通常，我们配置JVM的堆内存基本和操作系统内存各占一半（50%），由于引进了doc values所以我们可以考虑把JVM的堆内存设置得小一些，比如我们可以在一个64G的服务器上设置JVM堆内存为4 - 16GB比设置堆内存为32G更加高效。
```default
Doc      Terms  
---------------------------------------------------------------
Doc_1 | brown, dog, fox, jumped, lazy, over, quick, the  
Doc_2 | brown, dogs, foxes, in, lazy, leap, over, quick, summer  
Doc_3 | dog, dogs, fox, jumped, over, quick, the  
---------------------------------------------------------------
```
<br>



### 相关性

默认情况下，返回结果是按相关性倒序排列的。但是什么是相关性？相关性如何计算？每个文档都有相关性评分，用一个正浮点数字段 _score 来表示。_score 的评分越高，相关性越高。查询语句会为每个文档生成一个 _score 字段。评分的计算方式取决于查询类型 不同的查询语句用于不同的目的：fuzzy 查询会计算与关键词的拼写相似程度，terms 查询会计算 找到的内容与关键词组成部分匹配的百分比，但是通常我们说的relevance是我们用来计算全文本字段的值相对于全文本检索词相似程度的算法。<br>
Elasticsearch的相似度算法 被定义为 检索词频率/反向文档频率【TF/IDF】，包括以下内容：<br>

```default
这里我们以检索词 `honeymoon`，字段 `tweet` 为例：
检索词频率：检索词 `honeymoon` 在这个文档的 `tweet` 字段中的出现次数。
反向文档频率：检索词 `honeymoon` 在索引上所有文档的 `tweet` 字段中出现的次数。
字段长度准则：在这个文档中， `tweet` 字段内容的长度，内容越长，_score值越小。
```

单个查询可以联合使用 TF/IDF 和其他方式，比如短语查询中检索词的距离或模糊查询里的检索词相似度。<br>
相关性并不只是全文本检索的专利。也适用于 yes|no 的子句，匹配的子句越多，相关性评分越高。<br>
如果多条查询子句被合并为一条复合查询语句 ，比如 bool 查询，则每个查询子句计算得出的评分会被合并到总的相关性评分中。<br>
我们有一️整章着眼于相关性计算和如何让其配合你的需求控制相关度，详见：[点击](https://www.elastic.co/guide/cn/elasticsearch/guide/current/controlling-relevance.html)
我们可以使用explain参数 在开发中了解_score得来的依据，例如：
```json
GET /_search?explain 
{
   "query"   : { "match" : { "tweet" : "honeymoon" }}
}
```
返回的结果中包含_explanation字段，包含了 检索词频率、反向文档频率以及字段长度准则等信息。<br>



### 分析器（analyzer）
ES提供了开箱即用的字符过滤器、分词器和token 过滤器。 这些可以组合起来形成自定义的分析器以用于不同的目的。<br>
```default
Set the shape to semi-transparent by calling set_trans(5)
```
常用的内置分析器包括：
标准分析器（standard）：它根据 Unicode 联盟 定义的 单词边界 划分文本。删除绝大部分标点。最后，将词条小写。它会产生：
```default
set, the, shape, to, semi, transparent, by, calling, set_trans, 5
```
简单分析器（simple）：简单分析器在任何不是字母的地方分隔文本，将词条小写。它会产生：
```default
set, the, shape, to, semi, transparent, by, calling, set, trans
```
空格分析器（whitespace）：空格分析器在空格的地方划分文本。它会产生：
```default
Set, the, shape, to, semi-transparent, by, calling, set_trans(5)
```
语言分析器（xxx）：特定语言分析器可用于 很多语言。它们可以考虑指定语言的特点。例如， 英语 分析器附带了一组英语无用词（常用单词，例如 and 或者 the，它们对相关性没有多少影响），它们会被删除。 由于理解英语语法的规则，这个分词器可以提取英语单词的词干。英语 分词器会产生下面的词条：
```default
set, shape, semi, transpar, call, set_tran, 5
```
`分析器`实际上是将字符`过滤器`、`分词器`、`词条过滤器` 这三个功能封装到了一个包里：<br>
```default
字符过滤器[char_filter]：
首先，字符串按顺序通过每个 字符过滤器 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 & 转化成 and

分析器[tokenizer]：
其次，字符串被 分词器 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条（token）

词条过滤器[filter]：
最后，词条按顺序通过每个 token 过滤器 。这个过程可能会改变词条（例如，小写化 Quick ），删除词条（例如，像a、and、the等无用词），或者增加词条（例如，像jump和leap这种同义词）
```
为索引库创建一个自定义的分析器，这里我们创建的分析器不是全局的，它仅仅存在于我们定义的my_index索引中：
```json
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type": "mapping",
                    "mappings": [ "&=> and " ]
                }
            },
            "filter": {
                "my_stopwords": {
                    "type": "stop",
                    "stopwords": [ "the", "a" ]
                }
            },
            "analyzer": {
                "my_analyzer": {
                    "type": "custom",
                    "char_filter": [  "html_strip", "&_to_and" ],
                    "tokenizer": "standard",
                    "filter": [ "lowercase", "my_stopwords" ]
                }
            }
        }
    }
}
```
使用 analyze API 来 测试这个新的分析器：
```json
GET /my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "The quick & brown fox"
}

//下面的缩略结果展示出我们的分析器正在正确地运行：
{
  "tokens": [
    { "token": "quick", "start_offset": 4, "end_offset": 9, "type": "<ALPHANUM>", "position": 1 },
    { "token": "and", "start_offset": 10, "end_offset": 11, "type": "<ALPHANUM>", "position": 2 },
    { "token": "brown", "start_offset": 12, "end_offset": 17, "type": "<ALPHANUM>", "position": 3 },
    { "token": "fox", "start_offset": 18, "end_offset": 21, "type": "<ALPHANUM>", "position": 4 }
  ]
}
```
设置某个字段所使用的分析器：这个分析器现在是没有多大用处的，除非我们告诉 Elasticsearch在哪里用上它。我们可以像下面这样把这个分析器应用在一个 string 字段上：
```json
PUT /my_index/_mapping/my_type
{
    "properties": {
        "title": {
            "type": "text",
            "analyzer": "my_analyzer"
        }
    }
}
```
分析器是怎样起作用的？当我们 索引 一个文档，它的全文域被分析成词条以用来创建倒排索引。但是，当我们在全文域搜索的时候，我们需要将查询字符串通过相同的分析过程 ，以保证我们搜索的词条格式与索引中的词条格式一致。测试某个text字段的分析器是否起作用：
```json
GET /my_index/_analyze 
{
  "field": "title",
  "text": "The quick & brown fox"
}
```
<br>



### 查询与过滤

ES 使用的查询语言（DSL） 拥有一套查询组件，这些组件可以以无限组合的方式进行搭配。这套组件可以在以下两种情况下使用：
```default
过滤情况（filtering context）和查询情况（query context）。当使用于 过滤情况 时，查询被设置成一个“不评分”或者“过滤”查询。即，这个查询只是简单的问一个问题：“这篇文档是否匹配？”。回答也是非常的简单，yes或者no，二者必居其一。

> created 时间是否在 2013 与 2014 这个区间？
> status 字段是否包含 published 这个单词？
> lat_lon 字段表示的位置是否在指定点的 10km 范围内？

当使用于 查询情况 时，查询就变成了一个“评分”的查询。和不评分的查询类似，也要去判断这个文档是否匹配，同时它还需要判断这个文档匹配的有 _多好_（匹配程度如何）。 此查询的典型用法是用于查找以下文档：

> 查找与 full text search 这个词语最佳匹配的文档
> 包含 run 这个词，也能匹配 runs 、 running 、 jog 或者 sprint
> 包含 quick 、 brown 和 fox 这几个词 — 词之间离的越近，文档相关性越高
> 标有 lucene 、 search 或者 java 标签 — 标签越多，相关性越高 

一个评分查询计算每一个文档与此查询的 _相关程度_，同时将这个相关程度分配给表示相关性的字段 _score，并且按照相关性对匹配到的文档进行排序。这种相关性的概念是非常适合全文搜索的情况，因为全文搜索几乎没有完全 “正确” 的答案。
```
自ES问世以来，查询与过滤（queries and filters）就独自成为 Elasticsearch 的组件。但从 ES2.0开始，过滤（filters）已经从技术上被排除了，同时所有的查询（queries）拥有变成不评分查询的能力。然而，为了明确和简单，我们用"filter"这个词表示不评分、只过滤情况下的查询。你可以把 "filter"、"filtering query" 和 "non-scoring query" 这几个词视为相同的。相似的，如果单独地不加任何修饰词地使用 "query" 这个词，我们指的是 "scoring query"。<br>
过滤查询（Filtering queries）只是简单的检查包含或者排除，这就使得计算起来非常快。考虑到至少有一个过滤查询的结果是 “稀少的”（很少匹配的文档），并且经常使用不评分查询（non-scoring queries），结果会被缓存到内存中以便快速读取，所以有各种各样的手段来优化查询结果。相反，评分查询（scoring queries）不仅仅要找出匹配的文档，还要计算每个匹配文档的相关性，计算相关性使得它们比不评分查询费力的多。同时，查询结果并不缓存。多亏倒排索引（inverted index），一个简单的评分查询在匹配少量文档时可能与一个涵盖百万文档的filter表现的一样好，甚至会更好。但是在一般情况下，一个filter 会比一个评分的query性能更优异，并且每次都表现的很稳定。过滤（filtering）的目标是减少那些需要通过评分查询（scoring queries）进行检查的文档。通常的规则是，使用 查询（query）语句来进行 全文 搜索或者其它任何需要影响 相关性得分 的搜索。除此以外的情况都使用过滤（filters)。<br><br>



## 安装

**elasticsearch** <br>

1、解压
2、配置，编辑文件 elasticsearch.yml，请参考 [重要参数配置](https://www.elastic.co/guide/cn/elasticsearch/guide/current/important-configuration-changes.html)
```yaml
network.host: 0.0.0.0
cluster.name: es-dev
node.name: es_001
node.master: true
node.data: false
path.data: /xxx/xxx
path.logs: /xxx/xxx
path.plugins: /xxx/xxx
#候选节点个数/2+1 例如你有10个节点（能保存数据，同时能成为 master），法定数就是 6
discovery.zen.minimum_master_nodes: 2
discovery.zen.ping.unicast.hosts: ["host1", "host2:port"]
```
注意：
```
注意1：
你可以通过逗号分隔指定多个目录。数据可以保存到多个不同的目录，如果将每个目录分别挂载不同的硬盘，
这可是一个简单且高效实现一个软磁盘阵列（ RAID 0 ）的办法。Elasticsearch 会自动把条带化（注：
RAID 0 又称为 Stripe（条带化），在磁盘阵列中,数据是以条带的方式贯穿在磁盘阵列所有硬盘中的） 
数据分隔到不同的目录，以便提高性能。

注意2：
由于 ELasticsearch 是动态的，你可以很容易的添加和删除节点，但是这会改变这个法定最小节点个数。 
你不得不修改每一个索引节点的配置并且重启你的整个集群只是为了让配置生效，这将是非常痛苦的一件事情。基于这个原因， minimum_master_nodes （还有一些其它配置）允许通过 API 调用的方式动态进行配置。 当你的集群在线运行的时候，你可以这样修改配置，这将成为一个永久的配置，并且无论你配置项里配置的如何，这个将优先生效。当你添加和删除 master 节点的时候，你需要更改这个配置。
PUT /_cluster/settings
{
	"persistent" : {
	    "discovery.zen.minimum_master_nodes" : 2
	}
}

注意3：
最好使用单播：这意味着你的单播列表不需要包含你的集群中的所有节点， 它只是需要足够的节点，当一个新节点联系上其中一个并且说上话就可以了。如果你使用 master 候选节点作为单播列表，你只要列出三个就可以了。 这个配置在 elasticsearch.yml 文件中：
discovery.zen.ping.unicast.hosts: ["host1", "host2:port"]
```
3、启动和访问
```shell
$ bin/elasticsearch -d
$ curl 'http://localhost:9200/?pretty‘

#集群中默认数据传输端口：9300
#WEB端口：9200
```
4、插件
```shell
# 获取x-pack安装包(Do NOT put the file in the Elasticsearch plugins directory): 
$ wget https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-5.6.5.zip 
$ bin/elasticsearch-plugin install x-pack 
# 或者
$ bin/elasticsearch-plugin install file:///opt/software/x-pack-5.6.5.zip 
```
<br><br>



**kibana** <br>
1、解压
2、配置，编辑 kibana.yaml 文件
```yaml
server.host: 0.0.0.0
server.name: "xxx"
elasticsearch.url: "http://nimbusz:9200"
```
3、启动和访问(初始用户名密码：elastic/changeme)
```shell
$ nohup bin/kibana > /dev/null 2>&1 &
$ curl http://nimbusz:5601
```
4、插件
```shell
$ bin/kibana-plugin install x-pack 或者
$ bin/kibana-plugin install file:///opt/software/x-pack-5.6.5.zip
```






