---

title: ElasticSearch(三) 基础入门
tags: Elasticsearch
author: Clown95

---


# Elasticsearch基础入门
当ElasticSearch的实例并运行时，我们如何向ElasticSearch推送数据并构建查询?为了提供这些功能,ElasticSearch对外公开了一个设计精巧的API。这个API是基于REST的，并在实践中能轻松整合到任何支持HTTP协议的系统中去。它使用典型的HTTP方法，诸如GET,POST.DELETE,PUT来实现资源的获取，添加，修改，删除等操作。

我们常用的交互方式是`Curl命令` `Kibnna 图像界面`

![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1558630943065.png)

 curl常用参数介绍： 
 - X 指定http的请求方法有：`HEAD` `GET` `POST` `PUT` `DELETE` 
 - d 指定要传输的数据 
 - H 指定http请求头信息 
 -  i  显示响应的头信息
	
## 创建Index
我们可以用REST API创建一个`PUT`请求，到一个由`索引名称`，`类型名称`和`ID`组成的URL来创建索引。具体如：`http:\\localhost:9200/<index>/<type>/[<id>]`。

`索引名称`和`类型`是必需的，而`id`部分是`可选`的。如果不指定ID，ElasticSearch会为我们生成一个ID。

但是，如果不指定ID，应该使用HTTP的`POST`请求而不是`PUT`请求。如果创建索引使用原先不存在的ID。 如果具有相同`类型`和`ID`的文档已存在，则会被覆盖，这就是`更新`。

 在6.0版本以后只能有一个`type`，所以现在`type`没有什么意义,建议使用`doc`作为`type`。

>POST和PUT都可以用于创建，二者之间的区别：  PUT是幂等方法，POST不是。所以PUT用户更新，POST用于新增比较合适。 

### 指定ID创建
说了这么多我们现在来使用`Curl`创建第一个索引，我们插入一个电影信息并指定ID,这里使用索引的名称为`movies`，类型名称是`doc`，id是`1`。

```bash
 curl -H "Content-Type: application/json" -XPUT "http://localhost:9200/movies/doc/1" -d'
{
    "title": "大话西游",
    "director": "刘镇伟",
    "year": 1994
}'
```
curl提交了带有`Head`的一个`PUT`请求的URL,并且使用 `-d`参数来提交数据，d即data。

> 注意： 新版本的es 必须要指定`head`   `-H "Content-Type: application/json"`

如果复制上面命令，到`Kibana` 或者`sense` ，会直接转成Kibana命令：
```bash
PUT /movies/doc/1
{
    "title": "大话西游",
    "director": "刘镇伟",
    "year": 1994
}
```
对比两种写法，很明显是使用`kibana`比较方便，并且`kibana`支持自动补全功能。

当然也可以反转成 `curl`命令,只需要点击编辑框右边的小板头然后点击`Copy as cURL`即可。

>为了学习效果更好，curl和kibana两种方式我们会都写出来。

提交成功后，我们可以看到返回了一个json信息:
```json
{
  "_index": "movies",
  "_type": "doc",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```
从Json信息中我们可以得到，`_index`索引名称 、` _type`类型、` _id `等，并且我们可以通过result来显示我们的操作。
这里我们先提前注意下`_version` 这个字段，目前我们都值是`1`

当然我们也可以不提交任何数据,只进行创建，如果不提交数据，那么我们只能有索引名称， 索引类型和ID都不能有
例如：
```bash
PUT /test 
```
> 注意：实际项目里一般是不会直接这样创建 index 的，这里仅为演示。一般都是通过创建 mapping 手动定义 index 或者自动生成 index 。

### 不指定ID创建

接着我们再创建一个不指定ID索引:

Curl命令:
```bash
 curl -H "Content-Type: application/json" -XPOST "http://localhost:9200/movies/movie" -d'
{
    "title": "阿飞正传",
    "director": "王家卫",
    "year": 1990
}'
```

Kibana命令:
```bash
POST /movies/movie
{
    "title": "阿飞正传",
    "director": "王家卫",
    "year": 1990
}
```
响应信息：
```json
	{
  "_index": "movies",
  "_type": "doc",
  "_id": "UWsaCGsBoJ5yZPbogTdL",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 1
}
```

从Json中可以看到，系统为我们指定了一个ID ,值为`UWsaCGsBoJ5yZPbogTdL` 。

## 查看index 是否存在

如果你想做的只是检查索引是否存在,你对内容完全不感兴趣可以使用`HEAD`方法来代替`GET`。`HEAD`请求不会返回响应体，只有HTTP头：

### index存在

现在我们来查询下movies中ID 为1的索引：
curl 命令:
```bash
curl -i -XHEAD "http://localhost:9200/movies/doc/1"
```
Kibana命令:
```bash
HEAD /movies/doc/1
```
响应内容：
```bash
HTTP/1.1 200 OK
content-type: application/json; charset=UTF-8
content-length: 222
```
如果返回200 OK状态，则索引存在

### index不存在

我们再来查询ID为100的索引:
curl 命令:
```bash
curl -i -XHEAD "http://localhost:9200/movies/doc/100"
```
Kibana命令:
```bash
HEAD /movies/doc/100
```
响应内容：
```bash
HTTP/1.1 404 Not Found
content-type: application/json; charset=UTF-8
content-length: 61
```
如果不存在返回 404 Not Found

## 查询Index

我们已经学习了索引的创建,现在我们来了解下怎查询索引

### 查询指定ID索引
我们首先来查询下指定ID的索引，方法是通过`ID`，使用`GET`来检索它。
具体是通过ID从ElasticSearch中检索文档可发出URL的`GET`请求,`http://localhost:9200/<index>/<type>/<id>`

Curl命令：
```bash
curl -XGET 'http://localhost:9200/movies/doc/1'
```
Kibana命令:
```bash
GET /movies/doc/1
```
响应信息：
```json
{
  "_index": "movies",
  "_type": "doc",
  "_id": "1",
  "_version":1,
  "found": true,
  "_source": {
    "title": "大话西游",
    "director": "刘镇伟",
    "year": 1994

  }
}
```
### 查询所有索引

目前我们已经有3个索引了，现在来查询所有的索引的文档 ,可以通过指定`_search`端点来查询索引信息。

- `_search` 是用来指定所有索引，所有type下的所有数据都搜索出来
- `intdex1/_serarch` 则是指定一个index，搜索出它所有tpye的数据,

我们先不指定索引来查询下：
Curl命令：
```bash
curl -XGET 'http://localhost:9200/_search'
```
Kibana命令:
```bash
GET /_search
```
接着我们指定索引来查询：
Curl命令：
```bash
curl -XGET 'http://localhost:9200/movies/_search'
```
Kibana命令:
```bash
GET /movies/_search
```

我们来简单的解读下它返回的json
```json
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
    "total": 2,
    "max_score": 1,
    "hits": [
      {
        "_index": "movies",
        "_type": "doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "title": "大话西游",
          "director": "刘镇伟",
          "year": 1994
        }
      },
      {
        "_index": "movies",
        "_type": "doc",
        "_id": "UWsaCGsBoJ5yZPbogTdL",
        "_score": 1,
        "_source": {
          "title": "阿飞正传",
          "director": "王家卫",
          "year": 1990
        }
      }
    ]
  }
}
```
- `took`：Elasticsearch执行此次搜索所用的时间(单位：毫秒)
- `timed_out`:  告诉我们此次搜索是否超时
- `_shards` : 搜索了多少个分片，以及搜索成功/失败分片的计数
- `hits` : 搜索结果
- `hits.total`：符合搜索条件的文档数量
- `hits.max_score`：本次搜索的所有结果中，最大的相关度分数是多少，每一条document对于search的相关度，越相关，_score分数越大，排位越靠前
- `hits.hits`：实际返回的搜索结果对象数组(默认只返回前10条)，默认按照`_score`降序排列
- `hits._source`：索引详情信

## 更新Index
现在，在索引中有了两部电影信息，接下来我们来了解如何更新它的文档。更新的语法和创建的语法一致，我们只需要使用相同的索引，然后`PUT`需要更新到字段请求即可。

### 更新整个文档

例如,我们想要给`movies/movie/1`索引的文档添加一个`genres`字段。

Curl命令：
```base
curl  -H "Content-Type: application/json" -XPUT "http://localhost:9200/movies/doc/1" -d'
{
    "title": "大话西游",
    "director": "刘镇伟",
    "year": 1994 ,
	"genres": ["喜剧","爱情"]
}'
```
Kibana命令:
```bash
PUT /movies/doc/1
{
    "title": "大话西游",
    "director": "刘镇伟",
    "year": 1994 ,
	"genres": ["喜剧","爱情"]
}
```
响应消息：
```json
{
  "_index": "movies",
  "_type": "doc",
  "_id": "1",
  "_version": 2,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 2,
  "_primary_term": 1
}
```
我们来先看下响应消息中的`result`,它的值是`updated`，说明我们完成了一个更新的操作。
我们再来看下`_version`，它的值为2，之前我们的值是1，`_version`就是es的内部版本控制，具体介绍我们留在后面再说，这边仅作了解。

### _update更新部分字段

刚刚我们为`/movies/doc/1`添加了一个分类信息，但是却请求了整个文档。像 "title" 、"director"、 "year" 这些内容我们都没有进行更改，所以我们没有必要提交这些字段的内容，这时候我们就引入`_updata` ，它可对索引文档信息进行部分更新。

例如我们还是为之前的索引添加一个actors字段：

Curl命令：
```bash
curl  -H "Content-Type: application/json" -XPOST "http://localhost:9200/movies/doc/1/_update" -d'
{
  "doc":{"actors": ["周星驰","朱茵"]}
}'
```
Kibana命令:
```bash
POST /movies/doc/1/_update
{
  "doc":{"actors": ["周星驰","朱茵"]}
}
```
> 注意：json里`doc`必须有，否则会报错。
 
 响应信息：
 ```json
 {
  "_index": "movies",
  "_type": "doc",
  "_id": "1",
  "_version": 3,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 3,
  "_primary_term": 1
}
 ```
 通过响应信息我们可以看到 `_version`的值又增加了，现在为3。 `result`显示的操作依然是 `updated`。
 
Elasticsearch的更新是怎么回事呢？在内部，Elasticsearch标记旧的文档为删除，并添加了一个完整的新文档。旧版本文档不会立即消失，但你也不能去访问它。Elasticsearch会在你继续索引更多数据时清理被删除的文档。

es 文档更新到过程如下：
1.	 从旧文档中检索JSON
2.	 修改它
3.	 删除旧文档
4.	 索引新文档
  
## 版本控制

### 内部版本控制
前面我们多次提到了`_version`,那么`_version`它到底什么什么？`_version`是内部版本控制，用来保证数据一致性的。

当使用	index API更新文档的时候，我们读取原始文档，做修改，然后将整个文档(whole	document)一次性重新索引。最近的索引请求会生效——Elasticsearch中只存储最后被索引的任何文档。如果其他人同时也修改了这个文档，他们的修改将会丢失。

很多时候， 这并不是一个问题。或许我们主要的数据存储在关系型数据库中，然后拷贝数据到Elasticsearch中只是为了可以用于搜索。或许两个人同时修改文档的机会很少。亦或者偶尔的修改丢失对于我们的工作来说并无大碍。但有时丢失修改是一个很严重的问题。

elasticsearch为每个文档自动设置一个version属性, version从1开始, 当文档发生更新，删除操作时version都会自增1, 在获取或查询文档是version作为文档的一部分返回。

版本号默认是不返回的，我们可以手动开启返回版本号，例如：
```bash
GET movies/movie/_search
{
  "version": true, 
  "query": {
      "match_all": {
      }
  }
}
```

ElasticSearch采用了乐观锁来保证数据的一致性，也就是说，当用户对document进行操作时，并不需要对该document作加锁和解锁的操作，只需要指定要操作的版本即可。当版本号一致时，ElasticSearch会允许该操作顺利执行，而当版本号存在冲突时，ElasticSearch会提示冲突并抛出异常（VersionConflictEngineException异常）。

如果我version不一致就会报错，例如：

Curl命令：
```bash
curl  -H "Content-Type: application/json" -XPUT "http://localhost:9200/movies/doc/1?version=3" -d'
{
    "title": "大话西游",
    "director": "刘镇伟",
    "year": 1994 ,
	"genres": ["喜剧","爱情","魔幻"],
	"actors": ["周星驰","朱茵","吴孟达","罗家英","蓝洁瑛","莫文蔚"]
}'
```
响应信息：
```json
{
  "_index": "movies",
  "_type": "doc",
  "_id": "1",
  "_version": 4,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 4,
  "_primary_term": 1
}
```
我们使用`version=3` 成功的完成了更新的操作，现在`version=4`

下面我们再来执行下kibana的命令，我们仍然使用`version=3`：
```bash
PUT /movies/doc/1?version=3
{
    "title": "大话西游",
    "director": "刘镇伟",
    "year": 1994 ,
	"genres": ["喜剧","爱情","魔幻"],
	"actors": ["周星驰","朱茵","吴孟达","罗家英","蓝洁瑛","莫文蔚"]
}
```
响应信息：
```json
{
  "error": {
    "root_cause": [
      {
        "type": "version_conflict_engine_exception",
        "reason": "[movie][1]: version conflict, current version [4] is different than the one provided [3]",
        "index_uuid": "z1drLcqVTzuQEb3PNVVeYA",
        "shard": "3",
        "index": "movies"
      }
    ],
    "type": "version_conflict_engine_exception",
    "reason": "[movie][1]: version conflict, current version [4] is different than the one provided [3]",
    "index_uuid": "z1drLcqVTzuQEb3PNVVeYA",
    "shard": "3",
    "index": "movies"
  },
  "status": 409
}
```
我们可以看到执行命令后报错，错误原因是"version conflict, current version [4] is different than the one provided [3]（版本冲突，当前版本[4]不同于[3]）" .

### 外部版本控制
`version`是内部版本控制，那么什么是`外部版本控制`呢？

elasticsearch在处理外部版本号时会与对内部版本号的处理有些不同。它不再是检查`_version`是否与请求中指定的数值相同,而是检查当前的`_version`是否比指定的数值小。如果请求成功，那么外部的版本号就会被存储到文档中的_version中。

通俗的来说外部的版本控制的要求是：
-  不需要版本号一致
-  指定的版本号要比索引的版本号大才可以
  
为了保持_version与外部版本控制的数据一致,使用`version_type=external` 。

我们来演示下：
```bash
PUT /movies/doc/1?version=6&version_type=external
{
      "title": "大话西游",
    "director": "刘镇伟",
    "year": 1994 ,
	"genres": ["喜剧","爱情","魔幻"],
	"actors": ["周星驰","朱茵","吴孟达","罗家英","蓝洁瑛","莫文蔚"]
}

```
响应信息：
```bash
{
  "_index": "movies",
  "_type": "doc",
  "_id": "1",
  "_version": 6,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 5,
  "_primary_term": 1
}
```
通过响应消息我们可以看到，我们提交成功，并且版本号由4变成了6。 

但是如果你使用内部版本控制的话，即使version指定的值比原来的大也不是不允许执行。
我们来验证下：
```bash
PUT /movies/doc/1?version=7
{
      "title": "大话西游",
    "director": "刘镇伟",
    "year": 1994 ,
	"genres": ["喜剧","爱情","魔幻"],
	"actors": ["周星驰","朱茵","吴孟达","罗家英","蓝洁瑛","莫文蔚"]
}
```
响应消息：
```bash
{
  "error": {
    "root_cause": [
      {
        "type": "version_conflict_engine_exception",
        "reason": "[movie][1]: version conflict, current version [6] is different than the one provided [7]",
        "index_uuid": "z1drLcqVTzuQEb3PNVVeYA",
        "shard": "3",
        "index": "movies"
      }
    ],
    "type": "version_conflict_engine_exception",
    "reason": "[movie][1]: version conflict, current version [6] is different than the one provided [7]",
    "index_uuid": "z1drLcqVTzuQEb3PNVVeYA",
    "shard": "3",
    "index": "movies"
  },
  "status": 409
}
```
通过响应消息，我们发现情况确实如此。

## 删除索引
### 删除单个索引
到目前为止我们已经简单了解索引的创建、更新和查询，现在我们来了解怎么删除索引。

还记得我们之前创建了一个没有手动指定ID的索引吗？

系统为我们自动指定ID为`UWsaCGsBoJ5yZPbogTdL`，现在我们就根据这个ID来删除这个索引。

Curl命令：
```bash
curl -XDELETE 'http://localhost:9200/movies/doc/UWsaCGsBoJ5yZPbogTdL' 
```
Kibana命令:
```bash
DELETE /movies/doc/UWsaCGsBoJ5yZPbogTdL
```
> 注意：系统生成的ID每个人都不一样，请注意修改。

响应消息：
```json
{
  "_index": "movies",
  "_type": "doc",
  "_id": "UWsaCGsBoJ5yZPbogTdL",
  "_version": 2,
  "result": "deleted",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 1
}
```
我们可以看到`result`的值是`deleted` ,说明我们完成的是删除操作。

### 删除指定名称的所有索引
删除指定名称的所有索引，只需要指定索引名称就行。
例如，我们删除我们之前创建的`movis`的所有索引
Curl命令：
```bash
curl -XDELETE 'http://localhost:9200/movies' 
```
Kibana命令:
```bash
DELETE /movies
```

### 删除所有索引
删除所有索引那就更简单了，我们只需要使用通配符 `*` 即可

Curl命令：
```bash
curl -XDELETE "http://10.211.55.8:9200/*"
```
Kibana命令:
```bash
DELETE *
```
> 注意：这种操作很危险，请慎用！！！

##  批量API（多文档处理）

前面我们掌握了单索引的操作，对于搜索引擎来说，单索引操作肯定是满足不了我们都要求的，并且单所以的效率也较低，所以我们现在来了解多索引操作。批量API用于通过在单个请求中进行多个索引/删除操作来批量上传或删除JSON对象,需要添加`_bulk`关键字来调用API 。

bulk API允许我们使用单一请求来实现多个文档的	create	、 	index	、 	update	或	delete	。 这对索引类似于日志活动这样的数据流非常有用，它们可以以成百上千的数据为一个批次按序进行索引，这样会大大提升效率。

bulk的格式：
```
{action:{metadata}}
{requstbody}
```
action 可选择值为：
action |	解释说明
--|--
create	|当文档不存在时创建
index	|创建新文档或替换已有文档
update	|局部更新文档
delete	|删除一个文档

批量API调用有两种方式，一是通过手动输入， 二是通过文本文件导入（如json文件）

## 批量创建文档
现在我们来使用批量API来，批量添加电影信息。

### 手动输入

Curl命令：
```bash
curl -H "Content-Type: application/json" -XPOST  "http://localhost:9200/_bulk" -d'
{"create":{"_index":"movies","_type":"doc","_id":"1"}}
{"title": "大话西游","director": "刘镇伟","year": 1994 ,"genres": ["喜剧","爱情","魔幻"],"actors": ["周星驰","朱茵","吴孟达","罗家英","蓝洁瑛","莫文蔚"]}
{"create":{"_index":"movies","_type":"doc","_id":"2"}}
{"title":"监狱风云1","director":"林岭东","year":1987,"genres":["犯罪","剧情","动作"],"actors": ["周润发","梁家辉","张耀扬"]}
{"index":{"_index":"movies","_type":"doc","_id":"3"}}
{"title":"监狱风云2","director":"林岭东","year":1991,"genres":["犯罪","剧情","动作"],"actors": ["周润发","徐锦江","陈松勇"]}
{"create":{"_index":"movies","_type":"doc","_id":"4"}}
{"title":"肖申克的救赎","director":"弗兰克·德拉邦特","year":1994,"genres":["犯罪","剧情"],"actors": ["蒂姆·罗宾斯","摩根·弗里曼"]}
{"create":{"_index":"movies","_type":"doc","_id":"5"}}
{"title":"霸王别姬","director":"陈凯歌","year":1993,"genres":["剧情","爱情","同性"],"actors": ["张国荣","巩俐","张丰毅"]}
{"create":{"_index":"movies","_type":"doc","_id":"6"}}
{"title":"三傻大闹宝莱坞","director":"拉库马·希拉尼","year":2009,"genres":["喜剧","爱情","歌舞","剧情"],"actors": ["阿米尔·汗","马德哈万","沙尔曼·乔什","卡琳娜·卡普"]}
'
```
可以看到我们既有`index` 也有`create` ,两个都是创建索引，区别是`index`所创建的文档已经存在，那么会替换原来的文档，而`create`不会去更新原来的文档。

> 注意：每个数据结尾要换行，这样curl 才能识别每行数据。

bulk请求不是原子操作——它们不能实现事务。每个请求操作时分开的，所以每个请求的成功与否不干扰其它操作。如果某条失败了，并不影响下一条继续执行。

下面Kibana命令，里面的数据不一样，也需要执行一下:
```bash
POST /_bulk
{"index":{"_index":"movies","_type":"doc","_id":"7"}}
{"title":"盗梦空间","director":"克里斯托弗·诺兰","year":2010,"genres":["科幻","动作","动作"],"actors": ["克里斯托弗·诺兰","莱昂纳多·迪卡普里奥"]}
{"create":{"_index":"movies","_type":"doc","_id":"8"}}
{"title":"侏罗纪公园","director":"史蒂文·斯皮尔伯格","year":1993,"genres":["科幻","惊悚","冒险"],"actors": ["山姆·尼尔","劳拉·邓恩","杰夫·高布伦 "]}
{"create":{"_index":"movies","_type":"doc","_id":"9"}}
{"title":"爱在日落黄昏时","director":"理查德·林克莱特","year":2004,"genres":["爱情","剧情"],"actors": ["伊桑·霍克","朱莉·德尔佩","弗农·多布切夫"]}
{"index":{"_index":"movies","_type":"doc","_id":"10"}}
{"title":"罗马假日","director":"威廉·惠勒","year":1953,"genres":["喜剧","剧情","爱情"],"actors": ["奥黛丽·赫本","格利高里·派克","埃迪·艾伯特"]}
{"create":{"_index":"movies","_type":"doc","_id":"11"}}
{"title":"卢旺达饭店","director":"特瑞·乔治","year":2004,"genres":["战争","剧情","历史"],"actors": ["唐·钱德尔","苏菲·奥康内多","杰昆·菲尼克斯"]}
{"index":{"_index":"movies","_type":"doc","_id":"12"}}
{"title":"萤火虫之墓","director":"高畑勋","year":1988,"genres":["战争","剧情","动画"],"actors": ["辰己努","白石绫乃","志乃原良子"]}
{"create":{"_index":"movies","_type":"doc","_id":"13"}}
{"title":"霸王花","director":"钱升玮","year":1988,"genres":["动作","剧情","犯罪"],"actors": ["胡慧中","惠英红","罗芙洛","柏安妮"]}
{"create":{"_index":"movies","_type":"doc","_id":"14"}}
{"title":"澳门风云3","director":"王晶","year":2016,"genres":["搞笑","动作"],"actors": ["周润发","刘德华","张家辉","张学友","余文乐"]}
```
### 使用文文本文件导入

使用`cat movies.json`或者`vim movies.json`写入文本,添加我们的请求正文：

```json
{"create":{"_index":"movies","_type":"doc","_id":"15"}}
{"title":"西游降魔篇","director":"周星驰","year":2013,"genres":["奇幻","冒险","搞笑"],"actors": ["文章","舒淇","黄渤","罗志祥","李尚正"]}
{"create":{"_index":"movies","_type":"doc","_id":"16"}}
{"title":"龙猫","director":"宫崎骏","year":1988,"genres":["动画","冒险","奇幻"],"actors": ""}
{"index":{"_index":"movies","_type":"doc","_id":"17"}}
{"title":"阿甘正传","director":"罗伯特·泽米吉斯","year":1994,"genres":["爱情","剧情"],"actors": ["汤姆·汉克斯","罗宾·怀特","加里·西尼斯"]}
{"create":{"_index":"movies","_type":"doc","_id":"18"}}
{"title":"阿凡达","director":"詹姆斯·卡梅隆","year":2009,"genres":["动作","冒险","奇幻"],"actors": ["萨姆·沃辛顿","佐伊·索尔达娜","西格妮·韦弗","史蒂芬·朗"]}
{"create":{"_index":"movies","_type":"doc","_id":"19"}}
{"title":"流浪地球","director":"","year":"","genres":"","actors": ""}
{"create":{"_index":"movies","_type":"doc","_id":"20"}}
{"title":"这个杀手不太冷","director":"","year":"","genres":"","actors": ""}
```
现在我们来插入数据,如果要提供文本文件输入curl，则必须使用 `--data-binary`标志而不是`-d`，因为`-d`的不能保留换行符。

Curl命令：
```bash 
curl -s -H "Content-Type: application/x-ndjson" -XPOST http://localhost:9200/_bulk --data-binary "@movies.json"
```
Kibana命令:
```bash
POST /_bulk
-binary @movies.json
```

## 批量请求数据量控制

我们之前说过，批量API 效率会比较高，因为它只有一个请求，但是如果数据很大的话，那么批量API的效率反而会降低。

整个批量请求需要被加载到接受我们请求节点的内存里，所以请求越大，给其它请求可用的内存就越小。有一个最佳的bulk请求大小。超过这个大小，性能不再提升而且可能降低。我们需要找到一个最佳的数量。

最佳大小，当然并不是一个固定的数字。它完全取决于你的硬件、你文档的大小和复杂度以及索引和搜索的负载。那么我们怎么找到这个最佳大小？

- 试着批量索引标准的文档，随着大小的增长，当性能开始降低，说明你每个批次的大小太大了。
-  开始的数量可以在1000~5000个文档之间，如果你的文档非常大，可以使用较小的批次。
- 通常着眼于你请求批次的物理大小是非常有用的。一千个1kB的文档和一千个1MB的文档大不相同。一个好的批次最好保持在5-15MB大小间。

## 批量更新文档
除了删除，更新请求跟创建都差不多，但是请求体中必须要有`doc`。

下面我们使用的是手动输入，当然你也可以使用文本导入，这里就不在演示了。
```bash
POST /movies/doc/_bulk
{"update": {"_id": "19" }}
{"doc": {"director":"郭帆","year":2019,"genres":["科幻","灾难"],"actors": ["吴京","屈楚萧","李光洁","吴孟达","赵今麦"]}}
{"update":{"_id":"20"}}
{"doc": {"director":"吕克·贝松","year":1994,"genres":["剧情","惊悚","犯罪"],"actors": ["让·雷诺加里·奥德曼","娜塔丽·波曼"]}}
```
通过响应文件可以知道成功更新
![enter description here](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559224470439.png)

## 批量获取文档

如果你需要从Elasticsearch中检索多个文档，相对于一个一个的检索，更快的方式是在一个请求中使用`multi-get`或者`mget` API。
`mget` API参数是一个docs数组，数组的每个节点定义一个文档的`_index`、`_type`、`_id元数据`

> 必须使用`docs`数组来指定需要提取的所有文档的索引，类型和ID

Curl命令：
```bash
curl -H "Content-Type: application/json" -XPOST http://localhost:9200/_mget -d '
{
   "docs":[
     {"_index":"movies", "_type":"doc", "_id":"1"},
     {"_index":"movies", "_type":"doc", "_id": "2"},
     {"_index":"movies", "_type":"doc", "_id": "3"},
     {"_index":"movies", "_type":"doc", "_id": "4"},
     {"_index":"movies", "_type":"doc", "_id": "5"},
     {"_index":"movies", "_type":"doc", "_id": "6"},
     {"_index":"movies", "_type":"doc", "_id": "7"},
     {"_index":"movies", "_type":"doc", "_id": "8"},
     {"_index":"movies", "_type":"doc", "_id": "9"},
     {"_index":"movies", "_type":"doc", "_id": "10"} 
   ]
}'
```
Kibana命令:
```bash
POST /_mget
{
   "docs":[
     {"_index":"movies", "_type":"doc", "_id":"11"},
     {"_index":"movies", "_type":"doc", "_id": "12"},
     {"_index":"movies", "_type":"doc", "_id": "13"},
     {"_index":"movies", "_type":"doc", "_id": "14"},
     {"_index":"movies", "_type":"doc", "_id": "15"},
     {"_index":"movies", "_type":"doc", "_id": "16"},
     {"_index":"movies", "_type":"doc", "_id": "17"},
     {"_index":"movies", "_type":"doc", "_id": "18"},
     {"_index":"movies", "_type":"doc", "_id": "19"},
     {"_index":"movies", "_type":"doc", "_id": "20"} 
   ]
}
```
> 如果POST请求不指定index 和type，那么请求体里面的`_index`和`_type` 可以是任意存在 index 和type（但是两个要匹配），不一定都是相同的索引。

>  如果一个或多个文档没有找到，那么求是返回状态码200，不会影响到其他内容的查询，原因是mget请求本身成功了。如果想知道每个文档是否都成功了，你需要检查found标志。

## 批量删除文档

批量删除文档和创建、更新的用法有点区别，批量删除他不需要请求体。

批量删除也是可以使用文本导入，我们也不演示了。

==我们都数据都还有用处，所以下面的批量删除命令只做演示，不要执行！不要执行！ 不要执行！ #FF5722==

Curl命令：
```bash
curl -H "Content-Type: application/json" -XPOST http://localhost:9200/movies/doc/_bulk -d'
{"delete":{"_id":"1"}}
{"delete":{"_id":"2"}}
{"delete":{"_id":"3"}}
{"delete":{"_id":"4"}}
{"delete":{"_id":"5"}}
'
```
Kibana命令:
```bash
POST /movies/doc/_bulk
{"delete":{"_id":"6"}}
{"delete":{"_id":"7"}}
{"delete":{"_id":"8"}}
{"delete":{"_id":"9"}}
{"delete":{"_id":"10"}}
```
### 综合使用
当然批量API 也可以混合使用，

```bash
POST /movies/doc/_bulk
{"create":{"_index":"movies","_type":"doc","_id":"21"}}
{"title":"楚门的世界","director":"彼得·威尔","year":1998,"genres":["科幻","剧情"],"actors": ["金·凯瑞","劳拉·琳妮","艾德·哈里斯"]}
{"create":{"_index":"movies","_type":"doc","_id":"22"}}
{"title":"V字仇杀队","director":"","year":"","genres":"","actors": ""}
{"update":{"_id":"22"}}
{"doc":{"director":"詹姆斯·麦克提格","year":2006,"genres":["动作","惊悚","科幻"],"actors": ["娜塔莉·波特曼","雨果·维文","鲁珀特·格雷夫斯","斯蒂芬·瑞"]}}
{"delete":{"_id":"21"}}
'
```
我们分别查看 ID 是21 ，22 的索引

```bash
POST /_mget
{
   "docs":[
     {"_index":"movies", "_type":"doc", "_id":"11"},
     {"_index":"movies", "_type":"doc", "_id": "12"}
	 ]
}
```
响应消息：
```bash
{
  "docs": [
    {
      "_index": "movies",
      "_type": "doc",
      "_id": "21",
      "found": false
    },
    {
      "_index": "movies",
      "_type": "doc",
      "_id": "22",
      "_version": 2,
      "found": true,
      "_source": {
        "title": "V字仇杀队",
        "director": "詹姆斯·麦克提格",
        "year": 2006,
        "genres": [
          "动作",
          "惊悚",
          "科幻"
        ],
        "actors": [
          "娜塔莉·波特曼",
          "雨果·维文",
          "鲁珀特·格雷夫斯",
          "斯蒂芬·瑞"
        ]
      }
    }
  ]
}
```
通过响应信息我们可以发现 ID=21的索引没被发现， ID=22的索引文档已经更新。

## 索引别名

在Elasticsearch所有的API中，对应的是一个或者多个索引。Elasticsearch 可以对一个或者多个索引指定别名，通过别名可以查询到一个或者多个索引的内容。在内部，Elasticsearch会自动把别名映射到相应的索引上。可以对别名编写过滤器或者路由，在系统中别名不能重复，也不能和索引名重复。其实Elasticsearch的别名机制有点像数据库中的视图。

例如:为索引`movies`增加一个别名`films`。

Curl命令：
```bash
curl -H "Content-Type: application/json -XPOST "http://localhost:9200/_aliases" -d'
{
  "actions": [
    {
      "add": {
        "index": "movies",
        "alias": "films"
      }
    }
  ]
}'
```
Kibana命令:
```bash
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "movies","alias": "films"
      }
    }
  ]
}
```
别名没有修改的语法，当需要修改别名的时候，可以先删除别名，然后再增加别名。

删除别名也很简单，只需要把`add`改成`remove`,例如：
Curl命令：
```bash
curl -H "Content-Type: application/json -XPOST "http://localhost:9200/_aliases" -d'
{
  "actions": [
    {
      "remove": {
        "index": "movies",
        "alias": "films"
      }
    }
  ]
}'
```
Kibana命令:
```bash
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "movies",
        "alias": "films"
      }
    }
  ]
}
```
## 重建索引
我们虽然可以给索引添加新的类型，或给类型添加新的字段，但是不能添加新的分析器或修改已有字段。

修改在已存在的数据最简单的方法是重新索引：创建一个新配置好的索引，然后将所有的文档从旧的索引复制到新的上。

为了更高效的索引旧索引中的文档，使用`scan-scroll`来批量读取旧索引的文档，然后将通过`bulk API`来将它们推送给新的索引。
使用格式如下：
```bash
GET /old_index/_search?search_type=scan&scroll=1m
{
    "query": {
        "range": {
            "date": {
                "gte":  "2014-01-01",
                "lt":   "2014-02-01"
            }
        }
    },
    "size":  1000
}
```
`scan`是扫描的意思，用来扫描旧的索引相当于写入内容到新索引， `scroll` 是滚动索引，相当于遍历的作用，这个我们后面会具体讲解。

## 使用_cat查看Elasticsearch状态
我们可以使用`_cat`命令查看Elasticsearch状态。

参数| 用法  |描述
--|--|--
allocation	| GET/_cat/allocation | 查询每个节点上分配的分片（shard）的数量和每个分片（shard）所使用的硬盘容量
shards	| GET/_cat/shards  | 输出节点包含分片的详细信息
nodes	| GET/_cat/nodes  | 输出当前集群的拓扑结构（包括当前节点所在的地方和整个集群的相关信息等
indices	| GET/_cat/indices   | 查询指定索引的相关信息
 count |  GET/_cat/count  | 	快速查询当前整个集群或者指定索引的document的数量
 health  | GET/_cat/health |  查询当前集群的健康信息
plugins | GET/_cat/plugins | 输出每个节点正在运行的插件信息

> 更多内容请看官方文档 https://www.elastic.co/guide/en/elasticsearch/reference/current/cat.html#intro

