<h1 align="center">
  <a href="https://www.metarank.ai">
    <img width="120" src="https://raw.githubusercontent.com/metarank/metarank/master/doc/img/logo.svg" alt="Metarank logo" />
  </a>
  <p>Metarank 中文版</p>
</h1>

<p align="center">非官方简体中文翻译与学习指南</p>

<p align="center">
  <a href="https://github.com/metarank/metarank">官方仓库</a> ·
  <a href="https://docs.metarank.ai">官方文档</a> ·
  <a href="doc/zh-CN/README.md">中文文档</a> ·
  <a href="TRANSLATION_STATUS.md">翻译进度</a> ·
  <a href="RELEASE_NOTES.zh-CN.md">中文版本说明</a> ·
  <a href="doc/zh-CN/glossary.md">术语表</a>
</p>

> [!IMPORTANT]
> 本仓库是 [metarank/metarank](https://github.com/metarank/metarank) 的非官方中文翻译。Metarank 的核心代码、品牌与原始文档归上游项目及其贡献者所有。本仓库不代表 Metarank 官方，也不应被理解为译者原创了 Metarank 的核心能力。

## Metarank 是什么？

[Metarank](https://metarank.ai) 是一个开源排序服务，用于在既有搜索或推荐候选集之上完成个性化重排。它可以接收内容候选、用户和上下文信息，计算排序特征，通过学习排序模型返回新的顺序，并接收点击、购买等反馈事件。

它适合构建：

- 个性化搜索与学习排序（Learning-to-Rank，LTR）；
- 热门内容、相似内容和协同过滤推荐；
- 基于用户行为的实时个性化；
- 语义搜索与神经重排；
- 多个排序模型的并行服务。

Metarank 主要负责**候选集之后的排序与反馈闭环**。它不是一套包含内容召回、流量分配、显著性检验和实验决策在内的完整推荐平台。

## 中文文档从哪里开始？

| 目标 | 建议入口 |
|---|---|
| 先理解项目定位 | [Metarank 简介](doc/zh-CN/intro.md) |
| 在本机跑通示例 | [快速开始](doc/zh-CN/quickstart/quickstart.md) |
| 选择 Docker 或 JAR | [安装与环境准备](doc/zh-CN/installation.md) |
| 设计反馈数据 | [事件格式](doc/zh-CN/event-schema.md) |
| 接入排序服务 | [JSON API](doc/zh-CN/api.md) |
| 配置特征和模型 | [配置概览](doc/zh-CN/configuration/overview.md) |
| 深入计数、文本和多样性 | [详细特征文档](doc/zh-CN/README.md#详细特征与推荐模型) |
| 配置热门、相似或语义推荐 | [推荐模型文档](doc/zh-CN/README.md#详细特征与推荐模型) |
| 准备运行与部署 | [运行与部署文档](doc/zh-CN/README.md#运行与部署) |
| 统一推荐系统术语 | [中英术语表](doc/zh-CN/glossary.md) |
| 了解视频内容分发应用 | [Metarank × FlowLens 案例](doc/zh-CN/cases/video-distribution-flowlens.md) |
| 查看翻译覆盖范围 | [翻译进度](TRANSLATION_STATUS.md) |

完整英文文档仍保留在 [`doc/`](doc/) 目录中。中文文档位于 [`doc/zh-CN/`](doc/zh-CN/)，文件路径尽量与上游对应，方便比较和同步。`zh-v0.1.0` 已覆盖入门、核心 API、主要特征、推荐模型、数据与状态、CLI、单机/Docker 部署和生产运行边界。

## 一分钟运行

以下示例使用官方 RankLens 数据和内存存储，在本机完成数据导入、模型训练和 API 启动。

```bash
curl -O -L https://github.com/metarank/metarank/raw/master/src/test/resources/ranklens/events/events.jsonl.gz
curl -O -L https://raw.githubusercontent.com/metarank/metarank/master/src/test/resources/ranklens/config.yml

docker run -i -t -p 8080:8080 \
  -v "$(pwd)":/opt/metarank \
  metarank/metarank:0.8.0 standalone \
  --config /opt/metarank/config.yml \
  --data /opt/metarank/events.jsonl.gz
```

启动完成后，可向 `http://localhost:8080/rank/xgboost` 发送候选内容列表。完整步骤、请求示例和反馈闭环见[中文快速开始](doc/zh-CN/quickstart/quickstart.md)。

> 中文文档固定使用明确版本号，避免 `latest` 指向预发布版本。具体稳定版本应以官方发布页和上游安装文档为准。

## 中文版维护原则

- 不翻译 API 路径、字段名、配置键、命令、类名和代码标识符；
- 首次出现的重要术语采用“中文（English）”形式；
- 代码块尽量与上游保持一致，只翻译说明和注释；
- 对上游原文之外的解释明确标注为“译者说明”或“案例”；
- 不修改 Metarank 的核心算法和运行逻辑；
- 每次同步上游后更新翻译状态与对应提交。

## 与 FlowLens 的关系

本仓库包含一篇独立案例，用于解释 Metarank 与 [FlowLens](https://github.com/athenazhang-junzi/FlowLens) 在内容分发链路中的互补关系：

- Metarank：特征计算、模型打分、在线重排和反馈采集；
- FlowLens：数据质量、策略诊断、多目标评估和决策边界；
- FlowLens LiveLab：实验健康检查和决策过程演示。

目前这些项目**没有完成代码级线上集成**。案例只用于说明系统边界和产品方法，不代表真实平台接入、线上 A/B 测试或业务收益。

## 上游与许可

- 上游仓库：[metarank/metarank](https://github.com/metarank/metarank)
- 中文镜像：[athenazhang-junzi/metarank-zh](https://github.com/athenazhang-junzi/metarank-zh)
- 许可协议：[Apache License 2.0](LICENSE)
- 翻译协作：[CONTRIBUTING.zh-CN.md](CONTRIBUTING.zh-CN.md)

本仓库保留上游 `LICENSE`。重新分发或继续修改时，请继续遵守 Apache License 2.0，并保留适用的版权、许可和归属信息。
