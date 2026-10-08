# Standalone 单机模式

> 英文原文：[`doc/deploy/standalone.md`](../../deploy/standalone.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

Metarank 可以直接作为 JAR 或 Docker 容器在单机运行，无需先配置 Kubernetes 或云服务。

## 运行模式

- `import`：导入历史点击链并计算特征；
- `train`：训练模型；
- `serve`：启动排序 API；
- `standalone`：依次执行 `import`、`train` 和 `serve`；
- `validate`：检查配置和事件数据。

Standalone 适合：

- 零外部依赖的本地体验；
- 配置验证；
- 单机或较低负载的预发布环境。

限制：

- 单节点限制反馈摄取和推理吞吐；
- 训练与推理处于同一进程，训练可能造成延迟尖峰或 OOM；
- 不适合作为可水平扩展的生产架构。

## JAR

```bash
java -jar metarank.jar standalone \
  --data /path/to/events.json \
  --config /path/to/config.yml
```

## Docker

```bash
docker run -p 8080:8080 \
  -v /host/data:/data \
  metarank/metarank:0.8.0 standalone \
  --data /data/events.json \
  --config /data/config.yml
```

启动过程会：

1. 导入历史数据并计算统计；
2. 训练配置中的模型；
3. 启动实时排序 API。

完整演示见[快速开始](../quickstart/quickstart.md)。生产环境应将训练与 `serve` 分离。
