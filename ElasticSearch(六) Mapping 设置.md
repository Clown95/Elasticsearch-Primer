---

title: ElasticSearch(六) Mapping 设置
tags: ElasticSearch
author: Clown95

---

# Mapping 设置
Mapping类似数据库中的表结构定义,主要作用如下:
- 定义Index下的字段名( Field Name )
- 定义字段的类型,比如数值型、字符串型、布尔型等
- 定义倒排索引|相关的配置,比如是否索引、记录position等

## 查看Mapping
我们可以通过`_mapping`端点来查看索引的mapping
```bash
GET /movies/_mapping
```
得到的响应信息：
```json
{
	"movies": {
		"mappings": {
			"movie": {
				"properties": {
					"actor": {
						"type": "text",
						"fields": {"keyword": {"type": "keyword","ignore_above": 256}}
					},
					"actors": {
						"type": "text",
						"fields": {"keyword": {"type": "keyword","ignore_above": 256}}
					},
					"director": {
						"type": "text",
						"fields": {"keyword": {"type": "keyword","ignore_above": 256}}
					},
					"genres": {
						"type": "text",
						"fields": {"keyword": {"type": "keyword","ignore_above": 256}}
					},
					"title": {
						"type": "text",
						"fields": {"keyword": {"type": "keyword","ignore_above": 256}}
					},
					"year": {"type": "long"}
				}
			}
		}
	}
}
```
我们可以从返回json中可以看到 `movies` 这是我们的索引名， `movie`这是类型，
`properties`表示字段.我们有   `director`、`title`、 `year`和`genres`  字段。

## 自定义Mapping
自定义Mpping 是使用`PUT`请求，请求正文跟我们`GET`的内容一致。
```bash
PUT test_index
{
  "mappings": {
    "doc": {
      "properties": {
        "title": {
          "type": "text"
        },
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        }
      }
    }
  }
}
```
自定义Mapping需要注意几点：

- Mapping中的字段类型一旦设定后,禁止直接修改,因为Lucene实现的倒排索引生成后不允许修改。因为lucene已经建立好索引，为了效率它不允许修改。

- 如果更改，需要重新建立新的索引,然后做reindex操作。
	reindex操作步骤：

	1. 备份索引
	```bash
	POST _reindex
	{
	  "source": {"index":"旧索引"},
	  "dest": {"index":"备份索引"}
	}
	```
	2. 删除旧索引
	3. 新索引建立mapping
	4. 还原索引
	```bash
	POST _reindex
	{
	  "source": {"index":"备份索引"},
	  "dest": {"index":"新索引"}
	}
	```
	
- 虽然不允许我们修改字段，但是允许我们新增字段，可以通过`dynamic`参数来控制字段的添加。
	- true (默认)允许自动新增字段
	- false不允许自动新增字段,但是文档可以正常写入,但无法对字段进行查询等操作
	- strict文档不能写入,报错
	
	具体用法：
	```bash
	PUT test_index1
	{
	  "mappings": {
		"doc": {
		  "dynamic": false,
		  "properties": {
			"user": {
			  "properties": {
				"name": {
				  "type": "text"
				},
				"social_networks": {
				  "dynamic": true,
				  "properties": {}
				}
			  }
			}
		  }
		}
	  }
	}
	```
	定义后test_index1这个索引下不能自动新增字段，但是在user.social_networks下可以自动新增子字段.
		
## copy_to
- 将该字段复制到目标字段，实现类似_all的作用
- 不会出现在_source中，只用来搜索

我们来演示一下 :

```bash
PUT my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "first_name": {
          "type": "text",
          "copy_to": "full_name" 
        },
        "last_name": {
          "type": "text",
          "copy_to": "full_name" 
        },
        "full_name": {
          "type": "text"
        }
      }
    }
  }
}
```
给上面的索引插入信息：
```bash
PUT my_index/doc/1
{
  "first_name": "John",
  "last_name": "Smith"
}
```

使用`full_name` 进行搜索：
```bash
GET my_index/_search
{
  "query": {
    "match": {
      "full_name": { 
        "query": "John Smith",
        "operator": "and"
      }
    }
  }
}
```
响应内容：
```json
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "my_index",
        "_type": "doc",
        "_id": "1",
        "_score": 0.5753642,
        "_source": {
          "first_name": "John",
          "last_name": "Smith"
        }
      }
    ]
  }
}
```
## index
控制当前字段是否索引，默认为true，即记录索引，false不记录，即不可搜索。

```bash
PUT my_index1
{
  "mappings": {
    "doc": {
      "properties": {
        "cookie": {
          "type": "text",
          "index": "false" 
        }
      }
    }
  }
}
```
我们来插入下数据：
```bash
PUT my_index1/doc/1
{
  "cookie" :"user=123&pwd=123"
}
```

我们来搜索下看会出现什么结果：
```bash
GET my_index1/_search
{
  "query": {
    "match": {
       "cookie": "123"
    }
  }
}
```
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559114623317.png)
通过响应信息我们发现不能搜索。
## index_options
index_options参数控制将哪些信息添加到倒排索引，以用于搜索和突出显示，可选的值有：-docs，freqs，positions，offsets。
- docs：只索引 doc id
- freqs：索引 doc id 和词频，平分时可能要用到词频
- positions：索引 doc id、词频、位置，做 proximity or phrase queries 时可能要用到位置信息
- offsets：索引doc id、词频、位置、开始偏移和结束偏移，高亮功能需要用到offsets

使用方法跟前面几个都差不多：
```bash
PUT my_index2
{
  "mappings": {
    "doc": {
      "properties": {
        "cookie": {
          "type": "text",
          "index_options": "offsets" 
        }
      }
    }
  }
}
```
## null_value
当字段遇到null值时的处理策略,默认为null ,即空值,此时es会忽略该值。可以通过设定该值设定字段的默认值
```bash
PUT my_index3
{
  "mappings": {
    "doc": {
      "properties": {
        "cookie": {
          "type": "text",
          "null_value": "NULL" 
        }
      }
    }
  }
}
```
## 数据类型
1. 核心数据类型（Core datatypes）

	- 字符型：string，string类型包括text 和 keyword    
	- 数字型：long, integer, short, byte, double, float 
	- 日期型：date
	- 布尔型：boolean
	- 二进制型：binary
	- 范围类型integer_ range、float _range、long. _range、double_ range、date_ range

		text类型被用来索引长文本，在建立索引前会将这些文本进行分词，转化为词的组合，建立索引。允许es来检索这些词语。text类型不能用来排序和聚合。

		 Keyword类型不需要进行分词，可以被用来检索过滤、排序和聚合。keyword 类型字段只能用本身来进行检索

2. 复杂数据类型（Complex datatypes）

   -  数组类型（Array datatype）：数组类型不需要专门指定数组元素的type

   - 对象类型（Object datatype）：_ object _ 用于单个JSON对象
  
   - 嵌套类型（Nested datatype）：_ nested _ 用于JSON数组；
	
3. 地理位置类型（Geo datatypes）

    - 地理坐标类型（Geo-point datatype）：_ geo_point _ 用于经纬度坐标；
    
    - 地理形状类型（Geo-Shape datatype）：_ geo_shape _ 用于类似于多边形的复杂形状；

4. 特定类型（Specialised datatypes）

   - IPv4 类型（IPv4 datatype）：_ ip _ 用于IPv4 地址；
   
   - Completion 类型（Completion datatype）：_ completion _提供自动补全建议；
   
   - Token count 类型（Token count datatype）：_ token_count _ 用于统计做了标记的字段的index数目，该值会一直增加，不会因为过滤条件而减少。
   
   - mapper-murmur3c类型：通过插件，可以通过 _ murmur3 _ 来计算 index 的 hash 值；
   
   - 附加类型（Attachment datatype）：采用 mapper-attachments插件，可支持_ attachments _ 索引，例如 Microsoft Office 格式，Open Document 格式，ePub, HTML 等。

## Mapping支持属性

- "store":false   是否单独设置此字段的是否存储而从_source字段中分离，默认是false，只能搜索，不能获取值

- "index": true   分词，不分词是：false，设置成false，字段将不会被索引
   
- "analyzer":"standard "   指定分词器,默认分词器为standard analyzer

- "boost":1.23   字段级别的分数加权，默认值是1.0

- "doc_values":false   对not_analyzed字段，默认都是开启，分词字段不能使用，对排序和聚合能提升较大性能，节约内存

- "fielddata":{"format":"disabled"}   针对分词字段，参与排序或聚合时能提高性能，不分词字段统一建议使用doc_value

- "fields":{"raw":{"type":"string","index":"not_analyzed"}}    可以对一个字段提供多种索引模式，同一个字段的值，一个分词，一个不分词
            
- "ignore_above":100    超过100个字符的文本，将会被忽略，不被索引

- "include_in_all":ture   设置是否此字段包含在_all字段中，默认是true，除非index设置成no选项

- "index_options":"docs"   4个可选参数docs（索引文档号） ,freqs（文档号+词频），positions（文档号+词频+位置，通常用来距离查询），offsets（文档号+词频+位置+偏移量，通常被使用在高亮字段）分词字段默认是position，其他的默认是docs

- "norms":{"enable":true,"loading":"lazy"}   分词字段默认配置，不分词字段：默认{"enable":false}，存储长度因子和索引时boost，建议对需要参与评分字段使用 ，会额外增加内存消耗量

- "null_value":"NULL"   设置一些缺失字段的初始化值，只有string可以使用，分词字段的null值也会被分词

- "position_increament_gap":0   影响距离查询或近似查询，可以设置在多值字段的数据上火分词字段上，查询时可指定slop间隔，默认值是100

- "search_analyzer":"ik"   设置搜索时的分词器，默认跟ananlyzer是一致的，比如index时用standard+ngram，搜索时用standard用来完成自动提示功能

- "similarity":"BM25"   默认是TF/IDF算法，指定一个字段评分策略，仅仅对字符串型和分词类型有效

- "term_vector":"no"   默认不存储向量信息，支持参数yes（term存储），with_positions（term+位置）,with_offsets（term+偏移量），with_positions_offsets(term+位置+偏移量) 对快速高亮fast vector highlighter能提升性能，但开启又会加大索引体积，不适合大数据量用

## 多字段特性
允许对同一个字段采用不同的配置,比如分词,常见例子如对人名实现拼音搜索，只需要在人名中新增一个子字段为pinyin即可,当然这样需要插件支持，能够支持汉字转洴音。

```bash
PUT my_index4
{
  "mappings": {
    "doc": {
      "properties": {
        "username": {
          "type": "text",
          "fields": {
            "pingyin":{
              "type": "text",
              "analyzer": "pingyin"
            }
          }
        }
      }
    }
  }
}

```

## Dynamic Mapping


创建索引的时候,可以预先定义字段的类型以及相关属性，这样就能够把日期字段处理成日期，把数字字段处理成数字，把字符串字段处理字符串值等。

ES是依靠JSON文档的字段类型来实现自动识别字段类型，支持的类型如下：

JSON 类型|ES 类型
--|--
null|忽略
boolean|boolean
浮点类型|float
整数|long
object|object
array|由第一个非 null 值的类型决定
string|匹配为日期则设为date类型（默认开启）；匹配为数字则设置为 float或long类型（默认关闭）；设为text类型，并附带keyword的子字段


## 日期的自动识别

日期的自动识别可以自行配置日期格式,以满足各种需求

-默认是["strict_ date_ optional time"yyy/MM/dd HH:mm:ss llyyyy/MM/dd Z"]
- strict_ date_ optional _time是ISO datetime的格式,完整格式类似下面:
	- YYYY-MM-DDThh:mm:ssTZD (eg 1997-07-16T19:20:30+01:00）
	
- dynamic_ _date_ formats可以自定义日期类型

- date_ detection可以关闭日期自动识别的机制 ，例如我只想把时间当成字符输出。

自定义日期识别格式
```bash
PUT my_index5
{
  "mappings": {
    "doc": {
      "dynamic_date_formats": ["MM/dd/yyyy"]
    }
  }
}
```
关闭日期自动识别机制
```bash
PUT my_index5
{
  "mappings": {
    "doc": {
      "date_detection": false
    }
  }
}
```
## 数字的自动识别
- 字符串是数字时，默认不会自动识别为整形，因为字符串中出现数字完全是合理的
	- numeric_detection 参数可以开启字符串中数字的自动识别

```bash
PUT my_index6
{
  "mappings": {
    "doc": {
      "numeric_detection": true
    }
  }
}
```

## Dynamic templates
允许根据ES自动识别的数据类型、字段名等来动态设定字段类型，可以实现如下效果：

- 所有字符串类型都设定为keyword类型,即默认不分词
- 所有以message开头的字段都设定为text类型,即分词
- 所有以long_开头的字段都设定为long类型
- 所有自动匹配为double类型的都设定为float 类型,以节省空间

## Dynamic templates API

API如下：
```
"dynamic_templates": [
        {
          "my_template_name": {
            "match_mapping_type": "double",
            "mapping": {
              "type": "float"
            }
          }
        }
      ]
````
`dynamic_templates` 是一个数组，可以指定多个匹配规则
`my_template_name`  是我们的模板名称
`match_mapping_type`  是指定匹配规则


匹配规则一般有如下几个参数：

- match_mapping_type 匹配ES自动识别的字段类型，如boolean，long，string等
- match, unmatch 匹配字段名
- match_pattern 匹配正则表达式
- path_match, path_unmatch 匹配路径

下面我们来简单的演示几个匹配规则：


- 设置 double类型设置为 float类型 节省空间
	```bash
	PUT my_index7
	{
	  "mappings": {
		"doc": {
		  "dynamic_templates": [
			{
			  "double_as_float": {
				"match_mapping_type": "double",
				"mapping": {
				  "type": "float"
				}
			  }
			}
		  ]
		}
	  }
	}
	```

- 以message开头的都设置为text类型
	```bash
	PUT my_index8
	{
	  "mappings": {
		"doc": {
		  "dynamic_templates": [
			{
			  "message_as_text": {
				"match_mapping_type": "string",
				"match":"message*",
				"mapping": {
				  "type": "text"
				}
			  }
			}
		  ]
		}
	  }
	}
	```

- 字符串默认使用keyword类型
	```bash
	PUT my_index9
		{
		  "mappings": {
			"doc": {
			  "dynamic_templates": [
				{
				  "strings_as_keyword": {
					"match_mapping_type": "string",
					"mapping": {
					  "type": "keyword"
					}
				  }
				}
			  ]
			}
		  }
		}
	```
	
## 索引模板
索引模板，主要用于在新建索引时自动应用预先设定的配置，简化索引创建的操作步骤。
- 可以设定索引的setting和mapping
- 可以有多个模板，根据order设置，order大的覆盖小的配置
- 索引模板API，endpoint（端点）为 _template

索引模板API,如下所示：
```bash
PUT _template/template_1
{
  "index_patterns": ["test*"],
  "order": 2,
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "doc": {
      "_source": {
        "enabled": false
      },
      "properties": {
        "name": {
          "type": "keyword"
        }
      }
    }
  }
}
```
`index_patterns` 是匹配索引的名称
`order` 是顺序配置
`settings` 是索引的配置

上面我们创建索引模板，匹配 `test` 开头的索引.

现在我们来添加几个索引

```bash
PUT  test123
PUT  test_index123
```
接着我们查找这些索引的Mapping

```bash
GET test123
GET test_index123
```

![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559119782265.png)

通过响应结果我们发现，这些索引自动添加了 name的 字段， 但是这个字段我们再创建索引的时候并未指定， 这就是索引模板为我们自动匹配到的。

