---
title: 网络概述
description: Overview of Docker networks and networking concepts
keywords: networking, bridge, routing, routing mesh, overlay, ports
redirect_from:
- /engine/userguide/networking/
- /engine/userguide/networking/dockernetworks/
- /articles/networking/
---

Docker containers 和 services 如此强大的原因之一是您可以将它们连接在一起，或将它们连接到非 Docker workloads。
Docker containers 和 services 甚至不需要知道它们是否部署在 Docker 上，或者它们的对等方是否也是 Docker workloads。
无论您的 Docker 宿主机运行 Linux、Windows 还是两者的混合，您都可以使用 Docker 以与平台无关的方式管理它们。

本主题定义了一些基本的 Docker 网络概念，并为您充分利用这些功能 “设计” 和 “部署” 应用程序做好准备。

## 这个话题的范围

本主题**并没有**进入操作系统讲解关于 Docker 网络的具体细节，
所以你不会找到在 Linux 上 Docker 操纵 `iptables` 的有关信息，在 Windows 服务器上是如何操纵路由规则，
你也不会找到在 Docker 如何形成封装数据包或处理加密。
请查阅[Docker 和 iptables](iptables.md).。

此外，本主题不提供任何有关如何创建、管理和使用 Docker 网络的教程。
每个部分都包含指向相关教程和命令参考的链接。

## 网络驱动

Docker 的网络子系统是可插拔的，使用驱动。默认情况下存在多个驱动程序，并提供核心网络功能：

- `bridge`: 
  默认的网络驱动程序。如果您不指定驱动程序，这就是您正在创建的网络类型。
  **当您的应用程序运行在独立的容器中时，通常会使用桥接网络。**
  请参阅[桥接网络](bridge.md)。

- `host`: 
  对于独立容器，去掉容器和 Docker 主机之间的网络隔离，直接使用主机的网络。
  请参阅[使用主机网络](host.md)。

- `overlay`: 
  覆盖网络将多个 Docker daemons 连接在一起，并使 swarm services 之间能够相互通信。
  您还可以使用覆盖网络来便利 swarm service 和一个独立容器之间的通信，或者在不同的 Docker daemons 上的两个独立容器之间的通信。
  这种策略消除了在这些容器之间进行操作系统级（OS-level）路由的需要。
  请参阅[覆盖网络](overlay.md)。

- `ipvlan`: 
  IPvlan 网络让用户可以完全控制 IPv4 和 IPv6 寻址。
  VLAN 驱动程序在此基础上构建，可为对底层网络集成感兴趣的用户提供对第 2 层 VLAN 标记甚至 IPvlan L3 路由的完全控制。
  请参阅[IPvlan 网络](ipvlan.md)。

- `macvlan`: 
  Macvlan 网络允许您为容器分配 MAC 地址，使其在您的网络上显示为物理设备。
  Docker 守护进程通过容器的 MAC 地址将流量路由到容器。
  `macvlan` 在处理期望直接连接到物理网络而不是通过 Docker 主机的网络堆栈路由的遗留应用程序时，使用驱动程序有时是最佳选择。
  请参阅[Macvlan 网络](macvlan.md)。

- `none`: 
  对于这个容器，禁用所有网络。通常与自定义网络驱动程序结合使用。
  `none` 不适用于群服务。
  请参阅[禁用容器网络](none.md)。

- [Network plugins](/engine/extend/plugins_services/): 
  您可以通过 Docker 安装和使用第三方网络插件。
  这些插件可从 [Docker Hub](https://hub.docker.com/search?category=network&q=&type=plugin) 或第三方供应商处获得。
  请参阅供应商的文档以安装和使用给定的网络插件。

### 网络驱动总结

- 在同一个 Docker 宿主机上的多个容器进行通信时，**用户定义的桥接网络**是最佳选择。
- 当网络栈不应与 Docker 主机隔离，但您希望容器的其他方面被隔离时，**Host 网络**是最佳选择。
- 当不同 Docker 宿主机上运行的容器需要进行通信时，或者当多个应用程序使用 swarm 服务协同工作时，**Overlay 网络**是最佳选择。
- 当您从 VM 迁移安装或需要容器看起来像物理主机上的网络时，**Macvlan 网络**是最佳选择，每个主机都有唯一的 MAC 地址。
- **第三方网络插件**允许您将 Docker 与专门的网络栈集成。

## 网络教程

现在您了解了 Docker 网络的基础知识，请使用以下教程加深您的理解：

- [单机网络教程](network-tutorial-standalone.md)
- [Host 网络教程](network-tutorial-host.md)
- [Overlay 网络教程](network-tutorial-overlay.md)
- [Macvlan 网络教程](network-tutorial-macvlan.md)