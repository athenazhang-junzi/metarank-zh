# Metarank JSON API

> 英文原文：[`doc/api.md`](../api.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`  
> 版本说明：本文覆盖全部主要端点、请求字段和响应字段，压缩了重复的 HTTP 握手输出。

Metarank API 用于把排序服务接入应用：

- [`/feedback`](#反馈)：接收内容、用户、排序和交互事件；
- [`/train/<model name>`](#训练)：使用持久化点击链训练模型；
- [`/rank/<model name>`](#排序)：对显式传入的候选内容进行个性化重排；
- [`/recommend/<model-name>`](#推荐)：从推荐模型中召回内容；
- [`/flush`](#刷新会话缓冲区)：立即结束缓冲区中的点击链；
- [`/inference/...`](#llm-推理)：文本编码和 Cross-Encoder 打分；
- [`/metrics`](#prometheus-指标)：输出应用与 JVM 监控指标。

## 反馈

**端点**：`POST /feedback`

该端点接收 `item`、`user`、`ranking` 和 `interaction` 事件。完整结构见[事件格式](event-schema.md)。

排序和交互会先在会话缓冲区中合并为点击链。默认情况下，某个会话连续 `30m` 没有活动后，点击链才会结束、写入训练存储，并依据级联点击模型生成合成 `impression`。该时间由 `core.clickthrough.maxSessionLength` 控制。

需要立即结束缓冲区内容时，可以调用 [`/flush`](#刷新会话缓冲区)。

### 响应

```json
{"accepted":1,"status":"ok","tookMillis":3,"updated":0}
```

- `accepted`：本批成功处理的事件数；
- `status`：无错误时为 `ok`；
- `tookMillis`：处理本批数据所用毫秒数；
- `updated`：重新计算的底层排序特征数量。

### 示例

```bash
curl http://localhost:8080/feedback \
  -H 'Content-Type: application/json' \
  -d '{
    "event": "ranking",
    "id": "ranking-1",
    "items": [
      {"id":"72998"}, {"id":"589"}, {"id":"134130"}
    ],
    "user": "alice",
    "session": "alice1",
    "timestamp": 1661431894711
  }'
```

## 训练

**端点**：`POST /train/<model name>`

**请求体**：无

该端点使用已持久化的点击链数据训练指定模型，可以在需要时重新调用。自动化重训练方法见英文[模型重训练指南](../howto/model-retraining.md)。

### 响应

响应包含：

- `features`：各特征及其模型权重；
- `iterations`：每轮训练耗时、测试指标和训练指标；
- `sizeBytes`：模型大小。

```json
{
  "features": [
    {"name": "vote_avg", "weight": 629.0},
    {"name": "profile", "weight": [1202.0, 373.0, 627.0, 145.0]}
  ],
  "iterations": [
    {
      "id": 0,
      "millis": 274,
      "testMetric": 0.5787768851757988,
      "trainMetric": 0.593075630098252
    }
  ],
  "sizeBytes": 843792
}
```

## 排序

**端点**：`POST /rank/<model name>`

**查询参数**：

- `explain: boolean`：设为 `true` 时，在响应中附带计算后的特征值。

排序端点对请求中的候选内容进行个性化重排。调用方必须在 URL 中明确指定模型名称。

### 请求

```json
{
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "timestamp": "1599391467000",
  "user": "user1",
  "session": "session1",
  "fields": [
    {"name": "query", "value": "jeans"},
    {"name": "source", "value": "search"}
  ],
  "items": [
    {"id": "item3", "fields": [{"name": "relevancy", "value": 2.0}]},
    {"id": "item1", "fields": [{"name": "relevancy", "value": 1.0}]},
    {"id": "item2", "fields": [{"name": "relevancy", "value": 0.1}]}
  ]
}
```

- `id`：请求标识。之后提交 `ranking` 事件及对应 `interaction.ranking` 时使用相同值；
- `user`：可选的访客 ID；
- `session`：可选的会话 ID；
- `timestamp`：事件发生时间；
- `fields`：可选的排序级上下文；
- `items`：待重排的候选内容；
- `items.id`：内容 ID，应与 `item` 元数据事件一致；
- `items.fields`：可选的内容级上下文，例如 Elasticsearch 产生的 BM25 分数；
- `items.label`：可选的显式相关性判断。

### 响应

```json
{
  "took": 5,
  "items": [
    {
      "item": "item2",
      "score": 2.0,
      "features": [{"name": "popularity", "value": 10}]
    },
    {
      "item": "item3",
      "score": 1.0,
      "features": [{"name": "popularity", "value": 5}]
    }
  ]
}
```

- `took`：处理请求所用毫秒数；
- `items.item`：内容 ID；
- `items.score`：个性化模型计算的排序分数；
- `items.features`：模型计算的特征值，仅在请求启用 `explain=true` 时返回，结构随特征类型变化。

> 模型分数主要用于同一次请求中的排序。除非模型与校准过程明确支持，否则不要把它直接解释为点击概率。

## 推荐

**端点**：`POST /recommend/<model-name>`

该端点调用[推荐模型](../configuration/recommendations.md)召回推荐内容。

### 请求

```json
{
  "count": 10,
  "user": "alice1",
  "items": ["item4"]
}
```

- `count`：返回内容数量；
- `user`：可选的当前用户 ID；
- `items`：推荐上下文。相似内容场景可以只传一个内容，购物车式推荐可以同时传多个内容。

### 响应

```json
{
  "took": 5,
  "items": [
    {"item": "item2", "score": 2.0},
    {"item": "item3", "score": 1.0},
    {"item": "item1", "score": 0.5}
  ]
}
```

- `took`：处理请求所用毫秒数；
- `items.item`：内容 ID；
- `items.score`：推荐模型计算的分数。

## 刷新会话缓冲区

**端点**：`POST /flush`

**请求体**：无

该端点不等待 `core.clickthrough.maxSessionLength` 超时，立即结束当前会话缓冲区中的全部点击链，将它们写入训练存储，并生成合成 `impression`。

它适合：

- 演示；
- 集成测试；
- 明确知道不会再有交互到达的批量发送程序。

`/flush` 会结束**所有**尚未关闭的会话。若刷新之后又有某次排序对应的交互到达，该交互会形成退化的单内容点击链。因此只应在相关会话确实结束时调用。

响应格式与 `/feedback` 相同：

```json
{"accepted":0,"status":"ok","tookMillis":2,"updated":4}
```

## LLM 推理

Metarank 提供文本编码和 Cross-Encoder 打分 API，可用于混合搜索等场景。

### Bi-Encoder 文本编码

**端点**：`POST /inference/encoder/<name>`

使用配置中的 `<name>` 模型把一批文本编码为向量。

```json
{
  "texts": [
    "Berlin is a capital city",
    "My cat is fast"
  ]
}
```

```json
{
  "took": 5,
  "embeddings": [
    [1, 2, 3, 4],
    [0, 7, 2, 1]
  ]
}
```

### Cross-Encoder 相似度

**端点**：`POST /inference/cross/<name>`

使用配置中的 `<name>` 模型，为一批查询与文档组合计算相似度。

```json
{
  "input": [
    {"query": "cat", "text": "my cat is fast"},
    {"query": "cat", "text": "it has V8 engine"}
  ]
}
```

```json
{
  "took": 5,
  "scores": [0.25, 0.01]
}
```

## Prometheus 指标

**端点**：`GET /metrics`

该端点以 Prometheus 格式输出应用和 JVM 指标，例如反馈事件量、排序请求量及排序延迟。

```text
# HELP metarank_feedback_events_total Number of feedback events received
# TYPE metarank_feedback_events_total counter
metarank_feedback_events_total 58441.0

# HELP metarank_rank_requests_total Number of /rank requests
# TYPE metarank_rank_requests_total counter
metarank_rank_requests_total{model="xgboost",} 5.0
```

## 接入顺序建议

1. 先通过 `/feedback` 写入 `item` 和可选的 `user` 元数据；
2. 调用 `/rank/<model>` 对候选集重排；
3. 应用展示结果后回传对应 `ranking`；
4. 用户产生行为后回传关联的 `interaction`；
5. 确认点击链正确进入训练存储；
6. 调用 `/train/<model>` 或配置定期重训练；
7. 使用 `/metrics` 监控运行状态。
