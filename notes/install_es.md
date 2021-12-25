# 安装和配置

[上一篇：Elasticsearch 简介](/notes/es_basics.md)

## 安装 Elasticsearch

1. 进入 Elasticsearch [官方下载页面](https://www.elastic.co/cn/downloads/elasticsearch)，选择适合你系统的版本下载。这里我们以ARM构架的 MacOS 为例（MacBook Air M1 2020）。
2. 将文件解压缩到指定地址，此处个人习惯地址为 `/usr/local/`：
   ```sh
   tar -xzf Downloads/elasticsearch-7.16.2-darwin-aarch64.tar.gz -C /usr/local
   ```
3. 确保安装了 `JDK1.8` 以上版本，同时确保正确设置了变量环境：
   ```sh
   # 查看变量
   echo $JAVA_HOME
   # 如果没有返回任何值，则查看<java-path>
   /usr/libexec/java_home
   # 设置环境变量
   export JAVA_HOME=<java-path>
   ```
4. 接下来可以直接通过 binary file 来启动 Elasticsearch：
   ```sh
   /usr/local/elasticsearch-7.16.2/bin/elasticsearch
   ```
   或者在shell配置文件中添加 `alias elasticsearch="/usr/local/elasticsearch-7.16.2/bin/elasticsearch"` 方便启动
5. 如果启动过程中遇到 `AccessDenied` 报错，则需要设置 elasticsearch dir 权限：
    ```sh
    chown -R $USER:staff /usr/local/elasticsearch-7.16.2
    ```
6. 默认本地端口为 `http://localhost:9200/`，若ES启动成功进入页面后会看到如下信息：
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

### Elasticsearch 目录下都有什么？            
```
./
  - bin/* 启动文件 binary files
  - config/ 配置文件
     - elasticsearch.yml Elasticsearch配置文件
     - jvm.options Java虚拟机配置
     - log4j2.properties 日志配置文件
  - data/* 数据
  - lib/* 依赖jar包
  - logs/* 日志
  - modules/* 功能模块
  - plugins/* ES插件
```

### 配置 Elasticsearch
- `设置集群名`：
    ```yml
    cluster.name: <diy-name>
    ```
- `解决跨域问题`：Elasticsearch 的默认端口是 `9200`，为了允许被其他端口访问，我们需要进行基础的配置：在 `elasticsearch.yml` 中添加
    ```yml
    https.cors.enabled: true
    https.cors.allow-origin: "*"
    ```

### 安装 Elasticsearch-head
Elasticsearch-head 是 Elasticsearch 集群的 web front-end，可以提供 Elasticsearch 的可视化。使用 Google chrome 浏览器可以直接安装 [Chrome插件](https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm)， 也可以参考[https://github.com/mobz/elasticsearch-head](https://github.com/mobz/elasticsearch-head)。Head 的默认端口是 `http://localhost:9100/`。

### 安装IK分词器

IK分词插件的安装方法可以在其 [Github Repo](https://github.com/medcl/elasticsearch-analysis-ik) 上找到，这里建议使用 option2 `elasticsearch-plugin` 来安装。请特别注意插件版本需要与 Elasticsearch 版本对应，否则安装过程中报错

![es-head](/figures/es-head.png)

## 安装 Kibana

1. 进入 Kibana[ 官方下载页面](https://www.elastic.co/cn/downloads/kibana)，选择适合你系统的版本下载。这里我们以ARM构架的 MacOS 为例（MacBook Air M1 2020）。
2. 将文件解压缩到指定地址，此处个人习惯地址为 `/usr/local/`：
   ```sh
   tar -xzf Downloads/kibana-7.16.2-darwin-aarch64.tar.gz -C /usr/local
   ```
3. 接下来可以直接通过 binary file 来启动 Elasticsearch：
   ```sh
   /usr/local/kibana-7.16.2/bin/kibana
   ```
   或者在shell配置文件中添加 `alias kibana="/usr/local/kibana-7.16.2-darwin-aarch64/bin/kibana"` 方便启动
4. 如果启动过程中遇到 `AccessDenied` 报错，则需要设置 elasticsearch dir 权限：
    ```sh
    chown -R $USER:staff /usr/local/kibana-7.16.2-darwin-aarch64
    ```

[下一篇：Elasticsearch 核心概念](/notes/es_concepts.md)