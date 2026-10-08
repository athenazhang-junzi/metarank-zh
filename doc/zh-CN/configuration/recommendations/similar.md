# 相似内容推荐

> 英文原文：[`doc/configuration/recommendations/similar.md`](../../../configuration/recommendations/similar.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

`similar` 推荐模型根据其他用户的隐式行为，返回与当前上下文相似的内容。

典型场景：

- 内容详情页的“你可能还喜欢”，上下文是当前单个内容；
- 购物车页的“其他用户也购买”，上下文是购物车中的多个内容。

## 配置

```yaml
models:
  similar:
    type: als
    interactions: [click, like, purchase]
    factors: 100
    iterations: 100
```

- `interactions`：用于训练的交互类型；
- `factors`：隐式因子维度，默认 `100`；
- `iterations`：训练迭代次数，默认 `100`。

上游给出的经验范围：

- `factors`：50–500；
- `iterations`：50–300。

更高取值可能增加表达能力，但训练会变慢，也可能增加过拟合风险。可先从较低值开始，逐步提高并同时观察训练时间和离线指标。

## 底层方法

Metarank 使用基于隐式反馈的矩阵分解协同过滤方法。ALS 家族算法把稀疏的用户—内容交互矩阵分解为较小的稠密向量：

- 用户隐式向量；
- 内容隐式向量。

交互模式相似的内容通常具有相近向量。Metarank 随后：

1. 计算内容 Embedding；
2. 建立 HNSW 索引；
3. 调用 `/recommend/<model>` 时，通过 k-NN 查找相近向量。

## 优点与限制

优点：

- 推理速度快；
- 可以支持较大的内容库；
- 方法和部署结构相对直接。

限制：

- 依赖历史交互，冷启动内容较弱；
- 相比 BERT4Rec 等神经序列方法，表达能力有限；
- 当前上游文档明确说明该相似内容实现不是个性化推荐；
- 热门内容和高活跃用户可能主导隐式反馈矩阵。

语义冷启动可以结合[语义相似推荐](semantic.md)，但两种分数如何融合仍需由业务和评估方案定义。

请求与响应格式见[推荐 API](../../api.md#推荐)。
