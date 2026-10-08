# Metarank 中文文档

这里是 Metarank 的非官方简体中文文档。英文原文保留在上一级 [`doc/`](../) 目录，中文页面尽量沿用相同的相对路径。

## 推荐阅读路径

### 第一次接触 Metarank

1. [Metarank 是什么？](intro.md)
2. [安装与环境准备](installation.md)
3. [快速开始](quickstart/quickstart.md)
4. [事件格式](event-schema.md)
5. [JSON API](api.md)
6. [配置概览](configuration/overview.md)
7. [特征提取器](configuration/feature-extractors.md)
8. [支持的排序模型](configuration/supported-ranking-models.md)
9. [中英术语表](glossary.md)

### 理解完整排序闭环

建议按以下顺序理解：

```text
候选内容 → /rank 重排 → 展示排序结果 → /feedback 回传 → 生成训练样本 → 重新训练模型
```

中文核心参考已经覆盖事件、API、配置、排序模型和主要特征类型。部署与集成页面仍在后续阶段，未翻译部分请阅读对应[英文原文](../_toc.md)。

### 详细特征与推荐模型

特征：

- [计数、窗口与比率](configuration/features/counters.md)
- [文本与神经匹配](configuration/features/text.md)
- [上游相关性与位置](configuration/features/relevancy.md)
- [排序多样性](configuration/features/diversity.md)
- [用户与会话特征](configuration/features/user-session.md)

推荐模型：

- [热门趋势推荐](configuration/recommendations/trending.md)
- [ALS 相似内容](configuration/recommendations/similar.md)
- [语义相似推荐](configuration/recommendations/semantic.md)

### 运行与部署

- [CLI](cli.md)
- [数据源](configuration/data-sources.md)
- [状态持久化](configuration/persistence.md)
- [点击模型](click-models.md)
- [时间戳格式](timestamp-formats.md)
- [部署概览](deploy/deployment-overview.md)
- [Docker](deploy/docker.md)
- [Standalone](deploy/standalone.md)
- [模型重训练](howto/model-retraining.md)
- [生产运行建议](dev/production-recommendations.md)

### 中文案例

- [Metarank × FlowLens：视频内容分发案例](cases/video-distribution-flowlens.md)

案例属于中文仓库新增内容，不是 Metarank 官方文档，也不代表两个项目已经完成代码集成。

## 文档状态

完整状态见仓库根目录的 [`TRANSLATION_STATUS.md`](../../TRANSLATION_STATUS.md)。遇到中英文不一致时，以对应版本的上游代码、英文文档和官方发布说明为准。

## 参与维护

翻译术语和提交规则见 [`CONTRIBUTING.zh-CN.md`](../../CONTRIBUTING.zh-CN.md)。同步官方仓库的方法见[上游同步说明](upstream-sync.md)。
