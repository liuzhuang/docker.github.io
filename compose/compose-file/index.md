---
description: Compose 文件参考
keywords: fig, composition, compose, docker
redirect_from:
- /compose/yml
- /compose/compose-file/compose-file-v1/
title: Compose file
toc_max: 4
toc_min: 1
---

##  参考和指南

这些主题描述了 Compose 格式的 Docker Compose 实现。Docker Compose  **1.27.0+** 实现了 [Compose Specification](https://github.com/compose-spec/compose-spec/blob/master/spec.md) 定义的格式。
以前的 Docker Compose 版本支持多种 Compose 文件格式 – 2, 2.x, 和 3.x。Compose 规范是统一的 2.x 和 3.x 文件格式，聚合了这些格式的属性。

## Compose 和 Docker 兼容性表格

Compose 文件格式有多个版本 - 2、2.x 和 3.x。下表提供了各种版本的快照。有关每个版本包含的内容以及如何升级的完整详细信息，请参阅 **[About versions and upgrading](compose-versioning.md)**。

{% include content/compose-matrix.md %}

## Compose documentation

- [User guide](../index.md)
- [Installing Compose](../install.md)
- [Compose file versions and upgrading](compose-versioning.md)
- [Sample apps with Compose](../samples-for-compose.md)
- [Enabling GPU access with Compose](../gpu-support.md)
- [Command line reference](../reference/index.md)
