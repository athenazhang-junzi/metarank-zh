# 机器学习特征提取器

> 英文原文：[`doc/configuration/feature-extractors.md`](../../configuration/feature-extractors.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

常见的学习排序任务往往使用相似的特征。只要输入数据遵循[事件格式](../event-schema.md)，Metarank 就可以帮助生成其中许多特征。

## 从事件映射到特征

Metarank 在在线推理和离线训练期间接收事件流，并把它们合并为访客点击链的统一视图：

- 对每个 `ranking` 事件，按内容关联 `item` 元数据，同时读取 `user` 元数据；
- 把点击、购买等 `interaction` 事件关联到对应排序；
- 特征提取器基于完整点击链计算特征值。

不同作用域的提取器依次使用内容、会话和交互数据。

## 通用配置字段

所有特征提取器都包含以下通用字段：

- `name`：必填字符串，特征名称，在整个配置中必须唯一；
- `refresh`：可选时间，不同提取器有不同默认值，控制特征更新频率；
- `ttl`：可选时间，默认 `90d`，控制特征值保留时间；
- `scope`：可选，控制特征如何计算和存储。

## 作用域（Scope）

特征值不是单个静态值，而是按时间记录的变更序列。`scope` 是某个特征实例的唯一键。

Metarank 提供四种预定义作用域：

- `global`：全局特征，例如服务器机房外温度；
- `item`：以内容 ID 为键，例如内容热度；
- `user`：以用户 ID 为键，例如历史会话数量；
- `session`：以会话 ID 为键，例如当前购物车内容数。

作用域也用于减少重复计算。例如全局特征在一次排序中的所有候选项上都相同，不需要为每个候选重复计算。离线生成训练数据时，这项优化可以明显减少计算量。

```yaml
- name: popularity
  type: number
  field: item.popularity
  scope: item
  refresh: 0s
  ttl: 90d
```

`scope: item` 表示从内容元数据中提取的 `popularity` 按内容 ID 存储。

## 支持的特征类型

### 通用特征

| 类型 | 用途 |
|---|---|
| [`number`](../../configuration/features/scalar.md#boolean-and-numerical-extractors) | 直接使用数值字段 |
| [`vector`](../../configuration/features/scalar.md#vector-extractor) | 将可变长度数值向量规整为固定长度数组 |
| [`boolean`](../../configuration/features/scalar.md#boolean-and-numerical-extractors) | 将布尔值编码为 1 或 0 |
| [`string`](../../configuration/features/scalar.md#string-extractors) | 对字符串或字符串列表进行 One-Hot 编码 |
| [`word_count`](../../configuration/features/generic.md#word-count) | 统计字符串中的单词数量 |
| [`list_size`](../../configuration/features/generic.md#list-size) | 统计字符串或数值列表长度 |
| [`time_diff`](../../configuration/features/generic.md#time-difference) | 计算当前时间与数值时间字段的秒数差 |
| [`field_match`](../../configuration/features/text.md#field_match) | 匹配排序级字段与内容字段 |

### 用户与会话特征

| 类型 | 用途 |
|---|---|
| [`ua/platform`](../../configuration/features/user-session.md#user-agent-field-extractor) | 将移动端、桌面端、平板等平台 One-Hot 编码 |
| [`ua/os`](../../configuration/features/user-session.md#user-agent-field-extractor) | 编码 iOS、Android、Windows、Linux、macOS 等系统 |
| [`ua/browser`](../../configuration/features/user-session.md#user-agent-field-extractor) | 编码 Chrome、Firefox、Safari、Edge 等浏览器 |
| [`interacted_with`](../../configuration/features/user-session.md#interacted-with) | 判断用户是否与具有相同字段的其他内容交互过 |
| [`referer_medium`](../../configuration/features/user-session.md#referer) | 表示用户的流量来源 |

`ip_country`、`ip_city`、`session_length` 和 `session_count` 在上游文档中仍标记为计划能力。

### 排序特征

| 类型 | 用途 |
|---|---|
| [`relevancy`](../../configuration/features/relevancy.md#ranking) | 使用排序请求传入的内容级相关性值 |
| [`position`](../../configuration/features/relevancy.md#position) | 内容在原排序中的位置 |
| [`diversity`](../../configuration/features/diversity.md) | 搜索或推荐结果多样性 |

### 交互特征

| 类型 | 用途 |
|---|---|
| [`interaction_count`](../../configuration/features/counters.md#interaction-counter) | 当前会话中的交互次数 |
| [`window_event_count`](../../configuration/features/counters.md#windowed-counter) | 滑动时间窗口内的交互事件数量 |
| [`rate`](../../configuration/features/counters.md#rate) | A 类事件除以 B 类事件，可用于 CTR/CVR |

### 日期与时间特征

| 类型 | 用途 |
|---|---|
| [`local_time`](../../configuration/features/datetime.md#local_time-extractor) | 将用户本地时间映射为季节性特征 |
| [`item_age`](../../configuration/features/datetime.md#item_age) | 计算内容距离更新时间经过了多久 |

## 使用提醒

- 定义特征不代表模型会自动使用；仍需在 `models.<name>.features` 中显式引用；
- `refresh` 越短，特征越新，但更新成本越高；
- `ttl` 应覆盖模型训练和在线推理所需的历史范围；
- 作用域错误可能造成用户之间或内容之间的数据串用；
- 训练与推理必须使用一致的特征定义。
