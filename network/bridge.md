---
title: 使用 bridge 网络
description: All about using user-defined bridge networks and the default bridge
keywords: network, bridge, user-defined, standalone
redirect_from:
- /engine/userguide/networking/default_network/custom-docker0/
- /engine/userguide/networking/default_network/dockerlinks/
- /engine/userguide/networking/default_network/build-bridges/
- /engine/userguide/networking/work-with-networks/
- /config/containers/bridges/
---

在网络方面，bridge 网络是在网段之间转发流量的链路层设备。
bridge 可以是在主机内核中运行的硬件设备或软件设备。

就 Docker 而言，一个 bridge 网络使用一个软件 bridge，允许连接到同一个 bridge 网络的容器进行通信，同时提供与未连接到该桥接网络的容器的隔离。
Docker 网桥驱动程序会自动在宿主机中安装规则，以便不同网桥网络上的容器无法直接相互通信。

Bridge 网络适用于在**同一个**Docker daemon 宿主机上运行的容器。
对于运行在不同 Docker daemon 宿主机上的容器之间的通信，您可以在操作系统级别管理路由，或者使用[overlay 网络](overlay.md)。

当您启动 Docker 时，会自动创建一个[默认的桥接网络]((#use-the-default-bridge-network))（也称为bridge），除非另有说明，否则新启动的容器会连接到它。
您还可以创建用户定义的自定义桥接网络。**用户定义的 bridge 网络优于默认的 bridge 网络**。

## 自定义 bridge 和默认 bridge 的区别

- **用户定义的 bridge 在容器之间提供自动 DNS 解析。**.

  默认桥接网络上的容器只能通过 IP 地址相互访问，除非您使用--link选项，这被认为是遗留的。
  在用户定义的桥接网络上，容器可以通过名称或别名相互解析。
  Containers on the default bridge network can only access each other by IP
  addresses, unless you use the [`--link` option](links.md), which is
  considered legacy. On a user-defined bridge network, containers can resolve
  each other by name or alias.

  想象一个具有 Web 前端和数据库后端的应用程序。
  如果您调用容器web和db，则db无论应用程序堆栈在哪个 Docker 主机上运行，Web 容器都可以连接到 db 容器。
  Imagine an application with a web front-end and a database back-end. If you call
  your containers `web` and `db`, the web container can connect to the db container
  at `db`, no matter which Docker host the application stack is running on.

  如果您在默认桥接网络上运行相同的应用程序堆栈，则需要手动创建容器之间的链接（使用 legacy--link 标志）。
  这些链接需要在两个方向上创建，因此您可以看到这对于需要通信的两个以上容器变得复杂。
  或者，您可以操作/etc/hosts容器内的文件，但这会产生难以调试的问题。
  If you run the same application stack on the default bridge network, you need
  to manually create links between the containers (using the legacy `--link`
  flag). These links need to be created in both directions, so you can see this
  gets complex with more than two containers which need to communicate.
  Alternatively, you can manipulate the `/etc/hosts` files within the containers,
  but this creates problems that are difficult to debug.

- **用户定义的 bridge 有更好的隔离性。**.

  所有没有--network指定的容器都附加到默认的网桥网络。这可能是一种风险，因为不相关的堆栈/服务/容器然后能够进行通信。
  All containers without a `--network` specified, are attached to the default bridge network. This can be a risk, as unrelated stacks/services/containers are then able to communicate.

  使用用户定义的网络提供了一个范围网络，其中只有连接到该网络的容器才能进行通信。
  Using a user-defined network provides a scoped network in which only containers attached to that network are able to communicate.

- **容器可以在运行过程中从用户定义的网络连接和分离。**.

  During a container's lifetime, you can connect or disconnect it from
  user-defined networks on the fly. To remove a container from the default
  bridge network, you need to stop the container and recreate it with different
  network options.

- **每个用户定义的网络都会创建一个可配置的网桥。**.

  If your containers use the default bridge network, you can configure it, but
  all the containers use the same settings, such as MTU and `iptables` rules.
  In addition, configuring the default bridge network happens outside of Docker
  itself, and requires a restart of Docker.

  User-defined bridge networks are created and configured using
  `docker network create`. If different groups of applications have different
  network requirements, you can configure each user-defined bridge separately,
  as you create it.

- **默认网桥网络上的链接容器共享环境变量。**.

  最初，在两个容器之间共享环境变量的唯一方法是使用--link标志链接它们。
  这种类型的变量共享对于用户定义的网络是不可能的。但是，有更好的方法来共享环境变量。一些想法：
  Originally, the only way to share environment variables between two containers
  was to link them using the [`--link` flag](links.md). This type of
  variable sharing is not possible with user-defined networks. However, there
  are superior ways to share environment variables. A few ideas:

  - 多个容器可以使用 Docker 卷挂载包含共享信息的文件或目录。

  - 多个容器可以一起使用 `docker-compose`，compose 文件可以定义共享变量。

  - 您可以使用 swarm 服务而不是独立容器，并利用共享[secrets](../engine/swarm/secrets.md)和[configs](../engine/swarm/configs.md)。

连接到同一个用户定义的桥接网络的容器有效地将所有端口相互暴露。
对于不同网络上的容器或非 Docker 主机可以访问的端口，必须使用 `-p` 或 `--publish` 标志发布该端口。

## 管理用户自定义的 bridge

使用 `docker network create` 命令创建用户自定义桥接网络。

```console
$ docker network create my-net
```

您可以指定子网、IP 地址范围、网关和其他选项。
有关详细信息，请参阅[docker 网络创建](../engine/reference/commandline/network_create.md#specify-advanced-options) 参考 或查看 `docker network create --help` 帮助说明。

使用 `docker network rm` 命令删除用户定义的桥接网络。
如果容器当前已连接到网络，请先[断开它们](#disconnect-a-container-from-a-user-defined-bridge)。

```console
$ docker network rm my-net
```

> **到底发生了什么？**
>
> 当您创建或删除用户定义的网桥或将容器与用户定义的网桥连接或断开连接时，
> Docker 使用特定于操作系统的工具来管理底层网络基础设施（例如在 Linux 上添加或删除网桥设备或配置 `iptables` 规则）。
> 这些细节应该被视为实现细节。让 Docker 为您管理用户定义的网络。

## 将容器连接到用户自定义的 bridge

创建新容器时，您可以指定一个或多个 `--network` 标志。
此示例将 Nginx 容器连接到 `my-net`  网络。
它还将容器中的 80 端口发布到 Docker 主机上的 8080 端口，以便外部客户端可以访问该端口。
连接到 `my-net` 网络的任何其他容器都可以访问 `my-nginx` 容器上的所有端口，反之亦然。

```console
$ docker create --name my-nginx \
  --network my-net \
  --publish 8080:80 \
  nginx:latest
```

要将正在运行的容器连接到现有的用户定义网桥，请使用 `docker network connect` 命令。
以下命令将已运行的 `my-nginx` 容器连接到已存在的 `my-net` 网络：

```console
$ docker network connect my-net my-nginx
```

## 断开容器和用户自定义 bridge 的连接

要将正在运行的容器与用户定义的网桥断开连接，请使用该 `docker network disconnect` 命令。
以下命令将 `my-nginx` 容器与 `my-net` 网络断开连接。

```console
$ docker network disconnect my-net my-nginx
```

## 使用 IPv6

如果您需要 Docker 容器的 IPv6 支持，则需要 在创建任何 IPv6 网络或分配容器 IPv6 地址之前启用Docker 守护程序上的选项并重新加载其配置。


If you need IPv6 support for Docker containers, you need to
[enable the option](../config/daemon/ipv6.md) on the Docker daemon and reload its
configuration, before creating any IPv6 networks or assigning containers IPv6
addresses.

创建网络时，您可以指定 `--ipv6` 标志启用 IPv6。
您无法选择性地禁用默认 `bridge` 网络上的 IPv6 支持。

## 启用从 Docker 容器到外界的转发

默认情况下，来自连接到默认网桥网络的容器的流量 不会转发到外部世界。
要启用转发，您需要更改两个设置。
这些不是 Docker 命令，它们会影响 Docker 主机的内核。

1.  配置 Linux 内核以允许 IP 转发。

    ```console
    $ sysctl net.ipv4.conf.all.forwarding=1
    ```

2.  将 `iptables` `FORWARD` 的策略从 `DROP` 更改到 `ACCEPT`。

    ```console
    $ sudo iptables -P FORWARD ACCEPT
    ```

这些设置不会在重新启动后持续存在，因此您可能需要将它们添加到启动脚本中。

## 使用默认的 bridge 网络

默认 `bridge` 网络被认为是 Docker 的遗留细节，不推荐用于生产用途。
配置它是一个手动操作，它有[技术缺陷](#differences-between-user-defined-bridges-and-the-default-bridge)。

### 将容器连接到默认的 bridge 网络

如果您没有使用 `--network` 标志指定网络，而您确实指定了网络驱动程序，则默认情况下您的容器将连接到默认 `bridge` 网络。
连接到默认 `bridge` 网络的容器可以通信，但只能通过 IP 地址进行通信，除非它们使用[legacy `--link` flag](links.md)标志链接 。

### 配置默认的 bridge 网络

要配置默认`bridge` 网络，请在`daemon.json`. 这是一个`daemon.json`指定了多个选项的示例。仅指定您需要自定义的设置。

```json
{
  "bip": "192.168.1.5/24",
  "fixed-cidr": "192.168.1.5/25",
  "fixed-cidr-v6": "2001:db8::/64",
  "mtu": 1500,
  "default-gateway": "10.20.1.1",
  "default-gateway-v6": "2001:db8:abcd::89",
  "dns": ["10.20.1.2","10.20.1.3"]
}
```

重新启动 Docker 以使更改生效。

### 将 IPv6 和默认 bridge 网络搭配使用

如果您为 IPv6 支持配置 Docker（请参阅[使用 IPv6](#use-ipv6)），
默认桥接网络也会自动配置为支持 IPv6。与用户定义的网桥不同，您无法在默认网桥上有选择地禁用 IPv6。

## 下一步

- 完成[独立网络教程](network-tutorial-standalone.md)
- 了解[从容器的角度网络](../config/containers/container-networking.md)
- 了解[覆盖网络](overlay.md)
- 了解[Macvlan 网络](macvlan.md)