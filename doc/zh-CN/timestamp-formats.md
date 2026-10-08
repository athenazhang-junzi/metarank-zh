# 时间戳格式

> 英文原文：[`doc/timestamp-formats.md`](../timestamp-formats.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

所有 Metarank 输入事件都包含 `timestamp`：

```json
{
  "event": "item",
  "id": "event-1",
  "timestamp": "1599391467000",
  "item": "product1",
  "fields": [
    {"name": "title", "value": "Nice jeans"}
  ]
}
```

JSON 双精度数值可能造成大整数精度问题，因此 Metarank 支持：

- 推荐：UTC Unix 毫秒数字符串，例如 `"1599391467000"`；
- 推荐：带时区的 ISO 8601 字符串，例如 `"2022-06-22T11:21:39Z"`；
- 不推荐：JSON 数值，例如 `1599391467000`。

接入时应统一时间单位和时区，并检查事件是否按时间顺序排列。不要混用秒与毫秒。
