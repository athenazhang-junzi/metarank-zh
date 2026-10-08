# Metarank 中文文档

这里是 Metarank 的非官方简体中文文档。英文原文保留在上一级 [`doc/`](../) 目录，中文页面尽量沿用相同的相对路径。

## 推荐阅读路径

### 第一次接触 Metarank

1. [Metarank 是什么？](intro.md)
2. [安装与环境准备](installation.md)
3. [快速开始](quickstart/quickstart.md)
4. [中英术语表](glossary.md)

### 理解完整排序闭环

建议按以下顺序理解：

```text
候选内容 → /rank 重排 → 展示排序结果 → /feedback 回传 → 生成训练样本 → 重新训练模型
```

事件格式、API、配置和模型文档将在后续阶段逐步翻译。翻译完成前，请阅读对应[英文原文](../_toc.md)。

### 中文案例

- [Metarank × FlowLens：视频内容分发案例](cases/video-distribution-flowlens.md)

案例属于中文仓库新增内容，不是 Metarank 官方文档，也不代表两个项目已经完成代码集成。

## 文档状态

完整状态见仓库根目录的 [`TRANSLATION_STATUS.md`](../../TRANSLATION_STATUS.md)。遇到中英文不一致时，以对应版本的上游代码、英文文档和官方发布说明为准。

## 参与维护

翻译术语和提交规则见 [`CONTRIBUTING.zh-CN.md`](../../CONTRIBUTING.zh-CN.md)。同步官方仓库的方法见[上游同步说明](upstream-sync.md)。
