---
title: Use macvlan networks
description: All about using macvlan to make your containers appear like physical machines on the network
keywords: network, macvlan, standalone
redirect_from:
- /engine/userguide/networking/get-started-macvlan/
- /config/containers/macvlan/
---

一些应用程序，尤其是遗留应用程序或监控网络流量的应用程序，希望直接连接到物理网络。在这种情况下，你可以使用`macvlan`网络驱动程序为每个容器的虚拟网络接口分配一个MAC地址，使其看起来是一个直接连接到物理网络的物理网络接口。在这种情况下，你需要指定你的多克尔主机使用的一个物理接口`macvlan`，还有的子网和网关`macvlan`。您甚至可以`macvlan`使用不同的物理网络接口来隔离您的网络。请记住以下几点：

- 由于 IP 地址耗尽或“VLAN 传播”，很容易无意中损坏您的网络，这种情况是您的网络中有大量不适当的唯一 MAC 地址。

- 您的网络设备需要能够处理“混杂模式”，即一个物理接口可以分配多个 MAC 地址。

- 如果您的应用程序可以使用网桥（在单个 Docker 主机上）或覆盖（跨多个 Docker 主机进行通信），这些解决方案从长远来看可能会更好。

## 创建一个 macvlan 网络

创建 `macvlan` 网络时，它可以处于桥接模式或 802.1q 中继桥接模式。

- 在桥接模式下，`macvlan` 流量通过主机上的物理设备。

- 在 802.1q 中继桥接模式下，流量通过 Docker 动态创建的 802.1q 子接口。这允许您在更细粒度的级别控制路由和过滤。

### 桥接模式

要创建与给定物理网络接口桥接的 `macvlan` 网络，请使用 `docker network create` `--driver macvlan`命令。
您还需要指定 `parent`，这是流量将在 Docker 主机上物理通过的接口。

```console
$ docker network create -d macvlan \
  --subnet=172.16.86.0/24 \
  --gateway=172.16.86.1 \
  -o parent=eth0 pub_net
```

如果您需要排除 `macvlan` 网络中使用的 IP 地址，例如当给定的 IP 地址已被使用时，请使用`--aux-addresses`：

```console
$ docker network create -d macvlan \
  --subnet=192.168.32.0/24 \
  --ip-range=192.168.32.128/25 \
  --gateway=192.168.32.254 \
  --aux-address="my-router=192.168.32.129" \
  -o parent=eth0 macnet32
```

### 802.1q 中继桥接模式

如果您指定一个包含点的 `parent` 接口名称，例如 `eth0.50`，Docker 会将其解释为 `eth0` 的子接口并自动创建子接口。

```console
$ docker network create -d macvlan \
    --subnet=192.168.50.0/24 \
    --gateway=192.168.50.1 \
    -o parent=eth0.50 macvlan50
```

### 使用 ipvlan 代替 macvlan

在上面的示例中，您仍在使用 L3 网桥。
您可以使用 `ipvlan` 替代，并获得 L2 桥接器。指定 `-o ipvlan_mode=l2`。

```console
$ docker network create -d ipvlan \
    --subnet=192.168.210.0/24 \
    --subnet=192.168.212.0/24 \
    --gateway=192.168.210.254 \
    --gateway=192.168.212.254 \
     -o ipvlan_mode=l2 -o parent=eth0 ipvlan210
```

## 使用 IPv6

如果您已将[Docker 守护程序配置为允许 IPv6](../config/daemon/ipv6.md)，则可以使用双栈 IPv4/IPv6 `macvlan`网络。

```console
$ docker network create -d macvlan \
    --subnet=192.168.216.0/24 --subnet=192.168.218.0/24 \
    --gateway=192.168.216.1 --gateway=192.168.218.1 \
    --subnet=2001:db8:abc8::/64 --gateway=2001:db8:abc8::10 \
     -o parent=eth0.218 \
     -o macvlan_mode=bridge macvlan216
```

## Next steps

- Go through the [macvlan networking tutorial](network-tutorial-macvlan.md)
- Learn about [networking from the container's point of view](../config/containers/container-networking.md)
- Learn about [bridge networks](bridge.md)
- Learn about [overlay networks](overlay.md)
- Learn about [host networking](host.md)
- Learn about [Macvlan networks](macvlan.md)
