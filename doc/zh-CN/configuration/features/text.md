# 文本匹配与神经特征

> 英文原文：[`doc/configuration/features/text.md`](../../../configuration/features/text.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

## `field_match`

`field_match` 用于把排序事件中的字段与内容字段匹配。典型场景是在搜索任务中，把 `query` 分别与标题、标签、类别和描述计算相关性。

支持的匹配方式：

- `bm25`：使用 Lucene 计算查询字段与内容字段之间的 BM25 分数；
- `ngram`：拆分为 N-gram 后计算交并比；
- `term`：使用 Lucene 进行语言相关的分词和词项匹配；
- `bi-encoder`：分别生成查询与内容向量并计算距离；
- `cross-encoder`：把查询与内容一起送入模型，直接计算匹配分数。

## 数据示例

内容元数据：

```json
{
  "event": "item",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "item": "item1",
  "timestamp": "1599391467000",
  "fields": [
    {"name": "title", "value": "red socks"},
    {"name": "category", "value": "socks"},
    {"name": "brand", "value": "buffalo"},
    {"name": "description", "value": "lorem ipsum dolores sit amet"}
  ]
}
```

排序上下文：

```json
{
  "event": "ranking",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "timestamp": "1599391467000",
  "user": "user1",
  "session": "session1",
  "fields": [
    {"name": "query", "value": "sock"}
  ],
  "items": [
    {"id": "item3"},
    {"id": "item1"},
    {"id": "item2"}
  ]
}
```

## BM25

BM25 需要词频和索引统计。使用前先通过 CLI 生成 `term-freq.json`，见英文 [`termfreq` 文档](../../../cli.md#bm25-term-frequencies-dictionary)。

```yaml
- name: title_match
  type: field_match
  rankingField: ranking.query
  itemField: item.title
  method:
    type: bm25
    language: english
    termFreq: "/path/to/term-freq.json"
```

## N-gram 匹配

以下配置使用 3-gram 比较 `ranking.query` 与 `item.title`：

```yaml
- name: title_match
  type: field_match
  itemField: item.title
  rankingField: ranking.query
  method:
    type: ngram
    language: en
    n: 3
  refresh: 0s
  ttl: 90d
```

## 词项匹配

```yaml
- name: title_match
  type: field_match
  itemField: item.title
  rankingField: ranking.query
  method:
    type: term
    language: en
```

`term` 和 `ngram` 都使用 Lucene 文本分析。支持：

| 代码 | 语言 |
|---|---|
| `generic` | 不做语言相关转换 |
| `en` | 英语 |
| `cz` | 捷克语 |
| `da` | 丹麦语 |
| `nl` | 荷兰语 |
| `et` | 爱沙尼亚语 |
| `fi` | 芬兰语 |
| `fr` | 法语 |
| `de` | 德语 |
| `gr` | 希腊语 |
| `it` | 意大利语 |
| `no` | 挪威语 |
| `pl` | 波兰语 |
| `pt` | 葡萄牙语 |
| `es` | 西班牙语 |
| `sv` | 瑞典语 |
| `tr` | 土耳其语 |
| `ar` | 阿拉伯语 |
| `zh` | 中文 |
| `ja` | 日语 |

处理流程包括分词、去停用词、对非 `generic` 语言进行词干化，然后根据内容与查询的词项或 N-gram 交并情况计算分数。

## Bi-Encoder

Bi-Encoder 分别计算查询和内容的 Embedding，再计算余弦或点积距离。语义更接近的查询—内容组合通常得到更高相似度。

```yaml
- type: field_match
  name: title_query_match
  rankingField: ranking.query
  itemField: item.title
  distance: cos
  method:
    type: bi-encoder
    model: metarank/all-MiniLM-L6-v2
    dim: 384
    itemFieldCache: /path/to/item.embedding
    rankingFieldCache: /path/to/query.embedding
```

- `distance`：可选，默认 `cos`，也可使用 `dot`；
- `model`：可选的 ONNX 模型；
- `dim`：必填，Embedding 维度；
- `itemFieldCache`、`rankingFieldCache`：可选的预计算缓存。

Metarank 支持从 Hugging Face Hub 获取模型，或从本地目录加载：

- `namespace/model`：从 Hub 获取；
- `file:///<path>/<to>/<model dir>`：加载本地 ONNX 模型。

可用模型可参考 [Metarank Hugging Face](https://huggingface.co/metarank)。

### 使用 CSV 预计算向量

性能敏感场景可以只使用离线预计算向量，而不在请求时编码：

```yaml
- type: field_match
  name: title_query_match
  rankingField: ranking.query
  itemField: item.title
  distance: cos
  method:
    type: bi-encoder
    dim: 384
    itemFieldCache: /path/to/item.embedding
    rankingFieldCache: /path/to/query.embedding
```

内容向量文件：

```text
item1,0,1,2,3,4,5
item2,5,4,3,2,1,9
item3,1,1,1,1,1,1
```

查询向量文件：

```text
bananas,0,1,2,3,4,5
red socks,5,4,3,2,1,9
stone,1,1,1,1,1,1
```

- 查询和内容向量都存在时，计算指定距离；
- 至少一侧缺失且只使用缓存时，产生缺失值；
- 配置在线模型时，缺失向量可以在请求时计算。

## Cross-Encoder

Cross-Encoder 不分别生成向量，而是把查询与内容文本一起输入网络，直接输出匹配分数。

与 Bi-Encoder 相比：

- 通常精度更高；
- 无法独立预计算通用内容向量；
- 每个查询—内容组合都需要推理，资源消耗更高。

```yaml
- type: field_match
  name: title_query_match
  rankingField: ranking.query
  itemField: item.title
  method:
    type: cross-encoder
    model: metarank/ce-msmarco-MiniLM-L6-v2
    cache: /path/to/ce.cache
```

可以离线预计算部分查询—内容得分并通过 CSV 缓存提供：

```text
query1,doc1,0.7
query1,doc2,0.1
query2,doc3,0.2
```

## 选择建议

| 方法 | 适用场景 | 主要代价 |
|---|---|---|
| BM25 | 关键词检索、已有词频统计 | 需要词频字典 |
| N-gram | 拼写、短文本和局部匹配 | 语义能力有限 |
| Term | 语言相关词项匹配 | 依赖分词质量 |
| Bi-Encoder | 大候选集语义匹配 | 需要向量和索引/缓存管理 |
| Cross-Encoder | 小候选集高精度重排 | 请求时计算成本高 |
