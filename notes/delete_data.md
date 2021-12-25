# 删除 Deleting Data

[下一篇：更新 Updating Data](/notes/update_data.md)

删除数据的常见做法是：
- 删除单个 document：
    ```
    DELETE /<index>/_doc/<doc_id>
    ```
- 根据 query 批量删除 documents：
    ```
    POST /conference/_delete_by_query
    {
        "query": {
            "match": {
            "name": "2021"
            }
        }
    }
    ```
- 删除整个 index：
    ```
    DELETE /<index>
    ```
- 关闭（不删除）index
    ```
    POST /<index>/_close
    ```

[下一篇：搜索 Searching Data](/notes/search_data.md)