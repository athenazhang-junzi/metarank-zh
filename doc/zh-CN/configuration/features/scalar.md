# 标量与类别特征

> 英文原文：[`doc/configuration/features/scalar.md`](../../../configuration/features/scalar.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

Metarank 提供三类基础提取器，把事件字段直接映射为模型特征：

- `boolean`：把 `true/false` 映射为 `1/0`；
- `number`：直接使用数值；
- `string`：对低基数字符串执行 One-Hot 或索引编码。

## 布尔值与数值

内容事件：

```json
{
  "event": "item",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "item": "product1",
  "timestamp": "1599391467000",
  "fields": [
    {"name": "availability", "value": true},
    {"name": "price", "value": 69.0}
  ]
}
```

```yaml
- name: availability
  type: boolean
  scope: item
  field: item.availability
  refresh: 0s
  ttl: 90d

- name: price
  type: number
  scope: item
  field: item.price
  refresh: 0s
  ttl: 90d
```

也可以读取排序级上下文和单候选字段：

```yaml
- name: banner_examined
  type: boolean
  scope: item
  field: ranking.banner_examined

- name: relevancy
  type: number
  scope: item
  field: ranking.relevancy
```

交互事件发生在排序之后，因此排序请求发生时不能把未来交互字段直接作为当前特征。

用户元数据同样可以映射为特征：

```yaml
- name: user_age
  type: number
  field: user.age
  scope: user
  refresh: 0s
  ttl: 90d
```

## 数值向量

数值列表长度可能变化或为空，需要通过固定规约转换成模型可以使用的固定维度。

```yaml
- name: sizes
  type: vector
  field: item.sizes
  scope: item
  reduce: [first, last, min, max, avg, random, sum, size, euclidean_distance, vector10]
  refresh: 0s
  ttl: 90d
```

支持的规约方式：

- `first`、`last`、`min`、`max`、`random`：取对应元素，空列表返回 0；
- `avg`、`sum`：均值与求和；
- `size`：列表长度；
- `euclidean_distance`：平方和开根号；
- `vectorN`：取前 N 个值，不足部分补 0。

固定维度 Embedding 可以使用 `vectorN` 直接展开。例如 10 维 `als_embedding` 使用 `reduce: vector10` 后会成为 10 个数值特征。

## 字符串 One-Hot 编码

低基数字段可以显式列出取值：

```yaml
- name: color
  type: string
  scope: item
  encode: onehot
  values: [red, green, blue]
  field: item.color
```

输入 `color: "red"` 会产生：

```text
color_red=1, color_green=0, color_blue=0
```

字符串数组也受支持，例如 `color: ["red", "blue"]` 会同时激活两个位置。

## 字符串索引编码

类别数量较多时，One-Hot 会显著增加训练数据维度，可以使用索引编码：

```yaml
- name: color
  type: string
  scope: item
  encode: index
  values: [red, green, blue]
  field: item.color
```

输入 `green` 会产生类似 `color=2` 的单值特征。

限制：

- 只能处理单一字段值；数组只使用第一个值；
- 空值编码为 0，已有类别从 1 开始；
- LightGBM 可以针对类别特征选择切分；
- 当前 Java 封装下，XGBoost 会把索引类别当作普通数值。

索引编码维度更低，通常更快；但编码方式仍应通过实际数据和离线评估选择，不能只依赖默认值。
