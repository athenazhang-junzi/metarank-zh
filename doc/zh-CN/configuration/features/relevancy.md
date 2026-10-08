# 相关性与位置特征

> 英文原文：[`doc/configuration/features/relevancy.md`](../../../configuration/features/relevancy.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

## 上游相关性

Metarank 的主要定位是卫星式二次重排服务。它假设已有上游系统产生候选内容，例如：

- 搜索：Elasticsearch、OpenSearch 或 Solr；
- 推荐：Spark MLlib ALS 等推荐模型输出；
- 电商：库存或商品数据库。

上游通常也会为每个候选产生相关性分数，例如：

- 搜索中的 BM25 或 TF-IDF；
- 推荐中的 Embedding 余弦相似度。

这些分数可以放入 `ranking.items[].fields`：

```json
{
  "event": "ranking",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "timestamp": "1599391467000",
  "user": "user1",
  "session": "session1",
  "fields": [
    {"name": "query", "value": "cat"},
    {"name": "source", "value": "search"}
  ],
  "items": [
    {"id": "item3", "fields": [{"name": "relevancy", "value": 2.0}]},
    {"id": "item1", "fields": [{"name": "relevancy", "value": 1.0}]},
    {"id": "item2", "fields": [{"name": "relevancy", "value": 0.1}]}
  ]
}
```

Metarank 0.5.11 及以前版本提供了现已弃用的 `relevancy` 提取器。0.5.12 以后应使用普通数值提取器：

```yaml
- name: relevancy
  type: number
  scope: item
  field: item.relevancy
```

数值提取器的详细行为见英文 [`scalar.md`](../../../configuration/features/scalar.md)。

## 多路召回

混合搜索可能同时使用：

- Elasticsearch/OpenSearch/Solr 的词项检索及 BM25；
- Pinecone、Vespa、Qdrant 等向量检索及余弦相似度。

应用从多个召回器获取 Top-N，合并后交给 Metarank。某些内容同时被两路召回，另一些只来自其中一路：

```json
{
  "event": "ranking",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "timestamp": "1599391467000",
  "user": "user1",
  "session": "session1",
  "fields": [
    {"name": "query", "value": "cat"}
  ],
  "items": [
    {"id": "item3", "fields": [{"name": "bm25", "value": 2.0}]},
    {
      "id": "item1",
      "fields": [
        {"name": "bm25", "value": 1.0},
        {"name": "cos", "value": 0.75}
      ]
    },
    {"id": "item2", "fields": [{"name": "cos", "value": 0.02}]}
  ]
}
```

- `item1` 同时来自词项检索与向量检索；
- `item2` 只来自向量检索；
- `item3` 只来自词项检索。

未被某一路召回时可以缺少对应分数。XGBoost 和 LightGBM 都支持缺失值，但训练数据中必须保留这种缺失模式，不能随意用 0 替代而改变语义。

## `position`：位置特征

`position` 是一种位置偏差处理方法，思路来自 PAL（Position-bias Aware Learning）：

- 离线训练时，特征值等于内容在历史排序中的真实位置；
- 在线推理时，所有候选使用相同的固定位置值。

模型在训练中学习位置对交互的影响；推理时把候选位置设为同一个常数，使位置因素对所有候选保持一致。

```yaml
- type: position
  name: position
  position: 5
```

选择固定值时可以：

1. 从平均排序长度的中间位置开始，例如平均展示 20 个内容时从 10 开始；
2. 在附近做网格搜索；
3. 使用 `metarank standalone` 比较离线 NDCG；
4. 同时检查时间切分、历史位置偏差和其他业务护栏。

位置特征可以降低部分偏差，但不能自动消除历史曝光选择偏差或证明线上无偏效果。
