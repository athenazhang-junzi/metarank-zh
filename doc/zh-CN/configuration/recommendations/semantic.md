# 语义相似推荐

> 英文原文：[`doc/configuration/recommendations/semantic.md`](../../../configuration/recommendations/semantic.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

`semantic` 是基于内容的推荐模型，只根据内容神经向量之间的距离计算相似度。

它不依赖用户反馈，因此适合缓解新内容的冷启动问题。

## 使用 BERT/ONNX 编码

```yaml
models:
  semantic:
    type: semantic
    encoder:
      type: bert
      model: metarank/all-MiniLM-L6-v2
      dim: 384
    itemFields: [title, description]
```

- `itemFields`：参与编码的内容字段；
- `encoder`：生成 Embedding 的方式；
- `model`：编码模型；
- `dim`：向量维度。

上游当前对 Embedding 的支持范围有限：

- `bert`：使用 Hugging Face 上 Sentence Transformers 的 ONNX 编码模型；
- `csv`：载入外部预先生成的内容向量字典。

## 使用 CSV 向量

```yaml
models:
  semantic:
    type: semantic
    encoder:
      type: csv
      dim: 384
      path: /opt/dic.csv
    itemFields: [title, description]
```

CSV 格式：

- 第 1 列是内容 ID；
- 第 2 到第 N+1 列是 N 维浮点向量。

```text
p1,1.0,2.0,3.0
p2,2.0,1.5,1.0
```

## 使用边界

- 语义相似不等于用户喜欢；
- 只使用标题和描述时，推荐容易集中在内容表面相似性；
- 文本缺失、语言不一致和字段拼接顺序会影响 Embedding；
- 模型版本、向量维度和 CSV 字典必须保持一致；
- 语义推荐可以支持冷启动，但仍需评估相关性、多样性和内容安全；
- 用户行为足够后，可以与协同过滤或排序模型组合，但融合权重需要单独验证。

调用方式见[推荐 API](../../api.md#推荐)。
