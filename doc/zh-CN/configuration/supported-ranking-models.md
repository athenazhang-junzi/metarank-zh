# 支持的排序模型

> 英文原文：[`doc/configuration/supported-ranking-models.md`](../../configuration/supported-ranking-models.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

排序模型通过 `models.<name>.type` 配置：

```yaml
models:
  default:
    type: lambdamart
```

## LambdaMART

LambdaMART 是优化 [NDCG](https://en.wikipedia.org/wiki/Discounted_cumulative_gain) 的学习排序模型。简化后的训练过程是：

1. 输入一次排序以及候选内容的相关性判断；
2. 判断可以来自点击等隐式反馈，也可以来自评分等显式反馈；
3. 每个内容带有类别、CTR 等机器学习特征；
4. 从排序中抽取内容对；
5. 模型学习判断内容对中哪个更相关；
6. 在数据集中的内容对和排序上重复训练。

最终目标是让判断值更高的内容排在更靠前的位置。

Metarank 支持两种后端：

- XGBoost：`rank:pairwise` 目标；
- LightGBM：`lambdarank` 目标。

### 基本配置

```yaml
models:
  <model name>:
    type: lambdamart
    backend:
      type: xgboost
      iterations: 100
      seed: 0
    weights:
      click: 1
    features:
      - foo
      - bar

    # selector:
    #   rankingField: source
    #   value: search

    # split: time=80%
    # eval: ["NDCG@10"]

    # warmup:
    #   sampledRequests: 100
    #   duration: 5s
```

- `backend`：必填，选择 `xgboost` 或 `lightgbm` 及其参数；
- `weights`：必填，指定用于训练的交互类型和权重；
- `features`：必填，指定模型使用的已定义特征；
- `selector`：可选，筛选哪些事件进入该模型；
- `split`：可选，训练集与测试集的切分策略，默认 `time=80%`；
- `eval`：可选，训练后计算的评估指标，默认 `NDCG@10`；
- `warmup`：可选，模型加载后的 API 预热配置。

支持的评估指标包括 `NDCG`、`NDCG@k`、`MAP`、`MAP@k` 和 `MRR`。

## 交互权重

用户交互可以是 `click`、`add-to-wishlist`、`purchase`、`like` 等。它们既能作为 CTR 等特征的来源，也能通过权重影响模型优化目标。

```yaml
weights:
  click: 1.0
  purchase: 3.0
```

更高权重表示训练时该交互产生更高的相关性判断。权重设计是目标定义，不等于已经证明该目标会带来业务收益。

## 事件选择器

同时服务多个模型时，可以通过 `selector` 把不同场景的排序和交互送入对应模型，例如区分搜索与推荐。

### 接受或拒绝全部

未配置选择器时默认接受全部事件。

```yaml
selector:
  accept: true
```

`true` 接受全部，`false` 拒绝全部。

### 排序字段选择器

```yaml
selector:
  rankingField: source
  value: search
```

只接受 `ranking.fields` 中包含 `source=search` 的事件：

```json
{
  "event": "ranking",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "timestamp": "1599391467000",
  "user": "user1",
  "session": "session1",
  "fields": [
    {"name": "source", "value": "search"}
  ],
  "items": [
    {"id": "item1", "fields": [{"name": "relevancy", "value": 1.0}]}
  ]
}
```

### 随机采样选择器

```yaml
selector:
  ratio: 0.5
```

随机接受约一半事件。它只是事件采样，不应直接替代稳定的用户级实验分桶。

### 交互位置选择器

```yaml
selector:
  maxInteractionPosition: 10
  minInteractionPosition: 3
```

只接受交互位置位于指定范围的点击链。两个边界都可省略，可用于排除浏览位置过深或过浅的会话。

### 排序长度选择器

```yaml
selector:
  minItems: 10
  maxItems: 20
```

只接受候选数量在指定范围内的点击链。

### 用户选择器

```yaml
selector:
  user: monitoring-bot
```

只接受指定用户的事件，可用于隔离监控机器人或服务账号。

### 周期时隙选择器

```yaml
selector:
  periodSeconds: 300
  secondFrom: 90
  secondTo: 110
```

周期从 Unix epoch 对齐。例如 `300` 秒周期会在 UTC 每个整五分钟重新开始。边界均包含在内，并且必须满足：

```text
0 <= secondFrom <= secondTo < periodSeconds
```

该选择器常与 `not` 组合，用于丢弃严格按固定间隔产生的合成流量：

```yaml
selector:
  not:
    and:
      - minItems: 2
        maxItems: 2
      - periodSeconds: 300
        secondFrom: 90
        secondTo: 110
```

### AND、OR、NOT

```yaml
selector:
  and:
    - rankingField: source
      value: search
    - or:
        - rankingField: segment
          value: test
        - ratio: 0.5
        - not:
            accept: false
```

- `and` 和 `or` 接收选择器列表；
- `not` 只接收一个嵌套选择器。

## 训练集与测试集切分

Metarank 支持三种策略：

- `random`：随机切分；
- `hold_last`：对每个拥有多次排序的会话，把最后一部分排序作为测试集；
- `time`：按时间戳切分。

可以指定比例，默认训练集为 80%。

```yaml
split: time=80%
```

示例：

- `random=80%`：随机把 80% 分配给训练集；随机切分可能造成标签泄漏，需要谨慎；
- `hold_last`：使用默认的 80% 比例，在会话内部保留最后部分用于测试；
- `time=80%`：用较早的 80% 训练、较晚的 20% 测试，更接近时间外推场景。

## XGBoost 与 LightGBM 参数

两种后端支持：

| 参数 | 默认值 | 含义 |
|---|---:|---|
| `iterations` | `100` | 模型中的树数量 |
| `learningRate` | `0.1` | 学习率；更高通常训练更快，但可能降低精度或稳定性 |
| `ndcgCutoff` | `10` | 只有前 N 个位置影响 NDCG |
| `maxDepth` | `8` | 树的最大深度 |
| `seed` | 随机 | 固定后可提高训练可复现性 |
| `sampling` | `0.8` | 每棵树使用的特征比例，用于降低过拟合 |
| `debias` | `false` | 启用后端原生的位置偏差处理 |

LightGBM 还支持：

| 参数 | 默认值 | 含义 |
|---|---:|---|
| `numLeaves` | `16` | 每棵树允许的叶子数量 |

参数调优应参考 [LightGBM](https://lightgbm.readthedocs.io/en/latest/Parameters-Tuning.html) 和 [XGBoost](https://xgboost.readthedocs.io/en/stable/parameter.html) 官方文档。

## `shuffle` 基线

`shuffle` 随机扰动原始顺序，可以在实验中作为较弱的排序基线：

```yaml
models:
  random:
    type: shuffle
    maxPositionChange: 5
```

`maxPositionChange` 控制单个内容最多偏离原位置多远。

## `noop` 基线

`noop` 完全保留输入顺序，适合把上游原始排序作为控制基线：

```yaml
models:
  baseline:
    type: noop
```

它没有额外参数，也不会修改排序结果。

## 使用边界

- 模型配置决定优化目标，但不等于完成业务目标验证；
- 随机事件采样不等于用户级 A/B 分桶；
- `noop` 可以提供原排序基线，但实验分析仍由外部系统承担；
- 使用历史曝光训练时，应检查位置偏差、日志选择偏差和时间泄漏；
- 训练指标改善不等于线上因果收益。
