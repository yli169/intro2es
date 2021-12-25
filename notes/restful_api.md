# REST 风格操作

[上一篇：Elasticsearch核心概念](/notes/es_concepts.md)

`REST` - `[RE]`presentational `[S]`tate `[T]`ransfer 表现层状态转移。
描述的是在网络中 `client` 和 `server` 的一种`交互形式`。其核心是：`URI 名词定位资源，HTTP 动词描述操作`：
- `Resource`：资源，即数据，如 newsfeed，friends 等；
- `Representational`：某种表现形式，比如 JSON，XML，JPEG等；
- `State Transfer`：状态变化。通过 HTTP 动词实现；

REST 的核心是设计具有 REST 风格（RESTful）的 API，从而通过一套统一的接口为 Web，iOS 和 Android 提供服务。

我们看一下一个 REST 风格的 `HTTP request` 是什么样子的。以下的 request 将会创建一个名为 `conference` 的索引，并在其中创建一个名为 `emnlp` 的文档用以保存给定数据：

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
在上面的 `request` 中：
- `PUT` 是一个 *method*，代表特定的动作；
- `/conference/_doc/emnlp` 是一个 *URI*，也就是我们进行操作的*资源*对象，其中：
  - `conference` 是 *index名*；
  - `_doc` 是 *默认 type名*，type 的概念在7.0版本之后被取消，每个 index 默认只有一个 type 就是 _doc；
  - `emnlp` 是 *文档id*；
- `{} 中的 JSON` 就是我们传输的*数据*；

**ESTful API Methods**

| Method | Action | Example URL |
| :-: | :-: | :- |
| GET | 获取资源 | /index/_doc/doc_id 通过id查询文档
| POST | 创建或更新资源 | <table> <td>/index/_doc/ 插入doc并随机生成id </td> </tr> <td>/index/_doc/doc_id 更新id对应的doc</td> </table> | a |
| PUT | 创建或更新资源 | /index/_doc/doc_id 当doc不存在时插入doc，否则更新id对应的doc |
|DELETE| 删除资源 | /index/_doc/doc_id 删除id对应的doc |

Elasticsearch本身提供丰富的 [REST API 库](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html)。

**参考**
- [【知乎：覃超】怎样用通俗的语言解释REST，以及RESTful？](https://www.zhihu.com/question/28557115/answer/48094438)

[下一篇：索引 Indexing Data](/notes/index_data.md)