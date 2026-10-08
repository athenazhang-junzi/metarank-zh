# 通用派生特征

> 英文原文：[`doc/configuration/features/generic.md`](../../../configuration/features/generic.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

## `word_count`

统计字符串字段中的词数，适合区分长短标题、不同内容形态等：

```yaml
- name: title_length
  type: word_count
  scope: item
  field: item.title
```

## 已移除的 `relative_number`

`relative_number` 已在 0.7.x 中弃用并移除。XGBoost 和 LightGBM 可以直接处理数值特征，应改用 [`number`](scalar.md#布尔值与数值)。

## `list_size`

统计字符串列表或数值列表的元素数量：

```yaml
- name: toggled_filters_count
  type: list_size
  field: filters
  source: item
```

列表长度只是结构信号，使用前仍需检查缺失值、异常长列表和上游字段定义是否一致。
