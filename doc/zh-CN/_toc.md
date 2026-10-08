# Metarank 中文文档

## 入门

- [Metarank 是什么？](intro.md)
- [安装与环境准备](installation.md)
- [快速开始](quickstart/quickstart.md)

## 核心参考

- [事件格式](event-schema.md)
- [JSON API](api.md)
- [配置概览](configuration/overview.md)
  - [特征提取器](configuration/feature-extractors.md)
    - [计数、窗口与比率](configuration/features/counters.md)
    - [文本与神经匹配](configuration/features/text.md)
    - [上游相关性与位置](configuration/features/relevancy.md)
    - [排序多样性](configuration/features/diversity.md)
    - [用户与会话特征](configuration/features/user-session.md)
  - [支持的排序模型](configuration/supported-ranking-models.md)
  - 推荐模型
    - [热门趋势推荐](configuration/recommendations/trending.md)
    - [ALS 相似内容](configuration/recommendations/similar.md)
    - [语义相似推荐](configuration/recommendations/semantic.md)

## 中文补充

- [中英术语表](glossary.md)
- [Metarank × FlowLens：视频内容分发案例](cases/video-distribution-flowlens.md)
- [上游同步说明](upstream-sync.md)

## 翻译状态

- [查看翻译进度](../../TRANSLATION_STATUS.md)
- [查看英文完整目录](../_toc.md)
