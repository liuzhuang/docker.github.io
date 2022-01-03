---
description: Configure logging driver.
keywords: docker, logging, driver
redirect_from:
- /engine/reference/logging/overview/
- /engine/reference/logging/
- /engine/admin/reference/logging/
- /engine/admin/logging/overview/
title: 配置日志驱动
---

Docker 包含多种日志记录机制，可帮助您从[正在运行的容器和服务中获取信息](index.md)。
这些机制称为日志驱动程序。
每个 Docker daemon 都有一个默认的日志驱动程序，除非您将其配置为使用不同的日志驱动程序，否则每个容器都会使用该驱动程序，简称为 “log-driver”。


默认情况下，Docker 使用 [json-file日志驱动程序](json-file.md)，它在内部将容器日志缓存为 JSON。
除了使用 Docker 附带的日志驱动程序，您还可以实现和使用[日志驱动程序插件](plugins.md)。

> **Tip: 使用“本地”日志驱动程序来防止磁盘耗尽** 
> 
> 默认情况下，不执行日志轮换。
> 因此，默认 [`json-file` 日志驱动](json-file.md) 存储的日志文件用于生成大量输出的容器可能会导致大量磁盘空间，从而导致磁盘空间耗尽。
>
> Docker 将 `json-file` 日志驱动程序（无日志轮换）作为默认设置，以保持与旧版本 Docker 的向后兼容性，以及将 Docker 用作 Kubernetes 的运行时的情况。
>
> 对于其他情况，建议使用 "local" 日志驱动程序，因为它默认执行日志轮换，并使用更高效的文件格式。
> 请参阅下面的[配置默认日志驱动程序](#configure-the-default-logging-driver)章节以了解如何将 “local” 日志驱动程序配置为默认值，
> 以及有关 “local” 日志驱动程序的更多详细信息的[本地文件日志驱动程序](local.md)页面。

## 配置默认日志驱动

要将 Docker 守护程序配置为默认使用特定的日志记录驱动程序，请将 `daemon.json` 配置文件中 `log-driver` 的值设置为日志记录驱动程序的名称。
有关详细信息，请参阅 [dockerd 参考手册](/engine/reference/commandline/dockerd/#daemon-configuration-file) 中的 “守护程序配置文件” 部分 。

默认的日志驱动是 `json-file`. 以下示例将默认日志记录驱动程序设置为 [`local` 日志驱动](local.md)：

```json
{
  "log-driver": "local"
}
```

如果日志驱动程序具有可配置的选项，您可以在 `daemon.json` 文件中将它们设置为带有键 `log-opts` 的 JSON 对象。
以下示例在 `json-file` 日志驱动程序上设置了四个可配置选项：

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3",
    "labels": "production_status",
    "env": "os,customer"
  }
}
```

重新启动 Docker 以使更改对新创建的容器生效。现有容器不使用新的日志配置。

> **Note**
>
> `daemon.json` 文件中的 `log-opts` 配置选项必须以字符串形式提供。
> 因此，boolean 和 numeric（例如上面示例中的 `max-file` 值 ）必须用引号 (`"`) 括起来。

如果未指定日志记录驱动程序，则默认为 `json-file`. 
要查找 Docker 守护程序的当前默认日志记录驱动程序，
请运行 `docker info` 并搜索 `Logging Driver`. 您可以在 Linux、macOS 或 Windows 上的 PowerShell 上使用以下命令：

{% raw %}
```console
$ docker info --format '{{.LoggingDriver}}'

json-file
```
{% endraw %}

> **Note**
> 
> 更改守护程序配置中的默认日志驱动程序或日志驱动程序选项只会影响配置更改后创建的容器。
现有容器保留创建时使用的日志驱动程序选项。
要更新容器的日志驱动程序，必须使用所需的选项重新创建容器。
请参阅下面的[为容器配置日志驱动](#configure-the-logging-driver-for-a-container) 部分以了解如何查找容器的日志记录驱动程序配置。

## 为容器配置日志驱动

当您启动一个容器时，您可以使用 `--log-driver` 标志将其配置为使用不同于 Docker daemon 默认值的日志记录驱动程序。
如果日志驱动程序具有可配置选项，您可以使用 `--log-opt <NAME>=<VALUE>` 标志的一个或多个实例来设置它们。
即使容器使用默认日志驱动程序，它也可以使用不同的可配置选项。

以下示例使用 `none` 日志驱动启动 Alpine 容器。

```console
$ docker run -it --log-driver none alpine ash
```

要查找正在运行的容器的当前日志驱动程序，如果守护程序正在使用 `json-file` 日志记录驱动，
请运行以下 `docker inspect` 命令，用容器名称或 ID 替换 `<CONTAINER>`：

{% raw %}
```console
$ docker inspect -f '{{.HostConfig.LogConfig.Type}}' <CONTAINER>

json-file
```
{% endraw %}

## 配置日志消息的传递方式（从容器到日志驱动）

Docker 提供了两种模式来将消息从容器传递到日志驱动程序：

- （默认）直接，阻塞式地从容器到驱动程序传递
- 非阻塞传递，将日志消息存储在中间每个容器的环形缓冲区中以供驱动程序使用

`非阻塞` 消息传递模式可以防止应用程序由于日志回压而阻塞。
当 STDERR 或 STDOUT 流阻塞时，应用程序可能会以意想不到的方式失败。

> **Warning**
>
> 当缓冲区已满且新消息入队时，内存中最旧的消息将被丢弃。
> 丢弃消息通常比阻止应用程序的日志写入过程更受欢迎。
{: .warning}

`mode` 日志选项用于控制是使用 `blocking`(默认)还是 `non-blocking` 消息传递。

`max-buffer-size` 日志选项控制循环缓冲区的大小，当 `mode` 设置为 `non-blocking` 时的中间消息存储。
`max-buffer-size` 默认为 1 兆字节。

以下示例以 `非阻塞模式` 和 `4 兆字节的缓冲区` 启动一个具有日志输出的 Alpine 容器：

```console
$ docker run -it --log-opt mode=non-blocking --log-opt max-buffer-size=4m alpine ping 127.0.0.1
```

### 在日志驱动中使用环境变量或标签

一些日志驱动程序将容器的 `--env|-e` 或 `--label` 标志的值添加到容器的日志中。
此示例使用 Docker daemon 的默认日志记录驱动程序（让我们假设 `json-file`）启动一个容器，但设置了环境变量 `os=ubuntu`。

```console
$ docker run -dit --label production_status=testing -e os=ubuntu alpine sh
```

如果日志驱动程序支持它，这会向日志输出添加其他字段。
以下输出由 `json-file` 日志驱动程序生成：

```json
"attrs":{"production_status":"testing","os":"ubuntu"}
```

## 支持的日志驱动

支持以下日志驱动程序。
如果适用，请参阅每个驱动程序文档的链接以了解其可配置选项。
如果您使用[日志驱动程序插件](plugins.md)，您可能会看到更多选项。

| Driver | Description |
|:-------|:------------|
| `none` | 没有可用于容器的日志并且`docker logs`不返回任何输出。|
| [`local`](local.md) | 日志以自定义格式存储，旨在将开销降至最低。|
| [`json-file`](json-file.md) | 日志格式为 JSON。Docker 的默认日志驱动程序。|
| [`syslog`](syslog.md) | 将日志消息写入`syslog`工具。该`syslog`守护程序必须在主机上运行。|
| [`journald`](journald.md) | 将日志消息写入`journald`. 该`journald`守护程序必须在主机上运行。|
| [`gelf`](gelf.md) | 将日志消息写入 Graylog 扩展日志格式 (GELF) 端点，例如 Graylog 或 Logstash。|
| [`fluentd`](fluentd.md) | 将日志消息写入`fluentd`（转发输入）。该`fluentd`守护程序必须在主机上运行。|
| [`awslogs`](awslogs.md) | 将日志消息写入 Amazon CloudWatch Logs。|
| [`splunk`](splunk.md) | `splunk` 使用 HTTP 事件收集器写入日志消息。|
| [`etwlogs`](etwlogs.md) | 将日志消息作为 Windows 事件跟踪 (ETW) 事件写入。仅在 Windows 平台上可用。|
| [`gcplogs`](gcplogs.md) | 将日志消息写入 Google Cloud Platform (GCP) Logging。|
| [`logentries`](logentries.md) | 将日志消息写入 Rapid7 Logentries。|

> **Note**
>
> 当使用 Docker 19.03 或以上，[`docker logs` 命令](../../../engine/reference/commandline/logs.md) 实现了 `local`, `json-file` 和 `journald`日志记录的驱动程序。
> Docker 20.10 及更高版本引入了 “双日志记录”，它使用本地缓冲区，允许您将`docker logs`命令用于任何日志记录驱动程序。
> 有关详细信息，请参阅 [reading logs when using remote logging drivers](dual-logging.md)。

## 日志驱动限制

- 读取日志信息需要解压轮换的日志文件，这会导致磁盘使用量暂时增加（直到读取轮换文件中的日志条目）和解压缩时增加的 CPU 使用率。
- Docker数据目录所在的主机存储容量决定了日志文件信息的最大大小。
