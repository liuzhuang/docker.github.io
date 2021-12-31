---
description: Description of labels, which are used to manage metadata on Docker objects.
keywords: Usage, user guide, labels, metadata, docker, documentation, examples, annotating
title: Docker object labels
redirect_from:
- /engine/userguide/labels-custom-metadata/
---

标签是一种将元数据应用于 Docker 对象的机制，包括：

- Images
- Containers
- Local daemons
- Volumes
- Networks
- Swarm nodes
- Swarm services

您可以使用标签来组织图像、记录许可信息、注释容器、卷和网络之间的关系，或者以对您的业务或应用程序有意义的任何方式。

## 标签键和值

标签是一个键值对，存储为字符串。您可以为一个对象指定多个标签，但每个键值对在一个对象中必须是唯一的。如果同一个键被赋予多个值，则最近写入的值会覆盖所有先前的值。

### Key 格式推荐

标签键是键值对的左侧。键是字母数字字符串，可能包含句点 ( .) 和连字符 ( -)。大多数 Docker 用户使用其他组织创建的映像，以下准则有助于防止无意中跨对象复制标签，特别是如果您计划使用标签作为自动化机制。

- 第三方工具的作者应该使用他们拥有的域的反向 DNS 符号作为每个标签键的前缀，例如com.example.some-label.

- 未经域所有者许可，请勿在标签密钥中使用域。

- 的com.docker.*，io.docker.*和org.dockerproject.*命名空间是由泊坞窗供内部使用保留的。

- 标签键应以小写字母开头和结尾，并且应仅包含小写字母数字字符、句点字符 ( .) 和连字符 ( -)。不允许使用连续句号或连字符。

- 句点字符 ( .) 分隔命名空间“字段”。没有命名空间的标签键保留供 CLI 使用，允许 CLI 用户使用较短的键入友好字符串交互式标记 Docker 对象。

这些指南目前尚未强制执行，其他指南可能适用于特定用例。

### 价值准则

标签值可以包含任何可以表示为字符串的数据类型，包括（但不限于）JSON、XML、CSV 或 YAML。唯一的要求是首先使用特定于结构类型的机制将值序列化为字符串。例如，要将 JSON 序列化为字符串，您可以使用JSON.stringify()JavaScript 方法。

由于 Docker 不会对值进行反序列化，因此在按标签值查询或过滤时，您不能将 JSON 或 XML 文档视为嵌套结构，除非您将此功能构建到第三方工具中。

## 管理对象上的标签

支持标签的每种类型的对象都有添加和管理它们的机制，并在它们与该类型的对象相关时使用它们。这些链接是开始学习如何在 Docker 部署中使用标签的好地方。

图像、容器、本地守护进程、卷和网络上的标签在对象的生命周期内是静态的。要更改这些标签，您必须重新创建对象。群节点和服务上的标签可以动态更新。

- Images and containers
  - [Adding labels to images](../engine/reference/builder.md#label)
  - [Overriding a container's labels at runtime](../engine/reference/commandline/run.md#set-metadata-on-container--l---label---label-file)
  - [Inspecting labels on images or containers](../engine/reference/commandline/inspect.md)
  - [Filtering images by label](../engine/reference/commandline/images.md#filtering)
  - [Filtering containers by label](../engine/reference/commandline/ps.md#filtering)

- Local Docker daemons
  - [Adding labels to a Docker daemon at runtime](../engine/reference/commandline/dockerd.md)
  - [Inspecting a Docker daemon's labels](../engine/reference/commandline/info.md)

- Volumes
  - [Adding labels to volumes](../engine/reference/commandline/volume_create.md)
  - [Inspecting a volume's labels](../engine/reference/commandline/volume_inspect.md)
  - [Filtering volumes by label](../engine/reference/commandline/volume_ls.md#filtering)

- Networks
  - [Adding labels to a network](../engine/reference/commandline/network_create.md)
  - [Inspecting a network's labels](../engine/reference/commandline/network_inspect.md)
  - [Filtering networks by label](../engine/reference/commandline/network_ls.md#filtering)

- Swarm nodes
  - [Adding or updating a swarm node's labels](../engine/reference/commandline/node_update.md#add-label-metadata-to-a-node)
  - [Inspecting a swarm node's labels](../engine/reference/commandline/node_inspect.md)
  - [Filtering swarm nodes by label](../engine/reference/commandline/node_ls.md#filtering)

- Swarm services
  - [Adding labels when creating a swarm service](../engine/reference/commandline/service_create.md#set-metadata-on-a-service--l---label)
  - [Updating a swarm service's labels](../engine/reference/commandline/service_update.md)
  - [Inspecting a swarm service's labels](../engine/reference/commandline/service_inspect.md)
  - [Filtering swarm services by label](../engine/reference/commandline/service_ls.md#filtering)
