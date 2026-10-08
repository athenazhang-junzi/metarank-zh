# 日期与时间特征

> 英文原文：[`doc/configuration/features/datetime.md`](../../../configuration/features/datetime.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

## `local_time`

`local_time` 解析用户本地日期时间，用于描述时段、星期和季节性差异。

```json
{
  "event": "ranking",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "timestamp": "1599391467000",
  "user": "user1",
  "session": "session1",
  "fields": [
    {"name": "localts", "value": "2021-12-03T10:15:30+01:00"}
  ],
  "items": [
    {"id": "item3"}, {"id": "item1"}, {"id": "item2"}
  ]
}
```

```yaml
- name: time
  type: local_time
  parse: time_of_day
  source: ranking.localts
```

`source` 可以引用单独字段，也可以引用 `ranking.timestamp`。自定义字段应为包含时区的 ISO 格式字符串。

支持的 `parse`：

| 值 | 输出 |
|---|---|
| `day_of_week` | 1–7，星期一为 1 |
| `time_of_day` | 本地时间映射到 0.0–23.99 |
| `day_of_month` | 1–31 |
| `month_of_year` | 1–12 |
| `year` | 四位年份 |
| `second` | 本地时间对应的 Unix 秒数 |

时间特征容易产生周期边界问题。星期、小时等周期变量是否需要进一步做正余弦编码，应结合模型和实际效果判断。

## `item_age`

`item_age` 计算内容创建或更新时间距离当前排序时刻经过了多久。

```json
{
  "event": "item",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "item": "product1",
  "timestamp": "1599391467000",
  "fields": [
    {"name": "created_at", "value": "2021-12-03T10:15:30+01:00"}
  ]
}
```

```yaml
- name: freshness
  type: item_age
  source: item.created_at
  refresh: 0s
  ttl: 90d
```

`source` 支持：

- ISO 8601 日期时间字符串；
- Unix 秒数；
- 以字符串表示的 Unix 秒数，避免 JSON 数值精度问题；
- 原生事件时间戳 `item.timestamp`。

新鲜度只是排序信号。若业务要求新内容最低曝光、过期内容强制下架等确定性规则，仍需要额外策略层。
