---
title: elasticsearch笔记
date: 2023-02-03 15:33:59
tags: "es"
categories: "es"
---

#### 检查集群的健康状况

- green：所有primary shard和replica shard都已成功分配，集群100%可用
- yellow：所有primary shard都已成功分配，但至少有一个replica shard缺失，此时集群所有功能都正常使用，数据不会丢失，搜索结果依然完整，但集群的可用性减弱
- red：至少一个primary shard缺失，部分数据不能使用，搜索只能返回部分数据，分配上的写入请求会返回一个异常，为了索引数据的完整性，需尽快修复集群

#### 查询文档方法

- Query String Search（直接在浏览器的URL地址栏输入搜索参数）

  ```
  http://172.16.22.133.9301/book_shop/it_book/_search?q=name:Java
  ```

  ![](https://s1.ax1x.com/2023/02/03/pSsU7B4.png)

  各参数含义：（1）took：检索时间（2）time_out：是否超出规定检索时间（3）_shards：查询的index分散成了多个分片，搜索请求会分发到所有primary shard或对应的replica shard，显示各分片是否查询成功（4）hits：命中文档的情况，total表示总数，max_score：Lucene底层对检索文档相关度的评分，hits：命中文档的详细数据

- Json格式参数查询

- 过滤检索，如查询name中包含java且price不大于80的商品

  ```
  GET book_shop/it_book/_search
  {
  	"query": {
  		"bool": {
  			"must": {
  				"match": {"name": "Java"}
  			},
  			"filter": {
  				"range": {
  					"price": {"lte": 80.0}
  				}
  			}
  		}
  	}
  }
  ```

- 全文检索，查询描述信息desc中包含“Java图书”的文档，只显示name和desc

  ```
  GET book_shop/it_book/_search
  {
  	"query": {
  		"match": {"desc": "Java图书"}
  	},
  	"_source": ["name","desc"]
  }
  ```

- 短语检索（phrase search）

  不对检索串进行分词处理，只有与检索文本完全一致才能作为结果返回

  ```
  GET book_shop/it_book/_search
  {
  	"query": {
  		"match_phrase": {
  			"desc": "Java图书"
  		}
  	},
  	"_source": ["name","desc"]
  }
  ```

什么是倒排索引：即反向索引，日常通过key找到value，这个是通过value找到key，倒排索引主要由两部分组成：“单词词典”和“倒排文件”

- 单词词典：由文档集合中出现的所有单词构成的字符串集合，单词词典内每条索引项记在单词本身一些信息及指向“倒排列表”的指针
- 倒排列表：记录出现过某个单词的所有文档的文档列表及单词所在位置信息，每条记录称为一个倒排项，根据倒排列表可知哪些文档包含某个单词
- 倒排文件：所有单词的倒排列表顺序存储在磁盘某个文件中，该文件即倒排文件，倒排文件是存储倒排索引的物理文件

#### 分词器

- 分析器的组成
  1. 字符过滤器：如去除html标签，将&替换为and等
  2. 分词器：按照某种规律，如根据空格、逗号等将文本块进行分解
  3. 标记过滤器：修改词（小写化处理等）、去掉词（由某一规则去掉无意义的词，如“a”）、增加词（增加同义词等）
- 分析器与分词器之间为包含关系
- 倒排索引核心原理-normalization：时态、单复数、同义词、大小写转换等

#### 索引模版

将已经创建好的某个索引的参数设置和索引映射保存下来，在创建新索引时指定模版名可以重用已定义好的模版中的设置和映射

#### 搜索

默认情况下should是可以不匹配任何一个条件的，可以用minimum_should_match控制至少匹配几个才能作为结果返回；在没有must的情况下，should至少匹配一个才可以返回

#### 全量替换

- 若id不存在则为创建，若id已存在则为全量替换操作
- document是不可变的，若要修改document的内容则进行全量替换，直接对document重新建立索引，替换里面的内容
- es会将老的document标记为deleted，当创建越来越多的document的时候，es会在适当的时机在后台自动删除标记为deleted的document
- document强制创建：PUT /index/type/id?op_type=create，PUT /index/type/id/_create
- document的删除：DELETE /index/type/id，将其标记为deleted，当数据越来越多的时候，在后台自动删除

#### 悲观锁和乐观锁并发控制方案

- 悲观锁即在对数据进行修改时加锁；优点即对应用程序是透明的不需要额外操作，缺点是并发能力很低，同一时间只能有一条线程操作数据
- 乐观锁即将拿到的版本号与es版本号进行对比，若相同则修改，不同则重新读取；优点即并发能力高，缺点是较麻烦