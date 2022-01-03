---
description: How to use logging driver plugins
title: 使用日志驱动插件
keywords: logging, driver, plugins, monitoring
redirect_from:
- /engine/admin/logging/plugins/
---

Docker 日志插件允许您扩展和自定义 Docker 的日志功能，除了[内置日志驱动](configure.md)的功能。
日志服务提供者可以[实现他们自己的插件](../../../engine/extend/plugins_logging.md) ，并使它们在 Docker Hub 或私有仓库上可用。
本主题展示了日志服务的用户如何配置 Docker 来使用这个插件。

## 安装日志驱动插件

要安装日志驱动插件，请使用 `docker plugin install <org/image>` 提供插件开发人员的信息。

您可以使用 `docker plugin ls` 列出所有已安装的插件，也可以使用 `docker inspect` 来检查特定插件。

## 将插件配置为默认日志驱动程序

安装插件后，您可以将 Docker daemon 配置为默认使用它，通过将 `daemon.json` 文件中的 `log-driver` 选项设置为插件名，
详细信息请参见[日志概览](configure.md#configure-the-default-logging-driver)。
如果日志驱动程序支持其他选项，您可以将它们设置为 `log-opts` 同一文件中数组的值。

## 配置一个容器来使用插件作为日志驱动

安装插件后，您可以通过将`--log-driver` 标志指定为`docker run`，将容器配置为使用插件作为其日志驱动程序，
详细信息请参见[日志概述](configure.md#configure-the-logging-driver-for-a-container)。
如果日志驱动程序支持其他选项，您可以使用一个或多个 `--log-opt` 标志来指定它们，以选项名称作为键，选项值作为值。