---
description: Describes how to use the json-file logging driver.
keywords: json-file, docker, logging, driver
redirect_from:
- /engine/reference/logging/json-file/
- /engine/admin/logging/json-file/
title: JSON 文件日志驱动
---


默认情况下，Docker 会捕获所有容器的标准输出（和标准错误），并使用 JSON 格式将它们写入文件中。JSON 格式用它的来源（`stdout` 或 `stderr`）和它的时间戳来注释每一行。每个日志文件仅包含有关一个容器的信息。

```json
{"log":"Log line is here\n","stream":"stdout","time":"2019-01-01T11:11:11.111111111Z"}
```

> *Warning*
>
> `json-file` 日志驱动使用基于文件的存储。
> 这些文件旨在由 Docker 守护程序独占访问。使用外部工具与这些文件交互可能会干扰 Docker 的日志系统并导致意外行为，应避免。

## 用法

要将 `json-file` 驱动用作默认日志记录驱动，请将 `daemon.json` 文件中 `log-driver` 和 `log-opt` 键设置为的适当值，该文件位于`/etc/docker/` 或 `C:\ProgramData\docker\config\daemon.json`上。
有关使用 `daemon.json` 更多信息，请参阅 [daemon.json](../../../engine/reference/commandline/dockerd.md#daemon-configuration-file)。

以下示例将日志驱动程序设置为 `json-file` 并设置 `max-size` 和 `max-file` 选项以启用自动日志轮换。

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3" 
  }
}
```

> **Note**
>
> log-opts配置daemon.json文件中的配置选项必须以字符串形式提供。因此，布尔值和数字值（max-file例如上面示例中的值 ）必须用引号 ( ")括起来。

重新启动 Docker 以使更改对新创建的容器生效。现有容器不使用新的日志配置。


 `docker container create` or `docker run` -> `--log-driver` flag：

```console
$ docker run \
      --log-driver json-file --log-opt max-size=10m \
      alpine echo hello world
```

### 可选项

`json-file` 日志驱动程序支持以下日志记录选项：

| 可选项 | 描述 | 示例值 |
|:------------|:---------|:----------|
| `max-size`     | The maximum size of the log before it is rolled. A positive integer plus a modifier representing the unit of measure (`k`, `m`, or `g`). Defaults to -1 (unlimited).                                          | `--log-opt max-size=10m`                          |
| `max-file`     | The maximum number of log files that can be present. If rolling the logs creates excess files, the oldest file is removed. **Only effective when `max-size` is also set.** A positive integer. Defaults to 1. | `--log-opt max-file=3`                            |
| `labels`       | Applies when starting the Docker daemon. A comma-separated list of logging-related labels this daemon accepts. Used for advanced [log tag options](log_tags.md).                                              | `--log-opt labels=production_status,geo`          |
| `labels-regex` | Similar to and compatible with `labels`. A regular expression to match logging-related labels. Used for advanced [log tag options](log_tags.md).                                                              | `--log-opt labels-regex=^(production_status|geo)` |
| `env`          | Applies when starting the Docker daemon. A comma-separated list of logging-related environment variables this daemon accepts. Used for advanced [log tag options](log_tags.md).                               | `--log-opt env=os,customer`                       |
| `env-regex`    | Similar to and compatible with `env`. A regular expression to match logging-related environment variables. Used for advanced [log tag options](log_tags.md).                                                  | `--log-opt env-regex=^(os|customer)`              |
| `compress`     | Toggles compression for rotated logs. Default is `disabled`.                                                                                                                                                  | `--log-opt compress=true`                         |


### 示例

此示例启动一个 `alpine` 容器，该容器最多可包含 3 个日志文件，每个文件不超过 10 兆字节。

```console
$ docker run -it --log-opt max-size=10m --log-opt max-file=3 alpine ash
```
