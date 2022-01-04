---
title: 使用 host 网络
description: All about exposing containers on the Docker host's network
keywords: network, host, standalone
---

如果您对容器使用 `host` 网络模式，则该容器的网络堆栈不会与 Docker 主机隔离（容器共享主机的网络命名空间），并且容器不会分配自己的 IP 地址。
例如，如果您运行绑定到端口 80 的容器并使用 `host` 网络，则容器的应用程序可在主机 IP 地址的端口 80 上使用。

> **Note**: 
>
> 由于使用时容器不拥有自己的IP地址 host模式的网络，端口映射不生效，并且-p，--publish，-P，和--publish-all选项都将被忽略，产生一个警告而不是：
> 
> Given that the container does not have its own IP-address when using
> `host` mode networking, [port-mapping](overlay.md#publish-ports) does not
> take effect, and the `-p`, `--publish`, `-P`, and `--publish-all` option are
> ignored, producing a warning instead:
>
> ```console
> WARNING: Published ports are discarded when using host network mode
> ```

主机模式网络可用于优化性能，并且在容器需要处理大量端口的情况下，因为它不需要网络地址转换 (NAT)，并且没有为每个端口创建“用户空间代理”。

主机网络驱动程序仅适用于 Linux 主机，在 Docker Desktop for Mac、Docker Desktop for Windows 或 Docker EE for Windows Server 上不受支持。

您还可以通过传递 `--network host` 给 `docker service create` 命令，将 `host` 网络用于 swarm 服务。
在这种情况下，控制流量（与管理 swarm 和服务相关的流量）仍然通过覆盖网络发送，但各个 swarm 服务容器使用 Docker 守护程序的主机网络和端口发送数据。这会产生一些额外的限制。
例如，如果服务容器绑定到端口 80，则在给定的 swarm 节点上只能运行一个服务容器。

## 下一步

- [浏览主机网络教程](network-tutorial-host.md)
- [从容器的角度了解网络](../config/containers/container-networking.md)
- 了解[桥接网络](bridge.md)
- 了解[覆盖网络](overlay.md)
- 了解[Macvlan 网络](macvlan.md)