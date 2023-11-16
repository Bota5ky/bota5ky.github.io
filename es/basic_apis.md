9300 端口为 Elasticsearch 集群间组件的通信端口，9200 端口为浏览器访问的 http 协议 RESTful 端口。

| Elasticsearch |  Index   | Type  | Documents | Fields |
| :-----------: | :------: | :---: | :-------: | :----: |
|     MySQL     | Database | Table |    Row    | Column |

ES 里的 Index 可以看做一个库，而 Types 相当于表，Documents 则相当于表的行。这里 Types 的概念已经被逐渐弱化，Elasticsearch 6.X 中，一个 index 下已经只能包含一个type，Elasticsearch 7.X 中，Type 的概念已经被删除了。

**创建索引**

```http
PUT hittp://127.0.0.1:9200/shopping

{
	"acknowledged": true,
	"shards_acknowledged": true,
	"index":"shopping
}
```

再次创建会报错 400 error

**获取所有索引**

```http
GET hittp://127.0.0.1:9200/_cat/indices?v
```

**删除索引**

```http
DELETE hittp://127.0.0.1:9200/shopping

{
	"acknowledged": true
}
```

**创建文档**

```http
POST hittp://127.0.0.1:9200/shopping/_doc
{
	"title":"小米手机",
	"category":"小米",
	"images":"http://www.gulixueyuan.com/xm.jpg",
	"price":3999.00
}
```

**创建自定义 id 文档**

```http
POST hittp://127.0.0.1:9200/shopping/_doc/1001
{
	"title":"小米手机",
	"category":"小米",
	"images":"http://www.gulixueyuan.com/xm.jpg",
	"price":3999.00
}
```

能保证幂等性，用 PUT 也可以，如果能确定是新增，把`_doc`改成`_create`也可以。

**查询文档**

```http
GET hittp://127.0.0.1:9200/shopping/_doc/1001
```

**查询所有文档**

```http
GET hittp://127.0.0.1:9200/shopping/_search
{
	"query":{
		"match_all":{
		}
	}
}
```

请求体可以省略

**覆盖更新**

```http
PUT hittp://127.0.0.1:9200/shopping/_doc/1001
{
	"title":"小米手机",
	"category":"小米",
	"images":"http://www.gulixueyuan.com/xm.jpg",
	"price":4999.00
}
```

**局部更新**

```http
POST hittp://127.0.0.1:9200/shopping/_update/1001
{
	"doc": {
		"title":"华为手机"
	}
}
```

**删除文档**

```http
DELETE hittp://127.0.0.1:9200/shopping/_doc/1001
```

**条件查询**

```http
GET hittp://127.0.0.1:9200/shopping/_search?q=category:小米
```

使用请求体的形式

```http
GET hittp://127.0.0.1:9200/shopping/_search
{
	"query":{
		"match":{
			"category":"小米"
		}
	}
}
```

分页查询

```json
{
	"query":{
		"match_all":{
		}
	},
  "from":0,
  "size":2,
  "_source":["title"],
  "sort":{
    "price":{
      "order":"desc"
    }
  }
}
```

size 表示每页多少条，_source 指定显示的字段。

**多条件匹配**

```json
{
	"query":{
		"bool":{
			"must":[
				{
					"match":{
						"category":"小米"
					}
				},
        {
					"match":{
						"price":1999.00
					}
				}
			]
		}
	}
}
```

或匹配就把`must`换成`should`