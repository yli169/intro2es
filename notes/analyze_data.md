# 分析 Analyzing Data

Elasticsearch 在将 document 倒排索引前，会 `analyze` document 主体：
- `Character filtering`：transform & filter characters；
- `Tokenization`: Breaking text into tokens；
- `Token filtering`：transform & filter tokens；
- `Token indexing`：在 index 中储存 tokens；

![example_6](/figures/example_6.png)

## 配置 Analyzer

配置 analyzer 的方式有：
- 创建 index 时在 settings 配置：
    ```json
    {
        "settings": {
            "index": {
                "analysis": {
                    "analyzer": {
                        "customAnalyzer": {
                            "type": "custom",
                            "filter": ["customFilter1", "customFilter2"],
                            "char_filter": ["customCharFilter"]
                        }
                    }
                }
            }
        }
    }
    ```

- 在 mapping 中 设定 field 的 analyzer：
    ```json
    "mappings":{
        "properties":{
            "name": {
                "type":"text",
                "analyzer":"customAnalyzer"
            }
        }
    }
    ```

- 在 elasticsearch.yml 配置 global analyzer：
    ```yml
    index:
      analysis:
        analyzer:
          customAnalyer:
            type: custom
            tokenizer: customTokenizer
            filter: [customFilter1, customFilter2],
            char_filter: customCharFilter
    ```


以英文为代表的印欧语系一般以`单词(word)`为最小单位，使用空格作为天然的分隔符。而中文的最小单位是`字(character)`，词语之间没有分割。我们在处理中文语句时`分词(Tokenization)`的难度往往要更大一些。

## 基础 Token Filters
- `lowercase`
- `length` 在给定长度 range 外的 token 都会被去除；
- `stop`：去除 stop-words；
- `truncate`：将 token 截断至给定长度；
- `trim`：去掉 token 前后空格；
- `limit token count`：设定给定 field 可以保存的最多 token 数；
- `reverse`：将每个 token 进行反转。比如将 *\*bar* 反转为 *rab\** 匹配起来会更快；
- `unique`：对重复的 token 只保留第一个；
- `ascii folding`：将其它编码转为 ASCII 编码；
- `synonym`：将 token 转为 同义词，如*automobile* -> *car*；

## 基础 Tokenizers

- `standard`：基于语法分词，较适用于欧州语言，最多保留 255 个 token，去除标点符号；
- `keyword`：不分词，直接将输入作为输出；
- `letter`：在非字母处分词；
- `lowercase`：
  - letter tokenizer +
  - lowercase filter;
- `whitespace`：在空格处分词；
- `pattern`：给定特定 pattern 进行分词
- `uax_url_email`：对 *address@example.com* 分词；
- `path_hierarchy`：对 */path/to/field* 路径进行分词；

## 内置 Analyzers

- `standard`: 
  - standard tokenizer +
  - standard token filter +
  - lowercase token filter +
  - stop filter;
- `simple`:
  - lowercase tokenizer +
  - split at nonletters;
- `whitespace`: 根据空格分词；
- `stop`：filter 掉 stop-words；
- `keyword`：提取整个 field 并生成单个 token；
- `pattern`：根据 pattern 分词；
- `snowball`：
  - standard tokenizer+
  - stop filter +
  - lowercase token filter +
  - snowball stemmer +
- LANGUAGE & MULTILINGUAL：支持多种语言；

## IK 分词器
Elasticsearch 默认使用 Standard 分词器，也能对中文分词，但效果并不好：

Request:
```
GET _analyze
{
  "analyzer": "default",
  "text": "搜索引擎"
}
```
Response:
```
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

我们可以使用基于 [Lucene IK analyzer](https://code.google.com/archive/p/ik-analyzer/) 的 `IK Analysis plugin` 更好的进行分词。IK插件提供了 `ik_smart` 和 `ik_max_word` 两种分词器：

**ik_smart** 做最粗粒度的拆分，即最小词数：

Request:
```
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "搜索引擎"
}
```
Response:
```
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

Request:
```
GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "搜索引擎"
}
```
Response:
```
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

## Stemming
将单词还原为词根，一般方法是：
- stemming 算法：
  - [stemmer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stemmer-tokenfilter.html)
  - [snowball](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-snowball-tokenfilter.html)
  - [porter_stem](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-porterstem-tokenfilter.html)
  - [`kstem`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-kstem-tokenfilter.html)
- [Stemming with dictionaries](https://www.elastic.co/guide/en/elasticsearch/reference/current/stemming.html#dictionary-stemmers)