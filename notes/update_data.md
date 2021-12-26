# 更新 Updating Data

[下一篇：索引 Indexing Data](/notes/index_data.md)

## 更新数据：

 - 完全覆盖原有 doc：
    ```
    PUT /test_index/_doc/doc1
    { "name": "Lee", "age": 29 }  
    ```
- Partially 更新原有 doc：
    ```
    POST /test_index/_update/doc1
    {
    "doc": { "age": 31  }
    } 
    ```
- 应对 被 update 的 doc 可能不存在的情况，使用 `upsert`：
    ```
    POST /test_index/_update/doc1
    {
        "doc": { "age": 31 },
        "upsert": { 
            "name": "Lee", 
            "age": 31
        }
    }
    ```
- 使用 `script` 基于已有值更新，用 `ctx._source` 访问指定 field：
    ```
    POST /conference/_update/emnlp
    {
        "script" : {
            "source": "ctx._source.people *= params.markup",
            "lang": "painless",
            "params" : {
                "markup" : 0.8
            }
        }
    }
    ```
 
## 版本控制

Elasticsearch 支持 `concurrency control`，每次文档的创建和更新都会有对应的 `version`，当版本出现 conflict 时会报错并取消当前 update。可以在下一次 update 中再次尝试。

下例中 update2 在 update1 前先完成 processing，当 处理 update1 时由于版本号冲突，系统报错：

![example_3](/figures/example_3.png)

可以设定 `retry_on_conflict` 让 Elasticsearch 在 conflict 后自动尝试再次更新；

[下一篇：搜索相关性](/notes/search_relevancy.md)