# 计数与比率特征

> 英文原文：[`doc/configuration/features/counters.md`](../../../configuration/features/counters.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

事件计数是简单而重要的排序信号，但实现时需要处理两个问题：

- 计数会随时间持续变化。训练和回测必须保留每个历史时点的计数，而不能把当前值泄漏到过去；
- 用户或会话级累计计数可以一直增加，但内容热度通常需要时间窗口。生命周期内累计 100 次点击，与昨天获得 100 次点击代表不同含义。

Metarank 提供两类计数器：

- `interaction_count`：持续累加的交互计数，适合会话内点击数、购物车大小等；
- `window_count`：按时间桶和窗口聚合，适合内容热度。

两者都可以设置 `interaction: impression`，统计 Metarank 依据点击链自动产生的合成曝光。`impression` 是[保留交互类型](../../event-schema.md#保留交互类型impression)，不要自行发送同名事件，否则会重复计数。

## 交互计数器

```yaml
- name: click_count
  type: interaction_count
  scope: item
  interaction: click
  refresh: 60s
  ttl: 90d
```

- `scope` 可以是内容、用户或会话；
- `interaction` 指定要统计的交互类型；
- `refresh` 控制在线推理读取值的更新频率，默认 `0s`；
- `ttl` 控制长期无更新时保留多久。

提高 `refresh` 可以降低特征存储的写入压力，但推理读取到的值可能稍旧。

## 窗口计数器

```yaml
- name: clicks
  type: window_count
  interaction: click
  scope: item
  count: 10
  bucket_size: 24h
  windows: [7, 14, 30, 60]
  refresh: 60s
  ttl: 90d
```

该配置会产生一组特征：

```text
clicks_7: 12
clicks_14: 34
clicks_30: 70
clicks_60: 124
```

- `count`：可选，只使用最近 N 次交互；
- `bucket_size`：单个时间桶大小；
- `windows`：聚合多少个时间桶；
- `refresh`：最短刷新周期。

窗口计数器内部使用环形缓冲区，每个时间桶对应一个计数器，数量由最大窗口决定。计数更新很频繁时，每次交互都重新聚合窗口会增加计算开销，通常可以把刷新周期设为数分钟。

## `rate`：比率特征

`rate` 用于计算点击率、转化率等“一个交互计数除以另一个交互计数”的特征。

```yaml
- name: ctr
  type: rate
  top: click
  bottom: impression
  bucket: 24h
  periods: [7, 30]
  scope: item
  refresh: 1h
  ttl: 90d
  normalize:
    weight: 10
```

- `top`：分子交互类型；
- `bottom`：分母交互类型；
- `bucket`：时间桶大小；
- `periods`：聚合周期；
- `scope`：聚合键；
- `normalize`：可选的低样本平滑配置。

示例使用合成 `impression` 作为分母。其生成逻辑见英文[点击模型文档](../../../click-models.md)。

## 比率归一化

低样本比率非常不稳定：

- 100 次曝光、50 次点击，CTR 为 50%；
- 2 次曝光、1 次点击，CTR 也为 50%。

两者点估计相同，但可信度不同。Metarank 可以把单内容比率与全局先验混合：

```text
rate = (a + item_clicks) / (b + item_impressions)
```

其中 `a / b` 与全局先验 CTR 成比例。使用固定缩放因子 `w` 时：

```text
item_ctr = (w + item_clicks) /
           (w * (global_impressions / global_clicks) + item_impressions)
```

假设全局有 100 次曝光、10 次点击，内容 A 有 2 次曝光、1 次点击：

| weight | 全局曝光 | 全局点击 | 内容曝光 | 内容点击 | 全局 CTR | 原始内容 CTR | 归一化 CTR |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 10 | 100 | 10 | 2 | 1 | 10.00% | 50.00% | 10.78% |
| 1 | 100 | 10 | 2 | 1 | 10.00% | 50.00% | 16.67% |
| 10 | 100 | 10 | 10 | 3 | 10.00% | 30.00% | 11.82% |
| 10 | 20 | 5 | 10 | 3 | 25.00% | 30.00% | 26.00% |

`weight` 的合适取值取决于数据：

- 常见范围为 5–10；
- 离群 CTR 越多，可以使用越高的权重；
- 默认值为 10。

这类平滑可以减少低点击量内容的极端比率，但不能代替置信区间、冷启动策略或在线实验。

## 按内容字段聚合

设置 `scope: item.<field_name>` 后，可以按内容字段值聚合比率。例如按品牌计算 CTR：

```yaml
- name: ctr
  type: rate
  top: click
  bottom: impression
  bucket: 24h
  periods: [7, 30]
  scope: item.brand
```

这种方法可以缓解单内容交互不足的冷启动问题，例如按类别、颜色或品牌聚合。

限制：

- 支持归一化；
- 聚合字段应为低基数；
- 内容字段必须是字符串，不支持数组。

## 按排序字段与内容聚合

设置 `scope: ranking.<field_name>` 后，可以在排序上下文和内容 ID 的组合上计算比率。例如针对每个查询词计算单内容 CTR：

```yaml
- name: ctr
  type: rate
  top: click
  bottom: impression
  bucket: 24h
  periods: [7, 30]
  scope: ranking.query
```

限制同样包括：字段应为低基数字符串，不支持数组。高基数查询词会显著增加状态规模并造成数据稀疏。
