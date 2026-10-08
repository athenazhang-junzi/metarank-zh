# Metarank 中英术语表

本表用于统一中文文档用词。代码、API、JSON 字段和 YAML 键保持英文原样。

| English | 推荐中文 | 说明 |
|---|---|---|
| candidate | 候选内容 / 候选项 | 上游召回后等待排序的内容 |
| retrieval | 召回 / 检索 | 从全量集合中取得候选集 |
| ranking | 排序；排序事件 | 结合语境判断算法过程或事件类型 |
| reranking | 重排 / 二次排序 | 对已有候选顺序再次打分排列 |
| Learning-to-Rank | 学习排序 | 首次出现可加缩写 LTR |
| LambdaMART | LambdaMART | 模型名不翻译 |
| feature | 特征 | 模型使用的输入信号 |
| feature extractor | 特征提取器 | 从事件、属性和上下文生成特征 |
| inference | 推理 | 使用已训练模型计算排序结果 |
| feedback | 反馈 | 排序展示和用户交互的回传数据 |
| interaction | 交互 | 点击、购买、喜欢等用户行为 |
| impression | 曝光 / 展示 | 应结合具体事件合同使用 |
| visitor | 访客 | 上游文档中的非登录或统一访问主体 |
| user profile | 用户画像 | 基于用户历史行为形成的特征状态 |
| session | 会话 | 一段连续访问过程 |
| click-through rate | 点击率 | 常用缩写 CTR |
| position bias | 位置偏差 | 排名位置导致的观察和点击差异 |
| debiasing | 去偏 | 降低位置等偏差对训练的影响 |
| data source | 数据源 | REST、文件或流式系统等输入 |
| persistence | 持久化 | 将状态保存在 Redis 等存储中 |
| model serving | 模型服务 | 对外提供模型推理接口 |
| baseline | 基线 | 用于比较的默认或简单策略 |
| guardrail metric | 护栏指标 | 防止主指标改善伴随重要副作用 |
| offline evaluation | 离线评估 | 基于历史或离线数据进行评估 |
| online experiment | 在线实验 | 在真实流量中进行的实验 |

## 保持英文的内容

- 产品与组件名：Metarank、Redis、Kafka、Pulsar、Kinesis；
- 模型和算法名：XGBoost、LightGBM、LambdaMART、ALS；
- API：`/rank`、`/feedback`、`/train`、`/recommend`；
- 字段：`event`、`id`、`items`、`user`、`session`、`timestamp`；
- 命令与配置键。

## 容易混淆的边界

- **召回不等于重排**：Metarank 的主要能力位于候选集之后；
- **ranking 事件不自动等于所有内容都被看见**：实际接入应只记录真实展示范围；
- **模型分数不等于点击概率**：除非模型和校准过程明确支持这种解释；
- **多模型服务不等于完整 A/B 平台**：用户分桶、实验检查和统计决策通常由外部系统承担；
- **离线提升不等于线上业务收益**：两者需要不同证据。
