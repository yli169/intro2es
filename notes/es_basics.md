# Elasticsearch 简介

## 什么是 ElasticSearch?

`Elasticsearch` 是一个底层基于 [Apache Lucene<sup>TM</sup>](https://lucene.apache.org/core/)，提供 [RESTful API](https://restfulapi.net/) 的实时的分布式检索和分析引擎。它同是也是一个分布式文档数据库，其中每个字段均可被索引，且每个字段的数据皆可被搜索。ES具有高扩展性，能够横向扩展至数以百计的服务器储存，并在极短的时间内储存，搜索和分析PB级数据，并且可以应对高并发场景。

## 为什么要使用 ElasticSearch?

相比起传统的关系型数据库如 MySQL，Elasticsearch 更适用于`海量数据`以及`全文搜索`的场景：
- MySQL将`索引(Index)`储存在`磁盘(Disk)`中，当数据量大到一定程度时，每次的IO会导致性能衰减。而 Elasticsearch 直接建立 index 放入`内存(RAM)'中，效率大大提高；
- 同时由于 Elasticsearch 天然是`分布式(Distributed)`的，具有高`扩展性(Scalability)`，可以很容易横向扩容，通过分片降低检索规模；
- 在进行基于分词后的`全文搜索`时，对于 MySQL 来说，因为 index 失效，会进行全表搜索。而对于 Elasticsearch 而言，每个字都可以高速找到`倒排索引`的位置，并迅速获得文档id列表，大大的提升了性能。

> `索引(Index)`是什么？索引是对数据库表中一列或多列的值进行排序，从而达到对数据库表中特定信息进行迅速访问的数据结构。比如在下图中，如果有一条SQL语句 `SELECT * FROM teacher WHERE id = 101`, 在没有 index 的情况下，想要找到该条数据，我们就需要对全标进行扫描，匹配 `id=101` 的数据。但如果有了 index，我们就可以迅速找到 `101` 对应的记录在磁盘中的地址，进一步读取出相对应的数据。
> ![index example](/figures/index_record.png)

## 使用 ELK Stack 进行开发

我们在开发过程中主要会用到 `ELK Stack` 中的项目。

`ELK` 是三个开源项目的首字母缩写：
- `[E]lasticsearch`：搜索和分析引擎；
- `[L]ogstash`：轻量级的日志收集处理框架，可以收集分散的多样化的日志，并进行自定义过滤分析处理，然后传输到指定的位置，比如某个服务器或者文件。从功能上来讲，它只做三件事情：
  - input：数据收集；
  - filter：数据加工，如过滤，修改等；
  - output：数据输出；
- `[K]ibana`：为 `Elasticsearch` 和 `Logstash` 提供数据可视化和挖掘的web端平台。它可以用于日志和时间序列分析，应用程序监控和运营智能使用案例；

现在，ELK Stack可以与第四个项目结合使用：`Beats` —— 一个轻量化平台，集合了多种单一用途数据采集器。它们从成百上千或成千上万台机器和系统向 Logstash 或 Elasticsearch 发送数据。

![elk](/figures/elk.svg)

在接下来的笔记中，我们将会介绍如何安装 ELK 并对他们进行基础的配置。

[下一篇：安装和配置](/notes/install_es.md)