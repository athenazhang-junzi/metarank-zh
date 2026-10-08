# 配置概览

> 英文原文：[`doc/configuration/overview.md`](../../configuration/overview.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`  
> 版本说明：本文覆盖全部配置区段和关键参数，并压缩了重复注释。

Metarank 使用 YAML 配置文件，主要包含：

- `state`：特征状态和已训练模型如何存储；
- `train`：点击链训练数据如何存储；
- `features`：如何把输入事件转换为机器学习特征；
- `models`：需要训练和在线使用哪些排序或推荐模型；
- `inference`：神经搜索推理模型；
- `api`：API 的监听地址和端口；
- `input`：事件数据源；
- `core`：点击链缓冲、匿名分析和错误上报等服务选项。

完整示例见英文 [`sample-config.yml`](../../configuration/sample-config.yml)。

## `state`：状态存储

`state` 保存在线推理需要的特征值和已训练模型。详细选项见英文[持久化文档](../../configuration/persistence.md)。

### 内存模式

```yaml
state:
  type: memory
```

数据只存在当前进程内，服务重启后全部丢失，适合本地测试。

### Redis 模式

```yaml
state:
  type: redis
  host: localhost
  port: 6379
  format: binary

  cache:
    maxSize: 4096
    ttl: 1h

  pipeline:
    maxSize: 128
    flushPeriod: 1s
    enabled: true

  auth:
    user: <username>
    password: <password>

  tls:
    enabled: true
    ca: <path/to/ca.crt>
    verify: full

  timeout:
    connect: 1s
    socket: 1s
    command: 1s
```

Redis 模式可以持久化特征与模型，并通过客户端缓存和流水线提升吞吐。`verify` 可取：

- `full`：验证证书和主机名；
- `ca`：只验证证书；
- `off`：关闭验证，不建议用于生产环境。

认证信息优先通过环境变量或安全的密钥管理方式提供，不应提交到公开仓库。

## `train`：训练数据存储

Metarank 会把以下信息合并成点击链（click-through）：

- 当时向用户展示的排序；
- 用户在看到排序后的点击、购买等交互；
- 生成该次排序时使用的特征值。

点击链随后被转换为 LambdaMART 使用的隐式判断列表。

### Redis

```yaml
train:
  type: redis
  ttl: 365d
```

无需额外管理文件，可以直接读取最新点击链进行定期训练，但大量历史数据可能消耗较多内存。

### 丢弃训练数据

```yaml
train:
  type: discard
```

不保存点击链，因此不能依赖这部分数据重新训练模型。

### 本地目录

```yaml
train:
  type: file
  path: /path/to/dir
  format: json
```

内存占用较低，但需要自行管理文件目录、备份和生命周期。`format` 可使用 `json` 或 `binary`。

### S3

```yaml
train:
  type: s3
  bucket: <bucket name>
  prefix: <prefix name>
  region: <aws region>
  compress: gzip
  partSizeBytes: 10485760
  partSizeEvents: 1024
  partInterval: 1h
  readConcurrency: 8
  endpoint: <endpoint URI>
  format: binary
  deduplicate: false
```

S3 适合 Kubernetes 等分布式部署。可选参数包括压缩方式、分片大小、分片事件数、上传周期、读取并发、自定义端点、格式和读取去重。

不要把长期 AWS 凭据硬编码到配置文件；优先使用环境变量或工作负载身份。

## `features`：特征

`features` 描述如何把输入事件映射为模型特征。完整类型见[特征提取器](feature-extractors.md)。

```yaml
features:
  - name: popularity
    type: number
    scope: item
    source: item.popularity
    ttl: 60d
    refresh: 1h

  - name: genre
    type: string
    scope: item
    source: item.genres
    values:
      - drama
      - comedy
      - thriller
```

多个模型可以共享同一组特征定义。Metarank 会统一计算特征，但某个模型只有在自己的 `features` 列表中显式引用后才会使用该特征。

- `ttl` 控制没有更新的特征保存多久；
- `refresh` 控制派生值的刷新频率；
- 更长的刷新周期通常提高吞吐，但在线推理可能读取到稍旧的数据。

官方 RankLens 示例配置位于 [`src/test/resources/ranklens/config.yml`](https://github.com/metarank/metarank/blob/master/src/test/resources/ranklens/config.yml)。

## `models`：模型

`models` 定义在线个性化使用的模型。模型名称会成为 API 路径的一部分，例如 `default` 对应 `/rank/default`。

```yaml
models:
  default:
    type: lambdamart
    backend:
      type: xgboost
      iterations: 100
      seed: 0
    weights:
      click: 1
    features:
      - popularity
      - genre

  similar:
    type: als
    interactions: [click]
    factors: 100
    iterations: 30

  trending:
    type: trending
    weights:
      - interaction: click
        decay: 1.0
        weight: 1.0
```

- `lambdamart`：对显式传入的候选集进行学习排序；
- `als`：根据交互学习相似内容；
- `trending`：基于交互和时间衰减生成热门内容；
- `shuffle`：随机扰动原顺序，可作为基线；
- `noop`：不改变原顺序，可作为控制基线。

可以同时配置多个模型，用于不同业务场景、特征组合或实验版本。多模型服务本身不包含完整的用户分桶和统计实验平台。

详细参数见[支持的排序模型](supported-ranking-models.md)和英文[推荐模型概览](../../configuration/recommendations.md)。

## `inference`：神经推理

该区段配置用于搜索重排的文本推理模型：

```yaml
inference:
  msmarco:
    type: cross-encoder
    model: metarank/ce-msmarco-MiniLM-L6-v2
```

模型名称 `msmarco` 会用于 `/inference/cross/msmarco` 等 API。更多信息见英文[推理模型文档](../../configuration/inference-models.md)。

## `api`：服务地址

该区段可选。默认监听所有网络接口的 `8080` 端口。

```yaml
api:
  port: 8080
  host: "0.0.0.0"
```

生产环境中应结合反向代理、网络策略、认证和 TLS 控制 API 暴露范围。

## `input`：数据源

该区段可选。默认情况下，应用通过[反馈 API](../api.md#反馈)提交事件。Metarank 还支持文件、Kafka、Pulsar 和 Kinesis 等输入。

### 文件

```yaml
input:
  type: file
  path: /home/user/ranklens/events/
  offset: earliest
  format: json
```

### Kafka

```yaml
input:
  type: kafka
  brokers: [broker1, broker2]
  topic: events
  groupId: metarank
  offset: earliest
  format: json
```

### Pulsar

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
```

### Kinesis

```yaml
input:
  type: kinesis
  region: us-east-1
  topic: events
  offset: earliest
  format: json
```

各数据源的 `offset` 和 `format` 选项不同，接入前应核对英文[数据源文档](../../configuration/data-sources.md)。

## `core`：服务级配置

```yaml
core:
  clickthrough:
    maxParallelSessions: 10000
    maxSessionLength: 30m

  tracking:
    analytics: true
    errors: true
```

### 点击链合并

排序发生后，交互往往延迟到达。Metarank 因此会缓冲尚未结束的排序与会话：

- `core.clickthrough.maxSessionLength`：连续多长时间没有活动后结束会话，默认 `30m`；
- `core.clickthrough.maxParallelSessions`：缓冲区允许同时等待的会话数量，默认 `10000`。

也可以调用 [`POST /flush`](../api.md#刷新会话缓冲区)立即结束所有缓冲点击链。

### 匿名使用分析

Metarank 默认启用匿名使用分析，并在服务启动时向 `https://analytics.metarank.ai` 发送服务模式、存储类型、模型类型、特征类型、操作系统、架构、JVM 版本和安装标识等信息。上游说明其不记录 IP，数据保存一年，收集器代码开源。

如不希望发送匿名分析，可配置：

```yaml
core:
  tracking:
    analytics: false
```

### 错误上报

错误收集使用 Sentry，默认开启。上游说明 Breadcrumbs 和 PII 跟踪均关闭。可以配置：

```yaml
core:
  tracking:
    errors: false
```

也可以通过环境变量同时关闭使用分析和错误上报：

```bash
METARANK_TRACKING=false
```

公开演示、企业数据或隐私敏感环境中，应在启动前明确检查这一设置。
