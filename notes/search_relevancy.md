# 搜索相关性

[上一篇：搜索 Searching Data](/notes/search_data.md)

当我们进行搜索时，每一篇匹配出的文档都会有一个对应的 `_score`，来衡量 document 和 query 的相关性。默认情况下，返回的文档会根据分数来排序。默认分数计算为 `TFIDFSimilarity`。

## TFIDFSimilarity

### TF-IDF

`TF-IDF` [T]erm [F]requency–[I]nverse [D]ocument [F]requency 是一种统计方法，用以评价一个 `term` 对于 一个 corpus 中的 `document` 的重要性程度。
- Term Frequency 词频

    $$tf_{t,d}=\frac{f_{t,d}}{\sum_{t'\in d}f_{t',d}}$$

    $f_{t,d}$ 是 term `t` 在 document `d` 中的 count。分母是 document `d` 中所有词数的 count。

    当一个 term 在 一篇 document 中出现的次数越多，则它对这篇文章来说就越重要。


- Inverse Document Frequency 逆向文档频率
  
    $$idf_t=\log{\frac{|D|}{|{d \in D: t \in d}|}}$$

    分子是整个 corpus 文档的总数量，分母是包含了 term `t` 的 document `d` 的数量。

    当一个 term 出现在越多篇文档中，则用它来区分文档的能力就越低，一般来说我们更喜欢辨识度高的 terms。

TF-IDF 的公式为：

$$tfidf_{t,d}=tf_{t,d} \times idf_t$$

Lucene 使用了稍为不同的 formula：

$$score_{q,d}=coord(q,d) \times queryNorm(q) \times \sum_{t \in q}(tf_{t,d} \times (idf_t)^2 \times t.getBoost() \times norm(t,d))$$

- $coord(q,d)$ 协调因子：
    $$coord(q,d)=\frac{overlap}{maxOverlap}$$

    overlap 是文档命中 query 中 term 的个数，maxOverlap 是 query 中总共 term 的个数。文档命中 query 越多越相关。

- $queryNorm$ 查询正则因子：
    $$queryNorm(q) = \frac{1}{\sqrt{\sum_{t \in q}(idf_t)^2}}$$

    正则因子是为了能够让不同 query 之间互相可以比较，但由于排序是针对单个 query 的，所以这个因子对排序没有实际影响。

- $tf_{t,d}$ 词频：
    $$tf_{t,d}=\sqrt{\frac{f_{t,d}}{\sum_{t'\in d}f_{t',d}}}$$

- $idf_t$ 逆向文档频率：
    $$idf_t=1+\log{\frac{|D|+1}{|{d \in D: t \in d}|+1}}$$

- $t.getBoost()$ 搜索中使用的 term boost。可以使给定的 term(s) 拥有更高权重。

- $norm(t,d)$ 字段长度正则值与权重提升的乘积：
    $$norm(t,d)=lengthNorm \times d.getBoost() \times \prod_{f: t \in f + f'==f}f.getBoost() $$

    $lengthNorm = \frac{1}{\sqrt{|q|}}$

    这里 boost 的 fields 是 term hit 到的 fields 和文档中与其同名的所有 fields 的权重。（？这里我不是非常理解）

### 其它计算方法
- [Okapi BM25](https://lucene.apache.org/core/9_0_0/core/org/apache/lucene/search/similarities/BM25Similarity.html)
- [DFR similarity](https://lucene.apache.org/core/9_0_0/core/org/apache/lucene/search/similarities/DFRSimilarity.html)
- [IB similarity](https://lucene.apache.org/core/9_0_0/core/org/apache/lucene/search/similarities/IBSimilarity.html)
- [LM Dirichlet similarity](https://lucene.apache.org/core/9_0_0/core/org/apache/lucene/search/similarities/LMDirichletSimilarity.html)
- [LM Jelinek Mercer similarity](https://lucene.apache.org/core/9_0_0/core/org/apache/lucene/search/similarities/LMJelinekMercerSimilarity.html)

## Boosting

可以通过 `boosting` 来提升一篇 document 的relevance：
- 在 `index-time` 进行提升 (Deprecated in 5.0)
    ```
    PUT /<index>
    {
        "mappings": {
            "properties": {
                "<field>": {
                    "type": "<type>",
                    "boost": 2 
                }
            }
        }
    }
    ```
- 在 `query-time` 进行提升
    ```
    POST _search
    {
        "query": {
            "match": {
                "<field>": {
                    "query": "<query>",
                    "boost": 2
                }
            }
        }
    }
    ```

当进行 multi-fields 和 full-text 搜索时：
- 对整个 multi-match 进行 boost：
    ```
    POST /index/_search
    {
        "query": {
            "multi_match": {
                "query": "elasticsearch big data",
                "field": ["name", "description"],
                "boost": 2.0
            }
        }
    }
    ```
- 对特定 field 进行 boost：
    ```
    POST /index/_search
    {
        "query": {
            "multi_match": {
                "query": "elasticsearch big data",
                "field": ["name^2", "description"]            }
        }
    }
    ```
- 对特定 term(s) 进行 boost：
    ```
    POST /index/_search
    {
        "query": {
            "query_string": {
                "query": "elasticsearch^3 AND big data"
            }
        }
    }
    ```

## 理解 scoring 过程

- 通过设定 params `explain=true` 来查看打分细节：
    ```
    GET /_search
    {
        "query": {
            "match": {
                "name": "EMNLP"
                }
        },
        "explain": true
    }
    ```

- 理解为什么 document 没有被 match：
    ```
    GET /conference/_doc/emnlp/_explain
    {
        "query": {
            "match": {
            "name": "ACL"
            }
        }
    }
    ```

## 使用 Rescoring 降低 计算开销
当在一些搜索中计算 score 的开销过大，我们可以使用 `rescore` 的feature 降低开销：首先先进行简单的搜索在所有的 documents 上进行开销小的计算，提取 `Top k=window_size` 篇文档。之后在剩下的文档中进行开销大的计算。


```
GET /_search
{
    "query" : {
        "match" : {
            "message" : {
                "query" : "the quick brown"
            }
        }
    },
    "rescore" : {
        "window_size" : 50,
        "query" : {
            "rescore_query" : {
                "match_phrase" : {
                "message" : {
                    "query" : "the quick brown",
                    "slop" : 2
                }
                }
            },
            "query_weight" : 0.7,
            "rescore_query_weight" : 1.2
        }
    }
}
```

## 修改 Scoring

- 使用 [function_score](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html) 对 document 的 score 进行修改。

    ```
    GET /_search
    {
        "query": {
            "function_score": {
            "query": { "match_all": {} },
            "boost": "5", 
            "functions": [
                {
                "filter": { "match": { "test": "bar" } },
                "random_score": {}, 
                "weight": 23
                },
                {
                "filter": { "match": { "test": "cat" } },
                "weight": 42
                }
            ],
            "max_boost": 42,
            "score_mode": "max",
            "boost_mode": "multiply",
            "min_score": 42
            }
        }
    }
    ```

- 使用 `script_score` 的 [script](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html) 提供 custom scoring：
    ```
    GET /_search
    {
        "query": {
            "script_score": {
                "query": {
                    "match": { 
                        "message": "elasticsearch" 
                    }
                },
                "script": {
                    "source": "sigmoid(doc['my-int'].value/10)"
                }
            }
        }
    }
    ```
    scripting APIs 非常 flexible，你可以选择 built-in 的 methods 也可以自己写：
    - [Painless](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless.html) scripting language；
    - Lucene [expressions](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-expression.html) language；
    - Search [templates](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-template.html)；
    - Advanced [JAVA](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-engine.html) scripts using script engines；