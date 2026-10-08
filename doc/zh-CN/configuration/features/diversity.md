# 排序多样性特征

> 英文原文：[`doc/configuration/features/diversity.md`](../../../configuration/features/diversity.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

`diversity` 计算当前候选在同一次排序中与其他内容有多大差异，支持数值字段、字符串字段和字符串数组。

## 数值字段

假设内容包含数值型价格：

```json
{
  "event": "item",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "item": "item1",
  "timestamp": "1599391467000",
  "fields": [
    {"name": "price", "value": 69.0}
  ]
}
```

排序事件：

```json
{
  "event": "ranking",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "timestamp": "1599391467000",
  "user": "user1",
  "session": "session1",
  "items": [
    {"id": "item1"},
    {"id": "item2"},
    {"id": "item3"}
  ]
}
```

配置：

```yaml
- name: price_diff
  type: diversity
  source: item.price
  ttl: 90d
  top: 10
```

- `source` 只接受 `item.*` 字段；
- `ttl` 控制内容字段保留时间；
- `top` 可选，只使用前 N 个候选计算中位数。

若价格为：

```text
p1=100, p2=200, p3=250, p4=300, p5=220
```

排序 `[p1, p2, p3, p4, p5]` 的整体中位数为 220，因此：

```text
p1=-120, p2=-20, p3=30, p4=80, p5=0
```

设置 `top: 3` 后，只用前三项计算中位数 200：

```text
p1=-100, p2=0, p3=50, p4=100, p5=20
```

候选列表很长时，限制 `top` 可以减少计算量，也能让特征更聚焦于用户最可能看到的头部内容。

## 字符串字段

字符串多样性适合标签、颜色、尺寸、作者和类别等低基数字段。支持 `string` 和 `string[]`。

```json
{
  "event": "item",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "item": "item1",
  "timestamp": "1599391467000",
  "fields": [
    {"name": "color", "value": "red"}
  ]
}
```

```yaml
- name: color_diff
  type: diversity
  source: item.color
  ttl: 90d
  top: 10
```

算法先统计排序中的标签频率，再计算单内容标签与整体频率的相对交集。例如：

```text
red=50%, green=30%, blue=20%
```

- 只有 `red` 的内容得分为 50%；
- 同时包含 `red` 和 `blue` 的内容得分为 70%。

## 使用边界

该特征描述当前候选与同批内容的差异，仍然需要由模型学习它与目标之间的关系。它不是一个强制去重规则：

- 模型可能仍然把相似内容排在一起；
- 作者去重、类目配额等确定性要求通常需要额外业务规则层；
- `top` 的选择会改变参照集合；
- 多样性改善可能与相关性、消费效率或内容覆盖发生权衡。
