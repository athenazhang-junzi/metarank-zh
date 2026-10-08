# 使用 Docker 运行 Metarank

> 英文原文：[`doc/deploy/docker.md`](../../deploy/docker.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

官方镜像发布在 [Docker Hub](https://hub.docker.com/r/metarank/metarank/tags)，支持：

- `linux/amd64`；
- `linux/arm64/v8`，可在 Apple Silicon 上原生运行。

生产和可复现实验应固定明确版本，不使用浮动的 `latest`。

## 查看 CLI

```bash
docker run metarank/metarank:0.8.0 --help
```

所有子命令见[中文 CLI](../cli.md)。

## 挂载数据

镜像使用 `/data` 处理本地输入输出：

```bash
docker run \
  -v /home/user/input:/data \
  metarank/metarank:0.8.0 \
  train --config /data/config.yml
```

尽量使用明确的宿主机绝对路径，并确认 Docker Desktop 已授权访问该目录。

## 内存

镜像默认 JVM Heap 为 1 GB，实际 RSS 会因 JVM 额外开销更高。可以通过 `JAVA_OPTS` 调整：

```bash
docker run \
  -e JAVA_OPTS="-Xmx5g" \
  metarank/metarank:0.8.0 \
  train --config /data/config.yml
```

训练通常比 `serve` 消耗更多内存，设置容器限制时应同时考虑 JVM Heap 和堆外内存。

## 端口

API 默认使用 `8080`：

```bash
docker run \
  -p 8080:8080 \
  metarank/metarank:0.8.0 \
  serve --config /data/config.yml
```

生产环境不应未经认证直接暴露端口，应结合反向代理、TLS、访问控制和网络策略。
