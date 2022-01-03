---
description: How to keep containers running when the daemon isn't available.
keywords: docker, upgrade, daemon, dockerd, live-restore, daemonless container
title: 在 Docker daemon 停机时，让容器保持存活
redirect_from:
- /engine/admin/live-restore/
---

默认情况下，当 Docker daemon 终止时，它会关闭正在运行的容器。
您可以配置 daemon，以便在 daemon 不可用时容器保持运行。
此功能称为 _live restore_。_live restore_ 有助于减少由于 daemon 崩溃、计划中断或升级 导致容器停机的时间。

> **Note**
>
> Windows 容器不支持 _Live restore_，但它适用于在 Windows Docker Desktop 上运行的 Linux 容器。

## 开启 live restore

有两种方法可以启用 live restore，以便在 daemon 变得不可用时使容器保持活动状态。**仅执行以下操作之一**。

* 将配置添加到 daemon 配置文件中。在 Linux 上，默认为 `/etc/docker/daemon.json`。在 Docker Desktop for Mac 或 Docker Desktop for Windows 上，从任务栏中选择 Docker 图标，然后点击 **Preferences** -> **Daemon** -> **Advanced**。

  - 使用以下 JSON 启用 `live-restore`。

    ```json
    {
      "live-restore": true
    }
    ```

  - 启动 Docker daemon。
    在 Linux 上，您可以通过重新加载 Docker 守护程序来避免重新启动（避免容器的任何停机时间）。
    如果使用 `systemd`，则使用命令 `systemctl reload docker`。
    否则，向进程 `dockerd` 发送 `SIGHUP` 信号。

* 如果您愿意，可以使用 `--live-restore` 标志手动启动 `dockerd` 进程。
  这可能会导致意外行为。

## 升级期间 Live restore

_Live restore_ 允许您在 Docker daemon 更新时保持容器运行，但仅在安装补丁版本 (`YY.MM.x`) 时才受支持，不适用于主要 (`YY.MM`) 守护程序升级。

如果您在升级期间跳过版本，守护程序可能无法恢复其与容器的连接。
如果守护程序无法恢复连接，则它无法管理正在运行的容器，您必须手动停止它们。

## 重启时 Live restore

如果守护程序选项（例如 bridge IP addresses 和 graph driver）未更改，则实时还原选项仅适用于还原容器。
如果这些守护程序级别的配置选项中的任何一个发生了更改，实时还原可能无法工作，您可能需要手动停止容器。

## Live restore 对运行中的容器的影响

如果守护进程长时间关闭，正在运行的容器可能会填满守护进程通常读取的 FIFO 日志。完整的日志会阻止容器记录更多数据。默认缓冲区大小为 64K。如果缓冲区已满，您必须重新启动 Docker 守护程序以刷新它们。

在 Linux 上，您可以通过更改 `/proc/sys/fs/pipe-max-size`. 
您不能在 Docker Desktop for Mac 或 Docker Desktop for Windows 上修改缓冲区大小。

## Live restore 和 swarm mode 集群

实时恢复选项仅适用于独立容器，不适用于 swarm 服务。
Swarm 服务由 Swarm 管理器管理。
如果群管理器不可用，群服务将继续在工作节点上运行，但在有足够的群管理器可用以维持法定人数之前无法进行管理。