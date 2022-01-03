---
description: How to start containers automatically
keywords: containers, restart, policies, automation, administration
redirect_from:
- /engine/articles/host_integration/
- /engine/admin/host_integration/
- /engine/admin/start-containers-automatically/
title: 自动启动容器
---

Docker 提供[重启策略](../../engine/reference/run.md#restart-policies---restart) 来控制您的容器是在退出时自动启动，还是在 Docker 重启时自动启动。
重启策略确保被 linked 的容器以正确的顺序启动。
Docker 建议您使用重启策略，并避免使用进程管理器来启动容器。

重启策略和 `dockerd` 命令的 `--live-restore` 标志不同。`--live-restore` 允许您在 Docker 升级期间保持容器运行，尽管网络和用户输入会被中断。

## 重启策略

要为容器配置重启策略，请在使用 `docker run` 命令时使用 `--restart` 标志。 `--restart` 标志的值可以是以下任何一项：

| Flag | Description |
|:------|:-----------|
| `no` | 不自动重启容器。（默认） |
| `on-failure[:max-retries]` | 如果容器因错误退出，则重新启动容器，这表现为非零退出代码。或者，使用该`:max-retries`选项限制 Docker 守护程序尝试重新启动容器的次数。|
| `always` | 如果容器停止，请始终重新启动容器。如果是手动停止，则只有在Docker daemon重启或容器本身手动重启时才会重启。（请参阅重启策略详细信息中列出的第二个项目符号） |
| `unless-stopped` | 与 `always` 类似，除了当容器停止（手动或其他方式）时，即使在 Docker 守护程序重新启动后也不会重新启动。 |

以下示例启动一个 Redis 容器并将其配置为始终重新启动，除非它被明确停止或重新启动 Docker。

```console
$ docker run -d --restart unless-stopped redis
```

此命令为正在运行的，名为 `redis` 容器的更改重启策略。

```console
$ docker update --restart unless-stopped redis
```

此命令将确保所有当前运行的容器将重新启动，除非停止。

```console
$ docker update --restart unless-stopped $(docker ps -q)
```

### 重启策略详情

使用重启策略时请记住以下几点：

- 重启策略只有在容器启动成功后才会生效。
  在这种情况下，启动成功意味着容器至少启动了 10 秒，并且 Docker 已经开始监控它。这可以防止根本没有启动的容器进入重启循环。

- 如果您手动停止一个容器，它的重启策略将被忽略，直到 Docker 守护进程重启或容器被手动重启。这是防止重新启动循环的另一种尝试。

- 重启策略仅适用于 _容器_。swarm services 的重启策略配置不同。请参阅 [flags related to service restart](../../engine/reference/commandline/service_create.md)。


## 使用进程管理器

如果重启策略不适合您的需求，例如当 Docker 外部的进程依赖于 Docker 容器时，您可以使用进程管理器，例如 
[upstart](http://upstart.ubuntu.com/),
[systemd](https://freedesktop.org/wiki/Software/systemd/), 或者
[supervisor](http://supervisord.org/) instead.

> **Warning**
>
> 不要尝试将 Docker 重启策略与主机级进程管理器结合使用，因为这会产生冲突。
{:.warning}

要使用进程管理器，请将其配置为使用您通常用于手动启动容器的相同 `docker start` 或 `docker service` 命令来启动您的容器或服务。
有关更多详细信息，请参阅特定进程管理器的文档。

### 在容器内使用进程管理器

进程管理器还可以在容器内运行以检查进程是否正在运行，如果没有，则启动/重新启动它。

> **Warning**
>
> 这些不是 Docker 感知的，只是监视容器内的操作系统进程。
> Docker 不推荐这种方法，因为它依赖于平台，甚至在给定 Linux 发行版的不同版本中也不同。
{:.warning}
