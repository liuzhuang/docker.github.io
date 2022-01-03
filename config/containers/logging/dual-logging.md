---
description: Learn how to read container logs locally when using a third party logging solution.
keywords: docker, logging, driver, dual logging, dual-logging, cache, ring-buffer, configuration
title: 将 docker 日志与远程日志驱动程序一起使用
---

## 概述


此前多克尔发动机20.10，该docker logs命令 可能只能通过登录使用的驱动程序，使用支持的容器 local，json-file或journald登录司机。但是，许多第三方日志驱动程序不支持使用本地读取日志docker logs

这在尝试以自动化和标准方式收集日志数据时产生了多个问题。只能通过第三方解决方案以第三方工具指定的格式访问和查看日志信息。

从 Docker Engine 20.10 开始，docker logs无论配置的日志驱动程序或插件如何，您都可以使用读取容器日志。此功能称为“双日志记录”，允许您以docker logs一致的格式在本地读取容器日志，无论使用何种日志驱动程序，因为引擎配置为将信息记录到“本地”日志记录驱动程序。有关其他信息，请参阅配置默认日志记录驱动程序。

双日志记录使用local日志驱动程序作为缓存来读取容器的最新日志。默认情况下，缓存已启用日志文件轮换，并且每个容器最多限制为 5 个文件，每个文件为 20MB（压缩前）。

请参阅配置选项部分以自定义这些默认值，或参阅禁用双日志记录 部分以禁用此功能。

Prior to Docker Engine 20.10, the [`docker logs` command](../../../engine/reference/commandline/logs.md)
could only be used with logging drivers that supported  for containers using the
`local`, `json-file`, or `journald` log drivers. However, many third party logging
drivers had no support for locally reading logs using `docker logs`

This created multiple problems when attempting to gather log data in an
automated and standard way. Log information could only be accessed and viewed
through the third-party solution in the format specified by that
third-party tool. 

Starting with Docker Engine 20.10, you can use `docker logs` to read container
logs regardless of the configured logging driver or plugin. This capability,
referred to as "dual logging", allows you to use `docker logs` to read container
logs locally in a consistent format, regardless of the log driver used, because
the engine is configured to log information to the “local” logging driver. Refer
to [Configure the default logging driver](configure.md) for additional information. 

Dual logging uses the [`local`](local.md) logging driver to act as cache for
reading the latest logs of your containers. By default, the cache has log-file
rotation enabled, and is limited to a maximum of 5 files of 20MB each (before
compression) per container.

Refer to the [configuration options](#configuration-options) section to customize
these defaults, or to the [disable dual-logging](#disable-the-dual-logging-cache)
section to disable this feature.

## 先决条件 
 
使用双日志记录不需要更改配置。如果配置的日志驱动程序不支持读取日志，Docker Engine 20.10 及更高版本会自动启用双日志记录。

以下示例显示了在docker logs具有和不具有双重日志记录可用性的情况下运行命令的结果：

No configuration changes are needed to use dual logging. Docker Engine 20.10 and
up automatically enable dual logging if the configured logging driver does not
support reading logs.

The following examples show the result of running a `docker logs` command with
and without dual logging availability:

### 没有使用双重记录

当容器配置了远程日志驱动，例如splunk，并且禁用了双日志记录时，尝试在本地读取容器日志时会显示错误：

When a container is configured with a remote logging driver such as `splunk`, and
dual logging is disabled, an error is displayed when attempting to read container
logs locally:

- 第 1 步：配置 Docker daemon

    ```console
    $ cat /etc/docker/daemon.json
    {
      "log-driver": "splunk",
      "log-opts": {
        "cache-disabled": "true",
        ... (options for "splunk" logging driver)
      }
    }
    ```

- 第二步：启动容器

    ```console
    $ docker run -d busybox --name testlog top 
    ```

- 第 3 步：读取容器日志

    ```console
    $ docker logs 7d6ac83a89a0
    Error response from daemon: configured logging driver does not support reading
    ```

### 使用双重记录

启用双日志缓存后docker logs，即使日志驱动程序不支持读取日志，也可以使用该命令读取日志。以下示例显示了一个守护程序配置，该配置使用splunk远程日志驱动程序作为默认值，并启用了双日志记录缓存：

With the dual logging cache enabled, the `docker logs` command can be used to
read logs, even if the logging driver does not support reading logs. The following
examples shows a daemon configuration that uses the `splunk` remote logging driver
as a default, with dual logging caching enabled:

- 第 1 步：配置 Docker daemon

    ```console
    $ cat /etc/docker/daemon.json
    {
      "log-driver": "splunk",
      "log-opts": {
        ... (options for "splunk" logging driver)
      }
    }
    ```

- 第二步：启动容器

    ```console
    $ docker run -d busybox --name testlog top 
    ```

- 第 3 步：读取容器日志

    ```console
    $ docker logs 7d6ac83a89a0
    2019-02-04T19:48:15.423Z [INFO]  core: marked as sealed                                          	 
    2019-02-04T19:48:15.423Z [INFO]  core: pre-seal teardown starting                                                                                                 	 
    2019-02-04T19:48:15.423Z [INFO]  core: stopping cluster listeners                                                                                             	 
    2019-02-04T19:48:15.423Z [INFO]  core: shutting down forwarding rpc listeners                                                                                 	 
    2019-02-04T19:48:15.423Z [INFO]  core: forwarding rpc listeners stopped
    2019-02-04T19:48:15.599Z [INFO]  core: rpc listeners successfully shut down
    2019-02-04T19:48:15.599Z [INFO]  core: cluster listeners successfully shut down	
    ```

> **Note**
>
> 对于支持读取日志的日志驱动程序，例如local、json-file 和journald驱动程序，在双日志记录功能可用之前或之后的功能没有区别。对于这些驱动程序，可以docker logs在两种情况下使用日志读取。
> For logging drivers that support reading logs, such as the `local`, `json-file`
> and `journald` drivers, there is no difference in functionality before or after
> the dual logging capability became available. For these drivers, Logs can be
> read using `docker logs` in both scenarios.


### 配置选项

The "dual logging" cache accepts the same configuration options as the
[`local` logging driver](local.md), but with a `cache-` prefix. These options
can be specified per container, and defaults for new containers can be set using
the [daemon configuration file](/engine/reference/commandline/dockerd/#daemon-configuration-file).

By default, the cache has log-file rotation enabled, and is limited to a maximum
of 5 files of 20MB each (before compression) per container. Use the configuration
options described below to customize these defaults.


| Option           | Default   | Description                                                                                                                                       |
|:-----------------|:----------|:--------------------------------------------------------------------------------------------------------------------------------------------------|
| `cache-disabled` | `"false"` | Disable local caching. Boolean value passed as a string (`true`, `1`, `0`, or `false`).                                                           |
| `cache-max-size` | `"20m"`   | The maximum size of the cache before it is rotated. A positive integer plus a modifier representing the unit of measure (`k`, `m`, or `g`).       |
| `cache-max-file` | `"5"`     | The maximum number of cache files that can be present. If rotating the logs creates excess files, the oldest file is removed. A positive integer. |
| `cache-compress` | `"true"`  | Enable or disable compression of rotated log files. Boolean value passed as a string (`true`, `1`, `0`, or `false`).                              |

## 禁用双日志缓存

Use the `cache-disabled` option to disable the dual logging cache. Disabling the
cache can be useful to save storage space in situations where logs are only read
through a remote logging system, and if there is no need to read logs through
`docker logs` for debugging purposes.

Caching can be disabled for individual containers or by default for new containers,
when using the [daemon configuration file](/engine/reference/commandline/dockerd/#daemon-configuration-file).

The following example uses the daemon configuration file to use the ["splunk'](splunk.md)
logging driver as a default, with caching disabled:

```console
$ cat /etc/docker/daemon.json
{
  "log-driver": "splunk",
  "log-opts": {
    "cache-disabled": "true",
    ... (options for "splunk" logging driver)
  }
}
```

> **Note**
>
> For logging drivers that support reading logs, such as the `local`, `json-file`
> and `journald` drivers, dual logging is not used, and disabling the option has
> no effect.

## 限制

- If a container using a logging driver or plugin that sends logs remotely
  suddenly has a "network" issue, no ‘write’ to the local cache occurs. 
- If a write to `logdriver` fails for any reason (file system full, write
  permissions removed), the cache write fails and is logged in the daemon log.
  The log entry to the cache is not retried.
- Some logs might be lost from the cache in the default configuration because a
  ring buffer is used to prevent blocking the stdio of the container in case of
  slow file writes. An admin must repair these while the daemon is shut down.
