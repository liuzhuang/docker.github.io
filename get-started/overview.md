---
description: Docker explained in depth
keywords: docker, introduction, documentation, about, technology, understanding
redirect_from:
- /introduction/understanding-docker/
- /engine/userguide/basics/
- /engine/introduction/understanding-docker/
- /engine/understanding-docker/
- /engine/docker-overview/
title: Docker 概述
---

Docker 是一个用于开发、传输和运行应用程序的开放平台。Docker 使您能够将应用程序和基础设施分开，以便您可以快速交付软件。
使用 Docker，您可以管理您的基础设施以与管理应用程序相同的方式。
通过利用 Docker 的快速传输、测试和部署代码的方法，您可以显著减少编写代码和在生产环境中运行代码之间的延迟。


## Docker 平台

Docker 提供了在轻量级隔离环境中打包和运行应用程序的功能，轻量级的隔离环境称为容器。隔离性和安全性允许您可以在给定主机上同时运行许多容器。
容器是轻量级的并且包含运行应用程序所需的一切内容，所以您无需依赖于当前主机安装的内容。
您可以在工作时轻松地共享容器，并确保与您共享的每个人都获得在同样运行方式。

Docker 提供工具和平台来管理容器的生命周期:

* 使用容器开发您的应用程序及其支持组件。
* 容器成为分发和测试应用程序的单元。
* 准备就绪后，将应用程序作为容器或编排服务部署到生产环境中。无论您的生产环境是本地数据中心、云提供商还是两者的混合，这都是一样的。

## 我可以用 Docker 做什么？

**快速、一致地交付您的应用程序**

Docker 允许开发人员在标准化环境中使用本地容器（提供应用程序和服务的容器），从而简化了开发生命周期。容器非常适合持续集成和持续交付 (CI/CD) 工作流。

考虑以下示例场景:

- 您的开发人员在本地编写代码并使用 Docker 容器与同事共享他们的工作。
- 他们使用 Docker 将他们的应用程序推送到测试环境中并执行自动化和手动测试。
- 当开发者发现 bugs 时，可以在开发环境中进行修复，并重新部署到测试环境中进行测试和验证。
- 测试完成后，为客户提供修复就像将更新的镜像推送到生产环境一样简单。

**响应式部署和扩展**

Docker 这种基于容器的平台适合可移植的 workloads。Docker 容器可以在开发人员的本地笔记本电脑上运行，无论是物理的还是虚拟的数据中心、云提供商或混合环境中的机器。

Docker 的可移植性和轻量级特性也使其易于动态管理 workloads，根据业务需求几乎实时地扩展或删除应用程序和服务。

**在同一硬件上运行更多 workloads**

Docker 轻巧且快速。
它为基于管理程序的虚拟机提供了一种可行且经济高效的替代方案，因此您可以使用更多的计算能力来实现您的业务目标。
Docker 非常适合高密度环境以及需要以更少资源完成更多任务的中小型部署。

## Docker 架构

Docker 使用客户端-服务器架构。
Docker 客户端与 Docker 守护进程对话，后者负责构建、运行和分发 Docker 容器的繁重工作。
Docker 客户端和守护程序可以运行在同一系统上，或者您可以将 Docker 客户端连接到远程 Docker 守护程序。
Docker 客户端和守护进程使用 REST API、UNIX 套接字或网络接口进行通信。
另一个 Docker 客户端是 Docker Compose，它允许您使用由一组容器组成的应用程序。

![Docker Architecture Diagram](/engine/images/architecture.svg)

### The Docker daemon

Docker 守护程序 (`dockerd`) 监听 Docker API 请求并管理 Docker 对象，如镜像、容器、网络和卷。
守护程序也可以与其他守护程序通信以管理 Docker 服务。

### The Docker client

Docker 客户端 (`docker`) 是许多 Docker 用户和 Docker 交互的主要方式。
当您使用诸如 `docker run`，客户端会将这些命令发送到`dockerd`, 它来实现它们。`docker` 命令使用 Docker API。
Docker 客户端可以与多个守护程序通信。

### Docker Desktop

Docker Desktop 是一款适用 Mac 和 Windows环境，并且易于安装的应用程序，使您能够构建和共享容器化应用程序和微服务。
Docker Desktop 包括 Docker守护程序 (`dockerd`)，Docker客户端 (`docker`) 、Docker Compose、Docker Content Trust、Kubernetes 和 Credential Helper。
有关更多信息，请参阅 [Docker Desktop](https://docs.docker.com/desktop/)。

### Docker registries

Docker _registry_ 存储 Docker 镜像。Docker Hub 是一个公共的任何人都可以使用的 registry，Docker 默认配置为在 Docker Hub 上查找镜像。
您甚至可以运行自己的私有 registry。

当您使用 `docker pull` 或 `docker run` 命令，所需要的镜像是从配置的 registry 中提取。当您使用 `docker push` 命令，您的镜像将被推送到配置的 registry 中。

### Docker objects

当您使用 Docker 时，您是在创建和使用镜像、容器、网络，卷、插件和其他对象。本节是对其中一些对象的简要概述。

#### Images

_image_ 是一个用于创建 Docker 容器的只读模板，包含了一些指令。通常，镜像是基于另一个镜像，还有一些额外的自定义配置。
例如，你可以构建一个基于 `ubuntu` 的镜像，安装 Apache web 服务器和您的应用程序，以及运行应用程序所需的配置详细信息。

您可以创建自己的镜像，也可以只使用其他人创建的镜像并在注册表中发布。
要构建自己的镜像，您需要创建一个 `Dockerfile` 使用简单的语法来定义创建和运行镜像所需的步骤它。
`Dockerfile` 中的每条指令都会在镜像中创建一个图层。当你更改 `Dockerfile` 并重建镜像，仅那些具有更改后被重建。这是使镜像如此轻巧、小巧的部分原因，与其他虚拟化技术相比，速度更快。

#### Containers

容器是镜像的可运行实例。您可以使用 Docker API 或 CLI 创建、启动、停止、移动或删除容器。您可以将容器连接到一个或多个网络，为其附加存储，甚至可以根据其当前状态创建新镜像。

默认情况下，容器与其他容器及其主机相对隔离。您可以控制容器的网络、存储或其他底层子系统与其他容器或主机之间的隔离程度。

容器由其镜像以及您在创建或启动它时提供给它的任何配置选项定义。当容器被移除时，未存储在持久存储中的对其状态的任何更改都会消失。

##### 示例 `docker run` 命令

以下命令运行 `ubuntu` 容器，交互式地连接到您的本地命令行会话，并运行/bin/bash。

```console
$ docker run -i -t ubuntu /bin/bash
```

运行此命令时，会发生以下情况 (假设您正在使用默认注册表配置):

1. 如果你没有 `ubuntu` 镜像在本地，Docker 从您的配置的注册表，就像您已经手动运行了 `docker pull ubuntu` 一样。

2. Docker 创建一个新容器，就像您运行了一个 `docker container create` 命令。

3. Docker 为容器分配一个读写文件系统，作为它的最后一层。这允许正在运行的容器在其本地文件系统中创建或修改文件和目录。

4. Docker 创建一个网络接口以将容器连接到默认网络，因为您没有指定任何网络选项。这包括为容器分配 IP 地址。默认情况下，容器可以使用主机的网络连接连接到外部网络。

5. Docker 启动容器并执行 `/bin/bash`。因为容器正在交互运行并连接到您的终端 (由于 `-i` 和  `-t` 标志)，您可以在输出记录到时使用键盘提供输入你的终端。

6. 当你打 `exit` 的时候终止 `/bin/bash` 命令，容器会停止但不会被移除。您可以重新启动它或将其删除。

![run-ubuntu-basic.png](/images/liuzhuang/run-ubuntu-basic.png)

## 底层技术

Docker 是用 [Go](https://golang.org/) 编写的，并利用 Linux 内核的几个特性来提供其功能。
Docker 使用一种叫做 `namespaces` 的技术提供隔离的工作空间，被称为容器。运行容器时，Docker 会为那个容器创建一组 **namespaces**。

这些命名空间提供了一层隔离。容器的每个方面都在单独的命名空间中运行，并且其访问权限仅限于该命名空间。

## Next steps
- Read about [installing Docker](../get-docker.md).
- Get hands-on experience with the [Getting started with Docker](index.md) tutorial.
