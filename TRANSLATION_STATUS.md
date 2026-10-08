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

## 第一阶段

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
| 事件格式 | `doc/zh-CN/event-schema.md` | 待翻译 |
| API | `doc/zh-CN/api.md` | 待翻译 |
| 配置概览 | `doc/zh-CN/configuration/overview.md` | 待翻译 |
| 特征提取器 | `doc/zh-CN/configuration/feature-extractors.md` | 待翻译 |
| 排序模型 | `doc/zh-CN/configuration/supported-ranking-models.md` | 待翻译 |

## 后续阶段

后续依次覆盖特征、推荐模型、数据源、持久化、部署、集成、生产运行和开发文档。未翻译页面可直接阅读 [`doc/`](doc/) 下的英文原文。

## 发现问题

如果中文内容与当前代码或上游英文文档不一致，请在 Issue 中同时提供：

1. 中文页面路径；
2. 对应英文页面路径；
3. 上游提交或版本号；
4. 建议修改内容及依据。
