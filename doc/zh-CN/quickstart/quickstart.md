# 快速开始

> 英文原文：[`doc/quickstart/quickstart.md`](../../quickstart/quickstart.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

> 版本说明：本文为中文精编，保留完整运行链路并缩短了上游的大候选列表示例；需要逐项对照时请查看英文原文。

本指南使用 Docker 在单机运行 Metarank：下载官方 RankLens 示例、导入历史事件、训练排序模型、启动 API，然后观察反馈事件如何改变个性化排序。

## 前置条件

- Docker Desktop，或 Linux Docker Engine；
- Linux、macOS 或 Windows 10 及以上；
- x86_64 或 arm64 架构；
- 为 Docker 预留至少 2 GB 内存。

以下命令固定使用 `metarank/metarank:0.8.0`。如需使用其他版本，请先核对[官方发布页](https://github.com/metarank/metarank/releases)和[安装说明](../installation.md)。

## 1. 下载示例数据

RankLens 是基于电影推荐场景构建的开放示例，其中包含三类关键数据：

- `item`：电影标题、类型、演员、热度等内容属性；
- `ranking`：一次展示给用户的有序候选列表；
- `interaction`：用户点击或喜欢了哪个内容。

下载配置和历史事件：

```bash
curl -o config.yml https://raw.githubusercontent.com/metarank/metarank/master/src/test/resources/ranklens/config.yml
curl -o events.jsonl.gz https://media.githubusercontent.com/media/metarank/metarank/master/src/test/resources/ranklens/events/events.jsonl.gz
```

确认文件存在：

```bash
ls -lh config.yml events.jsonl.gz
```

其中：

- `config.yml` 描述如何把事件转换成特征、训练标签和模型；
- `events.jsonl.gz` 保存用于训练的历史内容、排序和交互事件。

## 2. 启动 Metarank

在这两个文件所在的目录执行：

```bash
docker run -i -t -p 8080:8080 \
  -v "$(pwd)":/opt/metarank \
  metarank/metarank:0.8.0 standalone \
  --config /opt/metarank/config.yml \
  --data /opt/metarank/events.jsonl.gz
```

`standalone` 会连续完成：

1. 导入当前目录中的历史事件；
2. 根据配置生成训练样本；
3. 训练排序模型；
4. 在 `8080` 端口启动 API。

训练和启动可能需要一些时间。看到服务开始监听端口后，再打开另一个终端发送请求。

## 3. 发起第一次排序

向配置中的 `xgboost` 模型提供一个候选列表：

```bash
curl http://localhost:8080/rank/xgboost \
  -H 'Content-Type: application/json' \
  -d '{
    "event": "ranking",
    "id": "rank-before-feedback",
    "items": [
      {"id":"72998"}, {"id":"67197"}, {"id":"77561"},
      {"id":"68358"}, {"id":"79132"}, {"id":"103228"},
      {"id":"72378"}, {"id":"85131"}, {"id":"94864"},
      {"id":"68791"}, {"id":"93363"}, {"id":"112623"}
    ],
    "user": "alice",
    "session": "alice-session-1",
    "timestamp": 1661431886711
  }'
```

响应中的 `items` 已按模型得分重新排序：

```json
{
  "items": [
    {"item": "72998", "score": 0.96},
    {"item": "79132", "score": 0.78}
  ]
}
```

实际响应会包含请求中的全部候选，分数也会更精确。`score` 仅用于当前模型内排序，不应直接解释为点击概率或业务收益。

## 4. 回传实际展示结果

应用展示结果后，应向 `/feedback` 回传 `ranking` 事件。实际系统只应记录真正展示给用户的内容；分页场景中不要把尚未曝光的后续页面全部记为曝光。

```bash
curl http://localhost:8080/feedback \
  -H 'Content-Type: application/json' \
  -d '{
    "event": "ranking",
    "id": "shown-ranking-1",
    "items": [
      {"id":"72998"}, {"id":"79132"}, {"id":"68358"},
      {"id":"112623"}, {"id":"103228"}, {"id":"93363"}
    ],
    "user": "alice",
    "session": "alice-session-1",
    "timestamp": 1661431888711
  }'
```

## 5. 回传用户交互

假设用户点击了内容 `93363`，交互事件需要通过 `ranking` 字段关联到先前的展示：

```bash
curl http://localhost:8080/feedback \
  -H 'Content-Type: application/json' \
  -d '{
    "event": "interaction",
    "type": "click",
    "id": "click-1",
    "ranking": "shown-ranking-1",
    "item": "93363",
    "user": "alice",
    "session": "alice-session-1",
    "timestamp": 1661431890711,
    "fields": []
  }'
```

> 展示列表与点击内容必须保持一致。为了便于学习，可直接使用英文原文中的完整 RankLens 请求；生产系统应以真实展示事件为准。

## 6. 再次请求排序

使用相同用户和候选内容再次调用 `/rank/xgboost`。如果配置中的实时特征使用了刚才的交互，新的顺序可能发生变化。

需要注意：

- 一次点击不等于模型已经完成重新训练；
- 顺序变化可能来自实时画像或计数特征；
- 模型更新仍需要新的训练样本、训练过程和模型加载；
- 是否发生变化取决于具体配置和事件是否成功写入。

## 7. 理解这条闭环

```text
候选内容
   ↓
/rank/xgboost
   ↓
模型返回重排结果
   ↓
应用真实展示部分结果
   ↓
/feedback：ranking
   ↓
/feedback：interaction
   ↓
实时特征更新 / 后续训练样本
```

## 常见问题

### Docker 无法挂载当前目录

确认 Docker Desktop 已允许访问当前文件目录，并检查 `config.yml` 与 `events.jsonl.gz` 的绝对路径。

### 端口被占用

可以把宿主机端口改为其他值：

```bash
docker run -i -t -p 8081:8080 ...
```

此时使用 `http://localhost:8081` 访问 API。

### Apple Silicon 是否支持？

官方 Docker 镜像支持 `arm64/v8`。如果直接运行 JAR，还需要满足 JDK 和 `libomp` 要求，见[安装说明](../installation.md)。

### 为什么固定版本而不用 `latest`？

上游安装文档提示 `latest` 可能指向预发布版本。固定版本更容易复现和排查问题。

## 下一步

- 阅读英文[事件格式](../../event-schema.md)，理解 `item`、`user`、`ranking` 和 `interaction`；
- 阅读英文[API 文档](../../api.md)，了解 `/rank`、`/feedback`、`/train` 和 `/recommend`；
- 阅读英文[配置文档](../../configuration/overview.md)，了解特征与模型配置；
- 通过[中英术语表](../glossary.md)统一概念。
