# Metarank CLI

> 英文原文：[`doc/cli.md`](../cli.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`  
> 版本说明：本文保留全部子命令和关键参数，压缩了长篇日志与外部训练输出。

基本格式：

```bash
java -jar metarank.jar <command> <args>
```

查看帮助和版本：

```bash
java -jar metarank.jar --help
java -jar metarank.jar --version
```

## 子命令概览

| 命令 | 用途 |
|---|---|
| `import` | 导入历史事件并更新状态 |
| `train` | 使用点击链训练排序模型 |
| `serve` | 启动在线推理 API |
| `standalone` | 依次执行导入、训练和服务 |
| `validate` | 验证配置和事件数据 |
| `sort` | 按时间戳排序历史事件 |
| `autofeature` | 从已有数据生成参考特征配置 |
| `export` | 导出训练集供外部调参 |
| `termfreq` | 为 BM25 生成词频字典 |

## 常用输入参数

- `--config` / `-c`：配置文件路径；
- `--data` / `-d`：输入文件或目录；
- `--format` / `-f`：`json`、`snowplow:tsv`、`snowplow:json`；
- `--offset` / `-o`：`earliest`、`latest`、`ts=<timestamp>`、`last=<duration>`；
- `--sort-files-by` / `-s`：多个文件按名称或修改时间排序；
- `--validation` / `-v`：是否执行输入验证。

## 验证

```bash
java -jar metarank.jar validate \
  --config config.yml \
  --data events.jsonl.gz
```

验证包括：

- 配置能否加载；
- 事件是否按时间排序；
- 事件类型及数量；
- 特征引用的字段是否存在；
- 交互能否关联到排序；
- 交互内容是否存在元数据；
- 交互位置与类型是否合法。

验证可能把事件载入内存，大数据集需要关注 OOM。失败项目应修复后再训练，不能只因为其他检查通过就忽略阻断错误。

## 历史数据排序

Metarank 期望历史数据按时间戳升序。

单文件：

```bash
java -jar metarank.jar sort \
  --data unsorted_file.jsonl.gz \
  --out sorted_file.jsonl.gz
```

目录：

```bash
java -jar metarank.jar sort \
  --data /my_folder \
  --out sorted_file.jsonl.gz
```

## 自动生成特征配置

```bash
java -jar metarank.jar autofeature \
  --data /path/to/events.json \
  --out /path/to/config.yaml
```

可选参数：

- `--cat-threshold`：判断类别字段的最低频率阈值，默认 `0.003`；
- `--ruleset`：`stable` 或 `all`。

自动生成结果只是起点，仍需检查字段语义、数据泄漏、TTL、作用域和模型目标。

## 训练模型

```bash
java -jar metarank.jar train \
  --config /path/to/config.yaml \
  --model <model-name> \
  --split time=80%
```

未指定 `--model` 时，会顺序训练配置中的全部模型。

切分策略：

- `random=N%`：随机切分，可能造成未来信息泄漏；
- `time=N%`：按时间排序，前 N% 用于训练，默认方式；
- `hold_last=N%`：按用户分组，在用户内部按时间保留后段样本。

示例：

```bash
--split random=90%
--split random
--split time=80%
```

推荐优先使用时间切分，并根据应用场景额外检查日志选择偏差。

## 启动服务

```bash
java -jar metarank.jar serve --config config.yml
```

`serve` 只启动在线 API，不负责当前进程内的历史导入和模型训练，适合生产服务实例。

## Standalone

```bash
java -jar metarank.jar standalone \
  --config config.yml \
  --data events.jsonl.gz
```

它把 `import`、`train`、`serve` 合并执行，适合本地和演示，不建议直接作为生产架构。

## 导出训练数据

```bash
java -jar metarank.jar export \
  --config /path/to/config.yaml \
  --model <model-name> \
  --out /export/dir \
  --split time=80%
```

- XGBoost 后端导出带 `qid` 的 LibSVM 训练与测试文件；
- LightGBM 后端导出包含 `label` 和 `group` 的 CSV；
- 同时生成包含默认参数的 XGBoost 或 LightGBM 配置；
- `--sample` 可控制导出的点击链采样比例。

导出数据可以用于外部超参数搜索，但外部工具的结果仍需回填到 Metarank 配置并重新验证。

## BM25 词频字典

```bash
java -jar metarank.jar termfreq \
  --data <path-to-data> \
  --out /term-freq.json \
  --fields title,description \
  --language en
```

生成的字典可被多个 `field_match` 特征复用：

```yaml
- name: title_match
  type: field_match
  rankingField: ranking.query
  itemField: item.title
  method:
    type: bm25
    language: english
    termFreq: "/path/to/term-freq.json"
```

## 环境变量

配置文件路径也可以通过环境变量提供，常用于 Docker 和 Kubernetes：

```bash
METARANK_CONFIG=s3://bucket/prefix/config.yml
```
