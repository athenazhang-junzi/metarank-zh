# 上游同步说明

本仓库通过两个 Git remote 区分中文 Fork 和官方项目：

```text
origin    https://github.com/athenazhang-junzi/metarank-zh.git
upstream  https://github.com/metarank/metarank.git
```

## 查看当前远程仓库

```bash
git remote -v
```

## 获取上游更新

```bash
git fetch upstream
```

获取更新后，先查看差异：

```bash
git log --oneline --decorate --graph HEAD..upstream/master
git diff --stat HEAD..upstream/master
```

不要在未检查差异时直接覆盖中文文件。建议先在独立分支同步：

```bash
git switch -c sync-upstream-YYYYMMDD
git merge upstream/master
```

完成冲突处理和文档检查后：

1. 更新 [`TRANSLATION_STATUS.md`](../../TRANSLATION_STATUS.md) 中的基线提交；
2. 检查已翻译页面对应的英文文件是否发生变化；
3. 更新受影响的中文页面；
4. 检查相对链接和代码示例；
5. 合并同步分支。

## 为什么保留英文 `doc/`？

保留英文文档可以：

- 直接比较中英文内容；
- 降低同步上游时的冲突；
- 在中文页面尚未更新时提供准确原文；
- 避免把机器翻译误认为官方结论。
