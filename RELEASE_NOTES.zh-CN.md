# Metarank 中文核心文档 `zh-v0.1.0`

发布日期：2026-10-08  
上游基线：`b9c719fedba67b762476ee719570915f4768b6b9`

## 定位

这是 Metarank 非官方简体中文仓库的首个核心文档版本，目标是让中文读者能够：

1. 理解 Metarank 在搜索和推荐链路中的位置；
2. 使用 Docker 跑通官方示例；
3. 理解事件、API、配置、特征和排序模型；
4. 选择热门、ALS 相似内容和语义推荐；
5. 理解状态持久化、重训练和生产服务边界。

该版本为 55 篇上游 Markdown 文档中的 31 篇提供了同路径中文页面，并另外提供中文首页、导航、术语表、同步说明和 FlowLens 案例等原创内容。

## 已覆盖

- 中文首页、入门、安装和快速开始；
- `item`、`user`、`ranking`、`interaction` 事件；
- 排序、反馈、训练、推荐、刷新、神经推理和指标 API；
- LambdaMART、XGBoost、LightGBM、`noop` 和 `shuffle`；
- 标量、类别、向量、计数、CTR、文本、相关性、位置、多样性、用户与时间特征；
- Trending、ALS Similar 和 Semantic 推荐；
- 文件、Kafka、Pulsar、Kinesis 和 REST 数据源；
- 内存、Redis、MapDB 和 RocksDB 状态存储；
- CLI、Docker、Standalone、模型重训练和生产建议；
- 中文术语表、翻译协作规范、上游同步说明和 FlowLens 案例。

## 未覆盖

该版本不是全量逐篇翻译。以下内容继续使用英文原文：

- Kubernetes 和 Helm 详细配置；
- Snowplow 集成；
- Prometheus、自定义日志和预热专页；
- Cross-Encoder、协同过滤等独立教程；
- 自动特征工程与训练结果分析专页；
- 源码构建、性能报告和 Changelog。

未翻译不代表功能不可用。中文页面会链接到必要的英文原文。

## 证据与维护边界

- 中文文档以指定上游提交为基线，不保证自动跟随最新版本；
- 精编页面保留关键字段和操作流程，但可能压缩重复示例与日志；
- 性能数据和推荐参数来自上游说明，不代表本仓库已在所有环境复现；
- 当前未对全部 Docker、Redis、Kafka、Pulsar、Kinesis 和 Kubernetes 组合进行本机运行验证；
- 上游更新后应先比较英文文件差异，再更新对应中文页面和翻译基线。

## 许可与归属

Metarank 核心代码、品牌和原始文档归上游项目及贡献者所有。本仓库是非官方中文翻译与学习指南，保留 Apache License 2.0 和上游归属信息。
