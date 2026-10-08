# Metarank 中文翻译进度

## 基线

- 上游仓库：[`metarank/metarank`](https://github.com/metarank/metarank)
- 上游分支：`master`
- 当前翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`
- 基线日期：2026-10-06
- 状态更新时间：2026-10-08

该提交信息为 `Update pulsar-client, pulsar-client-admin to 4.2.5 (#1727)`。中文翻译与上游代码版本并非自动同步，阅读技术参数时应同时核对英文原文和官方发布说明。

## 状态说明

| 状态 | 含义 |
|---|---|
| 完整翻译 | 已逐段覆盖原文并完成基本链接检查 |
| 中文精编 | 保留关键内容和可运行步骤，压缩重复示例或宣传性内容 |
| 进行中 | 已建立页面，仍需补充或与上游逐段核对 |
| 待翻译 | 当前只有英文上游文档 |
| 中文原创 | 不是上游原文翻译，属于中文仓库新增内容 |

## 第一、二阶段：入门与核心参考

| 内容 | 中文文件 | 状态 |
|---|---|---|
| 仓库首页 | [`README.md`](README.md) | 中文原创 |
| 项目介绍 | [`doc/zh-CN/intro.md`](doc/zh-CN/intro.md) | 中文精编 |
| 快速开始 | [`doc/zh-CN/quickstart/quickstart.md`](doc/zh-CN/quickstart/quickstart.md) | 中文精编 |
| 安装与环境 | [`doc/zh-CN/installation.md`](doc/zh-CN/installation.md) | 完整翻译 |
| 中文文档导航 | [`doc/zh-CN/README.md`](doc/zh-CN/README.md) | 中文原创 |
| 中英术语表 | [`doc/zh-CN/glossary.md`](doc/zh-CN/glossary.md) | 中文原创 |
| 上游同步说明 | [`doc/zh-CN/upstream-sync.md`](doc/zh-CN/upstream-sync.md) | 中文原创 |
| 视频内容分发案例 | [`doc/zh-CN/cases/video-distribution-flowlens.md`](doc/zh-CN/cases/video-distribution-flowlens.md) | 中文原创 |
| 事件格式 | [`doc/zh-CN/event-schema.md`](doc/zh-CN/event-schema.md) | 中文精编 |
| API | [`doc/zh-CN/api.md`](doc/zh-CN/api.md) | 中文精编 |
| 配置概览 | [`doc/zh-CN/configuration/overview.md`](doc/zh-CN/configuration/overview.md) | 中文精编 |
| 特征提取器 | [`doc/zh-CN/configuration/feature-extractors.md`](doc/zh-CN/configuration/feature-extractors.md) | 中文精编 |
| 排序模型 | [`doc/zh-CN/configuration/supported-ranking-models.md`](doc/zh-CN/configuration/supported-ranking-models.md) | 中文精编 |

## 第三阶段：详细特征与推荐模型

| 内容 | 中文文件 | 状态 |
|---|---|---|
| 计数、窗口与比率 | [`doc/zh-CN/configuration/features/counters.md`](doc/zh-CN/configuration/features/counters.md) | 中文精编 |
| 文本与神经匹配 | [`doc/zh-CN/configuration/features/text.md`](doc/zh-CN/configuration/features/text.md) | 中文精编 |
| 上游相关性与位置 | [`doc/zh-CN/configuration/features/relevancy.md`](doc/zh-CN/configuration/features/relevancy.md) | 中文精编 |
| 排序多样性 | [`doc/zh-CN/configuration/features/diversity.md`](doc/zh-CN/configuration/features/diversity.md) | 中文精编 |
| 用户与会话特征 | [`doc/zh-CN/configuration/features/user-session.md`](doc/zh-CN/configuration/features/user-session.md) | 中文精编 |
| 热门趋势推荐 | [`doc/zh-CN/configuration/recommendations/trending.md`](doc/zh-CN/configuration/recommendations/trending.md) | 中文精编 |
| ALS 相似内容 | [`doc/zh-CN/configuration/recommendations/similar.md`](doc/zh-CN/configuration/recommendations/similar.md) | 中文精编 |
| 语义相似推荐 | [`doc/zh-CN/configuration/recommendations/semantic.md`](doc/zh-CN/configuration/recommendations/semantic.md) | 中文精编 |

## 发布收尾：`zh-v0.1.0`

| 内容 | 中文文件 | 状态 |
|---|---|---|
| 标量、向量与类别特征 | [`doc/zh-CN/configuration/features/scalar.md`](doc/zh-CN/configuration/features/scalar.md) | 中文精编 |
| 通用派生特征 | [`doc/zh-CN/configuration/features/generic.md`](doc/zh-CN/configuration/features/generic.md) | 中文精编 |
| 日期与时间特征 | [`doc/zh-CN/configuration/features/datetime.md`](doc/zh-CN/configuration/features/datetime.md) | 中文精编 |
| 时间戳格式 | [`doc/zh-CN/timestamp-formats.md`](doc/zh-CN/timestamp-formats.md) | 中文精编 |
| 数据源 | [`doc/zh-CN/configuration/data-sources.md`](doc/zh-CN/configuration/data-sources.md) | 中文精编 |
| 状态持久化 | [`doc/zh-CN/configuration/persistence.md`](doc/zh-CN/configuration/persistence.md) | 中文精编 |
| 点击模型 | [`doc/zh-CN/click-models.md`](doc/zh-CN/click-models.md) | 中文精编 |
| CLI | [`doc/zh-CN/cli.md`](doc/zh-CN/cli.md) | 中文精编 |
| 部署概览 | [`doc/zh-CN/deploy/deployment-overview.md`](doc/zh-CN/deploy/deployment-overview.md) | 中文精编 |
| Docker | [`doc/zh-CN/deploy/docker.md`](doc/zh-CN/deploy/docker.md) | 中文精编 |
| Standalone | [`doc/zh-CN/deploy/standalone.md`](doc/zh-CN/deploy/standalone.md) | 中文精编 |
| 模型重训练 | [`doc/zh-CN/howto/model-retraining.md`](doc/zh-CN/howto/model-retraining.md) | 中文精编 |
| 生产运行建议 | [`doc/zh-CN/dev/production-recommendations.md`](doc/zh-CN/dev/production-recommendations.md) | 中文精编 |

## 保留英文的低频页面

`zh-v0.1.0` 定位为核心功能中文版，不宣称 55 篇文档全部完成。Kubernetes、Snowplow、自定义日志、Prometheus 专页、源码构建、性能报告、部分搜索/推荐教程和 Changelog 继续保留英文原文。

完整范围与发布边界见 [`RELEASE_NOTES.zh-CN.md`](RELEASE_NOTES.zh-CN.md)。

## 发现问题

如果中文内容与当前代码或上游英文文档不一致，请在 Issue 中同时提供：

1. 中文页面路径；
2. 对应英文页面路径；
3. 上游提交或版本号；
4. 建议修改内容及依据。
