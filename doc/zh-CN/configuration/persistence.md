# 状态持久化

> 英文原文：[`doc/configuration/persistence.md`](../../configuration/persistence.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

Metarank 的特征和模型状态可以存储在：

- 内存：临时状态，重启后丢失；
- Redis 6+：独立的远程持久化；
- 文件：实验性磁盘后端，支持 MapDB 和 RocksDB。

通过配置文件中的 `state` 选择；未配置时默认使用内存。训练点击链的存储方式见[配置概览](overview.md#train训练数据存储)。

## 内存

```yaml
state:
  type: memory
```

适合：

- 本地 `standalone` 测试；
- 上线前验证配置；
- 不需要跨重启保留状态的演示。

不适合生产持久化，因为服务重启会丢失所有状态。

## Redis

```yaml
state:
  type: redis
  host: localhost
  port: 6379
  format: binary

  cache:
    maxSize: 1024
    ttl: 1h
    clientTracking: true

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

  db:
    state: 0
    values: 1
    rankings: 2
    models: 3
```

Redis 模式使用：

- Pipelining：批量发送写操作；
- 客户端缓存：缓存热点读取，并接收服务端失效通知。

需要注意：

- `cache.maxSize` 是每类底层特征缓存的大小，实际总缓存会乘以特征存储类型数量；
- 热点数据可能长期留在客户端缓存，但 Redis 的失效通知会降低脏读；
- 低延迟环境中把 `pipeline.maxSize` 提高到 128 以上通常收益有限；
- `flushPeriod` 越长，多实例之间看到更新的延迟越高。

### TLS

自签名证书需要指定 CA：

```yaml
tls:
  enabled: true
  ca: /tls/key.crt
```

系统受信 CA 签发的证书通常只需要：

```yaml
tls:
  enabled: true
```

验证级别：

- `full`：验证证书和主机名；
- `ca`：只验证证书；
- `off`：完全跳过验证，不应在生产环境使用。

### 认证

不要在公开配置中硬编码凭据。可以使用：

- `METARANK_REDIS_USER`；
- `METARANK_REDIS_PASSWORD`。

### 编码格式

- `json`：便于阅读和调试；
- `binary`：开销更低，通常更快、更省内存。

上游在 RankLens 数据上的说明称二进制约快 2 倍、内存约少 4 倍；这是特定数据和环境下的项目方结果，实际部署应自行压测。

### Redis 限制

- 需要 Redis 6+；
- 不支持 Redis Cluster；
- 不支持客户端跟踪的托管兼容服务可设置 `cache.maxSize: 0`；
- GCP Memorystore 等场景可能还需要 `clientTracking: false`。

## 实验性磁盘持久化

```yaml
state:
  type: file
  path: /path/to/dir
  format: binary
  backend:
    type: rocksdb
```

- MapDB：基于 mmap，适合较小数据；
- RocksDB：LSM-tree，适合较大数据。

磁盘后端会让 Metarank 服务本身变成有状态部署，需要自行处理磁盘可靠性、备份、扩容和故障恢复。

### RocksDB

```yaml
state:
  type: file
  path: /path/to/dir
  backend:
    type: rocksdb
    lruCacheSizeMb: 1024000000
    blockSize: 8192
```

- 更大的 LRU 缓存通常提高读取吞吐，但占用更多内存；
- `blockSize` 应结合底层磁盘块大小测试。

### MapDB

```yaml
state:
  type: file
  path: /path/to/dir
  backend:
    type: mapdb
    mmap: true
    maxNodeSize: 16
```

生产环境中，上游建议优先使用独立 Redis；磁盘模式仍属于实验性能力。
