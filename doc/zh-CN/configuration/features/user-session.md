# 用户与会话特征

> 英文原文：[`doc/configuration/features/user-session.md`](../../../configuration/features/user-session.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

## User-Agent 提取器

HTTP User-Agent 包含平台、操作系统和浏览器等信息，可能与用户行为差异相关：

- 移动端、桌面端或平板；
- iOS、Android、Windows、Linux 或 macOS；
- Safari、Chrome、Firefox 等浏览器；
- 是否为已知爬虫。

原始 User-Agent 很难直接用作特征。Metarank 使用 [UA-Parser](https://github.com/ua-parser) 的模式库，并提供以下映射：

- `platform`：`mobile`、`desktop`、`tablet`；
- `os`：`ios`、`android`、`windows`、`linux`、`macos`、`chrome os`；
- `browser`：`safari`、`chrome`、`firefox`、`opera`、`ie`、`other`；
- `bot`：是否为已知爬虫。

```yaml
- name: platform_feature
  type: ua
  source: ranking.ua
  field: platform
  refresh: 0s
  ttl: 90d
```

- `source` 指向排序事件中的 User-Agent 字段；
- `field` 可选 `platform`、`os`、`browser` 或 `bot`；
- User-Agent 来自每次排序请求，接入方需要稳定提供该字段。

设备特征可能成为代理变量。使用时应检查隐私、合规、公平性和业务必要性，不能基于设备价格等推测直接给用户贴未经验证的标签。

## `interacted_with`

该特征回答：当前用户是否曾与具有相同属性的其他内容发生指定交互？

```yaml
- name: clicked
  type: interacted_with
  interaction: click
  field: [item.color]
  scope: user
```

Metarank 会记录用户点击过的内容颜色集合，并与当前候选的颜色求交集。

- `interaction`：要跟踪的交互类型；
- `field`：内容字符串字段或字符串数组；
- `scope`：`user` 或 `session`。

也可以同时跟踪多个字段：

```yaml
- name: clicked
  type: interacted_with
  interaction: click
  field: [item.color, item.tags, item.brand]
  scope: user
```

用户作用域适合长期偏好，会话作用域更适合短期意图。应结合 `ttl` 和用户标识稳定性决定保留周期。

## Referer 来源

Metarank 可以解析用户、排序或交互事件中的 HTTP Referer，并提取流量媒介。解析器定义六类：

- `unknown`；
- `search`；
- `internal`；
- `social`；
- `email`；
- `paid`。

排序事件：

```json
{
  "event": "ranking",
  "id": "81f46c34-a4bb-469c-8708-f8127cd67d27",
  "timestamp": "1599391467000",
  "user": "user1",
  "session": "session1",
  "fields": [
    {"name": "referer", "value": "http://www.google.com"}
  ],
  "items": [
    {"id": "item1"},
    {"id": "item2"}
  ]
}
```

配置：

```yaml
- name: referer_medium
  type: referer
  source: ranking.referer
  scope: user
```

Google 来源会被识别为 `search`，并 One-Hot 编码。例如：

```text
[0, 1, 0, 0, 0, 0]
```

提取器会记住接收到的多个来源。用户先从 Google 进入，再在站内浏览时，画像可能同时包含 `search` 和 `internal`：

```text
[0, 1, 1, 0, 0, 0]
```

`source` 可以来自 `user`、`ranking` 或 `interaction`，`scope` 可以使用用户或会话。处理完整 URL 时应遵守隐私要求，避免把查询参数中的敏感信息写入特征系统。
