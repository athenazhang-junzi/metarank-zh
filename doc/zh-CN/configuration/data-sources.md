# 数据源

> 英文原文：[`doc/configuration/data-sources.md`](../../configuration/data-sources.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

Metarank 有两个数据处理阶段：

- `import`：读取历史访客事件，生成当前系统状态和特征快照；
- `inference`：持续处理实时到达的事件。

## 连接器支持

| 数据源 | 历史导入 | 实时处理 |
|---|---:|---:|
| Apache Kafka | 支持 | 支持 |
| Apache Pulsar | 支持 | 支持 |
| AWS Kinesis Streams | 受保留周期限制 | 支持 |
| REST API | 支持批量发送 | 支持 |
| 文件 | 支持 | 不支持持续实时读取 |

Kinesis 的历史保留期有限。需要更长历史时，可以通过 Firehose 等方式把事件写入对象存储，再通过文件导入。

## 通用选项

### `offset`

- `earliest`：从数据源中最早可用消息开始；
- `latest`：只消费 Metarank 连接后新到达的事件；
- `ts=<timestamp>`：从指定历史时间开始；
- `last=<duration>`：只消费最近一段时间，例如 `1s`、`1m`、`1h`、`1d`。

### `format`

- `json`：支持 JSON Lines 和 JSON 数组；
- `snowplow:tsv`、`snowplow:json`：Snowplow 原生格式。

## 文件

```yaml
input:
  type: file
  path: /home/user/ranklens/events/
  offset: earliest
  format: json
  sort: name
```

`path` 可以是本机文件或目录。文件连接器支持：

- 根据扩展名自动识别 GZip 和 ZStandard；
- 从目录读取多个文件；
- 按名称或最后修改时间排序。

历史事件应按时间升序。文件顺序错误时可先使用 CLI `sort`。

## Apache Kafka

```yaml
input:
  type: kafka
  brokers: [broker1, broker2]
  topic: events
  groupId: metarank
  offset: earliest
  format: json
  options:
    client.id: metarank
    enable.auto.commit: "false"
```

`options` 可以传递 Kafka Consumer 原生参数。历史导入选择过去的 offset，实时服务通常使用 `latest`。

## Apache Pulsar

```yaml
input:
  type: pulsar
  serviceUrl: <pulsar service URL>
  adminUrl: <pulsar service HTTP admin URL>
  topic: events
  subscriptionName: metarank
  subscriptionType: exclusive
  offset: earliest
  format: json
  options:
    receiverQueueSize: 10
    acknowledgementsGroupTimeMicros: 50000
```

上游文档要求 Pulsar 2.8 及以上，并推荐 2.9 及以上。`subscriptionType` 可以是 `exclusive`、`shared` 或 `failover`。

## AWS Kinesis Streams

```yaml
input:
  type: kinesis
  region: us-east-1
  topic: events
  offset: earliest
  format: json
```

注意：

- 历史保留窗口限制了可直接导入的时间范围；
- 单分片吞吐限制可能使大规模历史导入较慢；
- 认证使用 AWS SDK 默认凭据链；
- 优先使用 IAM Role，手动凭据通过环境变量提供，不要写入仓库。

## REST API

`POST /feedback` 始终开启，无需额外配置，可以发送单个或批量事件：

```bash
java -jar metarank.jar serve --config conf.yaml
curl -d @events.json http://localhost:8080/feedback
```

大量历史数据更适合文件或流式连接器；REST 批量导入需要自行处理重试、顺序、幂等和请求大小。

完整端点说明见[JSON API](../api.md)。
