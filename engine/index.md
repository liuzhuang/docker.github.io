---
description: Engine
keywords: Engine
redirect_from:
- /edge/
- /engine/ce-ee-node-activate/
- /engine/misc/
- /linux/
- /manuals/ # TODO remove this redirect after we've created a landing page for the product manuals section
title: Docker Engine 概述
---

Docker Engine 是一种开源容器化技术，用于构建和容器化您的应用程序。Docker Engine 充当客户端 - 服务器应用程序，具有：

* 具有长时间运行的守护进程的服务器 [`dockerd`](/engine/reference/commandline/dockerd)。
* API 指定程序可以用来与 Docker 守护程序对话和指示的接口。
* 命令行界面 (CLI) 客户端 [`docker`](/engine/reference/commandline/cli/)。

CLI 使用 [Docker APIs](api/index.md) I通过脚本或直接 CLI 命令来控制 Docker 守护程序或与 Docker 守护程序交互。许多其他 Docker 应用程序使用底层 API 和 CLI。守护进程创建和管理 Docker 对象，例如图像、容器、网络和卷。

有关更多详细信息，请参阅 [Docker Architecture](../get-started/overview.md#docker-architecture)。

## Docker 使用指南

要更详细地了解 Docker 并回答有关使用和实现的问题，请查看 [overview page in "get started"](../get-started/overview.md)。

## 安装指南

[installation section](install/index.md) 您展示如何在各种平台上安装 Docker。

## 发布说明

现在可以在单独的 [Release Notes page](release-notes/index.md) 上找到当前系列中每个版本的更改摘要

## 功能弃用政策

随着对 Docker 的更改，有时可能需要删除现有功能或用更新的功能替换。在删除现有功能之前，它在文档中被标记为“已弃用”，并在 Docker 中保留至少 3 个稳定版本，除非另有明确说明。在那之后，它可能会被移除。

用户应注意每个版本的弃用功能列表，并计划尽快从这些功能迁移到替换功能（如果适用）。

弃用功能的完整列表可在 [Deprecated Features page](deprecated.md) 上找到 。

## Licensing

Docker 在 Apache 许可下获得许可，版本 2.0。有关完整的许可证文本，请参阅 [LICENSE](https://github.com/moby/moby/blob/master/LICENSE)。
