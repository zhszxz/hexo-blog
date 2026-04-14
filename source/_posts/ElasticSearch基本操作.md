---
title: ElasticSearch基本操作
date: 2025-09-22 16:06:37
tags:
- RestFul
category: ElasticSearch
---

# ElasticSearch基本操作



> es对文档的基本RestFul操作汇总

<!--more-->

| 操作类型                | 请求方式 | URL 格式                     | 说明                                     | 示例                                                         |
| ----------------------- | -------- | ---------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| 创建文档                | PUT      | `/{index}/_doc/{id}`         | 指定 ID 创建文档，若 ID 已存在则覆盖     | `PUT /users/_doc/1`携带 JSON 文档：`{"name":"张三","age":30}` |
| 创建文档（自动生成 ID） | POST     | `/{index}/_doc`              | 不指定 ID，ES 自动生成唯一 ID            | `POST /users/_doc`携带 JSON 文档：`{"name":"李四","age":25}` |
| 查看文档                | GET      | `/{index}/_doc/{id}`         | 通过 ID 查询单个文档                     | `GET /users/_doc/1`                                          |
| 更新文档（全量替换）    | PUT      | `/{index}/_doc/{id}`         | 同 “指定 ID 创建”，若文档存在则全量替换  | `PUT /users/_doc/1`携带新 JSON 文档                          |
| 更新文档（部分字段）    | POST     | `/{index}/_doc/{id}/_update` | 仅更新指定字段                           | `POST /users/_update/1`携带：`{"doc":{"age":31}}`            |
| 删除文档                | DELETE   | `/{index}/_doc/{id}`         | 通过 ID 删除文档                         | `DELETE /users/_doc/1`                                       |
| 批量操作                | POST     | `/_bulk`                     | 批量执行创建、更新、删除（需按特定格式） | 格式示例：`{"index":{"_index":"users","_id":"3"}}``{"name":"王五","age":28}` |



> 测试

1. 创建或全量替换文档

```json
PUT /index/_doc/1
{
  "name": "zhangsan",
  "age": 23
}
```



```json
{
  "_index" : "index",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

```



2. 创建索引并定义映射

```json
PUT /index
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "age": {
        "type": "long"
      }
    }
  }
}
```



3. 查询索引信息

```json
GET /index
```



```json
{
  "index" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "long"
        },
        "name" : {
          "type" : "text"
        }
      }
    },
    "settings" : {
      "index" : {
        "creation_date" : "1758529598685",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "uuid" : "t1PmwRFQRUSe0ixcmWUMbg",
        "version" : {
          "created" : "7060199"
        },
        "provided_name" : "index"
      }
    }
  }
}
```



4. 部分更新文档

```json
POST /index/_doc/1/_update
{
  "doc": {
    "name": "lisi"
  }
}
```



```json
{
  "_index" : "index",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```



5. 删除索引

```json
DELETE /index
```



6. ID查询文档

```json
GET /index/_doc/1
```



```json
{
  "_index" : "index",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "lisi",
    "age" : 23
  }
}
```



## 带条件的复杂查询

1. 关键字查询

```json
GET /index/_doc/_search?q=name:lisi
```



```json
{
  "took" : 27,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "name" : "lisi",
          "age" : 23
        }
      }
    ]
  }
}
```



2. 结构化的关键字查询

```json
GET /index/_doc/_search
{
  "query": {
    "match": {
      "name": "lisi"
    }
  }
}
```



```json
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "name" : "lisi",
          "age" : 23
        }
      }
    ]
  }
}
```



3. 指定返回字段的查询

```json
GET /index/_doc/_search
{
  "query": {
    "match": {
      "name": "狂神"
    }
  },
  "_source": [
    "name",
    "desc"
  ]
}
```



```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.86891425,
    "hits" : [
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.86891425,
        "_source" : {
          "name" : "狂神说java",
          "desc" : "一顿操作猛如虎，一看工资2500"
        }
      },
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 0.78038335,
        "_source" : {
          "name" : "狂神说前端",
          "desc" : "青涩大男孩"
        }
      }
    ]
  }
}
```



4. 对结果排序

```json
GET /index/_doc/_search
{
  "query": {
    "match": {
      "name": "狂神"
    }
  },
  "sort": {
    "age": {
      "order": "asc"
    }
  }
}
```



```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : null,
        "_source" : {
          "name" : "狂神说java",
          "age" : 25,
          "desc" : "一顿操作猛如虎，一看工资2500"
        },
        "sort" : [
          25
        ]
      },
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : null,
        "_source" : {
          "name" : "狂神说前端",
          "age" : 27,
          "desc" : "阳光开朗大男孩"
        },
        "sort" : [
          27
        ]
      }
    ]
  }
}
```



5. 分页

```json
GET /index/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 2
}
```



```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "lisi",
          "age" : 23
        }
      },
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "狂神说java",
          "age" : 25,
          "desc" : "一顿操作猛如虎，一看工资2500"
        }
      }
    ]
  }
}
```



6. 与条件查询

```json

GET /index/_doc/_search
{
  "query": {
    "match": {
      "bool": {
        "must": [
          {
            "match": {
              "name": "狂神"
            }
          },
          {
            "match": {
              "age": 23
            }
          }
        ]
      }
    }
  }
}
```



```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.6944115,
    "hits" : [
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.6944115,
        "_source" : {
          "name" : "狂神说java",
          "age" : 25,
          "desc" : "一顿操作猛如虎，一看工资2500"
        }
      }
    ]
  }
}
```



7. 或条件查询

```json
GET /index/_doc/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "name": "前端"
          }
        },
        {
          "match": {
            "name": "lisi"
          }
        }
      ]
    }
  }
}
```



```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.7199613,
    "hits" : [
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.7199613,
        "_source" : {
          "name" : "lisi",
          "age" : 23
        }
      },
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.2199391,
        "_source" : {
          "name" : "狂神说前端",
          "age" : 27,
          "desc" : "阳光开朗大男孩"
        }
      }
    ]
  }
}
```



8. 非条件查询

```json
GET /index/_doc/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "name": "狂神"
          }
        }
      ]
    }
  }
}
```



```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.0,
    "hits" : [
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.0,
        "_source" : {
          "name" : "lisi",
          "age" : 23
        }
      }
    ]
  }
}
```



9. 结果过滤

```json
GET /index/_doc/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "狂神"
          }
        }
      ],
      "filter": {
        "range": {
          "age": {
            "gt": 20,
            "lt": 26
          }
        }
      }
    }
  }
}
```



```json
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.69441146,
    "hits" : [
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.69441146,
        "_source" : {
          "name" : "狂神说java",
          "age" : 25,
          "desc" : "一顿操作猛如虎，一看工资2500"
        }
      }
    ]
  }
}
```



10. 数组或条件匹配

```json
GET /index/_doc/_search
{
  "query": {
    "match": {
      "name": "狂神 lisi"
    }
  }
}
```



```json
{
  "took" : 320,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.8132977,
    "hits" : [
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.8132977,
        "_source" : {
          "name" : "张三",
          "age" : 23,
          "desc" : "法外狂徒",
          "tags" : [
            "刑法",
            "国家饭碗",
            "惯犯"
          ]
        }
      },
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 0.49005115,
        "_source" : {
          "name" : "狂神说前端",
          "age" : 25,
          "desc" : "青涩大男孩",
          "tags" : [
            "暖男",
            "技术",
            "摄影"
          ]
        }
      },
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.49005115,
        "_source" : {
          "name" : "狂神说java",
          "age" : 27,
          "desc" : "一顿操作猛如虎，一看工资2500",
          "tags" : [
            "渣男",
            "海王",
            "交友"
          ]
        }
      }
    ]
  }
}
```



11. 精确查询

```json
GET /index/_doc/_search
{
  "query": {
    "term": {
      "name": "狂神说java"	//要求name为keyword或使用name的.keyword子字段
    }
  }
}
```



12. 关键词高亮

```json
GET /index/_doc/_search
{
  "query": {
    "match": {
      "name": "狂神"
    }
  },
  "highlight": {
    "fields": {
      "name": {}
    }
  }
}
```



```json
{
  "took" : 45,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.90630186,
    "hits" : [
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.90630186,
        "_source" : {
          "name" : "狂神说java",
          "age" : 27,
          "desc" : "一顿操作猛如虎，一看工资2500",
          "tags" : [
            "渣男",
            "海王",
            "交友"
          ]
        },
        "highlight" : {
          "name" : [
            "<em>狂</em><em>神</em>说java"
          ]
        }
      },
      {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 0.8182796,
        "_source" : {
          "name" : "狂神说前端",
          "age" : 25,
          "desc" : "青涩大男孩",
          "tags" : [
            "暖男",
            "技术",
            "摄影"
          ]
        },
        "highlight" : {
          "name" : [
            "<em>狂</em><em>神</em>说前端"
          ]
        }
      }
    ]
  }
}
```

