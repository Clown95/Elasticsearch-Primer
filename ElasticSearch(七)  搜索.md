---

title: ElasticSearch(七)  搜索
tags: ElasticSearch
author: Clown95

---

# 搜索

在前面，已经介绍了在ElasticSearch索引中处理数据的基础知识，现在是时候进行核心功能的学习了。
搜索主要有两种方式：
- URI Search
	- 操作简便,方便通过命令行测试
   -  但是仅包含部分查询语法

- Request Body Search
	- es 最常用的方式，查询丰富。
	- 提供的完备查询语法Query DSL(Domain Specific Language)

![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/query-dsl-multi-match.png)

## URI Search

URL检索是通过提供请求参数纯粹使用URI来执行搜索请求。
方法: `curl -XGET "http://localhost:9200/movies/doc/_search?参数`，多个参数用&分开。

常用参数如下：
参数 |描述
--|--
q |查询字符串（映射到query_string查询）
df | 在查询中不指定字段是默认查询的字段，如果不指定字段，ES会查询所有字段
analyzer| 分析查询字符串时要使用的分析器名称
sort | 排序，可以升序排序和降序排序
timeout |指定超时时间，默认为无超时
from | 返回的索引匹配结果的开始值，默认为0
size| 要返回的搜索条数，默认为10
default_operator| 要使用的默认运算符可以是AND或 OR，默认为OR

> 详细参数查看官方文档 ：https://www.elastic.co/guide/en/elasticsearch/reference/6.2/search-uri-request.html

下面我们来了解下怎么使用这些参数：

### q
我们首先来了解下 `q`的使用，现在我们来查找`监狱风云`的信息 :

```bash
GET /movies/doc/_search?q=%E7%9B%91%E7%8B%B1%E9%A3%8E%E4%BA%91
```
>E7%9B%91%E7%8B%B1%E9%A3%8E%E4%BA%91 是监狱风云的 url编码
>在线转码工具 ：http://tool.oschina.net/encode?type=4

当然我们也可以指定字段查询，例如：
```bash
GET /movies/doc/_search?q=title:%E7%9B%91%E7%8B%B1%E9%A3%8E%E4%BA%91
```

响应信息：
```bash
	{
  "took": 14,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 5.9509215,
    "hits": [
      {
        "_index": "movies",
        "_type": "doc",
        "_id": "2",
        "_score": 5.9509215,
        "_source": {
          "title": "监狱风云1",
          "director": "林岭东",
          "year": 1987,
          "genres": [
            "犯罪",
            "剧情",
            "动作"
          ],
          "actors": [
            "周润发",
            "梁家辉",
            "张耀扬"
          ]
        }
      },
      {
        "_index": "movies",
        "_type": "doc",
        "_id": "3",
        "_score": 3.8119292,
        "_source": {
          "title": "监狱风云2",
          "director": "林岭东",
          "year": 1991,
          "genres": [
            "犯罪",
            "剧情",
            "动作"
          ],
          "actors": [
            "周润发",
            "徐锦江",
            "陈松勇"
          ]
        }
      },
      {
        "_index": "movies",
        "_type": "doc",
        "_id": "14",
        "_score": 1.9059646,
        "_source": {
          "title": "澳门风云3",
          "director": "王晶",
          "year": 2016,
          "genres": [
            "搞笑",
            "动作"
          ],
          "actors": [
            "周润发",
            "刘德华",
            "张家辉",
            "张学友",
            "余文乐"
          ]
        }
      }
    ]
  }
}
```
	
我们可以看到每个被匹配出来的索引信息都有多个`_score` 和一个`max_score` ,这就是相关性评分。
默认情况下，Elasticsearch根据结果相关性评分来对结果集进行排序，所谓的「结果相关性评分」就是文档与查询条件的匹配程度。很显然，排名第一的`监狱风云1`的`title`字段明确的写到`监狱风云`。但是为什么`澳门风云3`也会出现在结果里呢？首先我们使用的是ES的默认分词器，它会把中文内容分成单个汉字，所以只要包含`监狱风云`中任意一个字的文档都会被匹配出来。又因为`监狱风云1` 完全匹配了`监狱风云` 四个字，而`澳门风云3`只匹配了`风云`两个字，所以`监狱风云1`的`_score`比`澳门风云3`的`_score`高
我们先了解下这个概念，后面我们再详细介绍

### sort
我们再来了解下排序`sort` ，我们根据year字段升序排列,例如：
```bash
GET /movies/doc/_search?q=title:%E7%9B%91%E7%8B%B1&sort=year:asc&pretty
```
asc 按升序排序，desc 按降序排序

### from和size 分页
`_serarch`返回的内容默认只有10个文档在hits数组中。我们的`movies`有20个左右的文档，那么我们如何看到其他文档？

和SQL使用LIMIT关键字返回只有一页的结果一样，Elasticsearch接受from和size参数，size: 表示查询多少条文档，默认10 ，from: 从第几行开始，默认0

现在我们想要每页显示5个数据，显示3页内容:

```bash
	GET /movies/doc/_search?&size=5&from=0
	GET /movies/doc/_search?&size=5&from=5
	GET /movies/doc/_search?&size=5&from=10	
```
	
### \_source过滤
 通常，GET	请求将返回文档的全部，存储在`	_source`	参数中。但是可能你感兴趣的字段只`title`。 请求个别字段可以使用`_source`参数。多个字段可以使用逗号分隔：
```bash
	GET	/movies/doc/1?_source=title,actors
```

### 操作符
Url Search 支持的操作符有：

- AND(&&), OR(||), NOT(!) 
- `+` `-`分别对应must和must not
- \>  <  >=  <=  
 	
> 注意：`+` `-`等符号url不能直接识别,要使用url  encode后的结果才可以。
	
现在我们来使用AND查询分类即`剧情`,又是`同性`
	
```bash
GET  movies/movie/_search?q=genres:(%e5%89%a7%e6%83%85 AND %e5%90%8c%e6%80%a7)&pretty
```
或者使用&& ，在下面的命令中我们把 &转成了%26
```bash
GET  movies/movie/_search?q=genres:(%e5%89%a7%e6%83%85 %26%26 %e5%90%8c%e6%80%a7)&pretty
```
我们再来使用 `+`,来进行多个匹配，只要任意满足一个即可匹配到
```bash
	GET  movies/movie/_search?q=genres:(%e5%89%a7%e6%83%85 %2B %e5%90%8c%e6%80%a7)&pretty
```
我们还可以指定查找的范围，例如我们查找大于1995年的信息：
```
curl -XGET "http://localhost:9200/movies/doc/_search?q=year:(>1995)"
```

查找大于1990且＜2000年的信息：
```
GET /movies/doc/_search?q=year:(>1990 AND <2000)
```
## Request Body Search
前面我们使用的Url search ,我们也可以使用请求正文来搜索信息，我们需要向请求正文中提供查询，这种方法是我们最常用的搜索方式。

为了使用ElasticSearch进行搜索，我们使用`_search`端点，可选择使用索引和类型。也就是说，按照以下模式向URL发出请求：
`http://localhost:9200/<index>/<type>/_search`。其中，index和type都是可选的。

请求正文是一个JSON对象，除了其它属性以外，它还要包含一个名称为“query”的属性。这种查询方法叫`查询DSL`。你可能想知道查询DSL是什么。它是ElasticSearch自己基于JSON的域特定语言，可以在其中表达查询和过滤器。
```json
{
    "query": {
           Query DSL here
    }
}
```

Query DSL基于JSON定义的查询语言,主要包含如下两种类型:
- 字段类查询 ：如term, match, range等，只针对某一 个字段进行查询
	字段类查询主要包括以下两类:
	- 全文匹配
	 针对text类型的字段进行全文检索,会对查询语句先进行分词处理,如match,match_ phrase等query类型
	
	- 单词匹配
	 不会对查询语句做分词处理,直接去匹配字段的倒排索引,如term, terms,range等query类型

- 复合查询：如bool查询等,包含一个或多个字段类查询或者复合查询语句。

## query_string查询
`query_string`  类似 `Url serarch ` 里面的参数`q`。
我使用`query_string` (查询字符串)，现在我们来查询`风云`所对应的文档。

```bash
POST movies/doc/_search
{
    "query": {
        "query_string": {
            "query": "风云"
        }
    }
}
```
## term查询
term 查询，可以用它处理数字（numbers）、布尔值（Booleans）、日期（dates）以及文本（text），而不是全文本字段。term是代表完全匹配，也就是精确查询，搜索前不会再对搜索词进行分词。 

下面我们查询指定的`year`字段：
```bash
GET /movies/_search
{
    "query" : {
      "term": {
        "year": {"value": 1994 }
      }
    }
}
```

6.X以上版本`term`好像是无法直接查询字符串内容，需要在字段后面添加`.keyword`,例如：

```bash
GET /movies/doc/_search
{
  "query": {
    "term": {
      "title.keyword": {
        "value": "大话西游"
      }
    }
  }
}
```
既然已经提到了`keyword`,我们就简单的说下,`title.keyword`，是es最新版本内置建立的field，就是不分词的。所以一个`title`过来的时候，会建立两次索引，一次是自己本身，是要分词的，分词后放入倒排索引；另外一次是基于title.keyword，不分词，保留256个字符最多，直接一个字符串放入倒排索引中。

如果一个字段既需要分词搜索，又需要精准匹配，我们就可以通过增加`keyword`字段来支持精准匹配。
```bash
{
    "type": "text",
    "fields": {
        "keyword": {
            "type": "keyword",
            "ignore_above": 256
        }
    }
}
```
这样相当于有`address`和`address.keyword`两个字段。

## terms查询
`term`查询对于查找单个值非常有用，但通常我们可能想搜索多个值。如果我们想搜索多个电影信息的文档该如何处理？

我们不需要使用多个` term` 查询，我们只要用` terms`查询,它相当于sql中的 `in` 。

使用的方法和`term`一样，现在我们来查询电影分类是`动作`和`剧情`的文档：
```bash
GET movies/movie/_search
{
  "query": {
    "terms": {
      "genres.keyword": ["剧情","动作"]
    }
  }
}
```
可以看到我们只是把terms查询的内容使用数组替代而已，其他基本上一样。
## match查询
还记得我们之前使用URL参数搜索`监狱风云`的时候，我们还演示了指定为`title`字段进行搜索。我们再来使用`match`查询来达到这样的效果，`match查询`它可以被认为是基本的属性搜索查询(就是通过特定的一个或多个属性来搜索)。

match搜索会先对搜索词进行分词，对于最基本的match搜索来说，只要搜索词的分词集合中的一个或多个存在于文档中即可，例如，当我们搜索`监狱风云`，搜索词会先分词为`监狱`和`风云`,只要文档中包含`监狱`和`风云`任意一个词，都会被搜索到 。
```bash
GET /movies/_search
{
    "query" : {
        "match" : { "title" : "监狱风云" }
    }
}
```
像这样指定搜索内容为`title`字段这样的设置称为`fields`，可用于指定要搜索的字段列表。如果不使用`fields`字段，ElasticSearch查询将默认自动生成的名为`_all`的特殊字段，来基于所有文档中的各个字段匹配搜索。
## fuzzy查询
我们再使用搜索引擎的时候，肯定出现过拼写错误的情况，比如说我们搜索`elasticsearch` 但是我不小心拼错成了`elasticsearhc`,但是搜索引擎仍然给我搜索出正确的结果。

![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559185250334.png)

使用`term`肯定是不行的，因为它是精确查找，所以我们引出`fuzzy`模糊查询。

为了方便演示，我们需要添加几个索引信息。
```bash
PUT  myname/doc/1
{
  "name" :"clown"
}
PUT  myname/doc/2
{
  "name" :"clwon"
}

PUT  myname/doc/3
{
  "name" :"colwn"
}
```
现在我们需要查询`clown` 这个文档：
```bash
GET myname/doc/_search
{
   "query": {
     "fuzzy": {
       "name": {
         "value": "clown",
         "fuzziness":  2
       }
     }
   }
}
```

`fuzziness`是你的搜索文本最多可以纠正几个字母去跟你的数据进行匹配，默认如果不设置，就是2。

## wildcard查询
wildcard（通配符）查询和prefix查询类似，也是一个基于词条的低级别查询。但是它能够让你指定一个模式(Pattern)，而不是一个前缀(Prefix)。它使用标准的shell通配符：?用来匹配任意字符，\*用来匹配零个或者多个字符。

我们来演示下：
```bash
GET myname/doc/_search
{
   "query": {
      "wildcard": {
        "name": {
          "value": "cl*n"
        }
      }
   }
}
  
```
## multi_match查询
如果我们需要指定多字段，可以使用`multi_match` ，来指定一个`fields`属性用来要搜索的字段数组。

```bash
GET /movies/_search?pretty
{
  "query": {
        "multi_match" : {
            "query" : "监狱风云",
            "fields" : ["title"]
        }
    }
}
```
我们可以看到上面的请求正文，指定了一个查询，并且使用`multi_match`进行多个匹配。然后通过`query` 指定查询内容，并通过`fields`限制查询的字段。

字段也可以通过通配符指定：
```bash
GET /movies/_search?pretty
{
  "query": {
        "multi_match" : {
            "query" : "监狱风云",
            "fields" : ["tit*"]
        }
    }
}
```
## filter过滤
filter 类似于 SQL 里面的where 语句，和上面的基础查询比起来，也能实现搜索的功能，同时 filter不计算相关性，并且可以将查询缓存到内存当中,因此，filter速度要快于query。tan
```bash
 "filter": {
                   Filter to apply to the query
     }
```

### 指定条件过滤
这时候我们就可以使用到过滤， 因为我知道监狱风云2是在1991年上映的，所以我使用`year`过滤出1991年的监狱风云
```bash
POST /_search
{
    "query": {
        "bool": {
            "must": {
                "query_string": {
                    "query": "监狱风云"
                }
            },
            "filter": {
                "term": { "year": 1991 }
            }
        }
    }
}

```
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1558545057353.png)

###   无查询条件过滤

在上面的示例中，使用过滤器限制查询字符串查询的结果。如果我们想只是单纯的过滤呢？也就是说，我们希望所有电影符合一定的标准。
在这种情况下，我们仍然在搜索请求正文中使用`query`属性。但是，我们不能只是添加一个过滤器，需要将它包装在某种查询中。

一个解决方案是修改当前的搜索请求，替换查询字符串 query 过滤查询中的`match_all`查询，这是一个查询，匹配一切内容。

比如说我想过滤出所有电影中上映时间是1994年的电影：

```bash
POST /_search
{
    "query": {
        "bool": {
            "must": {
                "match_all": {
                }
            },
            "filter": {
                "term": { "year": 1994 }
            }
        }
    }
}
```
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1558506955605.png)

###  过滤非空内容
```bash
GET /_search
{
  "query": {
    "bool": {
      "filter": {
        "exists": {
          "field": "title"
        }
      }
    }
  }
}
```

### 过滤器缓存

ElasticSearch提供了一种特殊的缓存，即过滤器缓存（filter cache），用来存储过滤器的结果，被缓存的过滤器并不需要消耗过多的内存（因为它们只存储了哪些文档能与过滤器相匹配的相关信息），而且可供后续所有与之相关的查询重复使用，从而极大地提高了查询性能。

注意：ElasticSearch并不是默认缓存所有过滤器，以下过滤器默认不缓存：
```
    numeric_range
    script
    geo_bbox
    geo_distance
    geo_distance_range
    geo_polygon
    geo_shape
    and
    or
    not
```
exists,missing,range,term,terms默认是开启缓存的

开启方式：在filter查询语句后边加上 `"_catch":true`

## constant_score
上面的查询我们有一个更简单的方法是使用常数分数查询，该查询能够包含一个查询或者一个过滤器，所有匹配文档的相关度分值都为1，不考虑TF/IDF：
```bash
curl -H "Content-Type: application/json"  -XPOST "http://localhost:9200/_search" -d'
{
    "query": {
        "constant_score": {
            "filter": {
                "term": { "year": 1994 }
            }
        }
    }
}'
```
## range查询
范围查询主要针对数值和日期类型，range 可以使用比较关键词，也可以使用自带参数。

比较关键词：
- gte − 大于和等于
- gt − 大于
- lte − 小于和等于
- lt − 小于

参数：
- from - 从什么时间或者数字开始，等同 gte
- to     - 到什么时间或者数字结束, 等同 lte
- include_lower - 是否包含范围的左边界，默认是true , 等同gt
- include_upper  - 是否包含范围的右边界，默认是true,等同 lt

我们先来查询下数值范围
```bash
POST movies/movie/_search
{
  "query": {
    "range": {
      "year": {
        "gte": 1990,
        "lte": 2000
      }
    }
  }
}
```

当然也可以使用:
```bash
POST movies/movie/_search
{
  "query": {
    "range": {
      "year": {
        "from": 1990,
        "to": 2000
      }
    }
  }
}
```

因为我们之前没有含日期的索引，所以我们现在来创建几个。
```bash
PUT  students/doc/1
{
  "name" :"Jay",
   "birth" :"1995-06-07"
}

PUT  students/doc/2
{
  "name" :"Arlis",
   "birth" :"1990-02-02"
}

PUT  students/doc/3
{
  "name" :"Nike",
   "birth" :"1980-02-02"
}
PUT  students/doc/4
{
  "name" :"Yves",
   "birth" :"2004-05-06"
}
```
好了现在我们已经有数据，开始查询：
```bash
POST students/doc/_search
{
  "query": {
    "range": {
      "birth": {
        "gte": "2000-01-01"
      }
    }
  }
} 
```

日期提供一种更友好的计算方式

```bash
POST students/doc/_search
{
  "query": {
    "range": {
      "birth": {
        "gte": "now-20y"
      }
    }
  }
}
```
`now` 代表当前系统时间， `20y` 代表20年 ， `now-20y` 即在现在的时间基础上减少20年

日期计算的操作符主要有三个 `+`  `-`  `/d`,例如：
- +1h 加一小时
- -1h 减一小时
-  /d 将时间舍入到天

## match_phrase查询
目前我们可以在字段中搜索单独的一个词，但是有时候你想要确切的匹配若干个单词或者短语(phrases). 我们只需要把`match`变为`match_phrase`即可。
`match_phrase`和`match`的区别，`match_phrase`首先分析查询字符串，从分析后的文本中构建短语查询，这意味着必须匹配短语中的所有分词，并且保证各个分词的相对位置不变,通俗的讲就是返回的结果，跟搜索内容的分词的顺序是一样的。

例如：
```bash
GET movies/movie/_search
{
  "query": {
    "match_phrase": {
      "title": "罗马假日"
    }
  }
}
```
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559130562833.png)

上面我们使用了`match_phrase`搜索了 `罗马假日`,能够得到索引信息。

下面我们搜索`假日罗马` 
```bash
GET movies/movie/_search
{
  "query": {
    "match_phrase": {
      "title": "假日罗马"
    }
  }
}
```
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559130592142.png)
我们发现没有得到任何结果。

## match_phrase_prefix 查询

我们还可以使用`match_phrase_prefix` 来查询前缀, 为什么是`match_phrase_prefix` 而不是`match_prefix`其实也很好理解，因为前缀是需要顺序的，必须在前面。
```bash
GET movies/movie/_search
{
  "query": {
    "match_phrase_prefix": {
      "title": "霸王"
    }
  }
}
```
## sort排序
当搜索的字段有多个时，可以对指定字段进行排序。注意的是文本内容不能进行排序。

我们使用`sort` 指定字段进行排序， 可以使用`order` 指定排序方法。

order的值可以为：
- asc 进行升序排序
- desc 进行降序排序

下面我们对`year`进行升序排序:
```bash
GET /movies/_search?pretty
{
   "query": {
        "match_all":{
        }
    },
  "sort": [
    {"year": {"order": "asc"}}
  ]
}
```
对`year`进行降序排序:

```bash
GET /movies/_search?pretty
{
   "query": {
        "match_all":{
        }
    },
  "sort": [
    {"year": {"order": "desc"}}
  ]
}
```
当一个字段的内容有多个值的时候，系统支持一些计算进行排序，包括min、max、sum、avg、median (中间值)。

mode  | 描述
--|--
min|选择最低值。
max|选择最高值。
sum|使用所有值的总和作为排序值。仅适用于基于数字的数组字段。
avg|使用所有值的平均值作为排序值。仅适用于基于数字的数组字段。
median|使用所有值的中位数作为排序值。仅适用于基于数字的数组字段。

我们来演示下用法：
```bash
GET /movies/_search?pretty
{
   "query": {
        "match_all":{
        }
    },
  "sort": [
    {"year": {"order": "asc","mode":"min"}}
  ]
}
```
## from size分页
实现分页搜索只需要指定`from`和`size`字段即可，例如：
```
GET /movies/_search?pretty
{
  "from": 0,
  "size": 15, 
   "query": {
        "match_all":{
        }
    }
}
```

> 注意：ES的from、size分页不是真正的分页，称之为浅分页。from+ size不能超过index.max_result_window 默认为10,000 的索引设置

## scroll滚动
我们在前面说过`from、size`分页是浅分页，而且最多只能显示10000个数据，如果一次性要查出来比如10万条数据，浅分页显然满足不了我们的要求。此时一般会采取用`scoll`滚动查询，一批一批的查，直到所有数据都查询完为止。

`scoll`搜索会在第一次搜索的时候，保存一个当时的视图快照，后续的对文档的改动（索引、更新或者删除）都只会影响后面的搜索请求。`scoll`采用基于`_doc`(不使用_score)进行排序的方式，性能较高.

为了提现查询的效果我们添加点数据。
```bash
wget https:   raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json

curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/account/_bulk" --data-binary "@accounts.json"
```
为了使用 scroll，初始搜索请求应该在查询中指定` scroll` 参数，这可以告诉 Elasticsearch 需要保持搜索的上下文环境多久，如 `scroll=1m` ,1m是一分钟的意思，如果数据很大，那么建议等待时间长一点。
```bash
GET /bank/account/_search?scroll=5m
{
  "query": {
    "range": {
      "age": {
        "gte": 20,
        "lte": 40
      }
    }
  }
}
```
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559199125272.png)

使用上面的请求返回的结果中包含一个`scroll_id`，这个 ID 可以被传递给 scroll API 来检索下一个批次的结果。

我们用这个`scroll_id`来查询
```bash
GET /_search/scroll
{
    "scroll" : "1m", 
    "scroll_id" : "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAE5FmcxbnNvb0ZKU0Fpb3BMRUFmeEJuaEEAAAAAAAABNxZnMW5zb29GSlNBaW9wTEVBZnhCbmhBAAAAAAAAAToWZzFuc29vRkpTQWlvcExFQWZ4Qm5oQQAAAAAAAAE7FmcxbnNvb0ZKU0Fpb3BMRUFmeEJuaEEAAAAAAAABOBZnMW5zb29GSlNBaW9wTEVBZnhCbmhB" 
}
```
当然这个响应消息还会返回一个`scroll_id`，如果我们需要不断的往下查找就需要不断的查询这些`scroll_id`。

## highlight高亮搜索
很多应用喜欢从每个搜索结果中高亮(highlight)匹配到的关键字，这样用户可以知道为什么这些文档和查询相匹配。在Elasticsearch中高亮片段是非常容易的，我们只需要添加`highlight`关键字。

请求信息:
```bash
GET students/doc/_search
{
    "query" : {
        "match" : {
            "title" : "卢旺达饭店"
        }
    },
    "highlight": {
        "fields" : {
            "title" : {}
        }
    }
}
```
响应信息：
```json
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 4.9041457,
    "hits": [
      {
        "_index": "movies",
        "_type": "doc",
        "_id": "11",
        "_score": 4.9041457,
        "_source": {
          "title": "卢旺达饭店",
          "director": "特瑞·乔治",
          "year": 2004,
          "genres": [
            "战争",
            "剧情",
            "历史"
          ],
          "actors": [
            "唐·钱德尔",
            "苏菲·奥康内多",
            "杰昆·菲尼克斯"
          ]
        },
        "highlight": {
          "title": [
            "<em>卢</em><em>旺</em><em>达</em><em>饭</em><em>店</em>"
          ]
        }
      }
    ]
  }
}
```
响应的内容与之前结果相同，但是在返回结果中会有一个新的部分叫做`highlight`，这里包含了来自`titile`字段中的文本，并且用`<em></em>`来标识匹配到的内容。

## simple_query_string 查询
`Simple_Query_String`类似`Query_String `,但是会忽略错误的查询语法,并且仅支持部分查询语法。

来看下面的例子 ，我们使用`|`查询包含罗马或者卢旺达的信息，但是符号附近有一些垃圾字符。
```bash
GET  movies/movie/_search
{
  "query": {
    "simple_query_string": {
      "query": "罗马 / | 、卢旺达",
      "fields": ["title"]
    }
  }
}
```
我们成功执行了命令。
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559132117755.png)


现在我们把`simple_query_string` 换成`query_string` ,在来试试。
```bash
GET  movies/movie/_search
{
  "query": {
    "query_string": {
      "query": "罗马 / | 、卢旺达",
      "fields": ["title"]
    }
  }
}
```
我们执行命令，发现报错
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559131952364.png)

## bool查询
bool 查询允许我们使用布尔逻辑将小的查询组成大的查询。

### AND查询
比如说我们需要同时匹配到 title 含有 `监狱` 和 `风云` 的索引信息。
请求内容：
```bash
GET /movies/_search
{
		"query":{
			"bool":
			{"must":[
				{"match":{"title":"监狱"}},
				{"match":{"title":"风云"}}
				]   
			}   
		}
	}
```
	在上述示例中，`bool` `must` 子句指定了所有匹配文档必须满足的条件。

-  **OR查询**
	相比之下，`bool` `should` 的组合，两个match查询并且返回所有`title`属性中包含 `监狱` 或 `西游` 的任意信息。

	请求内容：
	```bash
	GET /movies/_search
	{
		"query":{
			"bool":
			{"should":[
				{"match":{"title":"监狱"}},
				{"match":{"title":"西游"}}
				]   
			}   
		}
	}
	```
	在上述例子中`bool` `should` 子句指定了匹配文档只要满足其中的任何一个条件即可匹配。

### AND取反查询

	我们还可以使用 `bool` 和`must_not`, 查询属性中不包含匹配内容的文档。
	请求内容：
	```bash
	GET /movies/_search
	{
		"query":{
			"bool":
			{"must_not":[
				{"match":{"title":"监狱"}},
				{"match":{"title":"西游"}}
				]   
			}   
		}
	}
	```
	在上述例子中，`bool` `must_not`子句指定了其中的任何一个条件都不满足时即可匹配


### 布尔组合查询
我们可以组合 must 、should 、must_not 进行复杂的查询。

- A AND NOT B

```bash
GET /movies/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "监狱风云" } }
      ],
      "must_not": [
        { "match": { "year": 1991 } }
      ]
    }
  }
}
```

- A AND (B OR C)

```bash
GET /movies/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match_all": {}
        },
        {
          "bool": {
            "should": [
              {
                "match": {"genres": "同性"}
              },
              {"match": {"genres": "科幻"}
              }
            ]}
        }
      ]
    }
  }
}
```
## aggs聚合查询
lasticsearch有一个功能叫做聚合(aggregations)，它允许你在数据上生成复杂的分析统计。它很像SQL中的GROUP BY但是功能更强大。

### 准备数据
我们先插入一些数据:
```bash
POST /_bulk
{"index":{"_index":"goods","_type":"doc","_id":1}}
{"gname":"薯片","price":5.00,"classes":"零食","num":100}
{"index":{"_index":"goods","_type":"doc","_id":2}}
{"gname":"方便面","price":3.20,"classes":"零食","num":200}
{"index":{"_index":"goods","_type":"doc","_id":3}}
{"gname":"毛巾" ,"price":13.50, "classes":"日用品","num":60}
{"index":{"_index":"goods","_type":"doc","_id":4}}
{"gname":"面包" ,"price":4.50,"classes":"零食", "num":80}
{"index":{"_index":"goods","_type":"doc","_id":5}}
{"gname":"牙刷" ,"price":4.50, "classes":"日用品","num":100}
{"index":{"_index":"goods","_type":"doc","_id":6}}
{"gname":"可乐" ,"price":3, "classes":"零食","num":100}

```
### Max求最大值
现在我们来查询刚刚到商品中价格最高的：
```bash
GET goods/doc/_search
{
 "aggs": {
   "price_of_max": {
     "max": {
       "field": "price"
     }
   }
 }
}
```
可以在`aggregations`中得到查询的结果

```json
{
 ...
  "aggregations": {
    "maxprice": {
      "value": 13.5
    }
  }
```
### Sum求和
现在我们来统计所有商品的单价和：
如果我们想要屏蔽查询索引具体内容，可以使用`size` ，把查询数量设为0
```
GET goods/doc/_search
{
 "size" :0,
  "aggs": {
     "price_of_sum": {
         "sum": {
           "field": "price"
         }
     }
  }
}
```
响应信息：
```json
{
 ...
  "aggregations": {
    "price_of_sum": {
      "value": 33.700000047683716
    }
  }
}
```
可以看到和为 33.7

### avg求平均值
下面我们计算商品的平均价格
```bash
GET goods/doc/_search
{
  "size":0,
  "aggs": {
     "price_of_avg": {
         "avg": {
           "field": "price"
         }
     }
  }
}
```
响应信息：
```json
{
 ...
  "aggregations": {
    "price_of_cardi": {
      "value": 5.616666674613953
    }
  }
}
```

### cardinality求基数
```bash
GET goods/doc/_search
{
  "size":0,
  "aggs": {
     "price_of_cardi": {
         "cardinality": {
           "field": "price"
         }
     }
  }
}
```
### terms分组
我们来通过商品类别分组并统计商品数量：
```bash
GET goods/doc/_search
{
  "size":0,
  "aggs": {
     "price_group_by": {
         "terms": {
           "field": "classes.keyword"
         }
     }
  }
}
```
响应信息：
```json
...
...
{
  "aggregations": {
    "price_group_by": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "零食",
          "doc_count": 4
        },
        {
          "key": "日用品",
          "doc_count": 2
        }
      ]
    }
  }
}
```
我们可以看到零食的数量为4 ，日用品数量为2
## Source过滤

数据列过滤允许在查询的时候不显示原始数据，或者显示部分原始字段。

### 不显示原始文档
```bash
GET /_search
{
    "_source": false,
    "query" : {
        "term": {"year": {"value": 1994}}
    }
}
```
### 显示部分文档列
比如说我们只想显示` movies` 中`title`和`actors`信息。 
```bash
GET /_search
{
    "_source": "actors",
    "query" : {
        "term": {"year": {"value": 1994}}
    }
}
```
### 包含或者排除某些列
我们还可以使用 `includes` 包含某字段，或者使用`excludes`排除某字段。

例如： 
```bash
GET /_search
{
    "_source": {
        "includes": [ "title", "director" ],
        "excludes": [ "year" ]
    },
    "query" : {
      "term": {"year": {"value": 1994}}
    }
}
```
## 相关性评分	
我们曾经讲过，默认情况下，返回结果是按相关性倒序排列的。 但是什么是相关性？ 相关性如何计算？

每个文档都有相关性评分，用一个相对的浮点数字段` _score` 来表示 ` _score `的评分越高，相关性越高。查询语句会为每个文档添加一个 `_score` 字段,评分的计算方式取决于不同的查询类型。

不同的查询语句用于不同的目的：
- fuzzy 查询会计算与关键词的拼写相似程度
- terms查询会计算 找到的内容与关键词组成部分匹配的百分比
 
但是一般意义上我们说的全文本搜索是指计算内容与关键词的类似程度。
ElasticSearch的相似度算法被定义为 TF/IDF，即检索词频率/反向文档频率，包括一下内容：
- 检索词频率::
	检索词在该字段出现的频率，出现频率越高，相关性也越高。 字段中出现过5次要比只出现过1次的相关性高。
- 反向文档频率::
	每个检索词在索引中出现的频率，频率越高，相关性越低。 检索词出现在多数文档中会比出现在少数文档中的权重更低， 即检验一个检索词在文档中的普遍重要性。
- 字段长度准则::
	字段的长度是多少，长度越长，相关性越低。 检索词出现在一个短的 title 要比同样的词出现在一个长的 content 字段。
	
单个查询可以使用TF/IDF评分标准或其他方式，比如短语查询中检索词的距离或模糊查询里的检索词相似度。

相关性并不只是全文本检索的专属。也适用于yes|no的子句，匹配的子句越多，相关性评分越高。

如果多条查询子句被合并为一条复合查询语句，比如 bool 查询，则每个查询子句计算得出的评分会被合并到总的相关性评分中。

ElasticSearch提供了一个explain参数，将 explain 设为 true 就可以得到 `_score`详细的信息。
```bash
GET /movies/_search?explain=true
{
  "query": {
    "match": {
      "title": "龙猫"
    }
  }
}
```

具体的响应消息因为文本太多，自己执行命令查询 ，我们这里只分析几个重要信息

```
 "_shard": "[movies][3]"
 "_node": "g1nsooFJSAiopLEAfxBnhA",
```
  这里加入了该文档来自于哪个节点哪个分片上的信息，这对我们是比较有帮助的，因为词频率和 文档频率是在每个分片中计算出来的，而不是每个索引中。
  

`_explanation`会包含在每一个入口，告诉你采用了哪种计算方式，并让你知道计算的结果以及其他详情:
 
 ```
 "description":"score(doc=3,freq=1.0 = termFreq=1.0)), product of:"
```
  检索词的频率
 
 ```
 "description":"idf, computed as log(1 + (docCount - docFreq + 0.5) / (docFreq + 0.5)) from:"
 ```
 反向文档频率
 
 ```
  "description":"docCount",
 ```
 字符长度准则

## 加速检索建议
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/20180826132831209.png)
