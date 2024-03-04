# Elastic Search

## 安装配置

#### 初始配置

- 默认不支持root用户,需要进行配置才能允许
- 配置修改:

```conf
#config/elasticsearch.yml
network.host: 0.0.0.0

#config/jvm.options
-Xms128m
-Xmx128m

#/etc/sysctl.conf
vm.max_map_count = 655360
sysctl -p
```

#### 默认端口

- elastic search:9200
- kibana:5601

## Elastic Search 原理

#### Elastic Search 结构

ES和mysql之间的对应关系:

| Elastic Search |      MySQL       |
| :------------: | :--------------: |
|  Index(索引)   | Database(数据库) |
|   Type(类型)   |    Table(表)     |
| Document(文档) |     row(行)      |
|  Field(字段)   |    column(列)    |

- Elastic Search中索引是一种数据的集合,相当于只有一个表数据库(包含了数据库配置信息包括分片数量副本数量等信息;并且只能存储一类数据,因为要在一个Index中维护一个系列的倒排索引)
- Elastic Search中为什么管数据库叫索引:因为索引就是正排索引,包含了id等元数据,只不过将文档也存储了进来
- Elastic Search中一个文档对应一条数据,文档中的字段对应mysql中的列

#### 正排索引&倒排索引

- 正排索引:可以实现O(1)由id查询文档中的字段(存储文档的指针),但是不能快速查询字段符合条件的文档(eg:标题中含有"动漫推荐"的视频)

- 倒排索引:将文档内容进行分词,然后建立由词汇到文档id的映射,这样就能根据词汇查询文档信息

  ```
  正排索引:
  id		content
  --------------------
  1001	[2024冬] 憧憬成为魔法少女
  1002	[2024冬] 魔法少女育成计划
  
  倒排索引:
  keyword			id
  --------------------
  [2024冬]		  1001,1002
  魔法少女		1001,1002
  憧憬				1001
  成为				1001
  育成				1002
  计划				1002
  ```
  
#### 数据类型(数组不用单独定义)

- **字符串:**
  - **keyword**:**不会进行分词**直接建立倒排索引
  - **text**:**会进行分词**
- 数值:long, integer, short, byte, double, float
- 布尔值:boolean
- 时间:date
- GEO:地理信息

#### 分词(核心重点)

- ES默认使用**非字母字符分词**:不会拆分一个长的单词

```
GET _analyzer
{
	"analyzer":"english",
	"text":["Before Milky DD ~Naganami-sama to Boku~ (Kantai Collection -KanColle-) [English]"]
}
```

- 使用IK分词器

#### 文档得分机制

`Score = TF * IDF`

- TF:搜索文本中的各个词条在查询文本中出现了多少次,出现次数越多TF越高
- IDF:搜索文本中的各个词条在整个索引的所有文档中出现了多少次,出现次数越多IDF越低

## Elastic Search API

- 注意:Elastic Search的API遵循Restful风格,对于单个对象的操作都是**同一个URL**但是**请求方法不同**
  - PUT:更新操作,要求具有幂等性(PUT不管执行多少次都和执行一次结果一样)
  - POST:提交操作,一般提交具有数据
- 注意:Elastic Search默认不区分大小写

#### 索引操作

- 索引:aliases别名,mappings映射(字段类型),settings配置信息
- 索引建立之后,就不能再修改索引配置

```
PUT /索引名:创建索引
HEAD /索引名:判断索引是否存在
GET /索引名:获取指定索引信息
GET /_cat/indices?v:列举所有索引信息
DELETE /索引名:删除某个索引
```

- 映射:存储索引中字段的类型和是否建立倒排索引等信息

```
PUT /anime
{
	"aliases":{
		"animation": {},
		"animes":{}
	},
    "mappings":{
        "properties":{
        	"title":{"type":"text","analyzer": "english","search_analyzer": "english"},
        	"series":{"type":"keyword"},
            "characters":{"type":"keyword"},
            "tags":{"type":"keyword"},
            "artists":{"type":"keyword"},
            "groups":{"type":"keyword"},
            "language":{"type":"keyword"},
            "type":{"type":"keyword"},
            "pages":{"type":"short"},
            "upload":{"type":"date"},
            "readurl":{"type":"text","index":false}
        }
    }
}
```

#### 文档操作

- 添加/更新(全量修改,部分修改)

```
POST /索引名/_doc {JSON数据}	 		  :添加文档(自动生成ID并返回),不具有幂等性(不能用PUT)
PUT /索引名/_doc/自定义ID {JSON数据}	:添加/更新文档(自定义ID),幂等(如果不存在就添加,存在就全量修改)
POST /索引名/_update/指定ID {"doc":{部分JSON数据}} :更新文档(部分更新)
```

```
代码示例(添加/替换->1001的数据):
PUT /anime/_doc/1001
{
    "title": "Before Milky DD ~Naganami-sama to Boku~ (Kantai Collection -KanColle-) [English]",
    "series": ["kantai collection"],
    "characters": ["naganami"],
    "tags": [
        "big breasts",
        "sole female",
        "sole male",
        "stockings",
        "shotacon",
        "lactation",
        "very long hair",
        "breast feeding",
        "milking"
    ],
    "artists": ["yositama"],
    "groups": ["cuniculus"],
    "language": [ "translated","english" ],
    "type": ["doujinshi"],
    "pages": 24
}
部分添加
POST /anime/_update/1001
{
"doc": {
	"tags":[
		"big breasts",
        "sole female",
        "sole male",
        "stockings",
        "shotacon",
        "lactation",
        "very long hair",
        "breast feeding",
        "milking",
		"soft girl"
	]
}
}
```

- 查询(全部):

```
GET /索引名/_doc/指定ID		 		  :查询指定ID的文档
GET /索引名/_search		  		   :查询这个索引的全部文档
```

- 删除

```
DELETE /索引名/_doc/指定ID		:删除指定ID的文档
POST /索引名/_delete_by_query
{...}

```

#### 文档搜索

- 限制查询字段:_source

- 查询条件:query

  - match:分词然后查询
  - term:查询关键词(用来查询不能被拆分的词),仅对于keyword有效对text无效(需要设置子字段)
  - match_phrase:查询项分词之后顺序必须和原文一致(搭配slope)
  - range:范围查询:
    - gt:大于,lt:小于
    - gte,lte:大于等于,小于等于

  - bool:逻辑判断(多个查询条件)
    - should:或者(or)
    - must:并且(and)

- 分页查询

```
GET /索引名/_search
{
	"query": {
		"match":{
			"title":"zhangsan"
		}
	}
}
GET /索引名/_search
{
  "_source":["title","tags"],
  "query":{
    "term": {
      "title": "milking"
    }
  },
  "from":20,
  "size":10
}
GET /anime/_search
{
  "query":{
    "range": {
      "pages":{
        "gt": 20
      }
    }
  }
}
```



# 各个语言中的Elastic Search接口

## Python

#### 索引操作

```python
from elasticsearch import Elasticsearch
es = Elasticsearch(['http://192.168.88.18:9200'])
es.indices.create(index="index_name")
es.indices.delete(index="index_name")
```

#### 文档操作

- index(index,id,body):创建/全量更新(id:自己指定的ID参数)
- search(index,body):查询文档
- update(index,id,body):增量更新
- delete(index,id):删除文档

```
json_data = {'title': 'Before Milky DD ~Naganami-sama to Boku~ (Kantai Collection -KanColle-) [English]', 'series': ['kantai collection'], 'characters': ['naganami'], 'tags': ['big breasts', 'sole female', 'sole male', 'stockings', 'shotacon', 'lactation', 'very long hair', 'breast feeding', 'milking'], 'artists': ['yositama'], 'groups': ['cuniculus'], 'language': ['translated', 'english'], 'type': ['doujinshi'], 'pages': 24, 'readurl': 'https://nhentai.net/g/498719/'}
es.index(index='anime',body=json_data)

query = {
    "query": {
        "match_phrase": {
            "title": "milk"
        }
    }
}
es.search(index='animes',body=query)
```





- 更换分词器
- 批量添加

- selenium写一个ipynb:批量爬取漫画信息(配置数据放到最前面)
- 开发前端标签页面和搜索框
- 底层原理:时间复杂度/空间复杂度
- python使用ES接口修改,js使用ES接口查询
- 查看真正分词情况
- es整个流程

- https加密连接

- 时间处理

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  nKtnfDN*bTiJY6-xREfD

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  a82bc774415973ab6ce72cdfcca9685ddbea33a9ffeed8e536743ef55da9effe

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjEyLjIiLCJhZHIiOlsiMjA3LjE4MC4yMDYuMjA4OjkyMDAiXSwiZmdyIjoiYTgyYmM3NzQ0MTU5NzNhYjZjZTcyY2RmY2NhOTY4NWRkYmVhMzNhOWZmZWVkOGU1MzY3NDNlZjU1ZGE5ZWZmZSIsImtleSI6Img3N0dBSTRCdVExMjdnMUJWcFRuOlp2UjM2anVKU0hHRUFZeFI3VEdXbGcifQ==

ℹ️ Configure other nodes to join this cluster:
• Copy the following enrollment token and start new Elasticsearch nodes with `bin/elasticsearch --enrollment-token <token>` (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjEyLjIiLCJhZHIiOlsiMjA3LjE4MC4yMDYuMjA4OjkyMDAiXSwiZmdyIjoiYTgyYmM3NzQ0MTU5NzNhYjZjZTcyY2RmY2NhOTY4NWRkYmVhMzNhOWZmZWVkOGU1MzY3NDNlZjU1ZGE5ZWZmZSIsImtleSI6ImliN0dBSTRCdVExMjdnMUJWcFQ1OjBhLXllbE5LUVlpSy0tam1LaG5SSlEifQ==

  If you're running in Docker, copy the enrollment token and run:
  `docker run -e "ENROLLMENT_TOKEN=<token>" docker.elastic.co/elasticsearch/elasticsearch:8.12.2`
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Nhx9=B!.7#h

kibana_system: lfFd+-uW7qWTCi8z6zRy

- 架构部署(安全):es,kibana
- 分词器:三个部分配置
- 查询优化:match_phrase
- js,python接口

es存储聊天信息:

- 好友关系:set实现(redis)
- 