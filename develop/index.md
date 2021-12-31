---
title: 使用 Docker 开发
description: Overview of developer resources
keywords: developer, developing, apps, api, sdk
---

此页面包含一个资源列表，供想要使用 Docker 构建新应用程序的应用程序开发人员使用。

## 先决条件

通过[Get started](../get-started/index.md)中的学习模块了解如何构建映像并将其作为容器化应用程序运行。

## 在 Docker 上开发一个新应用

如果您刚刚开始在 Docker 上开发全新的应用程序，请查看这些资源以了解一些最常见的模式，以便从 Docker 中获得最大收益。

- 使用多阶段构建 [multi-stage builds](develop-images/multistage-build.md){: target="_blank" rel="noopener" class="_"} 来保持您的图像瘦小
- 使用 [volumes](../storage/volumes.md) 和 [bind mounts](../storage/bind-mounts.md){: target="_blank" rel="noopener" class="_"} 管理应用程序数据
- 使用 Kubernetes 扩展您的应用程序 [Scale your app with Kubernetes](../get-started/kube-deploy.md){: target="_blank" rel="noopener" class="_"} 
- 将您的应用扩展为 Swarm 服务 [Scale your app as a Swarm service](../get-started/swarm-deploy.md){: target="_blank" rel="noopener" class="_"} 
- 一般应用程序开发最佳实践 [General application development best practices](dev-best-practices.md){: target="_blank" rel="noopener" class="_"}

## 了解使用 Docker 开发特定于语言的应用程序

- [Docker for Java developers lab](https://github.com/docker/labs/tree/master/developer-tools/java/){: target="_blank" rel="noopener" class="_"} 
- [Port a node.js app to Docker lab](https://github.com/docker/labs/tree/master/developer-tools/nodejs/porting){: target="_blank" rel="noopener" class="_"}
- [Ruby on Rails app on Docker lab](https://github.com/docker/labs/tree/master/developer-tools/ruby){: target="_blank" rel="noopener" class="_"}
- [Dockerize a .Net Core application](../samples/dotnetcore.md){: target="_blank" rel="noopener" class="_"}
- [Dockerize an ASP.NET Core application with SQL Server on Linux](../samples/aspnet-mssql-compose.md){: target="_blank" rel="noopener" class="_"} using Docker Compose

## 使用 SDK 或 API 进行高级开发

在您可以编写 Dockerfiles 或 Compose 文件并使用 Docker CLI 后，通过使用 Docker Engine SDK for Go/Python 或直接使用 HTTP API 将其提升到一个新的水平。访问 [Develop with Docker Engine API](../engine/api/index.md) 部分，了解有关使用 Engine API 进行开发的更多信息，在何处可以找到适用于您选择的编程语言的 SDK，并查看一些示例。