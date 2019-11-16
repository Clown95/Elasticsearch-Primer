---

title: ElasticSearch(五) 倒排索引与分词
tags: ElasticSearch
author: Clown95

---

# 倒排索引与分词
## 倒排索引
Elasticsearch 使用一种称为`倒排索引`的结构，它适用于快速的全文搜索。一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。

为了更好的理解倒排索引，我们先说下《高性能MySql》这本书的目录和索引。
首先看下书的目录，它能够很方便的定位我们想要了解的知识点。

![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559063343392.png)

但是如果我们想要查询`auto-increment keys`的内容，目录上并没有出现这些具体的关键字，如果我们知道它所在的知识点也能快速的查询到，但是如果我们不知道它所在的知识点，那么我们只能一页一页的翻书来查找这个关键字，这样的效率无疑是很低。

但是如果我们有一个像下面这样的索引，我们就能很快的定位到具体的页码。

![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559063825751.png)

书与搜索引擎类似，我们可以抽象为：
- 目录页对应正排索引
- 索引页对应倒排索引

## 正排与倒排索引介绍
`正排索引`，是通过文档ID到文档内容、单词（文档的分词）

![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559064171953.png)

`倒排索引`，是通过单词获得文档ID，这些单词就是文档内容进行的分词。

 ![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559064267030.png)
 
 对比两张图，我们可以看到 `搜索引擎` 这个单词，在1，3文档，那么我们是如何查询包含`搜索引擎`的文档的？
 1. 通过倒排索引获得`搜索引擎`  对应的文档ID 有 1 和 3
 2. 通过正排索引查询 1 和 3 的完整内容
 3. 返回用户最终结果

## 倒排索引的组成
倒排索弓|是搜索引l擎的核心,主要包含两部分:
- 单词词典( Term Dictionary )
`单词词典`是倒排索引的重要组成部分， 他记录了所有文档的单词，所以他一般都比较大 。还会记录单词到倒排列表的关联信息。

- 倒排列表( Posting List )
`倒排列表` 记录了单词对应的文档集合,由倒排索引项( Posting )组成。
`倒排索引项`主要包含如下信息:
	- 文档Id ,用于获取原始信息
	- 单词频率( TF, Term Frequency) , 记录该单词在该文档中的出现次数,用于后续相关性算分
	- 位置( Position) ,记录单词在文档中的分词位置(多个) ,用于做词语搜索
	- 偏移( Offset ) ,记录单词在文档的开始和结束位置,用于做高亮显示

## 分词
分词是指将文本转换成一系列单词( term or token )的过程,也可以叫做文本分析。

比如说： `elasticsearch是最流行的搜索引擎`  这句话，我们可以分为`elasticsearch` 、`流行`、`搜索引擎`, 当然这个结果仅作为参考，具体的结果取决于你选用什么样的分词器。

分词器是es中专门处理分词的组件,英文为Analyzer ,它的组成如下:
- Character Filter -针对原始文本进行处理,比如去除html特殊标记符，特殊符号转换等
- Tokenizer - 将原始文本按照一定规则切分为单词
- Token Filters - 针对tokenizer处理的单词就行再加工,比如转小写、删除或新增等处理

## Analyze API
es提供了一个测试分词的api接口,方便验证分词效果, 断点是使用`_ analyze`
1. 可以直接指定analyzer进行测试

	```bash
	POST _analyze
	{
	  "analyzer": "standard",
	  "text": "hello world"
	}
	```
	analyzer 是我们选择的分词器 ，text 是我们要处理的内容。

	我们来看下响应信息
	```json
	{
	  "tokens": [
		{
		  "token": "hello",
		  "start_offset": 0,
		  "end_offset": 5,
		  "type": "<ALPHANUM>",
		  "position": 0
		},
		{
		  "token": "world",
		  "start_offset": 6,
		  "end_offset": 11,
		  "type": "<ALPHANUM>",
		  "position": 1
		}
	  ]
	}
	```
	- token - 分词结果
	- start_offset - 起始偏移
	- end_offset  - 结束偏移
	- position  - 分词位置

2. 可以直接指定索引中的字段进行测试

	这当我查询结果不是我们预期结果的时候，我们就可以使用它查看字段分词方式。
	```bash
	POST movies/_analyze
	{
	  "field": "title",
	  "text": "监狱风云"
	}
	```
	通过结果我们可以看到 es对中文分词默认是拆分为单个汉字，这就是我们搜索`剧情`的信息却能返回`爱情`的信息。

3.  可以自定义分词器进行测试

	我们还可以自定义分词器， 自己组合`Character Filter`、 `Tokenizer`和`Token Filters`

	```bash
	POST  _analyze
	{
	  "tokenizer": "standard",
	  "filter": ["lowercase"],
	  "text": "Hello Wordl"
	}
	```

- tokenizer将原始文本按照一定规则切分为单词( term or token ):
	- standard 按照单词进行分割
	- letter按照非字符类进行分割
	- whitespace按照空格进行分割
	- UAX URL Email按照standard分割,但不会分割邮箱和url
	- NGram和Edge NGram连词分割
	- Path Hierarchy 按照文件路径进行切割

- Token Filters 对于tokenizer输出的单词( term )进行增加、删除、修改等操作
	- lowercase将所有term转换为小写
	- stop删除stop words
	- NGram和Edge NGram连词分割
	- Synonym添加近义词的term

## 内置分词器的使用
es为我们提供了几个内置的分词器，下面我们列出了最重要的几个分析器：
- standard 分词器：(默认的)他会将词汇单元转换成小写形式，并去除停用词和标点符号，支持中文采用的方法为单字切分

```bash
POST _analyze
{
  "analyzer": "standard",
  "text": "I Love You Chian! 我爱你中国！"
}
```

- simple 分词器：首先会通过非字母字符来分割文本信息，然后将词汇单元统一为小写形式。该分析器会去掉数字类型的字符。

```bash
POST _analyze
{
  "analyzer": "simple",
  "text": "I123love456golang"
}
```
- Whitespace 分词器：仅仅是去除空格，对字符没有lowcase化,不支持中文；并且不对生成的词汇单元进行其他的标准化处理。

```bash
POST _analyze
{
  "analyzer": "whitespace",
  "text": "Chian is Num one"
}
```
- Pattern 分词器 ： 通过正则表达式自定义分隔符,默认的规则是`\W+`，即非字词的符号作为分隔符。
```bash
POST _analyze
{
  "analyzer": "pattern",
  "text": "good-good^study*day*day%up"
} 
```
- language 分词器：特定语言的分词器，大概有30多种，例如：arabic, armenian, basque, bengali, brazilian, bulgarian,catalan, cjk, czech, danish, dutch, english等，但是不支持中文。

## 中文分词

英文的分词比较好分主要是以空格为分割标准，但是中文就不一样了，中国的语言博大精深，相同的一句话根据语境的不同，可以解读出不同的意思，所以很难有一个明确的分词标准。

例如这句最经典的语句`下雨天留客天留我不留`,可以有五种意思：

- 下雨，天留客；天留，我不留。
- 下雨天，留客？天留，我不留。
- 下雨天，留客天，留我不留？
- 下雨天，留客天，留？我不留！
- 下雨天，留客天，留我不？留。

我们常用的中文分词系统有：
- IK
	- 实现中英文单词的切分, 支持ik_smart、ik_maxword等模式
	- 可自定义词库,支持热更新分词词典
	- https:   github.com/medcl/elasticsearch-analysis-ik
	
- jieba
	- python中最流行的分词系统,支持分词和词性标注
	- 支持繁体分词、自定义词典、并行分词等
	- https:   github.com/sing1ee/elasticsearch-jieba-plugin


分词会在如下两个时机使用:
- 创建或更新文档时( Index Time) ：会对相应的文档进行分词处理。索引时分词是通过配置Index Mapping中每个字段的analyzer属性实现的,不指定分词时,使用默认standard。

- 查询时( Search Time ) ,：会对查询语句进行分词。查询的时候可以通过`analyzer`指定分词器，还有就是设置 index  mapping的时候设置 `search_analyzer`实现搜索分词。

我们使用`IK`分词器对中文进行分词看看
```bash
POST _analyze
{
  "analyzer": "ik_smart",
  "text": "我爱你中国"
}
```
响应内容：
```json
{
  "tokens": [
    {
      "token": "我爱你",
      "start_offset": 0,
      "end_offset": 3,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "中国",
      "start_offset": 3,
      "end_offset": 5,
      "type": "CN_WORD",
      "position": 1
    }
  ]
}
```
我们可以看到`IK`很聪明的为我们分词了，不像`standard`分词器一个字一个字的分词。

