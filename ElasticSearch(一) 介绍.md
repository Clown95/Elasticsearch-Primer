---

title: ElasticSearch(一) 介绍
tags: ElasticSearch
author: Clown95

---
# ElasticSearch介绍
**Elasticsearch**是一个实时分布式和开源的全文搜索和分析引擎。 它可以从RESTful Web服务接口访问，并使用模式少JSON(JavaScript对象符号)文档来存储数据。它是基于Java编程语言，这使Elasticsearch能够在不同的平台上运行。使用户能够以非常快的速度来搜索非常大的数据量。

## Elasticsearch的特性
- Elasticsearch可扩展高达PB级的结构化和非结构化数据。
- Elasticsearch可以用来替代MongoDB和RavenDB等做文档存储。
- Elasticsearch使用非标准化来提高搜索性能。
- Elasticsearch是受欢迎的企业搜索引擎之一，目前被许多大型组织使用，如Wikipedia，The Guardian，StackOverflow，GitHub等。
- Elasticsearch是开放源代码，可在Apache许可证版本`2.0`下提供。

## Elasticsearch的主要概念

- **Index(索引)**  
ElasticSearch将它的数据存储在一个或多个索引( index)中。用比传统的关系型数据库领域来说，索引相当于SQL中的一个数据库，可以向索引写入文档或者从索引中读取文档。

- **Type(数据类型)**
ElasticSearch中每个文档都有与之对应的类型(type)定义。这允许用户在一个索引中存储多种文档类型，并为不同文档类型提供不同的映射。Type是相当于关系数据库中的“表”。每种类型都有一列字段，用来定义文档的类型。

- **Document(文档)** 
文档是ElasticSearch的最小存储单位，对所有使用ElasticSearch的案例来说，它们最终都可以归结为对文档的搜索。文档是存储在elasticsearch中的一个JSON文件。这是相当与关系数据库中表的一行数据。每个文档被存储在索引中，并具有一个类型和一个id。许多条 Document 构成了一个 Index。

- **Field(字段)**
一个文档可以有多个字段，相当于关系型数据库中的列。

- **Mapping(类型/映射)**
映射像关系数据库中的表结构，每个索引都有一个映射，它定义了索引中的每一个字段类型，以及一个索引范围内的设置。

- **Node (节点)** 
节点是一个Elasticsearch的运行实例，是集群的构成单元

- **Cluster(集群)** 
它是一个或多个节点的集合。 集群为整个数据提供跨所有节点的集合索引和搜索功能。

- **Shards(碎片)**  
索引被水平细分为碎片。这意味着每个碎片包含文档的所有属性，但包含的数量比索引少。水平分隔使碎片成为一个独立的节点，可以存储在任何节点中。主碎片是索引的原始水平部分，然后这些主碎片被复制到副本碎片中。

- **Replicasedit(副本)**
Elasticsearch允许用户创建其索引和分片的副本。 复制不仅有助于在故障情况下增加数据的可用性，而且还通过在这些副本中执行并行搜索操作来提高搜索的性能

![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/5313646-4ee3e7d2e8da5d7c.png)

## Elasticsearch的优点
*   Elasticsearch是基于Java开发的，这使得它在几乎每个平台上都兼容。
*   Elasticsearch是实时的，换句话说，一秒钟后，添加的文档可以在这个引擎中搜索得到。
*   Elasticsearch是分布式的，这使得它易于在任何大型组织中扩展和集成。
*   通过使用Elasticsearch中的网关概念，创建完整备份很容易。
*   与Apache Solr相比，在Elasticsearch中处理多租户非常容易。
*   Elasticsearch使用JSON对象作为响应，这使得可以使用不同的编程语言调用Elasticsearch服务器。
*   Elasticsearch支持几乎大部分文档类型，但不支持文本呈现的文档类型。

## Elasticsearch的缺点
*   Elasticsearch在处理请求和响应数据方面没有多语言和数据格式支持(仅在JSON中可用)，与Apache Solr不同，Elasticsearch不可以使用CSV，XML等格式。
*   Elasticsearch也有一些伤脑的问题发生，虽然在极少数情况下才会发生。

## Elasticsearch和RDBMS之间的比较
在Elasticsearch中，索引是类型的集合，因为数据库是RDBMS(关系数据库管理系统)中表的集合。每个表都是行的集合，就像每个映射都是JSON对象的Elasticsearch集合一样。
| Elasticsearch | 关系数据库 |
| --- | --- |
| 索引 | 数据库 |
| 碎片 | 碎片 |
| 映射 | 表 |
| 字段 | 字段 |
| JSON对象 | 元组 |

