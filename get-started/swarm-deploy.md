---
title: "部署到 Swarm"
keywords: swarm, swarm services, stacks
description: Learn how to describe and deploy a simple application on Docker Swarm.
redirect_from:
- /get-started/part4/
---

## 先决条件

- 按照方向和设置中的说明下载并安装 Docker 桌面。
- 完成第 2 部分中的应用程序容器化。
- 通过键入docker system info并查找消息Swarm: active（您可能需要向上滚动一点），确保在您的 Docker 桌面上启用了 Swarm 。

  如果 Swarm 没有运行，只需输入`docker swarm init`一个 shell 提示来设置它。


## 简介

既然我们已经演示了我们应用程序的各个组件作为独立容器运行并展示了如何使用 Kubernetes 部署它，让我们看看如何安排它们由 Docker Swarm 管理。Swarm 提供了许多用于扩展、联网、保护和维护容器化应用程序的工具，这些工具超出了容器本身的能力。

为了验证我们的容器化应用程序在 Swarm 上运行良好，我们将在我们的开发机器上使用 Docker Desktop 的内置 Swarm 环境来部署我们的应用程序，然后将其移交给在生产中的完整 Swarm 集群上运行。Docker Desktop 创建的 Swarm 环境功能齐全，这意味着它具有您的应用程序将在真实集群上享受的所有 Swarm 功能，可从您的开发机器上方便地访问。

## Describe apps using stack files

Swarm 绝不会像我们在本教程的上一步中那样创建单独的容器。相反，所有 Swarm 工作负载都被安排为服务，这些服务是可扩展的容器组，具有由 Swarm 自动维护的附加网络功能。此外，所有 Swarm 对象都可以并且应该在称为堆栈文件的清单中进行描述。这些 YAML 文件描述了 Swarm 应用程序的所有组件和配置，可用于在任何 Swarm 环境中轻松创建和销毁您的应用程序。

让我们编写一个简单的堆栈文件来运行和管理我们的公告板。将以下内容放入名为 的文件中bb-stack.yaml：

```yaml
version: '3.7'

services:
  bb-app:
    image: bulletinboard:1.0
    ports:
      - "8000:8080"
```

在这个 Swarm YAML 文件中，我们只有一个对象： a service，描述一组可扩展的相同容器。在这种情况下，您将只获得一个容器（默认），并且该容器将基于您在快速入门教程的第 2 部分中bulletinboard:1.0创建的映像。此外，我们已经要求 Swarm 将所有到达我们开发机器上的 8000 端口的流量转发到我们公告板容器内的 8080 端口。

> Kubernetes 服务和 Swarm 服务非常不同！尽管名称相似，但这两个协调器在术语“服务”中的含义却截然不同。在 Swarm 中，服务提供调度和网络设施，创建容器并提供将流量路由到它们的工具。在 Kubernetes 中，调度和网络是分开处理的：部署（或其他控制器）将容器的调度作为 Pod 处理，而服务只负责向这些 Pod 添加网络功能。

## 部署并检查您的应用程序

1.  将您的应用程序部署到 Swarm：

    ```console
    $ docker stack deploy -c bb-stack.yaml demo
    ```

    如果一切顺利，Swarm 将报告创建您所有的堆栈对象而没有任何抱怨：

    ```shell
    Creating network demo_default
    Creating service demo_bb-app
    ```

    请注意，除了您的服务之外，Swarm 还默认创建一个 Docker 网络来隔离部署为您的堆栈的一部分的容器。

2.  通过列出您的服务确保一切正常：

    ```console
    $ docker service ls
    ```

    如果一切顺利，您的服务将报告创建了 1/1 的副本：

    ```shell
    ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
    il7elwunymbs        demo_bb-app         replicated          1/1                 bulletinboard:1.0   *:8000->8080/tcp
    ```

    这表明您要求作为服务的一部分的 1/1 容器已启动并正在运行。此外，我们看到您的开发机器上的端口 8000 被转发到您的公告板容器中的端口 8080。

3.  打开浏览器并访问您的公告板localhost:8000；您应该会看到您的公告板，这与我们在快速入门教程的第 2 部分中将其作为独立容器运行时的情况相同。

4.  满意后，拆除您的应用程序：

    ```console
    $ docker stack rm demo
    ```

## 结论

至此，我们已经成功地使用 Docker Desktop 将我们的应用程序部署到我们开发机器上功能齐全的 Swarm 环境中。我们还没有对 Swarm 做太多事情，但现在大门已经打开：您可以开始向您的应用程序添加其他组件并利用 Swarm 的所有功能和强大功能，就在您自己的机器上。

除了部署到 Swarm 之外，我们还将我们的应用程序描述为一个堆栈文件。这个简单的文本文件包含我们在运行状态下创建应用程序所需的一切；我们可以将其检查到版本控制中并与我们的同事共享，从而使我们能够轻松地将应用程序分发到其他集群（例如可能在我们的开发环境之后出现的测试和生产集群）。

## Swarm 和 CLI 参考

本文中使用的所有新 Swarm 对象和 CLI 命令的更多文档可在此处获得：

 - [Swarm Mode](https://docs.docker.com/engine/swarm/)
 - [Swarm Mode Services](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/)
 - [Swarm Stacks](https://docs.docker.com/engine/swarm/stack-deploy/)
 - [`docker stack *`](https://docs.docker.com/engine/reference/commandline/stack/)
 - [`docker service *`](https://docs.docker.com/engine/reference/commandline/service/)
