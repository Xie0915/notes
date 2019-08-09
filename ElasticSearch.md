## **ElasticSearch**

[文档地址](http://wiki.jikexueyuan.com/project/elasticsearch-definitive-guide-cn/010_Intro/05_What_is_it.html)

### 入门



**是什么**

ES使用Java开发并使用Lucene作为核心，目的是提供RESTful API隐藏Lucene的复杂性，简化全文搜索

特点：

- 分布式实时文件存储，每个字段可以索引并搜索
- 分布式实时分析搜索
- 处理PB级结构化或非结构化



**安装**

```shell
curl -L -O http://download.elasticsearch.org/PATH/TO/VERSION.zip <1>
unzip elasticsearch-$VERSION.zip
cd  elasticsearch-$VERSION
```

运行

如果只有本地访问，修改配置文件elasticsearch.yml中network.hos为`network.host: 0.0.0.0`，在后台以守护进程方式运行，添加-d参数



**Java API**

节点客户端（在集群内）：无数据节点，不存储数据，但是能将请求转发到对应的节点上

传输客户端（不在集群内）：发送请求到远端集群，简单转发请求到集群中的节点

**HTTP** **API**

```SHELL
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```



**面向文档**

ES是**面向文档(document oriented)**的，这意味着它可以存储整个对象或**文档(document)**。然而它不仅仅是存储，还会**索引(index)**每个文档的内容使之可以被搜索。在Elasticsearch中，可以对文档（而非成行成列的数据）进行索引、搜索、排序、过滤。这种理解数据的方式与以往完全不同，这也是Elasticsearch能够执行复杂的全文搜索的原因之一。

ES使用JSON作为文档序列化格式



**索引**

对比关系型数据库,在Elasticsearch中，文档归属于一种**类型(type)**,而这些类型存在于**索引(index)**中.Elasticsearch集群可以包含多个**索引(indices)**（数据库），每一个索引可以包含多个**类型(types)**（表），每一个类型包含多个**文档(documents)**（行），然后每个文档包含多个**字段(Fields)**（列）

```
Relational DB -> Databases -> Tables -> Rows -> Columns
Elasticsearch -> Indices   -> Types  -> Documents -> Fields
```

假设建立一个员工目录



**搜索**

GET请求，给出索引，类型以及ID

```
GET /megacorp/employee/1

GET /{Indices}/{Types}/{id}
```

简单搜索

```
//默认返回前十个结果
GET /megacorp/employee/_search

//查询字符串
GET /megacorp/employee/_search?q=last_name:Smith
```

使用DSL（**Domain Specific Language**），以json请求体形式，能够构建更复杂的查询

```
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

使用过滤器

```
GET /megacorp/employee/_search
{
    "query" : {
        "filtered" : {
            "filter" : {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            },
            "query" : {
                "match" : {
                    "last_name" : "smith" 
                }
            }
        }
    }
}
```

**==全文搜索==**

传统数据库很难实现的功能，搜索特定字段

Query

```
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```

Result

```
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.16273327,
      "hits": [
         {
            ...
            "_score":         0.16273327,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_score":         0.016878016,
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

ES会根据结果相关性评分对结果进行排序

短语搜索

上述是对单个词的搜索，如果需要确切匹配某个短语，可以用match_phrase

```
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

改用上述查询，则结果会剔除掉只包含rock或者climbing的文档

**高亮**

```
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
```

加入highlight后，会自动高亮查询结果中的短语

```
"highlight": {
	"about": [
		"I love to go <em>rock</em> <em>climbing</em>" <1>
	]
}
```



**聚合**（aggregations）

```
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}

aggs:聚合关键字
terms:按内容聚合
avg:平均值
```





### 数据



**文档**

文档特指最顶层结构或者**根对象(root object)**序列化成的JSON数据（以唯一ID标识并存储于Elasticsearch中）

文档元数据

文档包含元数据，关于文档的信息，三个必须的元数据节点是：

| **_index** | **文档存储的地方** |
| ---------- | ------------------ |
| _type      | 文档代表的对象的类 |
| _id        | 文档唯一标识       |

我们的数据被存储和索引在**分片(shards)**中，索引只是一个把一个或多个分片分组在一起的逻辑空间

每个**类型(type)**都有自己的**映射(mapping)**或者结构定义，类型的**映射(mapping)**会告诉Elasticsearch不同的文档如何被索引



**索引**

文档通过`_index`,`_type`,`_id`唯一确定

ES中每一个文档都有一个版本号，当文档变化（包括删除）版本号就会变化



**获取**

GET 方法

返回元数据以及`_source`字段，`_source`字段就是创建索引时发给ES的原始文档

GET的返回包括`{"found": true}`，found为false时，HTTP响应码也会变为404



**存在**

检查文档是否存在用`HEAD`代替`GET`，通过http返回码标识





### 搜索

ES不仅会存储文档，也会索引文档内容使之能够被快速地查询

索引可以设计为：

1、使用结构化查询，并排序

2、全文检索，并按照关联性排序返回

3、上述两者结合





### 映射与分析

数据类型差异



分析和分析器

**字符过滤器：**