---
redirect_from:
- /reference/api/hub_registry_spec/
- /userguide/image_management/
- /engine/userguide/eng-image/image_management/
description: Documentation for docker Registry and Registry API
keywords: docker, registry, api,  hub
title: 管理镜像
---


使您的映像可供组织内部或外部其他人使用的最简单方法是使用 Docker 注册表，例如 [Docker Hub](#docker-hub)，或运行您自己的 [private registry](#docker-registry)。


## Docker Hub

Docker Hub是由 Docker, Inc. 管理的公共注册表。它集中了有关组织、用户帐户和映像的信息。它包括一个 Web UI、使用组织的身份验证和授权、CLI 和使用命令（如docker login、docker pull、 和docker push、评论、星标、搜索等）的 API 访问。

## Docker Registry

Docker Registry 是 Docker 生态系统的一个组件。注册表是一个存储和内容交付系统，保存命名的 Docker 镜像，可用于不同的标记版本。例如，distribution/registry带有标签2.0和的图像latest。用户通过使用 docker push 和 pull 命令（例如docker pull myregistry.com/stevvooe/batman:voice.

Docker Hub 是 Docker Registry 的一个实例。

## 内容信任

在网络系统之间传输数据时，信任是一个核心问题。特别是，当通过不受信任的媒体（例如互联网）进行通信时，确保系统运行的所有数据的完整性和发布者至关重要。您使用 Docker 将图像（数据）推送和拉取到注册表。内容信任使您能够验证通过任何渠道从注册中心接收的所有数据的完整性和发布者。

有关在 Docker 客户端上配置和使用此功能的信息，请参阅内容信任。
