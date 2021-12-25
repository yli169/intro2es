# 索引 Indexing Data

[上一篇：REST 风格操作](/notes/restful_api.md)

在这一篇笔记中我们将更加详细的讲解如何对数据进行索引. 以下实例都可以在 `Kibana` 的 `dev tools` 进行操作，在 `elasticsearch-head` 进行查看。你也可以使用 [cURL](https://curl.se/) 进行 command-line 操作，不过 kibana 更为简单和直观一些。

以下以 conference 为例：

- 创建 index：
  ```
  PUT /conference
  ```

- 创建新的 mapping：
    ```
    PUT /conference
    {
      "mappings": {
        "properties": {
          "name": { "type": "text" },
          "location": { "type": "text" },
          "main_event": {
            "properties": {
              "first_day": { "type": "date" },
              "last_day": { "type": "date" }
            }
          }
        }
      }
    }
    ```

    可以看到这里我们创建了一个包含了 1个`primary` 和 1个`replica` 共 2个`shards` 的 `node`，并将 名为 *conference* 的 `index` 存入其中。这个 `index` 中含有一个 default `type` 名为 *_doc*：

    ![example_0](/figures/example_0.png)

    同时我们看到 `replica shard` 现实为 *Unassigned*， 这是因为我们只有一个 node，而 `primary` 和它的 `replica` 不能在同一个 node 上，所以此时其实只有一个 shard 就是 `primary`。

- 向已经存在的 mapping 添加新 field：
    ```
    PUT /conference/_mapping
    {
        "properties": {
            "host": { "type": "text" }
        }
    }
    ```

- 创建 一个名为 *emnlp* 的 `document` 并存入数据：
    ```
    PUT /conference/_doc/emnlp
    {
      "name": "EMNLP 2021",
      "location": "Punta Cana, Dominican Republic",
      "main_event": {
        "first_day": "2021-11-07",
        "last_day": "2021-11-09"
      }
    }
    ```

    ![example_1](/figures/example_1.png)

- 使用 `POST` 同样可以创建文档：
    ```
    POST /conference/_doc
    {
      "name": "ACL-IJCNLP 2021",
      "location": "Virtual",
      "main_event": {
        "first_day": "2021-08-02",
        "last_day": "2021-08-04"
      }
    }
    ```

    ![example_2](/figures/example_2.png)

    那么它和 `PUT` 有什么不同呢：
    - `PUT` 是幂等的，无论执行多少次，不会创建超过一个资源。而 `POST` 不是幂等的，每次都会创建新的资源并生成随机id；
    - 使用 `PUT` 必须定义资源的 `完整URI` ；

那么 mapping 中定义的都有那些数据类型呢？

[下一篇：更新 Updating Data](/notes/update_data.md)