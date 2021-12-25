# 搜索 Searching Data

[上一篇：索引 Deleting Data](/notes/delete_data.md)

在这一篇笔记中我们将更加详细的讲解如何对数据进行搜索.

## 理解 Search Request

![example_4](/figures/example_4.png)

### 搜索范围
- 搜索整个 `cluster`：
    ```
    GET /_search
    ```
- 搜索整个 `index` ：
    ```
    GET /<index>/_search
    ```
- 搜索多个 `indices`：
    ```
    GET /<index1>,<index2>/_search
    GET /+<prefix>*,-<index>/_search
    ```

### Search Request 组成部分

  - `query`：搜索主体；
  - `size`：返回的最大文档数，类似于 SQL 的 LIMIT；
  - `from`：类似于 SQL 的 OFFSET；
  - `_source`：可以设定返回的 _source 内容，默认返回整个 _source；
  - `sort`：返回文档的排序顺序，默认根据 _score 排序；

```
GET /conference/_search
{
    "query" : {
        "term" : { "name" : "2021" }
    },
    "from" : 0, "size" : 2,
    "_source": ["name", "location"],
    "sort" : [
        {
            "people" : {
                "order" : "asc", 
                "mode" : "avg"
            }
        }
    ]
}
```

### 理解 Response

除了满足搜索条件的文件外，回复中还包含了其他有用的信息。首先让我们看一下，一个搜索回复是什么样子。

Request
```
GET /conference/_search
{
  "query": {
    "match" : {
      "name": "EMNLP"
    }
  }
}
```
Response
```json
{
  "took" : 1,  // 耗时 1ms 
  "timed_out" : false,  // 没有 time-out
  "_shards" : {
    "total" : 1,  // 一共只有 1个 (primary) shard
    "successful" : 1,  // 1个 shard 成功 replied
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,  // 一共有 1篇 匹配文档
      "relation" : "eq"  // 数字是否精确 eq： ==
                         //            gte：>=
    },
    "max_score" : 0.7549127,  // 所有匹配文档的最大分数
    "hits" : [  // 所有命中的匹配文档
      {
        "_index" : "conference",
        "_type" : "_doc",
        "_id" : "emnlp",
        "_score" : 0.7549127,  // 相关性分数
        "_source" : {
          "name" : "EMNLP 2021",
          "location" : "Punta Cana, Dominican Republic",
          "main_event" : {
            "first_day" : "2021-11-07",
            "last_day" : "2021-11-09"
          }
        },
      }
    ]
  }
}
```

## 常用 query 和 filter DSL

![example_5](/figures/example_5.png)

`Elasticsearch DSL` 是一个 high-level 的 library，类似于标准库。我们使用 `query` 进行匹配，使用 `filter` 进行筛选（filter 速度更快）：

```
GET /conference/_search
{
    "query": {
        "filtered": {
            "query": {
                "match": {
                    "location": "virtual"
                }
            },
            "filter": {
                "term": {
                    "name": "2021"
                }
            }
        }
    }
}
```

- `match_all` query 返回所有 documents 并将 _score 全部设为 1.0:
    ```
    GET /_search
    {
        "query": {
            "match_all": {}
        }
    }
    ```

### [Term-level queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/term-level-queries.html) 单词匹配
- [term](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html) 匹配单个 token：
    ```
    GET /_search
    {
        "query": {
            "term": {
                "name": "EMNLP"
            }
        }
    }
    ```
    也可以用来 filter：
    ```
    GET /_search
    {
        "query": {
            "filtered": {
                "query": {
                    "match_all": {}
                },
                "filter": {
                    "term": {
                        "name": "2021"
                    }
                }
            }
        }
    }
    ```
- [terms](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-terms-query.html) 匹配多个tokens：
    ```
     GET /_search
    {
        "query": {
            "terms": {
                "name": ["EMNLP", "ACL"]
            }
        }
    }   
    ```

### [Full text queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html) 全文匹配
- [match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html) 分词后匹配并计算分数：
    ```
    GET /_search
    {
        "query": {
            "match": {
                "name": "EMNLP 2021"
            }
        }
    }
    ```
- [query_string](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html) 匹配给定字符串：
    ```
    GET /_search
    {
        "query": {
            "query_string": {
            "query": "(EMNLP 2021) OR (ACL 2021)",
            "default_field": "name"
            }
        }
    }
    ```
- [multi-match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html) 在多个 fields 中搜索：
    ```
    {
        "query": {
            "multi_match": {
            "query": "EMNLP Dominican",
            "fields": ["name", "location"]
            }
        }
    }
    ```

### 复合 queries
- [bool](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html) query
    | query | description |
    | :- | :- |
    | must | 必须全部match，使用AND |
    | must_not | 必须不match，使用NOT |
    | should | 需要match，使用OR | 
    ```
    GET /_search
    {
        "query": {
            "bool": {
                "must": [
                    {
                        "term": { "name": "EMNLP" }
                    }
                ],
                "must_not": [
                    {
                        "term": { "name": "2020" }
                    }
                ],
                "should": [
                    {
                        "term": { "location": "dominican" }
                    }
                ],
            }
        }
    }
    ```
    bool query 也可以作为 filter 使用。

[下一篇：分析 Analyzing Data](/notes/analyze_data.md)