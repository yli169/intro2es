# REST风格操作

`REST` - `[RE]`presentational `[S]`tate `[T]`ransfer 表现层状态转移。
描述的是在网络中`client`和`server`的一种`交互形式`。其核心是：`URL名词定位资源，HTTP动词描述操作`：
- `Resource`：资源，即数据，如newsfeed，friends等；
- `Representational`：某种表现形式，比如JSON，XML，JPEG等；
- `State Transfer`：状态变化。通过HTTP动词实现；

REST的核心是设计具有REST风格（RESTful）的API，从而通过一套统一的接口为Web，iOS和Android提供服务。

## RESTful API Methods

| Method | Action | Example URL |
| :-: | :-: | :- |
| GET | 获取资源 | /index/_doc/doc_id 通过id查询文档
| POST | 创建或更新资源 | <table> <td>/index/_doc/ 插入doc并随机生成id </td> </tr> <td>/index/_doc/doc_id 更新id对应的doc</td> </table> | a |
| PUT | 创建或更新资源 | /index/_doc/doc_id 当doc不存在时插入doc，否则更新id对应的doc |
|DELETE| 删除资源 | /index/_doc/doc_id 删除id对应的doc |

我们注意到`PUT`和`POST`都可以创建和更新资源，那么它们有什么不同呢：
- 当资源已经存在时，二者并没有区别，都是更新已存在资源；
- `PUT`是幂等的，无论执行多少次，不会创建超过一个资源。而`POST`不是幂等的，每次都会创建新的资源并生成随机id；
- 使用`PUT`必须定义资源的`完整URI`；

Elasticsearch本身提供丰富的[REST API库](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html)。

## 简单操作实例
以下实例都可以在`Kibana`的`dev tools`进行操作，在`elasticsearch-head`进行查看。你也可以使用[cURL](https://curl.se/)进行command-line操作，不过kibana更为简单和直观一些。

### `PUT` Method
  
- 创建index和mapping type：
  ```
  PUT /test_index
  {
    "mappings": {
        "properties": {
        "name": { "type": "text" },
        "age": { "type": "long" }
        }
    }
  }
  ```
  ![example_0](/figures/example_0.png)
    
- 创建指定doc：
  ```
  PUT /test_index/_doc/doc1
  {
    "name": "Lee",
    "age": 30
  }
  ```
  ![example_1](/figures/example_1.png)
- 覆盖指定doc：
  ```
  PUT /test_index/_doc/doc1
  {
    "name": "Lee",
    "age": 29
  }  
  ```
  ![example_3](/figures/example_3.png)

### `POST` Method

- 创建doc并随机生成doc_id：
  ```
  POST /test_index/_doc/
  {
    "name": "Eric",
    "age": 25
  }
  ```
  ![example_2](/figures/example_2.png)

- 更新指定doc：
  ```
  POST /test_index/_update/doc1
  {
    "doc": {"age": 31 }
  } 
  ```
  ![example_4](/figures/example_4.png)

### `GET` Method
- 查看index信息
  ```
  GET /test_index
  ```
- 查看document信息
  ```
  GET /test_index/_doc/doc1
  ```
- 查看所有indices情况
  ```
  GET _cat/indices
  GET _cat/health
  ```

### `DELETE` Method
- 删除文档
  ```
  DELETE /test_index/_doc/doc1
  ```
- 删除index
  ```
  DELETE /test_index
  ```

我们将会在下两章中更详细的讲解Indexing和Searching的操作。

## 常见Client Error Status Code

Client error code占用 `4xx` 段

| Status Code | Description |
| :-: | :- |
| 400 Bad Request | 一般是因为request有syntax错误 |
| 401 Unauthorized | 缺少必须的user authentication |
| 403 Forbidden | client没有权限 |
| 404 Not Found | server无法找到请求的资源 |
| 405 Method Not Allowed | request的资源不可以被该method访问 |
| 406 Not Acceptable | server无法找到满足request criteria的资源 |
| 407 Proxy Authentication | 缺少必须的proxy Authentication |
| 408 Request Timeout | Server在时限内没有从client收到完整的request | 


**参考**
- [【知乎：覃超】怎样用通俗的语言解释REST，以及RESTful？](https://www.zhihu.com/question/28557115/answer/48094438)

[下一篇：如何Index新的数据](/notes/index_new_data.md)