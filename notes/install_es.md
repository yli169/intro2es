# 安装和配置

[上一篇：Elasticsearch简介](/notes/es_basics.md)

## 安装Elasticsearch
***
1. 进入Elasticsearch[官方下载页面](https://www.elastic.co/cn/downloads/elasticsearch)，选择适合你系统的版本下载。这里我们以ARM构架的MacOS为例（MacBook Air M1 2020）。
2. 将文件解压缩到指定地址，此处个人习惯地址为`/usr/local/`：
   ```sh
   tar -xzf Downloads/elasticsearch-7.16.2-darwin-aarch64.tar.gz -C /usr/local
   ```
3. 确保安装了`JDK1.8`以上版本，同时确保正确设置了变量环境：
   ```sh
   # 查看变量
   echo $JAVA_HOME
   # 如果没有返回任何值，则查看<java-path>
   /usr/libexec/java_home
   # 设置环境变量
   export JAVA_HOME=<java-path>
   ```
4. 接下来可以直接通过binary file来启动ES：
   ```sh
   /usr/local/elasticsearch-7.16.2/bin/elasticsearch
   ```
   或者在shell配置文件中添加 `alias elasticsearch="/usr/local/elasticsearch-7.16.2/bin/elasticsearch"` 方便启动
5. 如果启动过程中遇到`AccessDenied`报错，则需要设置elasticsearch dir权限：
    ```sh
    chown -R $USER:staff /usr/local/elasticsearch-7.16.2
    ```
6. 默认本地端口为`http://localhost:9200/`，若ES启动成功进入页面后会看到如下信息：
   ```json
   {
       "name" : "YOUR_DEVICE_NAME",
       "cluster_name" : "elasticsearch",
       "cluster_uuid" : "abcdefghijklmn",
       "version" : {
           "number" : "7.16.2",
           "build_flavor" : "default",
           "build_type" : "tar",
           "build_hash" : "123456abcdef",
           "build_date" : "2021-12-18T19:42:46.604893745Z",
           "build_snapshot" : false,
           "lucene_version" : "8.10.1",
           "minimum_wire_compatibility_version" : "6.8.0",
           "minimum_index_compatibility_version" : "6.0.0-beta1"
        },
        "tagline" : "You Know, for Search"
    }
   ```

### Elasticsearch下都有什么？            
```
./
  - bin/* 启动文件 binary files
  - config/ 配置文件
     - elasticsearch.yml ES配置文件
     - jvm.options Java虚拟机配置
     - log4j2.properties 日志配置文件
  - data/* 数据
  - lib/* 依赖jar包
  - logs/* 日志
  - modules/* 功能模块
  - plugins/* ES插件
```

### 配置Elasticsearch
- `解决跨域问题`：ES的默认端口是`9200`，为了允许被其他端口访问，我们需要进行基础的配置：在elasticsearch.yml中添加
    ```yml
    https.cors.enabled: true
    https.cors.allow-origin: "*"
    ```

### 安装Elasticsearch-head
Elasticsearch-head是ES集群的web front-end，可以提供ES的可视化。使用Google chrome浏览器可以直接安装[Chrome插件](https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm)， 也可以参考[https://github.com/mobz/elasticsearch-head](https://github.com/mobz/elasticsearch-head)。Es-head的默认端口是`http://localhost:9100/`。

### 安装IK分词器
以英文为代表的印欧语系一般以`单词(word)`为最小单位，使用空格作为天然的分隔符。而中文的最小单位是`字(character)`，词语之间没有分割。我们在处理中文语句时`分词(Tokenization)`的难度往往要更大一些。

Elasticsearch默认使用stanford分词器，也能对中文分词，但效果并不好：

```json
// Input
GET _analyze
{
  "analyzer": "default",
  "text": "搜索引擎"
}
// Results
{
  "tokens" : [
    {
      "token" : "搜",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "索",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "引",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "擎",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    }
  ]
}
```

我们可以使用基于 [Lucene IK analyzer](https://code.google.com/archive/p/ik-analyzer/) 的 `IK Analysis plugin` 更好的进行分词。IK插件提供了`ik_smart`和`ik_max_word`两种分词器：

**ik_smart** 做最粗粒度的拆分，即最小词数：

```json
// input
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "搜索引擎"
}
// results
{
  "tokens" : [
    {
      "token" : "搜索引擎",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 0
    }
  ]
}
```

**ik_max_word** 做最细粒度的拆分，即最大词数：

```json
// input
GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "搜索引擎"
}
// results
{
  "tokens" : [
    {
      "token" : "搜索引擎",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "搜索",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "索引",
      "start_offset" : 1,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "引擎",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 3
    }
  ]
}
```

IK分词插件的安装方法可以在其 [Github Repo](https://github.com/medcl/elasticsearch-analysis-ik) 上找到，这里建议使用 option2 `elasticsearch-plugin`来安装。请特别注意插件版本需要与Elasticsearch版本对应，否则安装过程中报错

![es-head](/figures/es-head.png)

## 安装Kibana
***
1. 进入Kibana[官方下载页面](https://www.elastic.co/cn/downloads/kibana)，选择适合你系统的版本下载。这里我们以ARM构架的MacOS为例（MacBook Air M1 2020）。
2. 将文件解压缩到指定地址，此处个人习惯地址为`/usr/local/`：
   ```sh
   tar -xzf Downloads/kibana-7.16.2-darwin-aarch64.tar.gz -C /usr/local
   ```
3. 接下来可以直接通过binary file来启动ES：
   ```sh
   /usr/local/kibana-7.16.2/bin/kibana
   ```
   或者在shell配置文件中添加 `alias kibana="/usr/local/kibana-7.16.2-darwin-aarch64/bin/kibana"` 方便启动
4. 如果启动过程中遇到`AccessDenied`报错，则需要设置elasticsearch dir权限：
    ```sh
    chown -R $USER:staff /usr/local/kibana-7.16.2-darwin-aarch64
    ```

[下一篇：Elasticsearch核心概念](/notes/es_concepts.md)