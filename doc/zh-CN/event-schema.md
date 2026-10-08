# 支持的事件

> 英文原文：[`doc/event-schema.md`](../event-schema.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

Metarank 接收一组预定义事件，用来描述访客行为和内容元数据：

1. `item`：内容元数据，描述系统需要了解的内容属性；
2. `user`：用户元数据，描述系统已知的访客属性；
3. `ranking`：排序事件，描述实际向访客展示了什么以及展示顺序；
4. `interaction`：交互事件，描述访客对排序结果执行的点击、喜欢、购买等动作。

## 通用事件格式

事件使用 JSON。每个事件都有以下必填字段：

- `id`：事件唯一标识，由接入应用生成；
- `timestamp`：事件时间，通常为从 `1970-01-01T00:00:00` 开始计算的毫秒数；其他受支持格式见英文[时间戳文档](../timestamp-formats.md)；
- `event`：事件类型，只能是 `item`、`user`、`ranking` 或 `interaction`。

多租户场景还可以使用可选的 `tenant` 字段。

### `fields` 参数

四类事件都可以携带 `fields`；对 `ranking` 和 `interaction` 而言该字段可省略。它可以提供搜索词、筛选条件、配送信息等额外上下文，供个性化模型生成特征。

```json
"fields": [
  {"name": "title", "value": "Your favourite cat"}
]
```

- `fields.name`：字段名称；
- `fields.value`：支持布尔值、字符串、数值、字符串列表或数值列表。

## `item`：内容元数据事件

`item` 事件用于新增内容或更新已有内容属性。无需提交业务中的全部字段，只需要发送可能用于排序特征的属性。

```json
{
  "event": "item",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "timestamp": "1599391467000",
  "item": "item1",
  "fields": [
    {"name": "title", "value": "Your favourite cat"},
    {"name": "color", "value": ["white", "black"]},
    {"name": "is_cute", "value": true}
  ]
}
```

- `id`：事件 ID；
- `item`：内容 ID；
- `fields`：内容属性数组。

## `user`：用户元数据事件

当系统掌握访客的额外信息时，可以使用 `user` 事件。例如用户注册后提供的年龄或性别。

```json
{
  "event": "user",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "timestamp": "1599391467000",
  "user": "user1",
  "fields": [
    {"name": "age", "value": 33},
    {"name": "gender", "value": "m"}
  ]
}
```

- `id`：事件 ID；当前版本尚未使用该值，但必须提供；
- `user`：访客 ID；
- `fields`：用户属性数组。

## `ranking`：排序事件

`ranking` 事件表示向访客展示了哪些内容，以及展示顺序。个性化算法用它理解用户看到了什么，并将后续交互关联到对应候选列表。

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

- `id`：排序请求标识，用于关联后续交互；应与[排序 API](api.md#排序) 请求中的值一致；
- `user`：可选的访客唯一标识；
- `session`：可选的会话标识，同一访客可以有多个会话；
- `fields`：可选的排序级上下文字段；
- `items`：实际展示给访客的内容；
- `items.id`：内容 ID，应与 `item` 事件中的 `item` 一致；
- `items.fields`：可选的内容级上下文字段；
- `items.label`：可选的显式相关性判断。

> 接入时应记录真正展示的内容。分页或滚动场景中，不应把尚未进入展示范围的全部候选都当作已经曝光。

## `interaction`：交互事件

`interaction` 事件表示访客对内容执行的动作，例如点击、喜欢或购买。`type` 必须与[配置](configuration/overview.md)中模型权重或特征引用的交互名称一致。

```json
{
  "event": "interaction",
  "id": "0f4c0036-04fb-4409-b2c6-7163a59f6b7d",
  "ranking": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "timestamp": "1599391467000",
  "user": "user1",
  "session": "session1",
  "type": "purchase",
  "item": "item1",
  "fields": [
    {"name": "count", "value": 1},
    {"name": "shipping", "value": "DHL"}
  ]
}
```

- `id`：交互事件标识；
- `ranking`：可选的父排序事件 ID。发生在内容详情页等排序场景之外的交互可以不提供；
- `user`：可选的访客 ID；
- `session`：可选的会话 ID；
- `type`：交互类型的内部名称；
- `item`：内容 ID，应与内容元数据中的 `item` 一致；
- `fields`：可选的交互上下文字段。

### 保留交互类型：`impression`

`impression` 是保留类型。Metarank 会根据[级联点击模型](../click-models.md#cascade-model-and-click-through-rate)，为访客检查过的内容自动生成合成 `impression` 事件。

不要自行发送 `type: impression`，否则自定义事件会与合成事件叠加，导致计数和比率特征重复统计。需要记录内容可见性时，应使用其他名称，例如 `view`。相关背景见上游 [issue #1554](https://github.com/metarank/metarank/issues/1554)。

## 接入检查清单

- 每个事件的 `id` 唯一；
- 时间戳格式一致；
- `item`、`user` 和 `session` 主键稳定；
- `ranking` 与 `interaction.ranking` 可以正确关联；
- 交互类型与配置完全一致；
- 没有自行发送保留的 `impression` 类型；
- 只记录真实展示范围。
