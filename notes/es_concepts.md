# ElasticSearch核心概念

[上一篇：ELK的安装和配置](/notes/install_es.md)

## 逻辑层面

### 文档 Documents
Elasticsearch是`document-oriented`，数据的最小单位就是`文档document`, 一般用`JSON`biaoshi 。文档有如下重要的属性：
- `self-contained`：一个文档同时包含`字段fields`和它们的`值values`； 
- `can be hierarchical`：文档中可以包含文档；
- `has a flexible structure`：并不依赖于`predefined schema`（在MySQL中必须要提前定义好columns）。在一个文档中可以忽略特定字段，也可以动态添加新字段；

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

尽管Elasticsearch可以灵活的忽略或新增field，但每个field的`类型type`非常重要。Elasticsearch会保存所有的fields到它们的类型以及一些其他设置的一个`映射mapping`，所以在ES中types有时候也称为`mapping types`。

`types`是`documents`的逻辑容器，就像是关系型数据库中，`tables`是`rows`的容器。每一个type中fields的定义合起来叫做一个`映射mapping`，例如[name]映射到[string]类型，[location]映射到[geo_point]类型。

我们说document是`schema-free`的，因为一个document并不需要包含mapping里定义的所有fields，同时可以加入新的fields。当新的field加入而Elasticsearch不确定其类型时，它会猜测可能的类型。然而猜测不一定是准确的，所以最安全的方式就是提前定义好所需要的mapping。

### 索引 Indices
`indices`是`mapping types`的容器。一个ES `index`就是一个独立的文档集合，类似于关系型数据库中的`database`。Index储存了mapping types中的所有fields以及其他设置。它们被存储在了各个`分片shards`上。

接下来我们来看在物理层面上，节点和分片是如何工作的。

## 物理层面

物理层面的两个主要概念是`节点(Node)`和`分片(Shard)`，简单来说：
- `Node`就是一个Elasticsearch进程；
- `Shard`就是一个Lucene索引，即一个包含倒排索引的文件目录；

![physical](/figures/nodes-shards.png)

### 节点 Nodes
一个`节点(node)`就是一个`ES进程`，通过配置文件或者启动时命名，启动之后，会分配一个UID，保存在data目录下。

节点的种类主要有：
- `Master Node`：每个node上都保存了cluster的状态信息，但只有Master Node可以修改cluster状态信息。每个node启动后都默认是一个Master-eligible node，可以通过参与选主流程成为Master node。当第一个node启动，它会将自己选举成Master node；
- `Data Node`: 可以保存数据的node，负责保存分片数据；
- `Coordinating Node`：负责接收Client请求，将请求分发到合适的node，最终把结果汇聚在一起。每个node默认都起到了Coordinating Node的职责；
- `Hot & Warm Node`：本质上还是Data Node，根据不同的硬件配置，可以实现Hot & Warm架构，降低集群部署成本；
- `Machine Learning Node`：负责跑机器学习Job，用来做异常检测；
- `Tribe Node`：连接到不同的clusters，并且支持将这些cluster当成一个单独cluster处理（5.3开始使用Cross Cluster Search处理）；

一个`集群(cluster)`至少有一个`节点(node)`，有着多个节点的集群可以将数据分布在多个服务器上。生产环境中一个node应该设置单一的角色。

### 分片 Shards
一个`分片(Shard)`就是一个底层的`Lucene Index`，即一个包含倒排索引的文件目录：
- `term dictionary`：将term映射到包含这个term的documents。搜索时，ES不需要扫描所有文件，只需要利用dictionary即可；
- `term frequencies`：ES可以快速获取term在一个文档中出现的频率，从而计算相关性。默认的排序算法是`TF-IDF`；

![lucene_index](/figures/lucene_index.png)

`Shards`分为两种类型：
- `主分片Primary Shard`：用以解决数据水平拓展问题。通过主分片将数据分布到集群内的所有节点上；主分片在索引创建时制定，后续不允许修改，除非`reindex`；
- `副本分片 Replica Shard`：用以解决数据高可用问题，是主分片的拷贝。当有多个节点时，副本分片和主分片不设置在同一节点上，当主分片节点不可用时，副本分片保证了数据的可用性。副本分片的数量可以在run-time动态调整；

默认情况下一个`Index`会被分为`5`个`primary shards`，每个`primary shard`有`1`个`replica shard`，所以一共有`10`个`shards`。

- `Indexing`时，当一个ES node收到`indexing request`后，会选择合适的shard来index这个document。选择shard的策略是均匀分布：给每个document计算一个`hashing id`，而每个shard都有一个等长的`hash range`，于是每一个shards被选择是等概率的。之后，node会把这个document发送选择好的shard所在的node上。

    ![sharding](/figures/indexing-shards.png)

- `Searching`时，node收到`searching request`后，根据`round-robin轮询算法`选择每一个可用的shard，将request分发到包含所有数据的一组shards上（可以是primary也可以是replica）。接下来收集好的数据被`aggregate`成一个单一的reply返回。

    ![searching](/figures/searching-shards.png)

设置分片时要做好容量规划：分片数量设置过小，可能导致单个分片数据量太大，进而导致数据重新分配耗时过大。另外可用性也会降低；反之如果分片数量设置过大，会影响相关性打分进而影响统计结果。另外单个节点过多分片影响性能，浪费资源。

[下一篇：REST API操作](/notes/rest_api.md)