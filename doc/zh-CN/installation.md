# 安装与环境准备

> 英文原文：[`doc/installation.md`](../installation.md)  
> 翻译基线：`b9c719fedba67b762476ee719570915f4768b6b9`

Metarank 提供 Docker 镜像和 JAR 包，可运行在 macOS、Windows 和 Linux 上。

## 使用 Docker

Docker 镜像发布在 [Docker Hub：`metarank/metarank`](https://hub.docker.com/r/metarank/metarank)。

- 官方镜像为多架构镜像，支持 `amd64` 和 `arm64/v8`；
- Apple Silicon Mac 可以原生运行对应架构镜像；
- `latest` 可能指向预发布版本，稳定使用时应固定明确版本；
- 本中文基线文档采用 `0.8.0` 示例。

验证镜像：

```bash
docker run metarank/metarank:0.8.0 --help
```

## 使用 JAR

Metarank 也可从官方 [Releases](https://github.com/metarank/metarank/releases) 页面下载 JAR。它包含 LightGBM 和 XGBoost 的原生库接口，对系统和架构有明确要求：

- Linux：x86_64/AArch64，JVM 21 及以上；
- Windows：x86_64、Windows 10 及以上，JVM 21 及以上；
- macOS：x86_64/AArch64、macOS 11 及以上，JVM 21 及以上。

运行帮助：

```bash
java -jar metarank.jar --help
```

### Java 版本

Metarank 当前在 JDK 21 和 25 上测试，不支持低于 JDK 21 的版本。尚未安装 JDK 时，可参考 [Eclipse Temurin](https://adoptium.net/installation/) 的安装说明。

### macOS 额外依赖

直接运行 JAR 时需要安装 `libomp`：

```bash
brew install libomp
```

缺少 `libomp` 时，训练模型可能出现与 LightGBM 动态库有关的 `UnsatisfiedLinkError`。

## 推荐选择

| 场景 | 建议方式 |
|---|---|
| 第一次体验 | Docker |
| 快速开始示例 | Docker standalone 模式 |
| 本地调试 JVM 应用 | JAR + JDK 21/25 |
| 生产部署 | 固定版本，并继续阅读上游部署文档 |

安装完成后进入[快速开始](quickstart/quickstart.md)。
