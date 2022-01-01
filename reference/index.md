---
title: 参考文档
description: This section includes the reference documentation for the Docker platform’s various APIs, CLIs, and file formats.
notoc: true
---

本节包括 Docker 平台的各种 API、CLI 和文件格式的参考文档。

## 文件格式

| 文件格式 | 描述 |
|:-------|:-----|
| [Dockerfile](/engine/reference/builder/) | 定义单个容器的内容和启动行为 |
| [Compose file](/compose/compose-file/) | 定义多容器应用 |


## 命令行 (CLIs)

| CLI | 描述 |
|:-------|:-----|
| [Docker CLI](/engine/reference/commandline/cli/) | Docker的主 CLI, 包括所有的 `docker` 命令 |
| [Compose CLI](/compose/reference/) | Docker Compose 的 CLI，它允许您构建和运行多容器应用程序 |
| [Daemon CLI (dockerd)](/engine/reference/commandline/dockerd/) | 管理容器的进程 |


## 应用程序编程接口 (APIs)

| API | 描述 |
|:-------|:-----|
| [Engine API](/engine/api/) | Docker 的主要 API，提供对守护程序的编程访问 |
| [Registry API](/registry/spec/api/) | 帮助镜像分发到 engine |
| [Docker Hub API](/docker-hub/api/latest/) | 与 Docker Hub 交互的 API | 

## 驱动程序和规格

| 驱动 | 描述 |
|:-------|:-----|
| [Image specification](/registry/spec/manifest-v2-2/)   | 描述 Docker 镜像的各种组件 |
| [Registry token authentication](/registry/spec/auth/)  | 概述 Docker 注册表身份验证方案|
| [Registry storage drivers](/registry/storage-drivers/) | 在使用 Registry 存储图像时启用对给定云提供商的支持 |

