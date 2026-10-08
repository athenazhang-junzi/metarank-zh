# 热门趋势推荐

> 英文原文：[`doc/configuration/recommendations/trending.md`](../../../configuration/recommendations/trending.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

`trending` 推荐模型用于突出当前热门内容。它不只是按累计次数排序，还支持：

- 组合点击、喜欢、购买等多种交互；
- 为不同交互配置不同权重；
- 通过时间衰减提高近期行为的重要性；
- 使用不同时间窗口构建短期趋势和长期畅销榜。

## 配置

在 `models` 中单独定义：

```yaml
models:
  yolo-trending:
    type: trending
    weights:
      - interaction: click
        decay: 0.8
        weight: 1.0
        window: 30d
      - interaction: like
        decay: 0.9
        weight: 1.5
        window: 60d
      - interaction: purchase
        decay: 0.95
        weight: 3.0
```

模型通过 [`/recommend/yolo-trending`](../../api.md#推荐) 调用。

该配置表示：

- 最终分数组合点击、喜欢和购买；
- 购买权重是点击的 3 倍，喜欢为 1.5 倍；
- 购买的时间衰减更慢；
- 点击和购买使用最近 30 天，喜欢使用最近 60 天。

如果 `window` 未设置，上游文档给出的默认值为 30 天；`decay` 和 `weight` 默认均为 `1.0`。

## 分数计算

单类交互的分数为：

```text
score = count * weight * decay ^ days_diff(now, timestamp)
```

配置多个交互类型时，各类分数相加得到最终分数。

上游建议的衰减范围：

- 一个月左右的周期：`0.8–0.95`；
- 更长周期：`0.95–0.99`。

这些值只是经验起点，应根据业务周期、事件量和离线回放调整。

## 使用边界

- 热门不等于个性化；
- 高权重购买可能让高频商品长期占据头部；
- 时间窗口太短会造成榜单抖动，太长会降低时效性；
- 机器人、运营活动或异常流量可能放大趋势；
- 正式使用前应设置内容安全、库存、作者去重和供给公平等外部规则。

请求与响应格式见[推荐 API](../../api.md#推荐)。
