# ElasticSearch 核心概念

[上一篇：安装和配置](/notes/install_es.md)

## 逻辑层面

### 文档 Documents
Elasticsearch 是 `document-oriented`，数据的最小单位就是`文档 document`, 一般用 `JSON` 表示 。文档有如下重要的属性：
- `self-contained`：一个文档同时包含`字段 fields` 和它们的`值 values`； 
- `can be hierarchical`：文档中可以包含文档；
- `has a flexible structure`：并不依赖于 `predefined schema`（在 MySQL 中必须要提前定义好 columns）。在一个文档中可以忽略特定字段，也可以动态添加新字段；

下面是一个文档实例：
```json
{
    "name": "Elasticsearch conference",
    "organizer": "Lee",
    "location": {
        "name": "Denver, Colorado, USA",
        "geo": "39.7392, -104.9847"
    },
    "members": ["Lee", "Jason"]
}
```

### 类型 Types

尽管 Elasticsearch 可以灵活的忽略或新增 field，但每个 field 的`类型 type` 非常重要。Elasticsearch 会保存所有的fields到它们的类型以及一些其他设置的一个`映射mapping`，所以在 Elasticsearch 中 types 有时候也称为 `mapping types`。

`types` 是 `documents` 的逻辑容器，就像是关系型数据库中，`tables` 是 `rows` 的容器。每一个type中fields的定义合起来叫做一个`映射 mapping`，例如 [name] 映射到 [string] 类型， [location] 映射到 [geo_point] 类型。

我们说 document 是 `schema-free` 的，因为一个 document 并不需要包含 mapping 里定义的所有 fields，同时可以加入新的 fields。当新的 field 加入而 Elasticsearch 不确定其类型时，它会猜测可能的类型。然而猜测不一定是准确的，所以最安全的方式就是提前定义好所需要的 mapping：`/index_name/type_name/doc_id`

#### 常见的 Field types

| Type | Description | Example values |
|:-:|:-|:-|
| [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) | 该类型的 fields 都被 `analyzed` 过：由一个 `analyzer` 将原始 string 分割为 list of terms； | "Lee", "EMNLP 2021" |
| [Numbers](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html) | | 123， 45.67 |
| [boolean](https://www.elastic.co/guide/en/elasticsearch/reference/current/boolean.html) |  | *true* or *false* |
| [Date](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html) |  | 2021-12-24T10:12:26.231+08:00 |
| [Arrays](https://www.elastic.co/guide/en/elasticsearch/reference/current/array.html) | | [first, secod, third] |

可以使用 [Multi-fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html) 对同一field 定义不同 type 或使用不同 analyzer。

`ES 7.0`及之后的版本取消了 types 的概念，每一个 index 只对应一个 mapping type 即 `_doc`：`/index_name/_doc/doc_id`

### 索引 Indices
`indices` 是 `mapping types` 的容器。一个 Elasticsearch `index` 就是一个独立的文档集合，类似于关系型数据库中的 `database`。Index 储存了 mapping types 中的所有 fields 以及其他设置。它们被存储在了各个`分片 shards` 上。

接下来我们来看在物理层面上，节点和分片是如何工作的。

## 物理层面

物理层面的两个主要概念是`节点(Node)`和`分片(Shard)`，简单来说：
- `Node` 就是一个 Elasticsearch 进程；
- `Shard` 就是一个 Lucene 索引，即一个包含`倒排索引`（[Appendix A](/notes/appendix_a.md)）的文件目录；

![physical](/figures/nodes-shards.png)

### 节点 Nodes
一个`节点(node)`就是一个 `Elasticsearch 进程`，通过配置文件或者启动时命名，启动之后，会分配一个 `UID`，保存在 `data` 目录下。

节点的种类主要有：
- `Master Node`：每个 node 上都保存了 cluster 的状态信息，但只有 Master Node 可以对其进行修改。每个 node 启动后都默认是一个 Master-eligible node，可以通过参与选主流程成为 Master node。当第一个 node 启动，它会将自己选举成 Master node；
- `Data Node`: 可以保存数据的 node，负责保存分片数据；
- `Coordinating Node`：负责接收 client 请求，将请求分发到合适的 node，最终把结果汇聚在一起。每个 node 默认都起到了 Coordinating Node 的职责；
- `Hot & Warm Node`：本质上还是 Data Node，根据不同的硬件配置，可以实现 Hot & Warm 架构，降低集群部署成本；
- `Machine Learning Node`：负责跑机器学习 Job，用来做异常检测；
- `Tribe Node`：连接到不同的 clusters，并且支持将这些 cluster 当成一个单独 cluster 处理（5.3版本开始使用 Cross Cluster Search 处理）；

一个`集群(cluster)`至少有一个`节点(node)`，有着多个节点的集群可以将数据分布在多个服务器上。生产环境中一个 node 应该设置单一的角色。

### 分片 Shards
一个`分片(Shard)`就是一个底层的 `Lucene Index`，即一个包含倒排索引的文件目录：
- `term dictionary`：将 term 映射到包含这个 term 的 documents。搜索时，Elasticsearch 不需要扫描所有文件，只需要利用 dictionary 即可；
- `term frequencies`：Elasticsearch 可以快速获取 term 在一个文档中出现的频率，从而计算相关性。默认的排序算法是 `TFIDFSimilarity`；

![lucene_index](/figures/lucene_index.png)

`Shards` 分为两种类型：
- `主分片 Primary Shard`：用以解决数据水平拓展问题。通过主分片将数据分布到集群内的所有节点上；主分片在索引创建时制定，后续不允许修改，除非 `reindex`；
- `副本分片 Replica Shard`：用以解决数据高可用问题，是主分片的拷贝。当有多个节点时，副本分片和主分片不设置在同一节点上，当主分片节点不可用时，副本分片保证了数据的可用性。副本分片的数量可以在 run-time 动态调整；

默认情况下一个 `Index` 会被分为 `5` 个 `primary shards`，每个 `primary shard` 有 `1` 个 `replica shard`，所以一共有 `10` 个 `shards`。

- `Indexing` 时，当一个 Elasticsearch node 收到 `indexing request` 后，会选择合适的 shard 来 index 这个 document。选择 shard 的策略是完全随机的：给每个 document 计算一个 `hashing id`，而每个 shard 都有一个等长的 `hash range`，于是每一个 shards 被选择是等概率的。之后，node 会把这个 document 发送选择好的 shard 所在的 node 上。

    ![sharding](/figures/indexing-shards.png)

- `Searching` 时，node 收到 `searching request` 后，根据 `round-robin 轮询算法`选择每一个可用的 shard，将 request 分发到包含所有数据的一组 shards 上（可以是 primary 也可以是 replica）。接下来收集好的数据被 `aggregate` 成一个单一的 reply 返回。

    ![searching](/figures/searching-shards.png)

设置分片时要做好容量规划：分片数量设置过小，可能导致单个分片数据量太大，进而导致数据重新分配耗时过大。另外可用性也会降低；反之如果分片数量设置过大，会影响相关性打分进而影响统计结果。另外单个节点过多分片影响性能，浪费资源。

[下一篇：REST 风格操作](/notes/restful_api.md)