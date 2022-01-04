---
title: 使用 macvlan 网络进行联网
description: Tutorials for networking using a macvlan bridge network and 802.1q trunk bridge network
keywords: networking, macvlan, 802.1q, standalone
---

本系列教程涉及连接到 `macvlan` 网络的网络独立容器。
在这种类型的网络中，Docker 主机在其 IP 地址处接受对多个 MAC 地址的请求，并将这些请求路由到适当的容器。
有关其他网络主题，请参阅[概述](index.md)。

## 目标

这些教程的目标是设置桥接 `macvlan` 网络并将容器连接到它，然后设置一个 802.1q 中继 `macvlan` 网络并将容器连接到它。

## 先决条件

- 大多数云提供商阻止 `macvlan` 网络。您可能需要物理访问您的网络设备。

- 在 `macvlan` 网络驱动程序仅适用于 Linux主机，并且不支持 Docker Desktop for Mac, Docker Desktop for Windows, 或 Docker EE for Windows Server.

- Linux 内核至少需要 3.9 版本，建议使用 4.0 或更高版本。

- 这些示例假设您的以太网接口是 `eth0`. 如果您的设备具有不同的名称，请改用该名称。

## Bridge 示例

在简单的桥接示例中，您的流量流过 `eth0`  并且 Docker 使用其 MAC 地址将流量路由到您的容器。
对于网络上的网络设备，您的容器似乎物理连接到网络。

1.  创建一个名为 `my-macvlan-net` 的 `macvlan` 网络。
    将 `subnet`、`gateway` 和 `parent` 值修改为在您的环境中有意义的值。

    ```console
    $ docker network create -d macvlan \
      --subnet=172.16.86.0/24 \
      --gateway=172.16.86.1 \
      -o parent=eth0 \
      my-macvlan-net
    ```

    您可以使用 `docker network ls` 和 `docker network inspect my-macvlan-net` 命令来验证网络是否存在并且是 `macvlan` 网络。

2.  启动一个 `alpine` 容器并将其附加到 `my-macvlan-net` 网络。
    这些 `-dit` 标志在后台启动容器，但允许您附加到它。
    `--rm` 标志表示容器在停止时被移除。

    ```console
    $ docker run --rm -dit \
      --network my-macvlan-net \
      --name my-macvlan-alpine \
      alpine:latest \
      ash
    ```

3.  检查 `my-macvlan-alpine` 容器并注意 `MacAddress` 密钥中的 `Networks` 密钥：

    ```none
    $ docker container inspect my-macvlan-alpine

    ...truncated...
    "Networks": {
      "my-macvlan-net": {
          "IPAMConfig": null,
          "Links": null,
          "Aliases": [
              "bec64291cd4c"
          ],
          "NetworkID": "5e3ec79625d388dbcc03dcf4a6dc4548644eb99d58864cf8eee2252dcfc0cc9f",
          "EndpointID": "8caf93c862b22f379b60515975acf96f7b54b7cf0ba0fb4a33cf18ae9e5c1d89",
          "Gateway": "172.16.86.1",
          "IPAddress": "172.16.86.2",
          "IPPrefixLen": 24,
          "IPv6Gateway": "",
          "GlobalIPv6Address": "",
          "GlobalIPv6PrefixLen": 0,
          "MacAddress": "02:42:ac:10:56:02",
          "DriverOpts": null
      }
    }
    ...truncated
    ```

4.  通过运行几个 docker exec` 命令查看容器如何查看自己的网络接口。

    ```console
    $ docker exec my-macvlan-alpine ip addr show eth0

    9: eth0@tunl0: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:10:56:02 brd ff:ff:ff:ff:ff:ff
    inet 172.16.86.2/24 brd 172.16.86.255 scope global eth0
       valid_lft forever preferred_lft forever
    ```

    ```console
    $ docker exec my-macvlan-alpine ip route

    default via 172.16.86.1 dev eth0
    172.16.86.0/24 dev eth0 scope link  src 172.16.86.2
    ```

5.  停止容器（Docker 因为`--rm`标记将其删除），并删除网络。

    ```console
    $ docker container stop my-macvlan-alpine

    $ docker network rm my-macvlan-net
    ```

## 802.1q trunked bridge 示例

在 802.1q 中继桥接示例中，您的流量流经 `eth0`（称为 `eth0.10`）的子接口，Docker 使用其 MAC 地址将流量路由到您的容器。
对于网络上的网络设备，您的容器似乎物理连接到网络。

1.  创建一个名为`my-8021q-macvlan-net`的`macvlan`网络。将`subnet`、`gateway`和`parent`值修改为在您的环境中有意义的值。

    ```console
    $ docker network create -d macvlan \
      --subnet=172.16.86.0/24 \
      --gateway=172.16.86.1 \
      -o parent=eth0.10 \
      my-8021q-macvlan-net
    ```

    您可以使用`docker network ls`和`docker network inspect my-8021q-macvlan-net` 命令来验证网络是否存在、是否为`macvlan`网络以及是否具有 parent `eth0.10`。可以`ip addr show`在Docker主机上使用来验证接口是否`eth0.10`存在并且有单独的IP地址

2.  启动一个`alpine容器`并将其附加到`my-8021q-macvlan-net` 网络。
    这些`-dit`标志在后台启动容器，但允许您附加到它。
    该`--rm`标志表示容器在停止时被移除。

    ```console
    $ docker run --rm -itd \
      --network my-8021q-macvlan-net \
      --name my-second-macvlan-alpine \
      alpine:latest \
      ash
    ```

3.  检查`my-second-macvlan-alpine`容器并注意`MacAddress` 密钥中的`Networks`密钥：

    ```none
    $ docker container inspect my-second-macvlan-alpine

    ...truncated...
    "Networks": {
      "my-8021q-macvlan-net": {
          "IPAMConfig": null,
          "Links": null,
          "Aliases": [
              "12f5c3c9ba5c"
          ],
          "NetworkID": "c6203997842e654dd5086abb1133b7e6df627784fec063afcbee5893b2bb64db",
          "EndpointID": "aa08d9aa2353c68e8d2ae0bf0e11ed426ea31ed0dd71c868d22ed0dcf9fc8ae6",
          "Gateway": "172.16.86.1",
          "IPAddress": "172.16.86.2",
          "IPPrefixLen": 24,
          "IPv6Gateway": "",
          "GlobalIPv6Address": "",
          "GlobalIPv6PrefixLen": 0,
          "MacAddress": "02:42:ac:10:56:02",
          "DriverOpts": null
      }
    }
    ...truncated
    ```

4.  通过运行几个`docker exec`命令查看容器如何查看自己的网络接口。

    ```console
    $ docker exec my-second-macvlan-alpine ip addr show eth0

    11: eth0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:10:56:02 brd ff:ff:ff:ff:ff:ff
    inet 172.16.86.2/24 brd 172.16.86.255 scope global eth0
       valid_lft forever preferred_lft forever
    ```

    ```console
    $ docker exec my-second-macvlan-alpine ip route

    default via 172.16.86.1 dev eth0
    172.16.86.0/24 dev eth0 scope link  src 172.16.86.2
    ```

5.  停止容器（Docker 因为`--rm` 标记将其删除），并删除网络。

    ```console
    $ docker container stop my-second-macvlan-alpine

    $ docker network rm my-8021q-macvlan-net
    ```

## 其他网络教程

现在您已经完成了网络的 `macvlan` 网络教程，您可能想要浏览这些其他网络教程：

- [Standalone networking tutorial](network-tutorial-standalone.md)
- [Overlay networking tutorial](network-tutorial-overlay.md)
- [Host networking tutorial](network-tutorial-host.md)