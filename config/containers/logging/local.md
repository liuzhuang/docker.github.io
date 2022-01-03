---
description: Describes how to use the local logging driver.
keywords: local, docker, logging, driver
redirect_from:
- /engine/reference/logging/local/
- /engine/admin/logging/local/
title: Local 文件日志驱动
---

`local` 日志驱动从容器的 stdout/stderr 捕获输出，并将其写入到为性能和磁盘使用而优化的内部存储。

默认情况下，`local` 驱动为每个容器保留 100MB 的日志消息，并使用自动压缩来减少磁盘大小。
100MB 默认值基于每个文件 20M 的默认大小和此类文件数量的默认计数 5（以考虑日志轮换）。

> *Note*
>
> `local` 日志驱动使用基于文件的存储。
> 文件格式和存储机制设计为由 Docker daemon 独占访问，不应由外部工具使用，因为在未来版本中实现可能会发生变化。

## 用法

要将 `local` 驱动用作默认日志记录驱动，请将`daemon.json`文件中`log-driver`和`log-opt`键设置为的适当值，该文件位于`/etc/docker/` 或 `C:\ProgramData\docker\config\daemon.json`上。
有关使用 `daemon.json` 更多信息，请参阅 [daemon.json](../../../engine/reference/commandline/dockerd.md#daemon-configuration-file)。


以下示例将日志驱动程序设置为 `local` 并设置 `max-size` 选项。

```json
{
  "log-driver": "local",
  "log-opts": {
    "max-size": "10m"
  }
}
```

重新启动 Docker 以使更改对新创建的容器生效。现有容器不使用新的日志配置。

您可以通过 `docker container create` 或 `docker run` 的 `--log-driver` 参数指定容器使用特定的日志驱动。

```console
$ docker run \
      --log-driver local --log-opt max-size=10m \
      alpine echo hello world
```

### 选项

`local` 日志驱动支持以下日志记录选项：

| 可选项 | 描述 | 示例值 |
|:-----|:-------|-----------|
| `max-size`  | The maximum size of the log before it is rolled. A positive integer plus a modifier representing the unit of measure (`k`, `m`, or `g`). Defaults to 20m.     | `--log-opt max-size=10m`   |
| `max-file`  | The maximum number of log files that can be present. If rolling the logs creates excess files, the oldest file is removed. A positive integer. Defaults to 5. | `--log-opt max-file=3`     |
| `compress`  | Toggle compression of rotated log files. Enabled by default.                                                                                                  | `--log-opt compress=false` |

### 示例

此示例启动一个 `alpine` 容器，该容器最多可包含 3 个日志文件，每个文件不超过 10 兆字节。

```console
$ docker run -it --log-driver local --log-opt max-size=10m --log-opt max-file=3 alpine ash
```
